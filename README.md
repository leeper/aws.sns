# AWS SNS Client Package

**aws.sns** is a simple client package for the Amazon Web Services (AWS) [Simple Notification Service (SNS)](https://aws.amazon.com/sns/) API, which can be used to trigger push messages to a variety of users, devices, and other endpoints. This might be useful for maintaining multi-platform mailing lists, or simply for creating a way to notify yourself when long-running code completes.

To use the package, you will need an AWS account and enter your credentials into R. Your keypair can be generated on the [IAM Management Console](https://aws.amazon.com/) under the heading *Access Keys*. Note that you only have access to your secret key once. After it is generated, you need to save it in a secure location. New keypairs can be generated at any time if yours has been lost, stolen, or forgotten. 

By default, all **cloudyr** packages look for the access key ID and secret access key in environment variables. You can also use this to specify a default region or a temporary "session token". For example:

```R
Sys.setenv("AWS_ACCESS_KEY_ID" = "mykey",
           "AWS_SECRET_ACCESS_KEY" = "mysecretkey",
           "AWS_DEFAULT_REGION" = "us-east-1",
           "AWS_SESSION_TOKEN" = "mytoken")
```

These can alternatively be set on the command line prior to starting R or via an `Renviron.site` or `.Renviron` file, which are used to set environment variables in R during startup (see `? Startup`).

If you work with multiple AWS accounts, another option that is consistent with other Amazon SDKs is to create [a centralized `~/.aws/credentials` file](https://blogs.aws.amazon.com/security/post/Tx3D6U6WSFGOK2H/A-New-and-Standardized-Way-to-Manage-Credentials-in-the-AWS-SDKs), containing credentials for multiple accounts. You can then use credentials from this file on-the-fly by simply doing:

```R
# use your 'default' account credentials
use_credentials()

# use an alternative credentials profile
use_credentials(profile = "bob")
```

Temporary session tokens are stored in environment variable `AWS_SESSION_TOKEN` (and will be stored there by the `use_credentials()` function). The [aws.iam package](https://github.com/cloudyr/aws.iam/) provides an R interface to IAM roles and the generation of temporary session tokens via the security token service (STS).


## Code Examples

The main purpose of Amazon SNS is to be able to push messages to different endpoints (e.g., Email, SMS, a Simple Queue Service queue, etc.). To do this, you have to create a *topic*, *subscribe* different endpoints (e.g., user email addresses) to that topic, and then *publish* to the topic. You can subscribe different types of endpoints to the same topic and, similarly, publish different messages to each type of endpoint simultaneously.

To create a topic, use `create_topic` and configure it using `set_topic_attrs`. The `name` argument in `create_topic` is a private label for you to keep track of topics. To use a topic, you need to use `set_topic_attrs` to configure a public display name that will be visible to subscribers:


```r
library("aws.sns")
topic <- create_topic(name = "TestTopic")
set_topic_attrs(topic, attribute = c(DisplayName = "Publicly visible topic name"))
```

```
## [1] TRUE
## attr(,"RequestId")
## [1] "d5069c6d-0616-5373-8b69-cca98694137b"
```

To add a subscription to a topic:


```r
subscribe(topic, "me@example.com", "email")
```

```
## [1] "pending confirmation"
## attr(,"RequestId")
## [1] "0dd167bd-8b99-59c5-ae20-37790b1ac9cd"
```

```r
#subscribe(topic, "1-111-555-1234", "sms") # SMS example
```

You can confirm the status of subscriptions using `list_subscriptions`:


```r
list_subscriptions(topic)
```

```
##         Endpoint        Owner Protocol     SubscriptionArn
## 1 me@example.com 920667304251    email PendingConfirmation
##                                       TopicArn
## 1 arn:aws:sns:us-east-1:920667304251:TestTopic
```

Subscriptions need to be confirmed by the endpoint. For example, an SMS endpoint will require an SMS response to an subscription invitation message. Subscriptions can be removed using `unsubscribe` (or whatever method is described in the invitation message); thus subscriptions can be handled by both users and administrator (you).

The endpoint will then receive a confirmation message, like the following, to confirm the subscription:

![Email confirmation message](http://i.imgur.com/8EK6jBu.png)

If they accept the invitation, the user will receive a confirmation of their subscription:

![Subscription confirmation screen](http://i.imgur.com/cK1KU3C.png)


To publish a message, use `publish`:


```r
publish(topic = topic, message = "This is a test message!", subject = "Hello!")
```

```
## [1] "343d9791-13da-5459-bb1d-c75593593451"
## attr(,"RequestId")
## [1] "6cfee596-6909-583b-98da-b552e98a1833"
```

By default, the message is sent to all platforms:

![Example message](http://i.imgur.com/nglMtZ9.png)


This may not be ideal if multiple dissimilar endpoints are subscribed to the same topic (e.g., SMS and email). This can be resolved by maintaining separate Topics or, more easily, by sending different messages to each type of endpoint:


```r
msgs <- list()
msgs$default = "This is the default message." # required
msgs$email = "This is a test email that will be sent to email addresses only."
msgs$sms = "This is a test SMS that will be sent to phone numbers only."
msgs$http = "This is a test message that will be sent to http URLs only."
publish(topic = topic, message = msgs, subject = "Hello!")
```

```
## [1] "7ba74da2-f27b-5ca7-80eb-1d6672f81caf"
## attr(,"RequestId")
## [1] "098f0756-35c2-5b90-a5cc-5792b5415b98"
```

In addition to the standard endpoints ("http", "https", "email", "email-json", "sms", "sqs", "application"), it is possible to create endpoints for mobile platform applications. [See the SNS Developer Guide for further details](http://docs.aws.amazon.com/sns/latest/dg/SNSMobilePush.html).

It is also possible to give other AWS accounts permission to view or publish to a topic using `add_permission`. For example, you may want to have multiple administrators who share responsibility for publishing messages to the topic. Permissions can be revoked using `remove_permission`.

## Installation

[![CRAN](https://www.r-pkg.org/badges/version/aws.sns)](https://cran.r-project.org/package=aws.sns)
![Downloads](https://cranlogs.r-pkg.org/badges/aws.sns)
[![Travis Build Status](https://travis-ci.org/cloudyr/aws.sns.png?branch=master)](https://travis-ci.org/cloudyr/aws.sns) 
[![codecov.io](https://codecov.io/github/cloudyr/aws.sns/coverage.svg?branch=master)](https://codecov.io/github/cloudyr/aws.sns?branch=master)

This package is not yet on CRAN. To install the latest development version you can install from the cloudyr drat repository:

```R
# latest stable version
install.packages("aws.sns", repos = c(cloudyr = "http://cloudyr.github.io/drat", getOption("repos")))
```

Or, to pull a potentially unstable version directly from GitHub:

```R
if(!require("ghit")){
    install.packages("ghit")
}
ghit::install_github("cloudyr/aws.sns")
```

---
[![cloudyr project logo](http://i.imgur.com/JHS98Y7.png)](https://github.com/cloudyr)
