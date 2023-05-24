# Deploying Medusa on Google Cloud

## How to Deploy a Medusa Commerce Backend Server using Google Cloud Compute Engine for free

In this tutorial, you will learn how to deploy a live server of the Medusa backend in the cloud using a Google Cloud Compute Engine Virtual Machine (VM).

## Introduction

[Medusa](https://medusajs.com/readme/) is a modular commerce engine that helps developers build rich digital commerce applications. It is free, open source, and uses Node.js. With Medusa, a developer can quickly build advanced commerce solutions without coding from scratch.

Medusa is very flexible and customizable. It can be hosted in the cloud using [IAAS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service) and [PAAS](https://en.wikipedia.org/wiki/Platform_as_a_service) solutions.

[Google Cloud](https://www.google.com/cloud/)'s [Compute Engine](https://cloud.google.com/compute) provides virtual machines suitable enough to host a Medusa backend.

This guide will teach you how to deploy a live server of the Medusa backend in the cloud using Google Cloud.

## Why Google Cloud?

Google Cloud offers a Free tier for some of its cloud services. You can host the Medusa server application within Google Cloud's Compute Engine service for free.

The free services of interest for hosting a Medusa server app are:

**Compute Engine:**

- 1 non-preemptible e2-micro VM instance per month in one of the following US regions:
    - Oregon: us-west1
    - Iowa: us-central1
    - South Carolina: us-east1
- 30 GB-months standard persistent disk
- 1 GB network egress from North America to all region destinations (excluding China and Australia) per month
- An external IP address.

For more information about all the free services offered by Google Cloud check out the [Google Cloud Free Program](https://cloud.google.com/free/docs/free-cloud-features) page.

## Prerequisites

- A Medusa backend server app. [Follow the Quickstart guide to create one](https://docs.medusajs.com/development/backend/install)
- A GitHub account. [Sign up for one here](https://github.com/signup)
- A Google Cloud account. [Sign up for a Google Cloud account here](https://console.cloud.google.com/freetrial/signup/)

## Step 1: Create Google Cloud Project

[Create a Google Cloud account](https://console.cloud.google.com/freetrial/signup) if you haven't gotten one already and [sign in](https://console.cloud.google.com/) to your Google Cloud console. 

In the Google Cloud console go to the [**Manage Resources**](https://console.cloud.google.com/cloud-resource-manager) page.

![Cloud Resource Manager](https://res.cloudinary.com/craigsims808/image/upload/v1684955482/articles/gcp-medusa/cloud-resource-manager_ytke3o.png)

Select **CREATE PROJECT** and a **New Project** window appears. Enter a `Project name` then click **CREATE**.

![New Project](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/new-project_cni1aq.png)

Go to the [**Project Selector**](https://console.cloud.google.com/projectselector2/home/dashboard) page and select the project you have just created.

![Select Project](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/new-project_cni1aq.png)

[Enable Billing](https://cloud.google.com/billing/docs/how-to/verify-billing-enabled) for your project if it isn't already enabled.

[Enable the Compute Engine API](https://console.cloud.google.com/apis/api/compute.googleapis.com/overview) for creating and running virtual machines.

![Enable Compute Engine API](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/enable-compute-engine-api_pxkduk.png)

## Step 2: Create a new VM instance

In the Google Cloud console, go to the [**Create an instance**](https://console.cloud.google.com/compute/instancesAdd) page.

Configure your new VM instance with the following details:
- **Name**: *`<any string as long as it starts with a small letter>`*
- **Region**: `us-central1(Iowa)`

> **NOTE:**
> *You can also use `us-east` or `us-west` as well. All are eligible for the free tier.*

- **Zone**: `us-central1-a`
- **Machine configuration**: `General purpose`
- **Series**: `E2`
- **Machine type**: `e2-micro (2 vCPU, 1 GB memory)

> **NOTE:** *You can always set up your VM with a much more powerful machine type with more memory and CPUs. This tutorial is showcasing what's possible using free tier options.*

![Machine Configuration](https://res.cloudinary.com/craigsims808/image/upload/v1684955484/articles/gcp-medusa/machine-configuration_ivxflh.png)

- **Boot disk Type**: `Standard persistent disk`
- **Boot disk Size**: `30 GB`
- **Image**: `Debian GNU/Linux 11 (bullseye)`
- **Allow HTTP traffic**: `Yes`
- **Allow HTTPS traffic**: `Yes`

![Boot disk Configuration](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/boot-disk-configuration_wsy0nw.png)

Click **CREATE** and wait for a short period for your instance to be created. After the instance is ready, it's listed on the [VM instances](https://console.cloud.google.com/compute/instances) page with a green status icon.

![VM Instance List](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/list-of-instances_rfcwc3.png)

In the list of VM instances, select **SSH** for the instance you have just created.

This will start an SSH session to your instance in your browser.

![SSH-in-browser](https://res.cloudinary.com/craigsims808/image/upload/v1684955484/articles/gcp-medusa/ssh-in-browser_l8u3vr.png)

This confirms that your instance is up and ready. Next, you will set up your Medusa server.

## Step 3: Install Node.js

Medusa supports LTS versions of Node.js (v16 or greater).

Install Node.js in your Medusa server.

```bash
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```

This installs version 16 of Node.

[Resolve the access permissions for npm](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally) using the following command:

```bash
mkdir ~/.npm-global && npm config set prefix '~/.npm-global' && echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.profile && source ~/.profile
```

## Step 4: Install Git

```bash
sudo apt install git-all
```

[Set up your Git Identity](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup).

## Step 5: Install Medusa CLI

```bash
npm install @medusajs/medusa-cli -g
```

## Step 6: Install and configure PostgreSQL

Medusa recommends using a PostgreSQL database for deployments.

Download and install PostgreSQL.

```bash
sudo sh -c \
  'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - \
  https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get -y install postgresql
```

Load the PostgreSQL console.

```bash
sudo -u postgres psql
```

Create a new user named `server_admin`.

```sql
CREATE USER server_admin WITH PASSWORD 'server_admin_pass';
```

Create a new database named `server_db` and make `server_admin` the owner.

```sql
CREATE DATABASE server_db OWNER server_admin;
```

Grant all privileges of `server_db` to `server_admin`.

```sql
GRANT ALL PRIVILEGES ON DATABASE server_db TO server_admin;
```

Exit the console.

```sql
exit
```

## Step 7: Setup Medusa app on your local machine

Create a [GitHub repository](https://github.com/new) to store your Medusa server app and clone it to your local machine.

In the work folder of the GitHub repo, set up a Medusa server app using the [Medusa Server Quickstart Guide](https://docs.medusajs.com/development/backend/install) instructions.

If you successfully followed the instructions, you should have a directory named `my-medusa-store` containing your Medusa server app and a server running on port `9000`.

Stop the server app on your local machine, then open up `medusa.config.js` in the root of your app folder.

Update `medusa.config.js` by commenting out the line database_database: `"./medusa-db.sql"`, and adding `database_url: DATABASE_URL,` in the `projectConfig` section.

```js
/** @type {import('@medusajs/medusa').ConfigModule["projectConfig"]} */
const projectConfig = {
  jwtSecret: process.env.JWT_SECRET,
  cookieSecret: process.env.COOKIE_SECRET,
  //database_database: "./medusa-db.sql",
  database_url: DATABASE_URL,
  database_type: DATABASE_TYPE,
  store_cors: STORE_CORS,
  admin_cors: ADMIN_CORS,
  // Uncomment the following lines to enable REDIS
  // redis_url: REDIS_URL
}
```

Edit `package.json` in the root of your app folder and change the `start` script as follows:

```json
  "scripts": {
    "clean": "cross-env ./node_modules/.bin/rimraf dist",
    "build": "cross-env npm run clean && tsc -p tsconfig.json",
    "watch": "cross-env tsc --watch",
    "test": "cross-env jest",
    "seed": "cross-env medusa seed -f ./data/seed.json",
    "start": "medusa migrations run && medusa start",
    "start:custom": "cross-env npm run build && node --preserve-symlinks index.js",
    "dev": "cross-env npm run build && medusa develop",
    "build:admin": "cross-env medusa-admin build"
  },
```

Commit and push the changes you made to GitHub.

```bash
git commit -a -m "Updated medusa.config.js & package.json" 
git push
```

## Step 8: Run Medusa Server in your VM

Clone your Medusa server app repo to your Google Cloud instance. Change directories to your Medusa server app repo.

Install the dependencies.

```bash
cd my-medusa-store
npm install
```

Create a `.env` file in the root of `my-medusa-store` and add the following code:

```toml
PORT=9000
JWT_SECRET=something
COOKIE_SECRET=something
DATABASE_TYPE=postgres
DATABASE_URL=postgres://server_admin:server_admin_pass@localhost:5432/server_db
```

>**NOTE:** 
>
>*Use other values for `JWT_SECRET` and `COOKIE_SECRET` besides `something` for better security.*

Seed your database.

```bash
npm run seed
```

Start your server.

```bash
npm run start
```

Your Medusa server app should start running, but it is not accessible over the internet yet. In the next step, we will configure the Network Firewall settings for your VM instance.

## Step 9: Open HTTP access to your Server

Go to the [Create a firewall rule](https://console.cloud.google.com/networking/firewalls/add) page and create a firewall rule with the following specifications:

- **Name:** `allow-http-9000`
- **Description:** `Allow port 9000 access to http-server
- **Target tags:** `http-server`
- **Source IPv4 ranges:** `0.0.0.0/0`
- **TCP Ports:** `9000`

![Create a firewall rule 1](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/create-firewall-rule-1_dq5goo.png)

![Create a firewall rule 2](https://res.cloudinary.com/craigsims808/image/upload/v1684955483/articles/gcp-medusa/create-firewall-rule-2_vyunwc.png)

Leave all the other options as is and click **CREATE** to create your firewall rule.

## Step 10: Test your Server

Visit `http://x.x.x.x:9000/health` in your browser, where `x.x.x.x` is your server's IP address. You should see an `OK` message. This confirms that your server is working.

![health check](https://res.cloudinary.com/craigsims808/image/upload/v1684755319/articles/oracle-medusa/health-check_wwfpfq.png)

Visit `http://x.x.x.x:9000/store/products` and you should see a list of products in your database.

![Products check](https://res.cloudinary.com/craigsims808/image/upload/v1684755320/articles/oracle-medusa/products-check_ctkes1.png)

## Further Steps

This tutorial was a minimal way of deploying a Medusa server on Google Cloud. Here are some more configurations you can make:

- [Install the MinIO plugin](https://docs.medusajs.com/plugins/file-service/minio) to handle image uploads to your Medusa backend.

- [Install PM2](https://pm2.io/docs/runtime/guide/installation/) to manage and ensure the availability of your Medusa server.

- [Install Nginx](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/) to configure a domain name.

- Add an SSL certificate for your domain name.

- Update your Medusa server with the Admin and storefront URLs.

- Change the URL in your Medusa storefront and admin based on the URL of your server.

- Set up a CI/CD pipeline between your development environment and your deployment.

- Secure your server.

## Conclusion

Hosting a Medusa server on Google Cloud is easy as seen in this tutorial. I hope this tutorial has provided sufficient knowledge on how you could host a Medusa server app on Google Cloud.

If there are any issues or questions, feel free to comment.