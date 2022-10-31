# A layered approach to security
## Introduction
This section covers an in-depth explaination on securing an **EC2-based environment**, from running vulnerability scans, limiting access with your firewall, setting up routing, network access control lists, all the way to isolating an instance in case of an incident. Applying the defenses explained in this document will **help reduce the impact** that an incident may have on your infrastructure.

## Securing EC2 instances
Elastic Compute Cloud services are the most widely used AWS service, in part because of the flexibility it allows us to have starting any virtual machine we might need. But the service being an **Infrastructure as a Service** offering, it means we have the most security responsibilities in the shared responsibility model variants. For this, we need to learn to properly protect our environment as best as possible. Fortunately, AWS gives us options to make this easier at scale.

### Vulnerability scanning with Amazon Inspector
Amazon Inspector is a managed AWS service that allows us to run **vulnerability scans** on our environments via an installed agent. Scans are automatically run and compared to a rule package that checks for compliance, from **CIS benchmark** scanning to **behavioral analysis** of your instance. At the end of an assessment, a **report** is exported about the findings.

To configure which targets you want to run the Inspector vulnerability scans on:
1. Install the Amazon Inspector agent on the target instances
2. Configure an assessment target by providing instance tags that will mark instances that should be scanned
3. Create an Assessment template linked to an assessment target, set the timeframe to run your scans and against which benchmark

After this, Amazon Inspector will start running assessments against all marked instances, and reporting any findings it comes across through its findings dashboard UI. We can then filter findings by severity, date or targets, and see a detail for each finding.

### Creating and securing EC2 key pairs
To securely access instances in an EC2 environment we use ***key pairs**. These are composed of a public key and a private key, and allow us to securely connect to these running resources in different ways. We can create them either at the time of creating an instance, or independently through the EC2 console. 

We can use a single key for multiple instances, but each instance can only have one attached key pair. We can also delete key pairs from AWS, but this does **not** delete the key from the instances that used it on creation, it just means it can't be used for new instances.

These keys can be used to access linux instances by using the `SSH` protocol by providing the ssh private key, or to windows instances through `Remote Desktop` by having to provide the private key to decrypt and retrieve the password for `Administrator`.

### Isolating instances for forensic investigation
#### Identifying targets
---
When you identify that an instance may have been compromised, this resource needs to be **isolated** for further investigation to make sure the potential security issue does not extend to further resources inside the organization. 

First, monitoring and logging. AWS provides several services and tools to implement logging and monitoring so we can check for abnormal behavior. The more information we have, the faster we can act on an intrusion. Our main toolset consists of:

- **AWS CloudTrail**, used to identify who and when called what AWS API task
- **AWS Config**, providing resource *inventory*, *change history*, *change user correlation* with CloudTrail, resource *relationships* and *compliance checks* against known standards of cloud security
- **Amazon CloudWatch**, which gathers resource usage from your instances (CPU, RAM, Disk space), and application log collection
- **VPC Flow Logs**, used to capture all traffic moving through your VPC between any network interfaces in it.

#### Isolating targets
---
Once the compromised instance is identified, we need to proceed with **isolation**. It needs to be removed from your production network, that way we'll make sure it can't access any more sensitive resources. The steps to isolate a compromised instance consist of:

- Create a **forensics security group** that allows you to quickly change the instance network access permissions, limiting the instance access to only the ports we'll use to access the instance, and only from known IP sources.
- Create a **snapshot** of the EC2 instance, allowing it to restore it in a secure environment if future analysis is needed
- If possible, take a **memory dump** of the instance
- Store a snapshots of the instance **logs**
- If needed, create a **limited-access s3 bucket** to store the memory dump and logs
- Create a new **separate**, **isolated** Forensics VPC
- Enable **logging**, such as VPC flow logs

The keys in an incident of this kind are to disconnect the instance from the production environment as **quickly** as possible, and **avoid making any changes** to preserve the state of the affected resource for future investigation.

### Using Systems Manager to administer EC2 instances
AWS Systems Manager is a very powerful AWS service that allows us to **administer and manage EC2 resources at scale** by the use of routines and playbooks without the need to open SSH or RDP port, which removes an attack vector from a security and compliance standpoint. This service also allows us to see the our EC2 service security compliance from a single **dashboard**.

Systems managers works by using **resource groups**, which are essentially groups of potential targets updated dynamically based on resource types and tags. These groups make it far easier to run batch tasks on a group of resources that need it, such as applying security patches.

#### Built-in insights
---
We can use Systems Manager Built-in insights to quickly check the **compliance state** of the resources in our infrastructure. We can currently check the following about resources in a given resource group:
- **Configuration compliance and history**, to see any misconfigurations we might have introduced
- **CloudTrail**, used to check any API interactions that have been executed against these resources
- **Personal Health Dashboard** to see if there are any present AWS outages affecting these resources specifically
- **Trusted Advisor**, which recommends potential improvements we can apply to upgrade cost optimization, performance, security or fault tolerance of a given resource

#### Actions
---
We can use AWS Systems Manager `Actions` to run a set of tasks against a given resource group, like applying a security patch, upgrading our operating system, or configuring maintentance windows. The available features for these actions are:

- **Automation**: Allows defining and running routine tasks on target systems, like applying the latest security patches every morning on a given resource group
- **Run command**: Similar to the `Automation` section, but intended for one-off or non-recurring maintenance tasks, like updating the `SSM Agent` on all instances
- **Session Manager**: Allows controlled, logged access to resources on our infrastructure, like allowing users to open a shell session on an istance without opening any ports to it
- **Distributor**: Allows creating packages, like software, that we can deploy to all target instances, like installing the Kinesis agent on a given group of instances
- **State Manager**: Allows using constant checks and desired state declarations to ensure resources avoids drift from our desired state:
  - Network settings
  - Software installation on startup
  - Keeping agents updated
  - Joining Windows agents to a domain
  - Running scripts on instances
- **Patch Manager**: Scans instances for missing security patches we should install, and apply these patches at scale with a few clicks, or patch them automatically as soon as patches are availble, after a certain time window, or on each resource's `maintenance window`
- **Maintenance Windows**: Allows establishing time windows for each resource where actions can be executed, for example when we have less running traffic on our infrastructure as to not affect production tasks with our maintenance routine

## Configuring infrastructure security
AWS provides a simple way to set up a Virtual Private Cloud to run resources inside of, but we may want more control over the creation process to properly configure the resources for security hardening, such as **Network Access Control Lists**. We'll have a look at what exactly a VPC is, how it works, and how to properly set it up.

### Understanding a VPC
A VPC is a **Virtual Private Cloud**, a type of AWS resource we use to isolate our environment inside of a slice of the AWS network and allow network segmentation in the necessary components for proper network security and compliance. A VPC essentially emulates the local area network you might have in your local datacenter.

The ideal VPC design separate resources in different subnets, some **public** (with public IP's on resources, and traffic routed directly through an *internet gateway*) and some **private** (where resources don't have a public IP, routing traffic through a *NAT gateway*), each public-private subnet pair in a different **availability zone**.ç

#### Creating a VPC using the wizard
---
We can create a VPC easily using the AWS-provided wizard, which sets up using defaults derived from some of the variables we provide, like the network CIDR, then sets up all of the necessary components inside the VPC, like subnets, NAT gateways, internet gateways, network ACL's and route tables.

#### Understanding the VPC components
---
VPC's are not really a single AWS element as is a virtual container with a group of AWS services that together are used to provide this service inside it. It consists of:
- **Subnets**, which delcare network ranges inside the VPC's `CIDR` and hold network-bound resources like EC2 instances. Apart from the first (network) and last (broadcast) IP's, AWS reserves the first 3 IP's in the range, for routing, AWS DNS, and one for future use
- **Internet Gateways**, which act as an exit point from the resources inside public subnets into the internet, attached to the VPC in general (we can also connect via a VPN connection to local datacenters with a Virtual Private Gateway)
- **NAT Gateways**, used to route traffic from private subnets into the internet running the task of public/private range translation, acting as a router , attached to a specific subnet
- **Route tables**, which are attached to subnets to define what route table the resources inside that subnet know and can use
- **Network Access Control Lists** control IP-level traffic flow from and to an AWS VPC subnet, like allowing or denying a certain IP range in or out for a certain port. The different rules inside a Network ACL are sorted by a priority number and applied by that order.
- **Security groups** handle resource-level traffic control at the port level. We allow specific IP's and Ports to send traffic into or out of a certain resource, like an EC2 instance or a NAT gateway

#### Creating a multi-subnet VPC manually
---
The steps to manually build a fault-tolerant Virtual Private Cloud on AWS are the following:
1. **Create a VPC**, selecting our network CIDR for the base networking. This will create a default Route Table and Network ACL for this VPC.
2. Create a **subnet** with a corresponding separate CIDR block (subdivision of the VPC CIDR) for public resources, another one for private ones, and name them accordingly. There is **no public/private distinction** in AWS subnets, this is defined by their route tables. Remember to put them in different **Availiability zones**.
3. **Create an internet gateway** and attach it to our VPC
4. Create a route table for our public subnet, which routes traffic for our VPC `CIDR` **locally** and outbound traffic to `0.0.0.0/0` through our **internet gateway**.
5. **Associate** this public route table to our *public-named* subnet
6. **Create a NAT gateway** to route private network traffic to the internet. choose to associate an Elastic IP with it. This NAT gateway is deployed in one of your **public subnets**.
7. Create a route table for our private subnet, which routes traffic for our VPC `CIDR` **locally** and outbound traffic to `0.0.0.0/0` through our **NAT gateway**

After this, we can create security groups with according limitations in one of our subnets and launch resources attached to them, and they will be able to communicate with the internet as required.

If we want to further protect our VPC from unwanted traffic, we can create a Network Access Control List which only allows traffic from our desired IP ranges and to certain ports, like `80` and `443` and attach it to our subnets.

## Implementing application security
Public applications that you host on AWS are often the target of intrusion attempts and attacks. You can use AWS services like `Web Application Firewall` to protect your environment proactively from attacks. We will create **rules** and configure **rule groups** (either ours for known attacks on our platform, or baseline protections like managed OWASP rulesets), attach them to **Web Access Control Lists**, and protect our access points like **CloudFront distributions** and **Application load balancers** with them.

### Protect your resources with AWS WAF
The steps to protect our infrastructure with the **AWS Web Application Firewall** service are:
1. Create a **Web Access Control List** we will attach rulesets to
2. **Associate** this Web ACL with the resources we want to protect, like a CloudFront distribution
3. **Add rules** (or rule groups) to this Web ACL. We can either use our own rules, or use **AWS managed rulesets** to protect against known attack surfaces, like cross-site scripting attempts. Choose the action to take if a rule is triggered (block the request? ask for a captcha?)

This way we'll protect requests to our environment with the `AWS Web Application Firewall` service.

### Using AWS Firewall Manager
Firewall Manager is a similar tool to AWS WAF, but dedicated to managing Web Application Firewall resources **across an entire AWS Organization**. To use this service, add your account to your AWS organization and select which account should act as a **Firewall Manager Administrative Account**. From then on, Firewall Manager resources will be managed from that account.

From there, we can create **standard** ruleset configurations for AWS WAF in all target accounts amd resource groups we will target. We will establish if our rules are added **first** or **last**, and every time WAF ACL's are created in an account, our organization-level rules will be attached to them.

### Managing Elastic Load Blancer security configuration
Elastic Load Balancers are a fundamental resource in AWS used to expose resources inside our infrastructure towards the open internet. They allow us to automatically scale capacity and load balance our requests between all of our targets, and they need to be protected accordingly.

#### Types of Elastic Load Balancers
---
We have three main types of Load Balancers in AWS:
- **Application Load Balancers**, used typically to redirect traffic when exposing web applications through HTTP or HTTPS. Offers several useful features like TLS termination and advanced routing based on request information
- **Network Load Balancers**, used in cases where a lower level of routing control is preferred in exchange of a higher performance, lower latency environment, These load balancers can easily support millions of incoming requests per second
- **Classic Load Balancers**, the classic AWS Load Balancer type, kept around as a legacy element that is now replaced by the other two types and is not recommended for use.

#### Managing encrypted requests
---
Load balancers can be either **internal** or **external**. Internal load balancers only listen to requests from inside of your VPC, but external load balancers listen to requests from the internet and resolve to a public IP accessible from the outside by default, so communication with our web applicatons should be **encrypted** using TLS.

We can configure a TLS access on our load balancer by creating a listener on port `443` and attaching a valid **certificate** to this listener. This will allow external clients to connect with HTTPS to your application through targets. TLS does not need to be handled by your application, as the Load balancer handles **TLS termination**. We can either bring our own certificate or create one with **Amazon Certificate Manager**.

### Securing an AWS API Gateway
AWS API Gateway is an AWS service that allows us to write a serverless API using AWS tools like lambda functions. This API access needs to be protected with **authentication** and **authorization** to prevent fraudulent access.

#### Controlling access to APIs
---
We can have different API endpoint **requirements depending on what this endpoint is used for**. For example, an API endpoint to request an authentication token needs to be public, since we need to be able to access it publicly to get our validation. But looking at your user profile should be an authenticated and authorized task, and thus needs to be handled properly. AWS provides **several methods** to handle these tasks:
- **IAM roles and policies**: We can attach roles and policies to a user authenticated to an API with IAM key ID and secret key pairs
- **IAM Tags** To further control if a user has access to an API endpoint, we can include conditionals in their policy to only allow them to access endpoints tagged in a certain way
- **Resource policies**: Instead of allowing a certain user in their role to use API endpoints, we can reverse the process and set an API resource policy that only allows certain identities to call this API. For example, certain CIDR's, VPC endpoints or IAM users/roles.
- **VPC Endpoint policies**: Similar to resource policies, we can permit invoking our API only through a certain VPC endpoint, and thus resources without access ti that VPC endpoint cannot make calls to the API. This ensures we privately control access to the API.
- **Lambda authorizers**: We use a lambda function to authorize and build the resulting policy for the user that authenticated into the API based on a proper access token sent to interact with the API further.
- **Amazon Cognito user pools**: Allows us to identify what resources a user has access to based on their user or group in a Cognito user pool in our account, authenticating using their available identity provider.

## DDoS protection
Distributed Denial of Service attacks are very common, and if executed correctly, can **bring down your infrastructure** and **stop your service** partially or completely if the attack capacity is large enough, costing a significant sales volume. Here we will look at AWS offerings to protect ourselves from these kinds of attacks with AWS shield and the AWS DDoS Response Team.

### Understanding a DDoS attack
During a Distributed denial of service attack an attacker will send **large amounts of traffic** towards your infrastructure, in an attempt to overload its capacity to answer to legitimate requests and thus stop service. The additional traffic is designed to overload the infrastructure that serves the application. There are several types of DDoS attacks:
- **SYN floods** attempt to open ongoing connections with the target system, but never answer to complete the sessions, thus exhausting the amount of available sessions for the web server and stopping service for new sessions.
- **HTTP floods** sends a very large amount of `GET` or `POST` requests to the server, so it has to use all of its processing power to answer them, gets overwhelmed and stops service.
- **Ping of Death** sends very large ping packets to the target that are larger than the maximum allowed size of a packet allowed by the ping spec, which results in a memory overflow in the target, hindering network performance as it handles and processes these large amounts of data.

### Protecting your environment using AWS Shield
AWS Shield is a amanged AWS service that **helps mitigate DDoS attacks** to your environment. It is divided between two available tiers:
- **AWS Shield Standard**: Available at **no additional cost** if you already have the AWS account. It automatically protects your infrastructure in an always-on model that detects potential typical DDoS indicators to mitigate them.
- **AWS Shield Advanced**: This tier adds, at a cost, further protection from DDoS attacks by extending monitoring to application traffic and supports **defending against large-scale DDoS attacks**. Incase of an attack that scales infrastructure, AWS Shield Advanced also comes with **cost protection** to reduce potential costs. This tier includes support from the AWS DDoS Response Team, who will assist on responding to and resolving this attack to your infrastructure.

## Incident response
As much defenses as you set up, as much as you try to limit what an attacker can do in your environment, incidents happen. Maybe a **misconfiguration** slips in, or an external malicious attacker manages to surpass your defenses. It is **impossible to avoid all security incidents**. Because of all of this, we beed to implement **incident response** policies for scenarios that appear.

### Where to start when implementing effective Incident Resonse
The most crucial step when responding to a security incident is preparation. Your staff needs to be properly prepared and trained to distinguish the criticality of a security incident, and there needs to be a plan of action for all standard potential attack paths you could see in your infrastructure. You can't prepare for everything, but for basic por clear attack paths, having a clear structure of incident response steps will greatly improve your response to this kind of attack.

Over time, these security runbooks will be adapted, extended and improved upon. Your staff should be properly trained in the usage of all of the security team's tools that are needed to respond to these incidents. Yo should also ensure that anyone who might need these resources has access to them before the incident happens, as this will remove back-and-forth and improve response time.

AWS's cloud adoption framerwork establishes four main controls to judge your readiness for cloud incident response:
- **Directive controls** *establish* a **governance**, **risk** and **compliance** model all of the security standards need to operate inside of
- **Preventive controls** *protect* you before an attack by mitigating risks and vulnerabilities 
- **Detective controls** *provide visibility* and transparency over the operation of your AWS resources
- **Responsive controls** *establish how to remediate* incidents or potential deviations from your security baselines

### Making use of AWS features
AWS provides a wide scope of general tools, features and services to help with these four domains of incident response tasks for proactive security monitoring, that we can approach from a security perspective to ease this protection task.

#### Logging
---
AWS offers a wide range of services and features to store and analyze logs from your infrastructure, from **investigative mesures** to **proactive security**. This is often overlooked and then regretted, as **having this data after an incident is incredibly valuable**. Logging provides a way to understand when your infrastructure is **normal**, and that helps identify abnormal behavior to identify incidents. Some examples of very useful logging services are **AWS CloudTrail logs**, **S3 access logs**, **CloudWatch logs**, and **CloudFront access logs**.
#### Threat detection and management
---
AWS offers services that **detect potential threats based on learned behavior** through machine learning and artifical intelligence. Similar to what was mentioned on the logging section, we can identify abnormalities by understanding what is normal. Such a service is **AWS GuardDuty**, which learns behavioral patterns from your infrastructure and alerts about potential threats. These services can also integrate with each other to centralize information, like sending GuardDuty findings to **AWS Security Hub**. Insights like:
- AWS users with suspicious activity
- S3 bucket with an unusual read/write volume
- EC2 intances using permissions they do not usually call for
### Responding to an incident
Now that we have an understanding on what the AWS security standards are and how we can gather forensics information and analyse findings, we can move to checking what actions we could take when an incident occurs
#### A forensic AWS account
---
As mentioned earlier in this section, isolation of the affected resource is key to not let an incident extend. We could create a different VPC on the same account, but the best practice is to have a **completely separate AWS account** we can run our analysis on. This ensures the resources on the source account are completely protected from potential infection. This also ensures any actions you take on the environment are **audtiable** through AWS CloudTrail.

#### Collating log information
---
Be sure you are able to **understand the logs** that are stored and will be used for potential analysis, so you can process them with your tooling and that you are able to read information inside these logs, as **understanding vital information quickly is key** for you to be able to respond to incidents and prepare proper reports. Also prepare to be able to **share** these logs between environments and parties as needed, to centralise your incident response dashboard so everyone has the same information available.

#### Resource isolation
---
If an instance is identified as being potentially compromised, we need to isolate it from the resources it could potentially access and move it to the isolated environment for further investigation. The process to do this is:
1. **Modify the security group** of the instance so that no unknown traffic can access that resource
2. Potentially, **remove the instance role** associated to the instance to avoid unwanted API calls
3. **Make an AMI** of the potentially compromised instance
4. **Share this AMI** with the forensics account
5. **Start a new instance** with the compromised instance AMI as a source on the forensics account
#### Copying data
---
Similar to the process established above, if we want to run analysis tasks in a storage medium like an **EBS volume**, we can create a snapshot in time of that storage object and share and copy it to the forensics account for mounting and further analysis in our forensics account.

#### Forensic instances
---
To analyze the information gathered from a security incident, it is very useful to run these checks from a common **instance that has all of the necessary tooling** to run these forensics analysis tasks. You should also further harden the security configuration of this instance and set up a strong auditing policy so there is **no doubts about potential data manipulation** if the data needs to be presented to custody or legal cases.

### A common approach to an infrastructure security incident
We can outline a generalized incident response and forensics analysis process we can follow for most incidents:
- **Capture**: Store as much potential *instance metadata information* you may need later before doing any potentially modifying tasks
- **Protect**: Enable *termination protection* to avoid the compromised instance being deleted and losing evidence
- **Isolate**: *Block potentially unwanted traffic* to and from the instance by changing the security group or updating the Network ACL's
- **Detach**: *Remove the instance from any autoscaling group* it's a part of, if applicable
- **Deregister**: If the instance is associated from any Elastic Load Balancers, *remove it from associated target groups* 
- **Snapshot**: Take snapshots of all necessary data from the storage volumes, and from the entire instance
- **Tag**: Highlight that resources that you created are for forensic investigation

## Securing connections to your AWS environment
This section will go over the several methods available to securely connect to your AWS infrastructure, be it as a user point of view, or from a hybrid cloud perspective from an on-premise datacenter

### Understanding your connection
Depending on your access needs to establish a secure connection between AWS or you, wether it's for your users or for your on-premise or for your infrastructure, you will have to choose one of the better fitting offerings from AWS. In this case, you need to decide if your best fit is **AWS VPN** or **AWS Direct Connect**.

### Using AWS VPN
With an **AWS site-to-site VPN**, a secure connection is established **between the on-premise location and your VPC** through a **VPN Gateway** (on AWS) connected to a **Customer Gateway** (on your on-premise location), the connection itself consisting of two fault-tolerant IPSec tunnels. After this connection is set up, you need to set up **routing** on both sides so you can properly communcate resources from one side to another. Remember to also update your security groups to allow access accordingly, as the IP sources are now your on-premise location. 

This connection is **limited to VPC resources**, and external services like S3 will still go through the public internet.

### Using AWS Direct Connect
This is a similar solution to using AWS VPN, with the difference that **communication does not occur through the public internet**, but is instead directly routed through a fiber connection through an **AWS Direct Connect location**, which is a datacenter set up by a **Direct Connect partner**. This solution comes with a significant investment in infrastructure, as you need to physically connect to a partner datacenter or colocation. This solution **allows access to services other than your VPC** through your private connection.

#### Controlling Direct Connect access using policies
---
As with other services in AWS, access to Direct Connect can be managed through **policies**. We can manage access to configuration of Direct Connect elements like we can for any other service, only allowing connection management to certain accounts or roles.