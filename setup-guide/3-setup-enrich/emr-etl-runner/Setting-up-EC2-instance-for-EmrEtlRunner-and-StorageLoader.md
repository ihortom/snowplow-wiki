[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [**Step 1: setup a Collector**](Setting-up-a-Collector) > [**Clojure collector setup**](setting-up-the-clojure-collector) > [[Setting up EC2 instance for EmrEtlRunner and StorageLoader]]

This tutorial assumes it's your first installation and you probably just want to checkout the platform. Thus many steps describe low-performance and unsecured installation. In real-world scenario you may want to fix that.

### Prepare your system

Before getting started you need to have:

- Account on [Amazon Web Services](http://aws.amazon.com/).
- Installed [AWS CLI](https://aws.amazon.com/cli/).
- IAM user, first one need to be created in [AWS Console](https://console.aws.amazon.com/iam/home?#users).
- IAM user need to have attached `AdministratorAccess`.
- Configured credentials on your local machine. (You can use `aws configure` for it).
- For some steps you may want to install [`jq`](https://stedolan.github.io/jq/). It's optional, but handy.

Everything else can be done from CLI.

### Setting up EC2 instance for EmrEtlRunner/StorageLoader

In the end of this step, you'll have an AWS EC2 instance, SSH access to it and key stored on local machine.

**1. Find your Default VPC ID**

We will refer to it as `{{ VPC_ID }}`.

```sh
$ aws ec2 describe-vpcs | jq -r ".Vpcs[0].VpcId"
```

**2. Create Security Group for SSH access**

On output you'll get `GroupId`. We will refer to it as `{{ SSH_SG }}`.

```sh
$ aws ec2 create-security-group \
    --group-name "EC2 SSH full access" \
    --description "Unsafe. Use for demonstration only" \
    --vpc-id {{ VPC_ID }} \
    | jq -r '.GroupId'
```

**3. Add rule allowing SSH access from anywhere**

```sh
$ aws ec2 authorize-security-group-ingress \
    --group-id {{ SSH_SG }} \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

**4. Create SSH key-pair named on the local machine**

We named it "snowplow-ec2" here.

```sh
$ aws ec2 create-key-pair --key-name snowplow-ec2 \
    | jq -r ".KeyMaterial" > ~/.ssh/snowplow-ec2.pem
$ chmod go-rwx ~/.ssh/snowplow-ec2.pem
```

**5. Run t2.small instance with Amazon Linux AMI with previously created SSH-key**

On output you will get your instance id. We will refer to it as `{{ INSTANCE_ID }}`.

```sh
$ aws ec2 run-instances \
    --image-id ami-60b6c60a \
    --count 1 \
    --instance-type t2.small \
    --key-name snowplow-ec2 \
    | jq -r '.Instances[0].InstanceId'
```

**6. Attach security group to Instance**

```sh
$ aws ec2 modify-instance-attribute \
    --instance-id {{ INSTANCE_ID }} \
    --groups {{ SSH_SG }}
```

**7. Check public IP-address of newly created Instance**

Further we will refer to it as `{{ PUBLIC_IP }}`.

```sh
$ aws ec2 describe-instances \
    --instance-ids {{ INSTANCE_ID }} \
    | jq '.Reservations[0].Instances[0].PublicDnsName'
```

**8. Log-in**

Fill-in `{{ PUBLIC_IP }}` from previous step.

```sh
$ ssh -i ~/.ssh/snowplow-ec2.pem ec2-user@{{ PUBLIC_IP }}
```

