# Security responsibility and Access Management
## Introduction
This section introduces you to the *shared responsibility model*: how the duties to keep security management in good condition are divided between AWS and you, and how to keep your security access control up to good condition.

## The shared responsibility model
the **Shared Responsibility Model** is the *guidelines* (not a legal contract) that AWS and you agree to follow to control security conditions on an environment, that generally mean AWS controls the security *of* the cloud, and you handle security *in* the cloud.


AWS has **three** different shared responsibility models we'll explore, each with varying levels of responsibility division:
- For **infrastructure** services
- For **container** services
- For **abstract** services

### The *infrastructure* Shared Responsibility Model
When talking about security *of* the cloud, AWS's responsibility, we talk about:
- AWS **Global infrastructure** (Regions, Availability Zones, Edge Locations)
- AWS **foundation services** (Compute, Databases, Storage, Networking)

On the other hand, we as users need to handle security **in** the cloud, as in, what we build on top of the AWS foundation:
- **Data-side data encryption** and integrity verification
- **Server-side encryption** for data and filesystems
- **Network traffic** protection, encryption and verification
- **Operating system**, **Network** and **Firewall** configuration
- **Platforms**, **applications** and **Identity and Access Management**
- **Customer data** protection


### the *container* Shared Responsibility Model
The word "container" usually refers to a technology that packages a software and all of its dependencies in a full package that can be run in most environments in the same way, like **Docker** or **Kubernetes**. This is not the case here.

With the word *container*, AWS refers to serices where the customer does not have access to some of the provied infrastructure or elements of its low-level operaton like the operatin system, for example **AWS Elastic MapReduce**, **Relational Database Service** or **Elastic Beanstalk**.

In this case the Responsibility model is similar to the *infrastructure* one, but some low-level duties (**Operating system**, **Platform**, **Applications**, **Identity and Access Management** anfirewallNetwork configuration**) are now handled by AWS instead of the user, for example, not having to manually handle OS updates for Relational Database Service instances.

### the *abstract* Shared Responsibility Model
This is the model used for fully managed services available on AWS and thus we do not have access to any infrastructure to manage, we kust interact with an endpoint. This covers services like Amazon **Simple Queue Service**, **DynamoDB** or **S3**.

In this model we just need to handle **customer data security** like client-side encryption and **access control** for users on the customer side, and AWS handles the rest, like server-side encryption or keeping data available between different availability zones

## Access management
Access management is one of the biggest and most important domains of AWS security. It handles *how* and *when* *who* or *what* can access your resources. Here, we will go in depth into **Identity and Access Management** and how we can implement it by provisioning **users**, **groups** and **roles**.

### Understanding the Identity and Access Management service
AWS IAM will probably be the first security service you use when you first set up an AWS account. By default, you log in with our **root user**, which has complete access to the environment and is a very large security risk if its credentials are leaked.

Due to this, AWS best practices indicate that this root user should not have **programmatic access keys** and should be protected with **multi-factor authentication**, and its use should be avoided as much as possible.

Instead you should create **IAM users** that are assigned to **groups**, which in turn have **policies** attached to them to grant the required permissions to them. This is an extra step used to reduce permission management overhead, as permissions on the group are assigned to all users within it.

That is the usual practice for user access. If instead **service** access is required, like an EC2 instance requiring access to S3, users with long-lived credentials are a bad practice. Instead, we'll create another type of identity: a **role**.

Roles are similar concepts to users in the sense tht they are an identity with an attached set of permissions they can use. The difference here is instead of logging in with a role, roles are **assumed** from other identities like **users** (from the same account, or from another *trusted* account), **services** like EC2 instances or Lambda functions, or **federated users**.

### Role types inside the IAM service
There are several different types of roles available, each with different objectives:
- **Service** roles, used within AWS to access one service from another, like reading S3 from an EC2 instance
- **User** roles, so a user on a main management account can access environments in different accounts without having a different user in each trusted account
- **Web identity** federated roles, which are used to give IAM permissions to federated users registered through an identity provider like Google Single sign-on instead of needing an IAM user
- **SAML 2.0** federated roles, used similar to the above option, but for providers like LDAP, for example Microsoft's Active Directory.

### Configuring multi-factor authentication
In addition to passwords, to avoid a credential leak allowing full account takeover to an attacker, every account that has access to the infrastructure should configure a device for **multi-factor authentication**. This ensure that, even with valid credentials for the environment, an attacker cannot access the resources in that account, since they do not have the second authentication device.

This device usually comes in the form of an **application** on your phone that gives you a 6-digit code that **changes** every 30 seconds, so this credential isn't long-lived. **hardware multi-factor authentication** devices are also an option, like Yubikeys.

Multi-factor authentication can also be a selective requirement on certain **policy actions**. For example, allow the user to create EC2 instances normally, but require an MFA device to be present on the account to delete an s3 bucket. This way, even if the MFA device is accidentally deleted from configuration, critical actions become unavailable.

## Working with access policies
We've talked a lot about attaching groups and roles to policies, but we haven't actually detiled what access policies actually are. an **access policy** is a set of AWS **API verbs** that apply to a set of **AWS objects**, like allowing access to listing all contents inside an S3 bucket, or allowing file uploads to a bucket with a specific name and prefix. These controls allow us to manage who can access what, how, and in what conditions.

Remember that AWS permission policies are **implicit deny**: if you didn't expressly specify that you can do a thing, you can't do the thing.

### The differences between policy types
Similarly to the different role types in AWS, we have different policy types for different tasks that we need to understand, each with a purpose:
- Identity-based policies
- Resource-based policies
- Permissions boundaries
- Access Control Lists
- Organization Service control policies

Let's go more in-depth into each one to understand them better.

#### Identity-based policies
---
This is usually the one you'll be most familiar with if you've worked with AWS for a long time. These are the policies you create inside the IAM service, and that allow the identity they attach to to perform a **set of actions** on a **set of resources**. These policies can be of one of the following types:
- **AWS-mamanged** policies, which come created with the account and are maintained by AWS for usual purposes, like read-only access or full control over a certain service
- **Customer-managed** policies, which are policy objects with custom definitions created by us
- **In-line** policies, which are policy definitions embedded directly into the object, either a user or a role.

#### Resource-based policies
---
**Similar to in-line identity policies** in the sense that they aren't their own object, but instead are attached directly to their target, but instad of being attached to identities, they are **attached to resource objects**. These are usually used on resources like **s3 buckets** to configure s3 bucket access policies. 

#### Permissions boundaries
---
Permissions boundaries are a policy declaration configured besides permissions policies. They indicate the **maximum amount of permissions** that an identity can have. For example, if the permission boundary only allows using the s3 service, but the identity policy configures Administrator Access to all services, the user associated with this policy won't be able to start an EC2 instance. **You can only have as many permissions as the permissions boundary indicates**. The rest of the permissions set on the identity policy will just be denied.

#### Access Control Lists
---
ACL's are used in the S3 Service in AWS to control access permissions on a **per-object** basis (as opposed to the resource-based policy, which is bucket-level), and are used to control public and cross-account access to each object. We can select object access configuration for:
- **Other AWS accounts**, based on their account ID
- **Public access**, so anyone with the bucket key can access it
- **S3 Log Delivery Group** as a specific permission to allow s3 server access logging write access, useful in a security and auditing scenario

#### Organization Service control policies
---
These special kinds of policies are **similar to Permissions Boundaries** used to control the maximum level of permissions available, but **used in an Organization environment** to control what allowed actions each account in the organizational unit it's applied to can have.

These are designed to simplify the management of permissions boundaries on a **multi-account organization environment**, to disable unnecessary potential permissions on certain accounts they might not need. As they are applied in cascade using Organizational Unit hierarchy, we can, for example, deny unused services at the root OU level, then fine-grain more blocking rules down the chain.

When the block control is considered, all of the passed down SCP's are compiled and the **block-before-apply policy is applied**, so to have an action verb available in an account, it needs to **not be denied anywhere on the chain**.

### Policy structure and syntax
Identity-based policies follow a certain structure and format to be valid. Here's an example that allows reading and writing to the `bucketexample` S3 bucket:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::bucketexample/*"
        }
    ]
}
 ```

The key fields here are:
- **Version**: Indicates the version of the policy language being used to interpret the policy. At the time of writing this, the latest version is `2012-10-17`.
- **Statement**: A group for the following parameters, that acts as a permission declaration. Each policy can have a number of statement elements inside it, as it's an array
- **Sid**: Sets a statement identification, a name to refer to this statement element as inside of the array. This has to be unique in this policy.
- **Effect**: Indicates what we want to do with the specified actions declared on this policy block. We can either **Allow** or **Deny** the actions.
- **Action**: Array of AWS action verbs we want to apply this policy block to. In this case, s3 API actions to read and write to s3 buckets.
- **Resource**: Specifies what resources this action applies to inside of AWS. In this case, we're limiting our read and write s3 bucket permissions to a bucket named `bucketexample`

We also have resource-based policies, which are essentially the same, but we need to add a **Condition** field to indicate what AWS **Principals** can access that resource:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::bucketexample/*"
            "Condition": {
                "Principal": {
                    "AWS": [
                        "arn:aws:iam::123456789012:user/read-write-s3-user"
                    ]
                }
            }
        }
    ]
}
```

- **Condition**: Establishes a condition to apply this permission. In this case, only the user with ARN `arn:aws:iam::123456789012:user/read-write-s3-user` can use the read/write bucket permission. Several different conditions and comparators can be used here, like checking that the session is authenticated with an MFA device.

### Cross-account access
As explained earlier on the section about assuming roles, an identity in one account can access resources with certain permissions in **other accounts**. In this process we have **two accounts**: the **Trusting**, and the **Trusted**, where the user in the Trusted account **assumes** a role in the Trusting account.

The steps to create a cross-account access role are:
1. **Create a role** on the trusting account with the desired permissions
2. Set "Another AWS ccount" as the **trusted identity**, and input the trusted account's **user arn** that will be able to assume this role
3. Create and attach a policy to the user on the Trusted account that allows them to assume that role on the main account with **Action** `sts:AssumeRole` and **Resource** being the role ARN you just created

Now the user on the Trusted account will be able to **Assume the role** in the Trusting account and see its resources.

### IAM policy management
As time goes on and your environment grows, policy volume will get harder to control, so remember the tools at your disposal to see where a policy is used and how it's configured. For each policy on the AWS console we have different tabs:
- **Permissions** indicates what permissions are configured for this policy, divided by AWS service
- **Policy usage** displays which identities are using this policy, wether it's users, groups or roles
- **Policy versions** displays the laast 5 versions of a policy, and we can set which one is used, and allows for quickly rolling back mistakes when configuring a permission set
- **Access advisor** displays information about policy usage events, which can help identify when a permission is not needed anymore and can be limited further to avoid potential security incidents in case of a credential leak

### Policy evaluation
Several steps are involved each time a policy permission is evaluated. Here's the steps followed in order:
1. **Authentication**, where AWS determines the `Principal` of a request and checks if it's allowed to use this policy
2. **Determining request context**, where the request is processed to check what parts of the policy should be applied, based on what actions are being requested and over what resources. In this step AWS checks **principals**, **environment data** and **resource data**
3. **Evaluating the policy**. Here AWS determines the different policy types being used and applies them in order. The policy evaluation order is
    1. **identity**-based
    2. **resource**-based
    3. IAM **Permissions boundaries**
    4. Organizations **Service Control Policies**
4. **Returning the permission result**, step in which, based on the policies considered, AWS will either *allow or deny the request*.

## Federated and Mobile access
Sometimes access to an AWS account needs to scale to hundreds or even thousands of users, and IAM user management **stops being a feasible task**. At some point, even a *possible* one due to service limitations, or maybe millions of users in a game that need to save progress in one of your databases? that's where Federated and Mobile access comes in.

### AWS Federated access
AWS Federated access allows to delegate identification of users to **external identity providers**, like for example Google Single Sign-On or Microsoft Active Directory with **SAML federation** or other forms of **social federation**. This removes the load of handling user identities through the `Identity and Access Management` service.

Once this set-up is configured, users will be able to access the AWS console through their identity provider with the AWS login portal, and upon login, will assume the **SAML 2.0 Federation**-type role we created and they will be able to use the role's permissions as expected. The same action can be configured using **Social identity providers**, like Facebook or Google (the "Login with Google" buttons are usually this)

Another option for authentication control is the AWS service **Amazon Cognito**, which was built specifically to handle millions of users accessing your application with secure log-in routines with security features like MFA device requirement enforcing, and once logged in, automatically generate **access tokens** you can handle through your application to gran them the necessary permissions. These accounts do not give access to AWS resources themselves, as they are intended for **application authentication**.

### Amazon Cognito Mobile access
Amazon cognito has two main internal elements that we need to understand: **user pools** and **identity pools**. The first one are, in essence, user directories that scale as we need them, granting access to existing users or allowing users to federate their access through social identity providers, and the user pool stores a **profile** for every user inside it. It also allows creating identity pool groups, assigning different permissions to each group as needed.

On the other hand, **identity pools** are used to connect a certain user with the AWS services your user needs to access. It does this by providing your user pool-issued user authentication token and returning temporary credentials to access these services needed by your application. User pool authenticates your users, identity pools give that user credential service access.

Authentication from the user to the user pool can be skipped by the use of **social login** identity providers, where the user logs into social login, and the social logins authenticate to the user pool and return a token, acting as a middle-man in the process.