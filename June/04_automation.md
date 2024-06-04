<i>This marks the approach taken on tasks to automate deployments</i>

(Tasks Carried out in) ---

- [bbtz\_add\_money\_basic\_fee :construction:](#bbtz_add_money_basic_fee-construction)
  - [Initial Challenges](#initial-challenges)
  - [Troubleshooting and Resolution](#troubleshooting-and-resolution)
    - [Docker Login Failures:](#docker-login-failures)
      - [Solution:](#solution)
    - [Missing Package Managers:](#missing-package-managers)
      - [Solution:](#solution-1)
    - [AWS Credentials and ECR Login:](#aws-credentials-and-ecr-login)
    - [EC2 Script Improvements:](#ec2-script-improvements)
      - [References:](#references)
      - [Conclusion](#conclusion)
- [bbtz\_second\_gen\_v2 :construction:](#bbtz_second_gen_v2-construction)
    - [At the end of the line :](#at-the-end-of-the-line-)
    - [Issue:](#issue)
    - [Initial Error Message:](#initial-error-message)
    - [Investigation:](#investigation)
      - [Solution:](#solution-2)
    - [Fix the Build Command:](#fix-the-build-command)
    - [Two approaches were considered:](#two-approaches-were-considered)
    - [Implemented Solution:](#implemented-solution)
      - [Verification:](#verification)
      - [References:](#references-1)
      - [Next Steps:](#next-steps)

****
# bbtz_add_money_basic_fee :construction:
- [ ] At the end of the line :
- Get the basics done to manage deployment to ec2 core production for use in live
- Ensure on push the actions to handle automation follow standard scalable structure
- Changed port allocation on codelevel to remove conflicts 

## Initial Challenges
- The initial workflow utilized the appleboy/ssh-action to deploy the container to the EC2 server. 
- However, authentication with SSH failed despite adding the public key to the 
  `.ssh/authorized_keys` file.

## Troubleshooting and Resolution

### <strong>Docker Login Failures:</strong>

The initial approach used aws ecr get-login-password to retrieve credentials for logging into ECR. 
- This resulted in errors.

#### Solution:

- We explored temporary credentials using `aws sts assume-role`. However, this also led to issues.
- Finally, we successfully resolved the login issue by switching to `aws-actions/configure-aws-credentials` and `aws-actions/amazon-ecr-login actions`. 
- These actions configure AWS credentials and log in to ECR more reliably within the workflow.

### <strong>Missing Package Managers:</strong>

- The script running on the EC2 instance required `jq` for parsing JSON output. 
- However, jq was not installed.

#### Solution:

- The script was updated to check the OS type (Ubuntu/Debian or Amazon Linux) and 
```bash
 install apt-get || yum
``` 
along with jq if necessary.
- Improved Workflow
- The provided YAML file showcases the final workflow with the implemented solutions. Here's a breakdown of the key changes:

### <strong>AWS Credentials and ECR Login:</strong>

- The aws-actions/configure-aws-credentials action is used to configure AWS credentials with the appropriate IAM role.
- The aws-actions/amazon-ecr-login action logs in to ECR using the configured credentials.

### <strong>EC2 Script Improvements:</strong>

- The script now checks for and installs jq if needed.
- Environment variables for AWS credentials are directly set within the script instead of relying on the previous unsuccessful `aws ecr get-login-password` approach.

#### References:

- AWS CLI documentation on aws sts assume-role: https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html
- AWS documentation on temporary credentials: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html
- GitHub Marketplace listing for aws-actions/configure-aws-credentials: https://github.com/aws-actions/configure-aws-credentials
- GitHub Marketplace listing for aws-actions/amazon-ecr-login: https://github.com/aws-actions/amazon-ecr-login

#### Conclusion
The workflow has been successfully updated to address the authentication and login issues encountered earlier. It now leverages AWS actions for a more reliable and secure approach to configuring credentials and logging into ECR.



# bbtz_second_gen_v2 :construction:
### At the end of the line :
- Get the fix on current workflow for deploy to ec2  development for use in dev-server
- Ensure on push the actions to handle automation follow standard scalable structure 
- Made stop to old container with current exposed port configured, against stopped container.

### <strong>Issue:</strong>

The workflow building a Docker image encountered an error 
```bash
"invalid tag": repository name must have at least one component".
``` 
- This prevented the image from being pushed to Amazon ECR.

### <strong>Initial Error Message:</strong>

```bash
ERROR: invalid tag "": repository name must have at least one component
```

### <strong>Investigation:</strong>

- The error message indicated an empty string was being used as the repository name in the docker build command.

Reviewed the workflow script and found the line:

```bash
docker build -t "" .
```

#### Solution:

### <strong>Fix the Build Command:</strong>
- We replaced the empty string with the name of the secret containing the ECR repository name. The updated line is:

```bash
docker build -t "${{ secrets.REPOSITORY_NAME }}" .
```
Push with "latest" Tag:

### <strong>Two approaches were considered:</strong>

- Separate Tag and Push: Build the image with the existing tag (e.g., Git commit SHA) and then use separate docker tag and docker push commands with the --all-tags flag to push both the original and "latest" tag.
- Push with --tag Option: Build the image and utilize the docker push command with the --tag option to specify an additional "latest" tag alongside the original tag.


### <strong>Implemented Solution:</strong>

- We opted for the second approach with the --tag option for simplicity.
- The updated workflow step includes:

```bash 
docker push "${{ secrets.AWS_REGISTRY_URL }}/${{ secrets.REPOSITORY_NAME }}:${TAG}" --tag "${{ secrets.AWS_REGISTRY_URL }}/${{ secrets.REPOSITORY_NAME }}:latest"
```
#### <strong>Verification:</strong>
- The workflow was re-run after the changes.
- The build process completed successfully.
- The Docker image was pushed to ECR with both the original tag (e.g., Git commit SHA) and the "latest" tag.

#### <strong>References:</strong>

Stack Overflow: https://stackoverflow.com/questions/55441517/docker-build-tag-not-setting-tag-repository-correctly

#### <strong>Next Steps:</strong>
Monitor future deployments for any issues related to image pushing.
Consider incorporating unit tests for the Docker build process to ensure consistency.