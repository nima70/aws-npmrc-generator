# AWS `.npmrc` Generator

## Overview

The reason I created this generator is that I found many problems with AWS CodeArtifact `.npmrc` configuration commands provided by AWS. As of 23/07/2024, AWS provides two methods for configuring the `.npmrc`:

1. Configuring using AWS CLI
2. Manual method: push and pull from your repository

## Problems

### Token Renewal

First of all, the token used to access the package manager must be renewed every 12 hours for both methods. This means that this process cannot be a one-time setup.

### Method 1 Problem: AWS CLI Configuration

Once this is set, you cannot access npm directly because your npmjs registry will be overwritten by the AWS CodeArtifact package manager. There is a solution: add the npmjs package manager manually every time you run this command. However, considering the token changes every 12 hours, this is a pain in the neck.

There is an alternative solution for Method 1 too. You can use the upstream and external options in the CodeArtifact package manager. However, this approach has two problems:

- **Storage Consumption**: All dependencies of your package will be stored inside your package, which is space-consuming. Why should we pay for something unnecessary?
- **Vendor Lock-In**: This locks you into AWS. Why should all my dependencies be locked to one vendor? What if we decide to change from AWS to another platform in the future?

### Method 2 Problem: Manual Configuration

The issue with the second method is that under the Windows environment, this approach does not work at the moment. This is due to a bug where the `.npmrc` file cannot read environment variables properly. I troubleshooted this for a long time and found out that the only way to make this work is by hard coding the token manually into the file. Considering that you have to refresh the token every 12 hours, this is also a pain in the neck.

## Opportunities for Automation

Regardless of Method 1 or 2, there are more opportunities for automating this task that I will explain shortly.

## Solution

I came up with a script that sets up the token refreshment and creates the `.npmrc` based on generic inputs. Indeed, all the parameters can be set through a `.env` file. Furthermore, I suggest a few scripts for the `package.json` file which could automate this task even further.

### How to Use

1. Create a `.env` file with the following content:

   ```
   CODEARTIFACT_DOMAIN=<your domain>
   CODEARTIFACT_REPOSITORY=<your repository>
   AWS_REGION=<your AWS region>
   CODEARTIFACT_DOMAIN_OWNER=<your domain owner>
   ```

2. Add the following script to your `package.json`:

   ```json
   "scripts": {
     "setup-npmrc": "node setup-npmrc.js",
     "install-packages": "npm install --userconfig ./.npmrc",
     "setup-and-install": "npm run setup-npmrc && npm run install-packages"
   }
   ```

3. Run the `setup-and-install` script to generate the `.npmrc` file and install your packages:
   ```bash
   npm run setup-and-install
   ```

This script handles the token refreshment and `.npmrc` creation seamlessly, ensuring a smooth workflow for your projects.
