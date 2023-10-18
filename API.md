# @seeebiii/ses-email-forwarding

This [AWS CDK](https://aws.amazon.com/cdk/) construct allows you to setup email forwarding mappings in [AWS SES](https://aws.amazon.com/ses/) to receive emails from your domain and forward them to another email address.
All of this is possible without hosting your own email server, you just need a domain.

For example, if you own a domain `example.org` and want to receive emails for `hello@example.org` and `privacy@example.org`, you can forward emails to `whatever@provider.com`.
This is achieved by using a Lambda function that forwards the emails using [aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder).

This construct is creating quite a few resources under the hood and can also automatically verify your domain and email addresses in SES.
Consider reading the [Architecture](#architecture) section below if you want to know more about the details.

## Examples

Forward all emails received under `hello@example.org` to `whatever+hello@provider.com`:

```javascript
new EmailForwardingRuleSet(this, 'EmailForwardingRuleSet', {
  // make the underlying rule set the active one
  enableRuleSet: true,
  // define how emails are being forwarded
  emailForwardingProps: [{
    // your domain name you want to use for receiving and sending emails
    domainName: 'example.org',
    // a prefix that is used for the From email address to forward your mails
    fromPrefix: 'noreply',
    // a list of mappings between a prefix and target email address
    emailMappings: [{
      // the prefix matching the receiver address as <prefix>@<domainName>
      receivePrefix: 'hello',
      // the target email address(es) that you want to forward emails to
      targetEmails: ['whatever+hello@provider.com']
    }]
  }]
});
```

Forward all emails to `hello@example.org` to `whatever+hello@provider.com` and verify the domain `example.org` in SES:

```javascript
new EmailForwardingRuleSet(this, 'EmailForwardingRuleSet', {
  emailForwardingProps: [{
    domainName: 'example.org',
    // let the construct automatically verify your domain
    verifyDomain: true,
    fromPrefix: 'noreply',
    emailMappings: [{
      receivePrefix: 'hello',
      targetEmails: ['whatever+hello@provider.com']
    }]
  }]
});
```

If you don't want to verify your domain in SES or you are in the SES sandbox, you can still send emails to verified email addresses.
Use the property `verifyTargetEmailAddresses` in this case and set it to `true`.

For a full & up-to-date reference of the available options, please look at the source code of  [`EmailForwardingRuleSet`](lib/email-forwarding-rule-set.ts) and [`EmailForwardingRule`](lib/email-forwarding-rule.ts).

#### Note

Since the verification of domains requires to lookup the Route53 domains in your account, you need to define your AWS account and region.
You can do it like this in your CDK stack:

```typescript
const app = new cdk.App();

class EmailForwardingSetupStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    new EmailForwardingRuleSet(this, 'EmailForwardingRuleSet', {
      // define your config here
    });
  }
}

new EmailForwardingSetupStack(app, 'EmailForwardingSetupStack', {
  env: {
    account: '<account-id>',
    region: '<region>'
  }
});
```

## Use Cases

- Build a landing page on AWS and offer an email address to contact you.
- Use various aliases to register for different services and forward all mails to the same target email address.

There are probably more - happy to hear them :)

## Install

### npm

```shell
npm i -D @seeebiii/ses-email-forwarding
```

Take a look at [package.json](./package.json) to make sure you're installing the correct version compatible with your current AWS CDK version.
See more details on npmjs.com: https://www.npmjs.com/package/@seeebiii/ses-email-forwarding

### Maven

```xml
<dependency>
  <groupId>de.sebastianhesse.cdk-constructs</groupId>
  <artifactId>ses-email-forwarding</artifactId>
  <version>4.0.1</version>
</dependency>
```

See more details on mvnrepository.com: https://mvnrepository.com/artifact/de.sebastianhesse.cdk-constructs/ses-email-forwarding/

#### Example Code

```java
package com.example;

import de.sebastianhesse.cdk.ses.email.forwarding.EmailForwardingProps;
import de.sebastianhesse.cdk.ses.email.forwarding.EmailForwardingRuleSet;
import de.sebastianhesse.cdk.ses.email.forwarding.EmailMapping;
import java.util.Arrays;
import software.amazon.awscdk.core.App;

import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Environment;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;

public class SesEmailForwardingJavaTestApp {
    public static void main(final String[] args) {
        App app = new App();

        new SesEmailForwardingJavaTestStack(app, "CdkEmailForwardingJavaTestStack", StackProps.builder()
                .env(Environment.builder()
                        .account("123456789") // TODO: replace with your account id
                        .region("us-east-1") // TODO: replace with your region
                        .build()
                )
                .build());

        app.synth();
    }

    static class SesEmailForwardingJavaTestStack extends Stack {
        public SesEmailForwardingJavaTestStack(final Construct scope, final String id) {
            this(scope, id, null);
        }

        public SesEmailForwardingJavaTestStack(final Construct scope, final String id, final StackProps props) {
            super(scope, id, props);

            EmailForwardingProps exampleProperties = EmailForwardingProps.builder()
                    .domainName("example.org")
                    // true if you own the domain in Route53, false if you need to manually verify it
                    .verifyDomain(true)
                    .fromPrefix("noreply")
                    .emailMappings(Arrays.asList(
                            EmailMapping.builder()
                                    .receiveEmail("hello@example.org")
                                    .targetEmails(Arrays.asList("email+hello@provider.com"))
                                    .build(),
                            EmailMapping.builder()
                                    .receiveEmail("privacy@example.org")
                                    .targetEmails(Arrays.asList("email+privacy@provider.com"))
                                    .build()
                            )
                    )
                    .build();

            EmailForwardingRuleSet.Builder.create(this, "example-rule-set")
                    .ruleSetName("example-rule-set")
                    .emailForwardingProps(Arrays.asList(exampleProperties))
                    .build();
        }
    }
}
```

### Python

```shell
pip install ses-email-forwarding
```

See more details on PyPi: https://pypi.org/project/ses-email-forwarding/

### .NET / C#

An artifact is pushed up to NuGet.org: https://www.nuget.org/packages/Ses.Email.Forwarding/

#### Project Scaffolding & Installation

```bash
# Create a new directory
mkdir ExampleApplication && cd ExampleApplication

# Scaffold a C# CDK project
cdk init --language csharp

# Add dependencies
cd src/ExampleApplication
dotnet add package Ses.Email.Forwarding
dotnet add package Amazon.CDK.AWS.SNS.Subscriptions

# Remove example stack and global suppressions (silenced by way of using discards)
rm ExampleApplicationStack.cs GlobalSuppressions.cs
```

#### Example Usage

```csharp
using Amazon.CDK;
using Amazon.CDK.AWS.SNS;
using Amazon.CDK.AWS.SNS.Subscriptions;
using SebastianHesse.CdkConstructs;
using Construct = Constructs.Construct;

namespace ExampleApplication
{
    public sealed class Program
    {
        public static void Main()
        {
            var app = new App();

            _ = new MailboxStack(app, nameof(MailboxStack), new StackProps
            {
                Env = new Environment
                {
                    // Replace with desired account
                    Account = "000000000000",

                    // Replace with desired region
                    Region = "us-east-1"
                }
            });

            app.Synth();
        }
    }

    public sealed class MailboxStack : Stack
    {
        public MailboxStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var notificationTopic = new Topic(this, nameof(EmailForwardingProps.NotificationTopic));

            // 'Bounce' and 'Complaint' notification types, in association with the domain being verified, will be sent
            // to this email address
            notificationTopic.AddSubscription(new EmailSubscription("admin@provider.com"));

            _ = new EmailForwardingRuleSet(this, nameof(EmailForwardingRuleSet), new EmailForwardingRuleSetProps
            {
                EmailForwardingProps = new IEmailForwardingProps[]
                {
                    new EmailForwardingProps
                    {
                        // If your domain name has already been verified as a domain identity in SES, this does not
                        // need to be toggled on
                        VerifyDomain = true,

                        // This is the prefix that will be used in the email address used to forward emails
                        FromPrefix = "noreply",

                        // This domain name will be used to send and receive emails
                        DomainName = "example.org",

                        // A list of mappings between a prefix and target email addresses
                        EmailMappings = new IEmailMapping[]
                        {
                            new EmailMapping
                            {
                                // Emails received by hello@example.org will be forwarded
                                ReceivePrefix = "hello",

                                // Emails will be forwarded to admin+hello@provider.com
                                TargetEmails = new []
                                {
                                    "admin+hello@provider.com"
                                }
                            }
                        },

                        // This notification topic be published to when events in association with 'Bounce' and
                        // 'Complaint' notification types occur
                        NotificationTopic = notificationTopic
                    }
                }
            });
        }
    }
}
```

## Usage

This package provides two constructs: [`EmailForwardingRuleSet`](lib/email-forwarding-rule-set.ts) and [`EmailForwardingRule`](lib/email-forwarding-rule.ts).
The `EmailForwardingRuleSet` is a wrapper around `ReceiptRuleSet` but adds a bit more magic to e.g. verify a domain or target email address.
Similarly, `EmailForwardingRule` is a wrapper around `ReceiptRule` but adds two SES rule actions to forward the email addresses appropriately.

This means if you want the full flexibility, you can use the `EmailForwardingRule` construct in your stack.

### Sending E-Mails over SMTP

You can also send emails over SES using this construct because it provides the basics for sending emails: a verified SES domain or email address identity.
You need to do the following if you're using the `EmailForwardingRuleSetConstruct`:

1. Set the `verifyDomain` or `verifyTargetEmailAddresses` to `true`.
2. [Create SMTP credentials in AWS SES](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/smtp-credentials.html?icmpid=docs_ses_console) and save them somewhere.
3. Setup your email program or application to use the SMTP settings.

## Architecture

The `EmailForwardingRuleSet` creates a `EmailForwardingRule` for each forward mapping.
Each rule contains an `S3Action` to store the incoming emails and a Lambda Function to forward the emails to the target email addresses.
The Lambda function is just a thin wrapper around the [aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder) library.
Since this library expects a JSON config with the email mappings, the `EmailForwardingRule` will create an SSM parameter to store the config.
(Note: this is not ideal because an SSM parameter is limited in the size and hence, this might be changed later)
The Lambda function receives a reference to this parameter as an environment variable (and a bit more) and forwards everything to the library.

In order to verify a domain or email address, the `EmailForwardingRuleSet` construct is using the package [@seeebiii/ses-verify-identities](https://www.npmjs.com/package/@seeebiii/ses-verify-identities).
It provides constructs to verify the SES identities.
For domains, it creates appropriate Route53 records like MX, TXT and Cname (for DKIM).
For email addresses, it calls the AWS API to initiate email address verification.

## TODO

- Encrypt email files on S3 bucket by either using S3 bucket encryption (server side) or enable client encryption using SES actions

## Contributing

I'm happy to receive any contributions!
Just open an issue or pull request :)

These commands should help you while developing:

 * `npx projen`          init [projen](https://github.com/projen/projen) and synthesize changes in [.projenrc.js](.projenrc.js) to the project
 * `yarn build`          compile typescript to js
 * `yarn watch`          watch for changes and compile
 * `yarn test`           perform the jest unit tests
 * `yarn eslint`         validate code against best practices

## Thanks

Thanks a lot to [arithmetric](https://github.com/arithmetric) for providing the NPM package [aws-lambda-ses-forwarder](https://github.com/arithmetric/aws-lambda-ses-forwarder).
This CDK construct is using it in the Lambda function to forward the emails.

## Author

[Sebastian Hesse](https://www.sebastianhesse.de) - Freelancer for serverless cloud projects on AWS.

## License

MIT License

Copyright (c) 2022 [Sebastian Hesse](https://www.sebastianhesse.de)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# API Reference <a name="API Reference" id="api-reference"></a>

## Constructs <a name="Constructs" id="Constructs"></a>

### EmailForwardingRule <a name="EmailForwardingRule" id="@nyawaya/ses-email-forwarding.EmailForwardingRule"></a>

A construct to define an email forwarding rule that can either be used together with {@link EmailForwardingRuleSet} or as a standalone rule.

It creates two rule actions:
- One S3 action to save all incoming mails to an S3 bucket.
- One Lambda action to forward all incoming mails to a list of configured emails.

The Lambda function is using the NPM package `aws-lambda-ses-forwarder` to forward the mails.

#### Initializers <a name="Initializers" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer"></a>

```typescript
import { EmailForwardingRule } from '@nyawaya/ses-email-forwarding'

new EmailForwardingRule(parent: Construct, name: string, props: EmailForwardingRuleProps)
```

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer.parameter.parent">parent</a></code> | <code>constructs.Construct</code> | *No description.* |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer.parameter.name">name</a></code> | <code>string</code> | *No description.* |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer.parameter.props">props</a></code> | <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps">EmailForwardingRuleProps</a></code> | *No description.* |

---

##### `parent`<sup>Required</sup> <a name="parent" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer.parameter.parent"></a>

- *Type:* constructs.Construct

---

##### `name`<sup>Required</sup> <a name="name" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer.parameter.name"></a>

- *Type:* string

---

##### `props`<sup>Required</sup> <a name="props" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.Initializer.parameter.props"></a>

- *Type:* <a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps">EmailForwardingRuleProps</a>

---

#### Methods <a name="Methods" id="Methods"></a>

| **Name** | **Description** |
| --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRule.toString">toString</a></code> | Returns a string representation of this construct. |

---

##### `toString` <a name="toString" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.toString"></a>

```typescript
public toString(): string
```

Returns a string representation of this construct.

#### Static Functions <a name="Static Functions" id="Static Functions"></a>

| **Name** | **Description** |
| --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRule.isConstruct">isConstruct</a></code> | Checks if `x` is a construct. |

---

##### ~~`isConstruct`~~ <a name="isConstruct" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.isConstruct"></a>

```typescript
import { EmailForwardingRule } from '@nyawaya/ses-email-forwarding'

EmailForwardingRule.isConstruct(x: any)
```

Checks if `x` is a construct.

###### `x`<sup>Required</sup> <a name="x" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.isConstruct.parameter.x"></a>

- *Type:* any

Any object.

---

#### Properties <a name="Properties" id="Properties"></a>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRule.property.node">node</a></code> | <code>constructs.Node</code> | The tree node. |

---

##### `node`<sup>Required</sup> <a name="node" id="@nyawaya/ses-email-forwarding.EmailForwardingRule.property.node"></a>

```typescript
public readonly node: Node;
```

- *Type:* constructs.Node

The tree node.

---


### EmailForwardingRuleSet <a name="EmailForwardingRuleSet" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet"></a>

A construct for AWS SES to forward all emails of certain domains and email addresses to a list of target email addresses.

It also verifies (or at least initiates verification of) the related domains and email addresses in SES.

The construct can be helpful if you don't want to host your own SMTP server but still want to receive emails to your existing email inbox.
One use case is if you're just building some sort of landing page and want to quickly setup email receiving for your domain without yet another separate email inbox.

This construct can...
- create a new receipt rule set (or use an existing one),
- attach a list of rules to forward incoming emails to other target email addresses,
- verify a given domain in SES (automatically if domain is managed by Route53, otherwise it'll just initiate the verification),
- initiate verification for all target email addresses that are provided for receiving the forwarded emails.

#### Initializers <a name="Initializers" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer"></a>

```typescript
import { EmailForwardingRuleSet } from '@nyawaya/ses-email-forwarding'

new EmailForwardingRuleSet(parent: Construct, name: string, props: EmailForwardingRuleSetProps)
```

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer.parameter.parent">parent</a></code> | <code>constructs.Construct</code> | *No description.* |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer.parameter.name">name</a></code> | <code>string</code> | *No description.* |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer.parameter.props">props</a></code> | <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps">EmailForwardingRuleSetProps</a></code> | *No description.* |

---

##### `parent`<sup>Required</sup> <a name="parent" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer.parameter.parent"></a>

- *Type:* constructs.Construct

---

##### `name`<sup>Required</sup> <a name="name" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer.parameter.name"></a>

- *Type:* string

---

##### `props`<sup>Required</sup> <a name="props" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.Initializer.parameter.props"></a>

- *Type:* <a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps">EmailForwardingRuleSetProps</a>

---

#### Methods <a name="Methods" id="Methods"></a>

| **Name** | **Description** |
| --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.toString">toString</a></code> | Returns a string representation of this construct. |

---

##### `toString` <a name="toString" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.toString"></a>

```typescript
public toString(): string
```

Returns a string representation of this construct.

#### Static Functions <a name="Static Functions" id="Static Functions"></a>

| **Name** | **Description** |
| --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.isConstruct">isConstruct</a></code> | Checks if `x` is a construct. |

---

##### ~~`isConstruct`~~ <a name="isConstruct" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.isConstruct"></a>

```typescript
import { EmailForwardingRuleSet } from '@nyawaya/ses-email-forwarding'

EmailForwardingRuleSet.isConstruct(x: any)
```

Checks if `x` is a construct.

###### `x`<sup>Required</sup> <a name="x" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.isConstruct.parameter.x"></a>

- *Type:* any

Any object.

---

#### Properties <a name="Properties" id="Properties"></a>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.property.node">node</a></code> | <code>constructs.Node</code> | The tree node. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.property.emailForwardingMappings">emailForwardingMappings</a></code> | <code>any[]</code> | *No description.* |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.property.ruleSet">ruleSet</a></code> | <code>aws-cdk-lib.aws_ses.IReceiptRuleSet</code> | *No description.* |

---

##### `node`<sup>Required</sup> <a name="node" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.property.node"></a>

```typescript
public readonly node: Node;
```

- *Type:* constructs.Node

The tree node.

---

##### `emailForwardingMappings`<sup>Required</sup> <a name="emailForwardingMappings" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.property.emailForwardingMappings"></a>

```typescript
public readonly emailForwardingMappings: any[];
```

- *Type:* any[]

---

##### `ruleSet`<sup>Required</sup> <a name="ruleSet" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSet.property.ruleSet"></a>

```typescript
public readonly ruleSet: IReceiptRuleSet;
```

- *Type:* aws-cdk-lib.aws_ses.IReceiptRuleSet

---


## Structs <a name="Structs" id="Structs"></a>

### EmailForwardingProps <a name="EmailForwardingProps" id="@nyawaya/ses-email-forwarding.EmailForwardingProps"></a>

#### Initializer <a name="Initializer" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.Initializer"></a>

```typescript
import { EmailForwardingProps } from '@nyawaya/ses-email-forwarding'

const emailForwardingProps: EmailForwardingProps = { ... }
```

#### Properties <a name="Properties" id="Properties"></a>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.domainName">domainName</a></code> | <code>string</code> | The domain name for which you want to receive emails using SES, e.g. `example.org`. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.emailMappings">emailMappings</a></code> | <code><a href="#@nyawaya/ses-email-forwarding.EmailMapping">EmailMapping</a>[]</code> | A list of email mappings to define the receive email address and target email addresses to which the emails are forwarded to. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.fromPrefix">fromPrefix</a></code> | <code>string</code> | A prefix that is used as the sender address of the forwarded mail, e.g. `noreply`. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.bucket">bucket</a></code> | <code>aws-cdk-lib.aws_s3.Bucket</code> | Optional: an S3 bucket to store the received emails. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.bucketPrefix">bucketPrefix</a></code> | <code>string</code> | Optional: a prefix for the email files that are stored on the S3 bucket. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.notificationTopic">notificationTopic</a></code> | <code>aws-cdk-lib.aws_sns.Topic</code> | Optional: an SNS topic to receive notifications about sending events like bounces or complaints. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.notificationTypes">notificationTypes</a></code> | <code>string[]</code> | Optional: a list of {@link NotificationType}s to define which sending events should be subscribed. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.verifyDomain">verifyDomain</a></code> | <code>boolean</code> | Optional: true if you want to verify the domain identity in SES, false otherwise. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps.property.verifyTargetEmailAddresses">verifyTargetEmailAddresses</a></code> | <code>boolean</code> | Optional: true if you want to initiate the verification of your target email addresses, false otherwise. |

---

##### `domainName`<sup>Required</sup> <a name="domainName" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.domainName"></a>

```typescript
public readonly domainName: string;
```

- *Type:* string

The domain name for which you want to receive emails using SES, e.g. `example.org`.

---

##### `emailMappings`<sup>Required</sup> <a name="emailMappings" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.emailMappings"></a>

```typescript
public readonly emailMappings: EmailMapping[];
```

- *Type:* <a href="#@nyawaya/ses-email-forwarding.EmailMapping">EmailMapping</a>[]

A list of email mappings to define the receive email address and target email addresses to which the emails are forwarded to.

> [EmailMapping](EmailMapping)

---

##### `fromPrefix`<sup>Required</sup> <a name="fromPrefix" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.fromPrefix"></a>

```typescript
public readonly fromPrefix: string;
```

- *Type:* string

A prefix that is used as the sender address of the forwarded mail, e.g. `noreply`.

---

##### `bucket`<sup>Optional</sup> <a name="bucket" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.bucket"></a>

```typescript
public readonly bucket: Bucket;
```

- *Type:* aws-cdk-lib.aws_s3.Bucket
- *Default:* A new bucket.

Optional: an S3 bucket to store the received emails.

If none is provided, a new one will be created.

---

##### `bucketPrefix`<sup>Optional</sup> <a name="bucketPrefix" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.bucketPrefix"></a>

```typescript
public readonly bucketPrefix: string;
```

- *Type:* string
- *Default:* inbox/

Optional: a prefix for the email files that are stored on the S3 bucket.

---

##### `notificationTopic`<sup>Optional</sup> <a name="notificationTopic" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.notificationTopic"></a>

```typescript
public readonly notificationTopic: Topic;
```

- *Type:* aws-cdk-lib.aws_sns.Topic
- *Default:* A new SNS topic.

Optional: an SNS topic to receive notifications about sending events like bounces or complaints.

The events are defined by `notificationTypes` using {@link NotificationType}. If no topic is defined, a new one will be created.

---

##### `notificationTypes`<sup>Optional</sup> <a name="notificationTypes" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.notificationTypes"></a>

```typescript
public readonly notificationTypes: string[];
```

- *Type:* string[]
- *Default:* ['Bounce', 'Complaint']

Optional: a list of {@link NotificationType}s to define which sending events should be subscribed.

---

##### `verifyDomain`<sup>Optional</sup> <a name="verifyDomain" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.verifyDomain"></a>

```typescript
public readonly verifyDomain: boolean;
```

- *Type:* boolean
- *Default:* false

Optional: true if you want to verify the domain identity in SES, false otherwise.

---

##### `verifyTargetEmailAddresses`<sup>Optional</sup> <a name="verifyTargetEmailAddresses" id="@nyawaya/ses-email-forwarding.EmailForwardingProps.property.verifyTargetEmailAddresses"></a>

```typescript
public readonly verifyTargetEmailAddresses: boolean;
```

- *Type:* boolean
- *Default:* false

Optional: true if you want to initiate the verification of your target email addresses, false otherwise.

If `true`, a verification email is sent out to all target email addresses. Then, the owner of an email address needs to verify it by clicking the link in the verification email.
Please note in case you don't verify your sender domain, it's required to verify your target email addresses in order to send mails to them.

---

### EmailForwardingRuleProps <a name="EmailForwardingRuleProps" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps"></a>

#### Initializer <a name="Initializer" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.Initializer"></a>

```typescript
import { EmailForwardingRuleProps } from '@nyawaya/ses-email-forwarding'

const emailForwardingRuleProps: EmailForwardingRuleProps = { ... }
```

#### Properties <a name="Properties" id="Properties"></a>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.domainName">domainName</a></code> | <code>string</code> | The domain name of the email addresses, e.g. 'example.org'. It is used to connect the `fromPrefix` and `receivePrefix` properties with a proper domain. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.emailMapping">emailMapping</a></code> | <code><a href="#@nyawaya/ses-email-forwarding.EmailMapping">EmailMapping</a>[]</code> | An email mapping similar to what the NPM library `aws-lambda-ses-forwarder` expects. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.fromPrefix">fromPrefix</a></code> | <code>string</code> | A prefix that is used as the sender address of the forwarded mail, e.g. `noreply`. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.id">id</a></code> | <code>string</code> | An id for the rule. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.ruleSet">ruleSet</a></code> | <code>aws-cdk-lib.aws_ses.IReceiptRuleSet</code> | The rule set this rule belongs to. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.bucket">bucket</a></code> | <code>aws-cdk-lib.aws_s3.Bucket</code> | A bucket to store the email files to. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.bucketPrefix">bucketPrefix</a></code> | <code>string</code> | A prefix for the email files that are saved to the bucket. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.enableLambdaLogging">enableLambdaLogging</a></code> | <code>boolean</code> | Enable log messages in Lambda function which forwards emails. |

---

##### `domainName`<sup>Required</sup> <a name="domainName" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.domainName"></a>

```typescript
public readonly domainName: string;
```

- *Type:* string

The domain name of the email addresses, e.g. 'example.org'. It is used to connect the `fromPrefix` and `receivePrefix` properties with a proper domain.

---

##### `emailMapping`<sup>Required</sup> <a name="emailMapping" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.emailMapping"></a>

```typescript
public readonly emailMapping: EmailMapping[];
```

- *Type:* <a href="#@nyawaya/ses-email-forwarding.EmailMapping">EmailMapping</a>[]

An email mapping similar to what the NPM library `aws-lambda-ses-forwarder` expects.

> [EmailMapping](EmailMapping)

---

##### `fromPrefix`<sup>Required</sup> <a name="fromPrefix" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.fromPrefix"></a>

```typescript
public readonly fromPrefix: string;
```

- *Type:* string

A prefix that is used as the sender address of the forwarded mail, e.g. `noreply`.

---

##### `id`<sup>Required</sup> <a name="id" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.id"></a>

```typescript
public readonly id: string;
```

- *Type:* string

An id for the rule.

This will mainly be used to provide a name to the underlying rule but may also be used as a prefix for other resources.

---

##### `ruleSet`<sup>Required</sup> <a name="ruleSet" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.ruleSet"></a>

```typescript
public readonly ruleSet: IReceiptRuleSet;
```

- *Type:* aws-cdk-lib.aws_ses.IReceiptRuleSet

The rule set this rule belongs to.

---

##### `bucket`<sup>Optional</sup> <a name="bucket" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.bucket"></a>

```typescript
public readonly bucket: Bucket;
```

- *Type:* aws-cdk-lib.aws_s3.Bucket
- *Default:* A new bucket will be created.

A bucket to store the email files to.

If no bucket is provided, a new one will be created using a managed KMS key to encrypt the bucket.

---

##### `bucketPrefix`<sup>Optional</sup> <a name="bucketPrefix" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.bucketPrefix"></a>

```typescript
public readonly bucketPrefix: string;
```

- *Type:* string
- *Default:* inbox/

A prefix for the email files that are saved to the bucket.

---

##### `enableLambdaLogging`<sup>Optional</sup> <a name="enableLambdaLogging" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleProps.property.enableLambdaLogging"></a>

```typescript
public readonly enableLambdaLogging: boolean;
```

- *Type:* boolean
- *Default:* true

Enable log messages in Lambda function which forwards emails.

---

### EmailForwardingRuleSetProps <a name="EmailForwardingRuleSetProps" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps"></a>

#### Initializer <a name="Initializer" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.Initializer"></a>

```typescript
import { EmailForwardingRuleSetProps } from '@nyawaya/ses-email-forwarding'

const emailForwardingRuleSetProps: EmailForwardingRuleSetProps = { ... }
```

#### Properties <a name="Properties" id="Properties"></a>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.emailForwardingProps">emailForwardingProps</a></code> | <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps">EmailForwardingProps</a>[]</code> | A list of mapping options to define how emails should be forwarded. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.enableRuleSet">enableRuleSet</a></code> | <code>boolean</code> | Optional: whether to enable the rule set or not. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.ruleSet">ruleSet</a></code> | <code>aws-cdk-lib.aws_ses.IReceiptRuleSet</code> | Optional: an existing SES receipt rule set. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.ruleSetName">ruleSetName</a></code> | <code>string</code> | Optional: provide a name for the receipt rule set that this construct creates if you don't provide one. |

---

##### `emailForwardingProps`<sup>Required</sup> <a name="emailForwardingProps" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.emailForwardingProps"></a>

```typescript
public readonly emailForwardingProps: EmailForwardingProps[];
```

- *Type:* <a href="#@nyawaya/ses-email-forwarding.EmailForwardingProps">EmailForwardingProps</a>[]

A list of mapping options to define how emails should be forwarded.

---

##### `enableRuleSet`<sup>Optional</sup> <a name="enableRuleSet" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.enableRuleSet"></a>

```typescript
public readonly enableRuleSet: boolean;
```

- *Type:* boolean
- *Default:* true

Optional: whether to enable the rule set or not.

---

##### `ruleSet`<sup>Optional</sup> <a name="ruleSet" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.ruleSet"></a>

```typescript
public readonly ruleSet: IReceiptRuleSet;
```

- *Type:* aws-cdk-lib.aws_ses.IReceiptRuleSet

Optional: an existing SES receipt rule set.

If none is provided, a new one will be created using the name provided with `ruleSetName` or a default one.

---

##### `ruleSetName`<sup>Optional</sup> <a name="ruleSetName" id="@nyawaya/ses-email-forwarding.EmailForwardingRuleSetProps.property.ruleSetName"></a>

```typescript
public readonly ruleSetName: string;
```

- *Type:* string
- *Default:* custom-rule-set

Optional: provide a name for the receipt rule set that this construct creates if you don't provide one.

---

### EmailMapping <a name="EmailMapping" id="@nyawaya/ses-email-forwarding.EmailMapping"></a>

#### Initializer <a name="Initializer" id="@nyawaya/ses-email-forwarding.EmailMapping.Initializer"></a>

```typescript
import { EmailMapping } from '@nyawaya/ses-email-forwarding'

const emailMapping: EmailMapping = { ... }
```

#### Properties <a name="Properties" id="Properties"></a>

| **Name** | **Type** | **Description** |
| --- | --- | --- |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailMapping.property.targetEmails">targetEmails</a></code> | <code>string[]</code> | A list of target email addresses that should receive the forwarded emails for the given email addresses matched by either `receiveEmail` or `receivePrefix`. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailMapping.property.receiveEmail">receiveEmail</a></code> | <code>string</code> | You can define a string that is matching an email address, e.g. `hello@example.org`. |
| <code><a href="#@nyawaya/ses-email-forwarding.EmailMapping.property.receivePrefix">receivePrefix</a></code> | <code>string</code> | A short way to match a specific email addresses by only providing a prefix, e.g. `hello`. The prefix will be combined with the given domain name from {@link EmailForwardingRuleProps}. If an email was sent to this specific email address, all emails matching this receiver will be forwarded to all email addresses defined in `targetEmails`. |

---

##### `targetEmails`<sup>Required</sup> <a name="targetEmails" id="@nyawaya/ses-email-forwarding.EmailMapping.property.targetEmails"></a>

```typescript
public readonly targetEmails: string[];
```

- *Type:* string[]

A list of target email addresses that should receive the forwarded emails for the given email addresses matched by either `receiveEmail` or `receivePrefix`.

Make sure that you only specify email addresses that are verified by SES. Otherwise SES won't send them out.

Example: `['foobar@gmail.com', 'foo+bar@gmail.com', 'whatever@example.org']`

---

##### `receiveEmail`<sup>Optional</sup> <a name="receiveEmail" id="@nyawaya/ses-email-forwarding.EmailMapping.property.receiveEmail"></a>

```typescript
public readonly receiveEmail: string;
```

- *Type:* string

You can define a string that is matching an email address, e.g. `hello@example.org`.

If this property is defined, the `receivePrefix` will be ignored. You must define either this property or `receivePrefix`, otherwise no emails will be forwarded.

---

##### `receivePrefix`<sup>Optional</sup> <a name="receivePrefix" id="@nyawaya/ses-email-forwarding.EmailMapping.property.receivePrefix"></a>

```typescript
public readonly receivePrefix: string;
```

- *Type:* string

A short way to match a specific email addresses by only providing a prefix, e.g. `hello`. The prefix will be combined with the given domain name from {@link EmailForwardingRuleProps}. If an email was sent to this specific email address, all emails matching this receiver will be forwarded to all email addresses defined in `targetEmails`.

If `receiveEmail` property is defined as well, then `receiveEmail` is preferred. Hence, only define one of them.

---



