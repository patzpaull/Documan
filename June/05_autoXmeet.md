
- [bbtz\_add\_money\_basic\_fee :construction:](#bbtz_add_money_basic_fee-construction)
  - [Overview](#overview)
  - [Steps Taken](#steps-taken)
    - [Setting Up GitHub Actions for CI/CD](#setting-up-github-actions-for-cicd)
    - [Deploying Docker Containers](#deploying-docker-containers)
    - [Configuring Nginx](#configuring-nginx)
    - [Managing SSL Certificates with Certbot](#managing-ssl-certificates-with-certbot)
  - [Challenges and Solutions](#challenges-and-solutions)
  - [Conclusion](#conclusion)
    - [Meeting synopsis on Dep calc and system talk](#meeting-synopsis-on-dep-calc-and-system-talk)

****
# bbtz_add_money_basic_fee :construction:
- [ ] At the end of the line : 
- proceed with container configurations as well as exposing the configurations and ports for stage to our new basic fee container 
- make use of the current nginx configurations send the dist unzip and copy to specified directory.
- Standard procedure to make an upload and expose the intended and talked about directories to the right 


## Overview

This document outlines the steps taken to deploy and configure various applications on AWS EC2 instances, as well as the challenges encountered and the solutions implemented. The primary focus was on setting up GitHub Actions for CI/CD, configuring Nginx for different subdomains, and ensuring SSL certificates were properly installed and renewed.

## Steps Taken

### Setting Up GitHub Actions for CI/CD

**Objective:** Automate the deployment process for Docker containers on AWS EC2.

**Steps:**
- **Generated SSH Keys:** Created RSA and ed25519 SSH keys to facilitate secure communication between GitHub Actions and the EC2 instance.
  - Command: `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
- **Stored SSH Keys:** Stored the private key as a GitHub secret (`EC2_PRIVATE_KEY`) and added the public key to the EC2 instance's `~/.ssh/authorized_keys` file.
- **Configured GitHub Actions:**
  - Configured job steps to start the SSH agent and add the private key.
  - Tested SSH connection to ensure the setup was correct.
  - Deployed the Docker container using `appleboy/ssh-action`.

**Challenges:**
- **Authentication Issues:** Encountered issues with SSH authentication despite correct key setup.
  - **Solution:** Added detailed debug steps and ensured the correct key format and permissions.
  - Command to fix permissions: `chmod 600 ~/.ssh/authorized_keys`

**References:**
- [Appleboy SSH Action Documentation](https://github.com/appleboy/ssh-action)

### Deploying Docker Containers

**Objective:** Deploy Docker containers on the EC2 instance and expose the necessary ports.

**Steps:**
- **Built and Tagged Docker Images:** Used GitHub Actions to build and tag Docker images.
  - Commands: `docker build -t <image-name> .`, `docker tag <image-name> <repository-url>`
- **Pushed Docker Images to ECR:** Configured GitHub Actions to push images to Amazon ECR.
  - Commands: `aws ecr get-login-password | docker login --username AWS --password-stdin <repository-url>`, `docker push <repository-url>`
- **Deployed Docker Containers on EC2:** Pulled and ran the Docker container on the EC2 instance.
  - Commands: `docker pull <repository-url>`, `docker run -d -p <host-port>:<container-port> <repository-url>`

**References:**
- [Amazon ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)

### Configuring Nginx

**Objective:** Set up Nginx to serve different subdomains and applications.

**Steps:**
- **Created Nginx Configuration Files:** Configured server blocks for various subdomains.
  - Example configuration for `stg.main-baridibaridi.com`:
    ```nginx
    server {
      listen 80;
      server_name stg.main-baridibaridi.com;
      location / {
        proxy_pass http://localhost:4003;
      }
    }
    ```
- **Tested and Reloaded Nginx Configuration:**
  - Commands: `sudo nginx -t`, `sudo systemctl reload nginx`

**Challenges:**
- **SSL Certificate Issues:** Encountered issues while generating and renewing SSL certificates.
  - **Solution:** Used Certbot for automated SSL certificate generation and renewal.
  - Command: `sudo certbot --nginx -d stg.main-baridibaridi.com`

**References:**
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Certbot Documentation](https://certbot.eff.org/)

### Managing SSL Certificates with Certbot

**Objective:** Ensure secure communication with HTTPS by managing SSL certificates.

**Steps:**
- **Installed Certbot:** Installed Certbot and the necessary plugins for Nginx.
  - Command: `sudo yum install -y certbot python3-certbot-nginx`
- **Generated SSL Certificates:** Used Certbot to obtain SSL certificates for various subdomains.
  - Command: `sudo certbot --nginx -d <subdomain>`
- **Configured Automatic Renewal:** Ensured certificates are renewed automatically.
  - Command: `sudo certbot renew --dry-run`

**Challenges:**
- **Certificate Loading Issues:** Encountered issues with Nginx failing to load certificates.
  - **Solution:** Ensured the correct paths and permissions for certificate files.

**References:**
- [Certbot User Guide](https://certbot.eff.org/docs/using.html)

## Challenges and Solutions

1. **SSH Authentication Issues:**
   - **Problem:** SSH authentication failed despite correct key setup.
   - **Solution:** Debugged by adding detailed steps and ensuring key permissions were correct. Configured SSH agent correctly within GitHub Actions.

2. **Docker Deployment Issues:**
   - **Problem:** Difficulty in correctly tagging and pushing Docker images to ECR.
   - **Solution:** Verified image tagging and repository URLs. Ensured correct login to ECR.

3. **Nginx Configuration Issues:**
   - **Problem:** Conflicts in server names and SSL certificate loading errors.
   - **Solution:** Verified server block configurations and SSL paths. Used Certbot for automated SSL management.

4. **SSL Certificate Issues:**
   - **Problem:** Issues with generating and renewing SSL certificates using Certbot.
   - **Solution:** Followed Certbot documentation to correctly generate and renew certificates. Ensured Nginx was correctly configured to use the certificates.

## Conclusion

The deployment and configuration of applications on AWS EC2 involved setting up CI/CD with GitHub Actions, deploying Docker containers, configuring Nginx, and managing SSL certificates with Certbot. Through detailed debugging and systematic problem-solving, we achieved successful deployment and configuration, ensuring secure and efficient operations.

### Meeting synopsis on Dep calc and system talk

- A slight background on this talk is that there are two Other relative projects that focus on an aspect of the business:     Depreciation of assets, Ac assets dep, Tools and procurement updates
```bash
depreciation calculation 
currently handled and managedby A Person 

Proposed Monthly with data fields like
-- Maintain and ensure accuracy for data gathered before export

Disaster scenarios and failover calculations and functions : can check against sample trends and average out the individual operations and to flag overlapping fields and heavy discrepancies 
-- Risks in operations on 
@ inventory list depreciation calculation (ON US )  ENd of the year essentials BY JANUARY   


For collecting errors and handle some functions 





Three techniques 

Basic calcultation rules : Check status and service module 
- if its the same within previous months keep to input same amount 
Monthly operation floe 
create purchase recored and upliad in inventory 

2 csv files exported purchase report and the montthly cost all target inventory list 
-- Zoho integration (Dep calculation software) -- use the export 

2 of the csv files have data that targets the features that will lead to depreciation calculation

questions on primary key - level of normalization [ Clear definition of the data for primary key ]  Brief the model number and serial number for primary keys model number and serial number s\

-- Some data needs to be interconnected from zoho creator and databse we will need to pulll with relevancy  

Time frame is 6 month for frontend and backend and match the feature points 
ZOHO knowledge transfer for the pre process the data 

Starting timescale:- 


data rendered in special format, display results 2 options 
index like fy24 - april -> show 

inventory list///////
internal profit///// same approach to get depreciation

## Zoho Analytics 


// Final Verdict 
Create a software that handles the exported data and does the intended operations to calulate depreciation cost in the inventory list as well as internal profit 


Refer to shared doc 
```