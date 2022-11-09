# Monitoring, logging and auditing
## Introduction
In this section we'll discover the **logging methods** available to us in the AWS platform. We will use these logging tools to learn **how to audit AWS events** for security incidents and uncover potential issues in our platform.

## Implementing logging mechanisms
With so many services available and being used to run your infrastructure, there is a **vast amount of information** being sent and received across your platform. With so much data, it's very important that you are able to **track and record what is happening** across your application for potential attack indicators to respond to. In this section, we will cover log gathering and configuration for:
- S3 Server Access Logs
- Flow Logs and traffic mirroring
- AWS CloudTrail logs
- The CloudWatch logging agent

### Implementing logging
For many organizations, implementing logging and monitoring can feel like an afterthought, or a non-priority feature. In reality, the opposite is true, where having a very strong logging and monitoring platform **helps surface problems and incidents** a lot faster, because having good tools and information available means we can **work with better and more precise information** when tackling an issue.

Many AWS services output very useful logs we can store to investigate if needed, and their output and storage **can usually be configured very easily**. This logging should also be implemented to fulfill auditing and governance needs. **However**, analyzing this information and responding to it accordingly can be complicated due to its **volume** and **complexity**.

Now that we have a basic understanding of AWS logging, lets see some examples of **useful logs in security and auditing** scenarios.

#### Amazon S3 logging
---
Amazon S3 is one of the most used AWS storage services due to its performance and versatility. Due to this, it is very important to understand the two main S3 service logging mechanisms:
- **Amazon S3 server access logging**, which details when a particular *bucket* is accessed, gathering information about that access like the requester identity, the requested `bucket name`, a `timestamp`, the performed `action`, a response `status code` and any `error codes`, if returned
- **Amazon S3 object-level logging**, which integrates heavily with CloudTrail and tracks access to s3 objects inside of a given bucket. This allows us to track if an object with sensitive information has been accessed, when, and by whom.


### Implementing VPC flow logs
Large AWS environments with multiple VPC's can have a very large amount of network traffic going through them. VPC Flow Logs allow us to **capture all of this communication** and **store it for analysis** inside of a bucket. If an attacker communicated with a command and control server outside of AWS from one of our instances, the logs will show it. VPC flow logs are created as an attachment to a given **subnet**, which then can output to a given S3 bucket.

### VPC Traffic Mirroring
VPC Traffic mirroring allows, as the name implies, to **duplicate traffic** from elastic network intefaces attached to a given resource, so this traffic can be sent to a **third-party service** to run **further threat analysis and traffic inspection**, for example, an appliance that analyzes traffic that is received on a given interface.

As opposed to the simplified log format of VPC Flow Logs, this allows you to run further **packet inspection**, as the information sent from the mirrored interface is intact. However, these packets can contain a lot of unnecessary information, so we can run **mirror filters** to only send in necessary data to our appliances. like a certain protocol, or data coming from a certain CIDR block.

### Using AWS CloudTail logs
As discussed in earlier sections, CloudTrail is an AWS service that **logs all AWS API calls** made in the account. This is a really powerful service to help us with **compliance and auditing** of API actions, as having a full API history allows us to analyze potential intrusion paths followed by attackers, **what** they did, **when** they did it, and **with what identity**. CloudTrail has some internal components we need to grasp to properly understand the service:
- **Trails** are the fundamental building blocks of CloudTrail. Thet contain information that details what you would like to monitor and track for that specific trail.
- **Events** are logs that are created every time an API event is called and are stored inside of a given trail. Every event provides insights about what was requested, who requested it, and what was the outcome of the request in terms of permissions and actions
- **Log files** contain the events that a given trail captured and are saved to s3 for archival purposes every short period of time
- Logs can, apart from being stored in s3, be sent to a **CloudWatch logs** log group to view them in that service for analysis and monitoring
- **API activity filters** provide a search and filtering functionality to filter events on a given Trail for its registered logs

#### Consolidating logs from multiple accounts into a single bucket
---
If we're running a **multi-account setup** inside of an AWS organization, checking separate CloudTrail trails individually can be a **significant effort**. The best practice and recommendation is to **consolidate** all CloudTrail logs into a single trail inside of an auditing and compliance account.

### Using the CloudWatch logging agent
We can use the CloudWatch agent to **gather logs from our application** by pointing it at log files on an instance, and send those application logs into a **consolidated CloudWatch log group and log bucket**. This will allow us to have a **centralised place to check application logs** easily, even if our application scales up horizontally.

If we already have the CloudWatch logging agent running on the instance, we just need to **add the log configuration** tho the CloudWatch agent configuration file, and **add the proper permissions** for the instance role to be able to write into our target CloudWatch log groups.

## Auditing and governance
Every organization is subjected to a level of governance and compliance, and this comes hand in hand with **security and compliance auditing**, wether internal or from a third-party. You must ensure you are able to **audit and retrieve information** about organization actions, to prove that you are gathering the proper data and storing it so that, if ever needed, you can **act on a security incident** regarding customer data accordingly.

In this section we will dive into what exactly an audit is, understanding AWS Artifact, securing the AWS CloudTrail service, auditing configuration changes through AWS Config, and maintaining compliance by using the Amazon Macie service.

### What is an audit?
There are many audits an external third-party can run on your organization, like **network security**, **data management**, **remediation management** or **change management**. These audits review your configuration, policies and procedures when handling different domains of your business. These audits are checked agaist **official standards** like **ISO 27001** for security management controls. AWS adheres to many global security and cloud-specific standards like **SOC II**, but also regional data security standards like **FedRAMP** and **HIPAA**.

Different business need to follow different security standards and cerifications depending on what fields they operate in, for example, you need to adhere to **HIPAA** if you handle medical data, or **PCI-DSS** if you store and handle credit card information. Let's discover how some AWS services can help us in the task of security auditing.
### Understanding AWS Artifact
Unlike other AWS services, instead of using AWS Artifact to create other resources in our infrastructure like EC2 or RDS, this service allows us to **download and view AWS security and compliance reports** and online agreements:
- **Reports** that have been carried out by external auditors in AWS and have issued relevant reports, certifications, accreditations and other attestations
- **Agreements** that allow customers to review and accept any agreements made with AWS related to this specific account, or for your organization

#### Accessing reports and agreements
---
Access these reports by opening the **AWS Artifact** service, where we have all of these documents available as necessary. Viewing these documents requires us to sign a *non-disclosure agreement*. with AWS relating to the information within that document.

### Securing AWS CloudTrail
As we have discussed in previous sections, AWS Cloudtrail is a **central service in auditing and compliance tasks** when necessary. As such, we need to **protect the data** that it generates with encryption, and have a way to **verify that the data has not been modified** by an attacker. CloudTrail has easy options for both these tasks that we will discuss.

#### Encrypting log files with SSE-KMS
---
For both the Cloudtrail trail and the archival s3 bucket log we store the entries for future verification, we need to apply **data encryption at rest**. We can do this with an AWS KMS key.

#### Enabling log file validation
---
When forensics investigation of an AWS account needs to be executed, part of that process is verifying that the Cloudtrail data entries have not been modified by a third-party. We can do this by enabling **log file validation** for a specific trail. After this, we'll be able to verify a trail's data consistency from a specific date onwards.

### Auditing configuration changes through AWS Config
With how many services AWS provides and can be part of your application, it's **easy to get lost on the changes** that are constantly running on your existing resources. You could still be **running unnecessary legacy infrastructure** which could represent a security risk, or miss a connection between two of your hundreds or thousands of networking connections that represents a **potential incident**.

Answering these questions is necessary for a security audit, but gathering this information is usually a very long and involved process we need to manually run. AWS Config is the service that helps answering these questions in an **automated**, **auditable** and **compliant** manner. This is thanks to several internal elements of this service:

- **Configuration items**: This represents a *configuration point-in-time snapshot* of a specific AWS resource: metadata, attributes, resource relationships, and configuration options. This is *updated every time the resource changes*.
- **Configuraton streams**: When a new Configuration Item is created, it is added to its Configuration stream, which is essentially an SNS topic inside AWS Config. This enables us to *notify about changes in critical resources* as we see fit.
- **Configuration history**: Thanks to storing Configuriation Items cronologically, we can see a history of the configuration of each resource in our infrastructure. *what* changed, *when* it was modified, and *who* ran that modification.
- **Configuration snapshots**: This element allows us to get a *point-in-time report* of how our entire infrastructure was configured at a selected time, and can be exported on demand as needed.
- **Configuration recorders**: A configuration recorder is the resource that *lets AWS Config gather information* streaming into the service. It is created when configuring the service, and it *allows us to pause AWS Config* if necessary.
- **AWS Config rules**: Config rules allow us to *monitor certain critical resources* inside of our infrastructure and *send us a notification* if their configuration steps *out of our established security standards* to apply *corrective actions*.
- **Resource relationships**: This feature allows us to see a *map of the relationships between resources* in our infrastructure. For example, if we get a notification that a security group has been modified, we can run an *AWS Config resource relationship report* and see which instances this security group is attached to.

#### The AWS config process
---
Here's the outlined process to start using AWS Config in your infrastructure:
1. Open the AWS config service and **create your configuration recorder**
2. AWS Config will identify and **discover supported resources**
3. For any detected supported resources, Config will **create a Configuration Item**
4. Any change detected in your Configuration Items will be sent to your **notification stream**

### Maintaining compliance with Amazon Macie
Amazon Macie is a managed service that uses Machine learning to **detect**, **protect** and **classify** data within your s3 buckets. By continuously running analysis on data access patterns, Amazon Macie can **identify suspicious activity** that could indicate a potential security threat.

#### Classifying data using Amazon Macie
Macie classifies bucket data through a series of automated data detection mechanisms, of which there are currently five:
- **Content type**, identifying the file contents by using the file type headers in each object
- **File extension**, detecting matches with the content type headers to check if the standard file extension is a match
- **Theme**, which detects certain keywords and sets data tags accordingly, like encrypted data, banking or confidential markings
- **Regex**, used to match certain data by common format, like dropbox links, CVE numbers, or private keys
- **Support vector machine-based classifier**, an entirely AWS-managed classifier hidden from the console that reads metadata and document length to classify data as internal tags like financial data, application logs, web languages, encryption keys, and more

After classifying this data, AWS will assign a **threat level** and data impact in case this data triggers a potential leak alert.

#### Amazon Macie data protection
Amazon Macie uses two main sources to train its internal model and detect potential data leaks:
- **AWS CloudTrail events** reviews API calls that have accessed data, and depending on its metadata, it will apply a risk score from 1 to 10
- **AWS CloudTrail errors** tracks access errors to data, like `AccessDenied` object access errors, a high count of which may imply an identity trying to scrape any data it can access, which get a risk score assigned as well