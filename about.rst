What is Reflex?
==================================

Reflex is a tool that enables organizations to enforce security best practices in their cloud environment. Reflex works by deploying resources which monitor your environment, and automatically detect or fix resources that are configured in an insecure manner. Best of all, Reflex is event driven, so problems are identified as they happen. No manual intervention or synchronized polling required.

Reflex Architecture
-----------------------
Reflex leverages the **CloudWatch Events** resource as the main source of active account monitoring within AWS. Our architecture uses this as the foundational message source forwarding to an **SQS Queue** target that will then be ingested by a custom **Lambda Function**. Once the logic in that message is evaluated by the Lambda function and the event is found to be non compliant, an alert will be sent out via a central **SNS Topic** to subscribed parties. If the specific rule allows for remediation
functionality, the remediation will take place and results of remediation will be included in the alert. 

.. image:: _static/reflex-architecture.png
   :width: 800pt

How Much Does Reflex Cost
----------------------------
Reflex itself is open source, and the Reflex tool is free to use. *However*, Reflex works by deploying resources in your AWS account, and there is a cost to deploying and running those resources.

**Any costs incurred while running Reflex are your responsibility.** Make sure you understand how Reflex works and are comfortable incurring any associated costs before you deploy resources.


Typical Monthly Cost
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Your cost to run Reflex will depend on a variety of factors, particularly the number of Reflex rules you deploy and how often activity occurs in your AWS account. Our experience has been that running these kinds of rules, even in large AWS environments, is inexpensive. The largest AWS users (ie large enterprises with dozens or hundreds of active developers) typically spend no more than $5 per rule per month. The average user, with only a few developers, should expect to spend a few cents per rule per month. And if you qualify for AWS' free tier, it is likely that there will be little to no cost at all.

The following information outlines the resources Reflex deploys, and should give you a starting point for estimating the cost of running Reflex in your AWS account.


CloudWatch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Reflex utilizes CloudWatch Event Rules to monitor events in your environment and trigger rules. Event Rules are free. Reflex also utilizes CloudWatch Logs, which have ingestion and archive costs if you choose to use those features. In most cases the cost of Logs should be free.

For more information see `AWS' CloudWatch Pricing Documentation <https://aws.amazon.com/cloudwatch/pricing/>`_.


SQS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The first million requests with SQS each month are free (if you qualify for the free tier), so for most users there should be no cost for SQS. If you do not qualify for the free tier, SQS costs $0.40 per million requests, so the cost for SQS should be negligible in most environments.

For more information see `AWS' SQS Pricing Documentation <https://aws.amazon.com/sqs/pricing/>`_.


Lambda
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Lambda compute costs are the main cost associated with running Reflex. As these costs are dependent on which rules you use in your environment, it can be hard to predict what these will be. However even for large organizations it shouldn't be more than a few dollars per rule per month.

For more information see `AWS' Lambda Pricing Documentation <https://aws.amazon.com/sqs/pricing/>`_.


SNS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
By default, reflex will create SNS Topics and publish messages to them to notify you of what is happening in your environment. SNS offers one thousand free email publishes per month, with a cost of $2.00 per 100,000 after that. For small organizations the cost of SNS should be low or nothing, but as always it depends on your environment and deployed rules.

For more information see `AWS' SNS Pricing Documentation <https://aws.amazon.com/sns/pricing/>`_.

Multi-region Support
----------------------------

In a default implementation of reflex, a single region is monitored for events. If you are using a single region and preventing access (whether via SCP or IAM), the default implementation of reflex should suffice. However, if you would like to monitor other regions, reflex provides a configuration option for forwarding CloudWatch Events to a central region for processing. 

This infrastructure is created similarly to the output of `reflex build` in that it is a collection of terraform files that are to deploy infrastructure. To create region-forwarding infrastructure, you can either extend the provider configuration (outlined below) or run `reflex region --region [region_to_forward]` in the same directory as your configuration file. 


.. code-block:: yaml

  providers:
   - aws:
      region: us-east-1
      forwarding_regions:
        - us-east-2
        - eu-west-1

Multi-account Support
----------------------------

In a default implementation of reflex, a single account is monitored for events. If your organization leverages many accounts and wishes to monitor them all through a single reflex deploy, we allow for the configuration of a multi-account forwarder to a central account. In this setup, there is an account deployed with normal reflex infrastructure, called a "parent" account and then many "child" accounts that forward their rule findings to the central account for processing. In the "child" accounts, you will find just event rules, SNS topics, and IAM roles for cross account describe/remediation.

Similarly to the multi-region build output, we can create multi-account output with a configuration block update like below. This will create separate output directories needed to be deployed separately with terraform in each account. *Note*: If specifying child accounts, it is required to specify parent accounts in the configuration below. Specifying neither child or parent accounts will create a single account build. 

.. code-block:: yaml

  providers:
   - aws:
      region: us-east-1
      parent_account: "123456789012"
      child_accounts:
        - "234567890123"
        - "345678901234"
