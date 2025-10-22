# AWS IAM (Identity and Access Management) Notes

### Architecture Diagram

🏗️ IAM Integration with AWS Services
```
                    +--------------------------+
                    |    IAM (Central Control) |
                    +--------------------------+
                                |
            +-------------------+---------------------+
            |                                         |
    +---------------+                         +---------------+
    |    Users       |                         | Applications  |
    |  (Developers)  |                         |  (EC2, Lambda)|
    +-------+--------+                         +-------+-------+
            |                                          |
            |                                          |
   +-------------------+                   +--------------------+
   | Assume IAM Role   |                   | Assume IAM Role    |
   +-------------------+                   +--------------------+
            |                                          |
   +-------------------+                   +--------------------+
   | Access AWS S3     |                   | Access DynamoDB    |
   +-------------------+                   +--------------------+
```



## 1. IAM User

### Definition
An IAM user represents a human or application that needs long-term access to your AWS account and services

### Key Points
- Has permanent credentials:
  - Username/password for AWS Console
  - Access key and secret key for CLI/API

- Permissions are given via:
  - Directly attached policies, or
  - Group membership, or
  - By assuming a role for temporary access

### Example
- **User**: DevUser
- **Policy**: AmazonEC2ReadOnlyAccess
- **Result**: Can view EC2 instances.

only the permmisions attached to the user are the actions user can perform on the aws 


### IAM Quotas:
- 5,000 IAM users per account.
- 10 managed policies per IAM role/user


## How to Create an IAM User

### Step 1: Open IAM Service
In the top search bar, type IAM and select it.
You will enter the IAM Dashboard.

### Step 2: Navigate to Users
In the left-hand menu, click **Users**.
Click **Create user** at the top.

### Step 3: Configure User Details
- **User name**: Enter a name for the user (e.g., DevUser1).

- **Access type**:
  - *Programmatic access* → For CLI, SDK, API access.
  - *AWS Management Console access* → For console login.
- Set a custom password or let AWS auto-generate.
- Decide if the user must reset the password at next login.

### Step 4: Set Permissions
You have three options:

1. **Attach existing policies directly**
   - Example: AmazonS3ReadOnlyAccess, AdministratorAccess.

2. **Add user to group**
   - If you have groups like Developers or Admins.
   - Users inherit permissions from the group.

3. **Copy permissions from existing user**

- Example: Attach AmazonEC2ReadOnlyAccess if the user only needs to view EC2 instances.

### Step 5: Add Tags (Optional)
- Tags are key-value pairs to help manage users (e.g., Department: Dev).
- Optional, but useful for large organizations.

### Step 6: Review and Create
- Review your settings.
- Click **Create user**.

### Step 7: Save Credentials
- You will see username and console password and Console login URL.
- **Important**: Save these securely. The secret key is shown only once.


--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

## 2. IAM Group

### Definition
A group is a collection of IAM users. You attach permissions to the group, and all members inherit those permissions.

### Example
- **Group**: Developers
- **Policy**: AmazonS3ReadOnlyAccess
- **Members**: DevUser1, DevUser2

Both users get read-only S3 access.

### Use Case
- Easier permission management when you have multiple users with the same access needs.
- when added user to group the user will get both user level and group level permission 
- If a user has a deny permmision of that policy attached to group then it gets more priority 

--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

## 3. IAM Roles and Policies

### 1. What is an IAM Role?
An IAM Role is an AWS identity with permissions but no long-term credentials (like username/password or access keys).

Roles are meant to be assumed by trusted entities (like EC2 instances, Lambda functions, users from another account, etc.).

When a role is assumed, temporary security credentials are provided.

> **Note**: It gives AWS CLI commands permissions without AWS credentials.

---

### 2. Types of IAM Roles

| **Role Type** | **Purpose / Use Case** | **Who Can Assume It** |
|---------------|------------------------|----------------------|
| **Service Role** | Assigned to an AWS service to allow it to interact with other AWS resources | AWS services (e.g., EC2, Lambda, ECS, SageMaker) |
| **EC2 Role (Instance Role)** | EC2 instance assumes this role to access AWS resources without embedding credentials | EC2 instances |
| **Cross-Account Role** | Allows users or services from another AWS account to access your account's resources | IAM users, roles, or services from another AWS account |

--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

## 4. Role Examples

### 1. Service Role

**Purpose**: Let an AWS service perform actions on your behalf.
**Who assumes it**: AWS service itself.

**Example**: Lambda writing logs to CloudWatch.

#### Steps:
- **Role Name**: Lambda-CloudWatch-Role
- **Permissions Policy**: CloudWatchLogsFullAccess
- **Trust Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Use Case**: When Lambda runs, it assumes this role to create logs in CloudWatch.



### 2. EC2 Instance Role

**Purpose**: Allow EC2 instance to access AWS resources without hardcoding credentials.
**Who assumes it**: EC2 instance.

**Example**: EC2 accessing S3 bucket.

#### Steps:
- **Role Name**: EC2-S3-Access-Role
- **Permissions Policy**: AmazonS3ReadOnlyAccess
- **Trust Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```


--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

### 3. Cross-Account Role

**Purpose**: Allow users or roles in another AWS account to access resources in your account.

> **Important**: Cross-account roles are always created in the account that owns the resources (Account A).
> Example: Account A has an S3 bucket, Account B wants access → Role is created in Account A.

**Role is created in Account A with the account ID of B with permissions to access the resources of Account A S3**

**Who assumes it**: IAM users or roles from another account.

**Example**: Dev account accessing Prod account S3 bucket.

#### Steps:
- **Role Name**: ReadS3FromDevAccount
- **Permissions Policy**: Read access to S3 bucket prod-data.
- **Trust Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::DEV_ACCOUNT_ID:user/DevUser" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### How Can the User/Role in Another Account Access Resources?

#### Step 1: Create the Role in Account A
- **Role Name**: CrossAccountS3ReadRole
- **Permissions**: Allow access to specific resources (e.g., S3 read-only).
- **Trust Policy**: Define who can assume it (Account B user or role)

**Trust Policy Example**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::ACCOUNT_B_ID:user/DevUser" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Permissions Policy Example**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::prod-data/*"
    }
  ]
}
```

#### Step 2: User in Account B assumes the role

User in Account B must have a permission policy like:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::ACCOUNT_A_ID:role/CrossAccountS3ReadRole"
    }
  ]
}
```

**Assume role using AWS CLI**:
Account B user (with your policy) runs:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_A_ID:role/CrossAccountS3ReadRole \
  --role-session-name DevSession
```

- STS returns temporary credentials.
- Those credentials have the permissions defined in Account A's role policy (for example AmazonS3ReadOnlyAccess).
- Using those temporary creds, the user can access S3 read-only, because that's what the role's permission policy allows.

### Alternative: Direct Role Attachment

#### In Account B (Assuming Account):
- **Role**: DevAppRole
- **Attach this IAM Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::ACCOUNT_A_ID:role/CrossAccountS3ReadRole"
    }
  ]
}
```

Now, any EC2 instance, Lambda, or service that uses DevAppRole can automatically assume the role from Account A programmatically (no manual CLI required).

---

## 5. When to Create What Role?

It depends on who assumes roles:

### If it's a Service Role (for AWS Service like EC2/Lambda)
```json
"Principal": {
  "Service": "ec2.amazonaws.com"
}
```

### If it's a Cross-Account Role
```json
"Principal": {
  "AWS": "arn:aws:iam::ACCOUNT_B_ID:role/SpecificRoleName"
}
```

### If it's a Same Account Role (IAM user assumes within same account)
You can point "AWS" to an IAM user or role in the same account.
```json
"Principal": { "AWS": "arn:aws:iam::111111111111:user/Santosh" }
```
So the user Santosh can assume this role.

--------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------

## 6. Example: IAM User assuming a Role

#### In Account A:
- **Role Name**: S3AdminRole
- **Trust Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::ACCOUNT_A_ID:user/DevUser" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

- **Permission Policy**: AmazonS3FullAccess

#### In Account A (same or another):
- **User**: DevUser
- **Policy attached**:
```json
{
  "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Resource": "arn:aws:iam::ACCOUNT_A_ID:role/S3AdminRole"
}
```

Now DevUser can:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::ACCOUNT_A_ID:role/S3AdminRole \
  --role-session-name S3AccessSession
```

And then access S3 using temporary credentials from the role.