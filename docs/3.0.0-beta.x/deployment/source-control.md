### Install and Configure Git versioning on your server

A convenient way to maintain your Strapi application and update it during and after initial development is to use [Git](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control). In order to use Git, you will need to have it installed on your EC2 instance. EC2 instances should have Git installed by default, so you will first check if it is installed and if it is not installed, you will need to install it.

The next step is to configure Git on your server.

#### 1. Check to see if `Git` is installed

If you see a `git version 2.x.x` then you do have `Git` installed. Check with the following command:

```bash
git --version
```

#### 2. **OPTIONAL:** Install Git.

::: tip
Only do if _not installed_, as above. Please follow these directions on [how to install Git on Ubuntu 18.04](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
:::

#### 3. Configure the global **username** and **email** settings: [Setting up Git - Your Identity](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)

After installing and configuring Git on your EC2 instance. Please continue to the next step.

### Prepare and clone Strapi project to server

These instructions assume that you have already created a **Strapi** project, and have it in a **GitHub** repository.

You will need to update the `database.json` file to configure Strapi to connect to the `RDS` database. And you will need to install an npm package called `pg` locally on your development server.
::: tip
The `pg` package install is only necessary if you are using **PostgresSQL** as your database.
:::

#### 1. Install `pg` in your Strapi project.

On your development machine, navigate to your Strapi project root directory:
`Path: ./my-project/`

```bash
npm install pg
```

#### 2. Edit the `database.json` file.

Copy/paste the following:

`Path: ./my-project/config/environments/production/database.json`:

```json
{
  "defaultConnection": "default",
  "connections": {
    "default": {
      "connector": "bookshelf",
      "settings": {
        "client": "postgres",
        "host": "${process.env.DATABASE_HOST || '127.0.0.1'}",
        "port": "${process.env.DATABASE_PORT || 27017}",
        "database": "${process.env.DATABASE_NAME || 'strapi'}",
        "username": "${process.env.DATABASE_USERNAME || ''}",
        "password": "${process.env.DATABASE_PASSWORD || ''}"
      },
      "options": {
        "ssl": false
      }
    }
  }
}
```

#### 3. Install the **Strapi Provider Upload AWS S3 Plugin**:

`Path: ./my-project/`.

This plugin will allow configurations for each active environment.

```bash
npm install strapi-provider-upload-aws-s3@beta
```

#### 4. Push your local changes to your project's GitHub repository.

```bash
git add .
git commit -m 'installed pg, aws-S3 provider plugin and updated the production/database.json file'
git push
```

#### 5. Deploy from GitHub

You will next deploy your Strapi project to your EC2 instance by **cloning it from GitHub**.

From your terminal and logged into your EC2 instance as the `ubuntu` user:

```bash
cd ~
git clone https://github.com/your-name/your-project-repo.git
```

Next, navigate to the `my-project` folder, the root for Strapi. You will need to run `npm install` to install the packages for your project.

`Path: ./my-project/`

```bash
cd ./my-project/
npm install
NODE_ENV=production npm run build
```
