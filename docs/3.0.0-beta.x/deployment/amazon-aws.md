# Amazon AWS

This is a step-by-step guide for deploying a Strapi project to [Amazon AWS EC2](https://aws.amazon.com/ec2/). This guide will connect to an [Amazon AWS RDS](https://aws.amazon.com/rds/) for managing and hosting the database. Optionally, this guide will show you how to connect host and serve images on [Amazon AWS S3](https://aws.amazon.com/s3/).

Prior to starting this guide, you should have created a [Strapi project](../getting-started/quick-start.md), to use for deploying on AWS. And have read through the [configuration](../getting-started/deployment.md#configuration) section.

### Amazon AWS Install Requirements and creating an IAM non-root user

- You must have an [Amazon AWS](aws.amazon.com/free) account before doing these steps.

Best practices for using **AWS Amazon** services state to not use your root account user and to use instead the [IAM (AWS Identity and Access Management) service](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html). Your root user is therefore only used for a very few [select tasks](https://docs.aws.amazon.com/general/latest/gr/aws_tasks-that-require-root.html). For example, for **billing**, you create an **Administrator user and Group** for such things. And other, more routine tasks are done with a **regular IAM User**.

#### 1. Follow these instructions for [creating your Administrator IAM Admin User and Group](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)

- Login as **root**.
- Create **Administrator** role.

#### 2. Next, create a **regular user** for the creation and management of your Strapi project

- Copy your **IAM Users sign-in link** found here: [IAM Console](https://console.aws.amazon.com/iam/home) and then log out of your **root user** and log in to your **administrator** user you just created.
- Return to the IAM Console by `searching for IAM` and clicking or going here: [IAM Console](https://console.aws.amazon.com/iam/home).
- Click on `Users`, in the left hand menu, and then click `Add User`:
  1. In the **Set user details** screen:
  - Provide a **User name**.
  - **Access Type**: Check both `Programmatic access` and `AWS Management Console access`.
  - `Autogenerate a password` or click `Custom password` and provide one.
  - **OPTIONAL:** For simplicity, `uncheck` the **Require password reset**.
  - Click `Next: Permissions`.
  2. In the **Set Permissions** screen, do the following:
  - Click `Create group`, name it, e.g. `Developers`, and then choose appropriate policies under **Policy Name**:
    - search for `ec2` and check `AmazonEC2FullAccess`
    - search for `RDS` and check `AmazonRDSFullAccess`
    - search for `s3` and check `AmazonS3FullAccess`
    - Click `Create group`
  - Click to `Add user to group` and check the `Developers` group, to add the new user.
  - Click `Next: Tags`.
  3. **Add tags** (optional)
  - This step is **optional** and based on your workflow and project scope.
  - Click `Next: Review`.
  4. **Review**
  - Review the information and ensure it is correct. Use `Previous` to correct anything.
  - Click `Create user`.
  5. **Success** - These are very **IMPORTANT CREDENTIALS**
     _If you do not do these steps you will have to reset your `Access key ID` and `Secret access key` later._
  - `Download the .csv file` and store it in a safe place. This contains the user name, login link, Access key ID and Secret access key.
  - **OPTIONAL:** Add these credentials to your \*Password manager\*\*.
  - Click on the `AWS Management Console Access sign-in link`. This will log you out of `Administrator`.

#### 3. `Log into` your **AWS Management Console** as your `regular user`

You may now proceed to the next steps.

#### Additional IAM User Resources

- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html).
- [Instructions to reset Access key ID and Secret access key](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html).

### Launch an EC2 virtual machine

Amazon calls a virtual private server, a **virtual server** or **Amazon EC2 instance**. To use this service you will `Launch Instance`. In this section, you will **establish IAM credentials**, **launch a new instance** and **set-up primary security rules**.

#### 1. From your **AWS Management Console** and as your **_regular_** user

- `Find Services`, search for `ec2` and click on `EC2, Virtual Servers in the Cloud`

#### 2. **Select Appropriate Region**

In the top menu, near your IAM Account User name, select, from the dropdown, the most appropriate region to host your Strapi project. For example, `US East (N.Virginia)` or `Asia Pacific (Hong Kong)`. You will want to remember this region for configuring other services on AWS and serving these services from the same region.

#### 3. Click on the blue `Launch Instance` button

- `Select` **Ubuntu Server 18.04 LTS (HVM), SSD Volume Type**
- Ensure `General purpose` + `t2.small` is `checked`.
  ::: tip
  `t2.small` is the smallest instance type in which Strapi runs. `t2.nano` and `t2.micro` **DO NOT** work. At the moment, deploying the Strapi Admin interface requires more than 1g of RAM. Therefore, **t2.small** or larger instance is needed.
  :::
- Click the grey `Next: Configure Instance Details` and `Next: Add Storage`
- In the **Step 4: Add Storage** verify the `General Purpose SSD (gb2)`, then click `Next: Add tags`.
- In the **Step 5: Add Tags**, add tags to suit your project or leave blank, then click `Next: Configure Security Group`.
- In the **Step 6: Configure Security Group**, configure the `security settings` as follows:
  - **Assign a security group:** Check as `Create a new security group`
  - **Security group name:** Name it, e.g. `strapi`
  - **Description:** Write a short description, e.g. `strapi instance security settings`
  - You should have a rule: **Type:** `SSH`, **Protocol:** `TCP`, **Port Range** `22`, **Source:** `0.0.0.0/0` (all IP addresses). If not, add it.
  - Click the grey `Add rule` to add each of these rules:
    - **Type:** `SSH`, **Protocol:** `TCP`, **Port Range** `22`, **Source:** `::/0`
    - **Type:** `HTTP`, **Protocol:** `TCP`, **Port Range** `80`, **Source:** `0.0.0.0/0, ::/0`
    - **Type:** `HTTPS`, **Protocol:** `TCP`, **Port Range** `443`, **Source:** `0.0.0.0/0, ::/0`
    - **Type:** `Custom TCP Rule`, **Protocol:** `TCP`, **Port Range** `1337`, **Source:** `0.0.0.0/0` **Description:** `Strapi for Testing Port`
      These rules are basic configuration and security rules. You may want to tighten and limit these rules based on your own project and organizational policies.
      ::: tip
      After setting up your Nginx rules and domain name with the proper aliases, you will need to delete the rule regarding port 1337 as this is for testing and setting up the project - **not for production**.
      :::
- Click the blue `Review and Launch` button.
- Review the details, in the **Step 7: Review Instance Launch**, then click the blue `Launch` button. Now, you need to **select an existing key pair** or **create a new key pair**. To create a new key pair, do the following:
  - Select the dropdown option `Create a new key pair`.
  - Name your the key pair name, e.g. `ec2-strapi-key-pair`
    ::: warning
    Download the **private key file** (.pem file). This file is needed, so note where it was downloaded.
    :::
  - After downloading the file, click the blue `Launch Instances` button.

Your instance is now running. Continue to the next steps.

### Install a PostgreSQL database on AWS RDS

Amazon calls their database hosting services **RDS**. Multiple database options exist and are available. In this guide, **PostgreSQL** is used as the example, and the steps are similar for each of the other database that are supported by Strapi. (**MySQL**, **MongoDB**, **PostgreSQL**, **MariaDB**, **SQLite**). You will set-up an **RDS instance** to host your `postgresql` database.

::: tip
**Amazon RDS** does **NOT** have a completely free evaluation tier. After finishing this guide, if you are only testing, please remember to [delete the database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html). Otherwise, you will incur charges.
:::

#### 1. Navigate to the `AWS RDS Service`

In the top menu, click on `Services` and do a search for `rds`, click on `RDS, Managed Relational Database Service`.

#### 2. Select your region

In the top menu bar, select the region that is the same as the EC2 instance, e.g. `EU (Paris)` or `US East (N. Virgina)`.

#### 3. Create the database

Click the orange `Create database` button.
Follow these steps to complete installation of a `PostgreSQL` database:

- **Engine Options:** Click on `PostgreSQL`, version **PostgreSQL 10.x-R1**
- **Templates:** Click on `Free Tier`.
- **Settings**
  - **DB instance identifier** Give a name to your database, e.g. `strapi-database`
  - **Credential Settings**: This is your `psql` database _username_ and _password_.
    - **Master username:** Keep as `postgres`, or change (optional)
    - `Uncheck` _Auto generate a password_, and then type in a new secret password.
- **Connectivity**, and **Additional connectivity configuration**: Set `Publicly accessible` to `Yes`.
- **OPTIONAL:** Review any further options (**DB Instance size**, **Storage**, **Connectivity**), and modify to your project needs.
- You need to give you Database a name. Under **Additional configuration**:
  - **Additional configuration**, and then **Initial database name:** Give your database a name, e.g. `strapi`.
- Review the rest of the options and click the orange, `Create database` button.

After a few minutes, you may refresh your page and see that your database has been successfully created.

### Configure S3 for image hosting

Amazon calls cloud storage services **S3**. You create a **bucket**, which holds the files, images, folders, etc... which then can be accessed and served by your application. This guide will show you have to use **Amazon S3** to host the images for your project.

#### 1. Navigate to the `Amazon S3`

In the top menu, click on `Services` and do a search for `s3`, click on `Scalable storage in the cloud`.

#### 2. Create the bucket

Click on the blue `Create bucket` button:

- Give your bucket a unique name, under **Bucket name**, e.g. `my-project-name-images`.
- Select the most appropriate region, under **Region**, e.g. `EU (Paris)` or `US East (N. Virgina)`.
- Click `Next`.
- Configure any appropriate options for your project in the **Configure Options** page, and click `next`.
- Under **Block public access**:
  - Uncheck `Block all public access` and set the permissions as follows:
    - `Uncheck` Block new public ACLs and uploading public objects (Recommended)
    - `Uncheck` Block public access to buckets and objects granted through any access control lists (ACLs)
    - `Check` Block public access to buckets and objects granted through new public bucket policies
    - `Check` Block public and cross-account access to buckets and objects through any public bucket policies
  - Select `Do not grant Amazon S3 Log Delivery group write access to this bucket`.
- Click `Next`.
- **Review** and click `Create bucket`.

### Configure EC2 as a Node.js server

You will set-up your EC2 server as a Node.js server. Including basic configuration and Git.

You will need your **EC2** ip address:

- In the `AWS Console`, navigate to the `AWS EC2`. In the top menu, click on `Services` and do a search for `ec2`, click on `Virtual Servers in the cloud`.
- Click on `1 Running Instance` and note the `IPv4 Public OP` address. E.g. `34.182.83.134`.

#### 1. Setup the `.pem` file

- You downloaded, in a previous step, your `User` .pem file. e.g. `ec2-strapi-key-pair.pem`. This needs to be included in each attempt to `SSH` into your `EC2 server`. Move your `.pem` file to `~/.ssh/`, follow these steps:
- On your local machine, navigate to the folder that contains your .pem file. e.g. `downloads`
- Move the .pem file to `~/.ssh/` and set file permissions:
  `Path:./path-to/.pem-file/`

```bash
mv ec2-strapi-key-pair.pem ~/.ssh/
chmod 400 ~/.ssh/ec2-strapi-key-pair.pem
```

#### 2. Log in to your server as the default `ubuntu` user:

::: tip
In the future, each time you log into your `EC2` server, you will need to add the path to the .pem file, e.g. `ssh -i ~/.ssh/ec2-strapi-key-pair.pem ubuntu@12.123.123.11`.
:::

```bash
ssh -i ~/.ssh/ec2-strapi-key-pair.pem ubuntu@12.123.123.11

Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1032-aws x86_64)

...

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-12.123.123.11:~$

```

### Configure Strapi Provider AWS S3 plugin

The next steps involve configuring Strapi to connect to the AWS S3 bucket.

#### 1. Locate your `IPv4 Public IP`:

- Login as your regular user to your `EC2 Dashboard`
- Click on `1 Running Instances`.
- Below, in the **Description** tab, locate your **IPv4 Public IP**

#### 2. Next, create your **Administrator** user, and login to Strapi:

- Go to `http://your-ip-address:1337/`
- Complete the registration form.
- Click `Ready to Start`

#### 3. Configure the plugin with your bucket credentials:

- From the left-hand menu, click `Plugins` and then the `cog` wheel located to the right of `Files Upload`.
- From the dropdown, under **Providers**, select `Amazon Web Service S3` and enter the configuration details:
  You can find the **Access API Token** and **Secret Access Token** in the **configuration.csv** file you downloaded earlier or if you saved them to your password manager.
  - **Access API Token**: This is your _Access Key ID_
  - **Secret Access Token**: This is your _Secret Access Key_
    Navigate back to your **Amazon S3 Dashboard**:
  - **Region**: Is your selected region, eg. `EU (Paris)`
  - **Bucket**: Is your bucket name, eg. `my-project-name-images`
  - Set your **Maximum size allowed(in MB)** to a value: eg. _10mb_
  - Select `ON`, for **Enable File Upload**
  - Click the `Save` button.

You may now test the image upload, and you will find your images being uploaded to **Amazon AWS S3**.

### Further steps to take

- You can **add a domain name** or **use a subdomain name** for your Strapi project, you will need to [install NGINX](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/) and [configure it](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/nodejs-platform-proxy.html).
  ::: tip
  After setting up **NGINX**, for security purposes, you need to disable port access on `Port 1337`. You may do this easily from your **EC2 Dashboard**. In `Security Groups` (lefthand menu), click the checkbox of the group, eg. `strapi`, and below in the `inbound` tab, click `Edit`, and delete the rule for `Port Range` : `1337` by click the `x`.
  :::
- To **install SSL**, you will need to [install and run Certbot by Let's Encrypt](https://certbot.eff.org/docs/using.html).

- Set-up [Nginx with HTTP/2 Support](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-with-http-2-support-on-ubuntu-18-04) for Ubuntu 18.04.

Your `Strapi` project has been installed on an **AWS EC2 instance** using **Ubuntu 18.04**.
