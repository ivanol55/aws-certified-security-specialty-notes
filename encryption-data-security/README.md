# Encryption and data security
## Introduction
The final section of these notes covers **encryption**. We'll look at different types of encryption used throughout AWS and how to **configure and implement encryption** across a variety of services available to us. Then we'll take a deep dive into **AWS Key Management System (KMS)**. We'll also look into **Amazon CloudHSM**, how this service differs from KMS, and when you should use one over the other.

## Managing key infrastructure
Understanding the management of encryption keys and implementing encryption is **key to understanding AWS security**, as it lays a big role in your infrastructure usage. It is **crucial for data protection** within the cloud, so we need to understand the different methods available to choose the **best option to build secure solutions** in the cloud.

### A simple overview of encryption
In today's data scenario, **having unencrypted data is not an acceptable security risk**. We need to have all the possible control over data, from controlled access with permissions to authenticated identities, and to make sure an attacker with access to the data can't use it without the decryption key. This is necessary to **protect sensitive and confidential information**.

This encryption should be applied both **at rest** when saved to a storage medium like an EBS volume or an s3 bucket, and **in transit** when sending it through the internet, like HTTPS communications, or being sent from an EC2 instance to an RDS database

Encryption uses different **mathematical algorithms** to **transform data from plaintext to ciphertext**. The only wat to decrypt data again is with the keys used to encrypt it in the first place. Encryption keys themselves are **really long strings of text**. The longer the key, the stronger the encryption.

### Symmetric encryption versus Asymmetric encription
There are two main ways of encrypting data: **symmetric** and **asymmetric**. The first one uses an encryption key to protect our data that can then be used to decrypt the ciphertext too. In the other hand, asymmetric encryption generates a **key pair** of a public key and a private key. The public key is used to encrypt your data that anyone can have, which then can only be decrypted by using your private key, which should be closely protected and guarded.

## Exploring AWS Key Management Service (KMS)
 AWS Key Management Service is essential if you want to safely encrypt and decrypt data in your AWS environment at scale. It's a managed service that allows creating, storing, automatically rotating and deleting encryption keys to protect data with native integration with AWS services such as RDS, S3, EBS, CloudTrail and many more.

KMS is a regional service, so you need to be wary of where your resources for this service are created. If you need a key for `us-east-1`, the ones you created in `eu-central-1` are not an option. Furthermore, you should remember that access to these keys is tightly controlled. So much so, that even AWS administrators cannot access them, which means if you delete a KMS key, not even AWS administrators can retrieve them back for you.

It is important to know that KMS is not designed to perform encryption in-transit, but instead used to manage keys for **at-rest data encryption**. in-transit encryption should be handled with other mechanisms, such as Secure Socket Layer

### Understanding the key components of AWS KMS
To understand how KMS works, we need to dive deeper into the different **components** that compose the service. Let's discuss further about:
- Customer master keys
- Data encryption keys
- Key material
- Key policies
- Grants

#### Customer master keys
The CMK is the **main building block** of the KMS service, it contains the key material to both encrypt and decrypt data. KMS supports both **symmetric** and **asymmetric** keys. These two key types have some differences to take into account:
- Symmetric keys can be used to **generate** symmetric data keys in addition to asymmetric data key pairs
- **Importing** your own key material is only supported for symmetric CMKs
- When using a **custom key store**, you can only store symmetric CMKs
- Asymmetric keys can be used either for encryption and decryption **OR** for signing and verification, not both

There are three different types of CMK's used by KMS:
- AWS-owned, which are **used by AWS to manage encryption on some services**, like DynamoDB tables or S3 server-side encryption. We do not have management access to these keys
- AWS-managed, which are the default AWS-managed keys for some services that **we can see, but not manage**, like the default EBS encryption key
- Customer-managed, keys that are **created and managed by us the users**, so we can update, rotate, delete them, modify their policies, or change their rotation period

It is possible to store your CMKs in a **custom key store**, instead of the KMS key store. These custom key stores can be created usen an AWS CloudHSM cluster that you own and manage. Having your own CloudHSM allows you to directly **control the Harware Security Modules** that are responsible for generating key material. CloudHSM will be discussed later in more detail.

#### Data encryption keys
Data encryption keys are generated by CMKs, however, **they do not reside inside KMS service**, instead are used outside of the service to **encrypt your data**. When a request is reveived by KMS to generate a data key, the associated CMK will create **two identical data encryption keys**: one in **plaintext**, one **encrypted**.

The process of using one key to encrypt the other is called **envelope encryption**. During the encryption process the plaintext key will be used to encrypt your data using an encryption algorithm. Once the encryption is complete, the plaintext key will be deleted and the encrypted data key will be stored with the encrypted data.

#### Key material
Key material is essentially the data that is used to encrypt and decrypt data stored in your infrastructure. This key material is stored inside of your CMK. When you create a Customer-managed CMKs, you can choose not to create any key material so you can import it in, but otherwise, it is created automatically with your CMK.

#### Key policies
The main function of a key policy when attached to a CMK is to **define who can use that key** for cryptographic operations like encrypting, decrypting, generating data keys, and administer it with actions such as importing key material, or revoking the key entirely. We can limit access in three main ways:
- **Via key policies**, as described above
- **Via key policies and IAM**, which also applies access rules from IAM policies attached to the identity trying to use the CMK
- **Via key policies and grants**, where access can also be delegated to other identities by creating grants and providing the grant tokens to the target identities

### Exploring AWS CloudHSM
AWS CloudHSM is a managed service used for data encryption. Being fully managed, many aspects of implementing and maintaining the HSM are **abstracted**, such as hardware provisioning, patching and backups. It also **scales** as needed on demand.

HSM stands for **Hardware Security Module**, which is dedicated security hardware and validated to FIPS 140-2 Level 3. This is used to securely create your own encryption keys.

Using AWS CloudHSM is rerquired when additional control and administrative power is needed over encryption than what KMS offers. With certain **compliance and regulation requirements** you might be required to manage an HSM as a cryptographic key store. In addition to key generation, an HSM also allows us to:
- Use **different encryption alhorithms**
- **Management of both symmetric and asymmetric cryptographic keys**, including both importing and exporting keys
- **Signing** and verifying signatures
- The ability to use a cryptographic hash function to calculate **HMAC**s

You can create a CloudHSM cluster and use it as a KMS key store in case regulations don't allow you to store your encryption keys in a potentially shared environment.

#### CloudHSM clusters
When you deploy CloudHSM, it is deployed as a **cluster**, by default 6 modules per account, but can be scaled from 1 to 28. The more HSM count we have, the higher the performance. For **resiliency**, your cluster is set up **across different availability zones** inside of its own **VPC managed by AWS**, to which you communicate through a set of created **Elastic Network Interfaces**.

To communicate with a CloudHSM cluster from unmanaged services, **client software** is needed. This software can be installed in EC2 instances that need to communicate with our CloudHSM through its managed ENIs

#### AWS CloudHSM users
AWS CloudHSM uses different types of **users** that we need to be aware of. **Different users have different permissions**, and these user types are defined as follows:
- Precrypto Office
- Crypto Office
- Crypto User
- Appliance User

Let's get into each one individually with more detail.

#### Precrypto office
When the very first HSM is created within your cluster, the HSM will contain a **Precrypto Office user** that contains a default username and password. When your first HSM is set up, you will need to connect to it and log in to the HSM to **activate** it. The only permissions this user has is to read HSM data and change its own password.

When connecting to the HSM, you will need to use these credentials to change its own user password. Doing this will **activate your HSM** and this Precrypto Office user** becomes a Crypto Office user**.

#### Crypto office
The Crypto Office user has grater permissions than the Precrypto user, as it has the ability to perform some **user management** tasks, such as user creation and deletion or password changes. It can also perform a number of **administrative-level operations** like:
- Zeroize data on the HSM, which allows this user to delete keys, certificates and data from the HSM
- Identify the number of HSMs within the cluster
- Obtain HSM metadata including IP addresses, model, serial number, and firmware and device IDs
- View synchronization satus across HSMs in the cluster

#### Crypto user
A crypto user inside of your HSM is able to **perform cryptographic operations** within the cluster, like:
- Perform encryption and decryption
- Create, delete, wrap, unwrap, and modify attributes of keys
- Signing and verifying
- Generating digests and HMACs
- Zeroize data and basic cluster information such as IP address and serial numbers

#### Appliance user
The appliance user exists in all HSMs and is used to carry out the **cloning and synchronization** actions between your HSM instances. This is **managed by AWS** itself to ensure your cluster is maintained and in sync. From the permissions perspective, the Appliance user is similar to the Crypto Office user, but it can't manage users or rotate passwords.

### AWS Secrets Manager
AWS Secrets Manager is a managed service that offers the ability to **safely store values** that should have controlled access by storing the **encrypted value** as an AWS object, and also providing extended features we can take advantage of like IAM access **granular control**, automatic **credential rotation** integration, or value **versioning**.

## Managing data security
In the previous section we learned about how we can generate encryption keys and key pairs with AWS Key Management Services and AWS CloudHMS. Now that we have these keys, we need to learn how to **properly use them** on the main services that AWS **integrates** with these keys:
- Amazon EBS
- Amazon EFS
- Amazon S3
- Amazon RDS
- Amazon DynamoDB

Let's explore each of these service integrations in more detail.

### Amazon EBS encryption
EBS volumes are used in several of non-managed AWS services, for example in EC2 instances. **These volumes should be encrypted** for security and compliance. When a volume is encrypted, there is **no performance loss** and it has a **very limited effect in latency**. Encryption is **transparent** and applications do not need to be adapted to support encryption.

We encrypt an EBS volume on creation by selecting the Customer Master Key we wanty to encrypt it with, either the default AWS one or one that we created. If a drive or snapshot was created without encryption, we need to **copy** it to an encrypted version of it by **cloning** it in AWS with our desired Customer Master Key. Same procedure if we want to **swap the CMK** a volume or snapshot is encrypted with.

To ensure new volumes are encrypted by default, we can **enable an EC2 setting** that will set new volumes to be set to be **encrypted by default** with the default AWS key for EBS.

### Amazon EFS
Amazon EFS is used for file-level storage in a **fully managed NFS experience** with an elastic scaling of the storage size. 

To protect stored data in this managed service we need to configure a Customer Master Key that we want to use to encrypt data when interacting with EFS, and the managed service will handle everything else to protect data with encryption **both at-rest and in-transit**. This protects **both the data itself and the file metadata** that's attached to the files.

### Amazon S3
S3 provides a fully managed file-level storage solution allowing to save objects up to 5 terabytes of size each. AWS provides **several S3 encryption mechanisms** depending on our requirements for at-rest storage encryption:
- **Server-side encryption with s3-managed keys (SSE-S3)**, which encrypts your files with s3-managed keys, like the default S3 key managed by AWS transparently on the server side. This happens entirely on S3 without KMS interaction.
- **Server-side encryption with kms-managed keys (SSE-KMS)**, encrypting your files using your provided KMS key, transparently on the server side as described on the KMS section.
- **Server-side encryption with customer-managed keys (SSE-C)**, Similar to the last option that encrypts the files transparently on s3, but the client needs to provide the key with the request instead of interacting with KMS. Similar to how KMS works, the encrypion key is encrypted and stored with the data, and the unencrypted key is deleted from memory.
- **Client-side encryption with kms-managed keys (CSE-KMS)**, similar to SSE-KMS, but instead an encryption key blob and encryption key are sent to the client, the objects are encrypted on the client, and that encrypted object blob is sent into the s3 bucket, together with the KMS cipher blob key stored as object metadata
- **Client-side encryption with customer-managed keys (CSE-C)**, Where the data is encrypted on the client side like the last method, but using a customer key instead of a KMS key, storing the encrypted data key as metadata with the object

### Amazon RDS
Amazon Relational Database Service allows us to create managed instances of popular database engines like MySQL, MariaDB and PostgreSQL. Much like other services that store data, **we need to follow compliance guidelines** to keep data protected both at-rest and in-transit. 

When creating a database, similarly to when configuring EFS, we'll **select which KMS key we want to use**, and data encryption at-rest of both the instance and the backup snapshots will be handled by AWS. As for in-transit encryption, each database needs to be configured to support and use **TLS encryption** for connections and data transfer. 

### Amazon DynamoDB
Amazon DynamoDB is afully mnaged key-value and document NoSQL database. It was designed for high-performance across multiple regions. Being fully managed, AWS is responsible for many of the maintenance tasks, such as high availability, patching and backups. Much like RDS and EFS, it comes with easy encryption options for both at-rest and in-transit.

As for encryption at-rest, it is enabled by default in all new DynamoDB tables and handled transparently server-side, and this option cannot be turned off or disabled. Also, unlike other services like RDS, you can swap the KSM key used for encryption at any time. All your data is kept encrypted by default. We can use either de default DynamoDB encryption key, a KMS customer-managed key or a KMS AWS-managed key.

Encryption in transit is automatic, as connection with DynamoDB is made using HTTPS, which encrypts that connecton with TLS. DynamoDB responses also use HTTPS.