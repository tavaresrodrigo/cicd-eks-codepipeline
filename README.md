# CI/CD WITH CODEPIPELINE AND EKS

Continuous integration (CI) and continuous delivery (CD) are essential in the world of microservices. Teams are more productive when they can make discrete changes frequently, release those changes programmatically and deliver updates without disruption. This simple web app with 2 microservices architecture follows the recommendations on [The Twelve Factors app methodology](https://12factor.net/) and can be easily applied in a production environment with just a few staging adaptations.

![two-tier-application.png](two-tier-application.png)

This CI/CD pipeline using AWS CodePipeline and EKS. The CI/CD pipeline will deploy a tier two Kubernetes service, after making a change to the GitHub repository a new image will be built and the deployment object will be patched rolling out a new version of the application in the Kubernetes cluster running on EKS.

![pipeline.png](pipeline.png)

## Deploying the pipeline

I assume you already have an EKS cluster up and running, so we will start by creating the IAM role in order to allow AWS CodeBuild to deploy a sample Kubernetes service using [IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). 

### Create the role: 
From the bastion host that allows you to kubectl against your EKS cluster, perform the commands below to create IAM role and policies for CodeBuild:
```shell
ACCOUNT_ID= YOUR AWS ACCOUNT NUMBER
TRUST="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Principal\": { \"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\" }, \"Action\": \"sts:AssumeRole\" } ] }"

echo '{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": "eks:Describe*", "Resource": "*" } ] }' > /tmp/iam-role-policy

aws iam create-role --role-name EksWorkshopCodeBuildKubectlRole --assume-role-policy-document "$TRUST" --output text --query 'Role.Arn'

aws iam put-role-policy --role-name EksWorkshopCodeBuildKubectlRole --policy-name eks-describe --policy-document file:///tmp/iam-role-policy
```

### MODIFY AWS-AUTH CONFIGMAP:

We need to add the role created in the previous step in order to allow CodeBuild to authenticate to the EKS cluster assuming the role.

```shell
ROLE="    - rolearn: arn:aws:iam::$ACCOUNT_ID:role/EksWorkshopCodeBuildKubectlRole\n      username: build\n      groups:\n        - system:masters"

kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml

kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"
```
### CONFIGURE GITHUB ACCESS TOKEN

In order for CodePipeline to receive callbacks from GitHub, we need to generate a personal access token on **Settings/Developer settings/Personal access tokens** since I'm not the owner of the organization, I used my public repository [tavaresrodrigo/cicd-eks-codepipeline](https://github.com/tavaresrodrigo/cicd-eks-codepipeline) to connect to CodePipeline.

## DEPLOYING THE PIPELINE 

We could use Terraform which is a great tool that offers Super portability since by using the same tool you have a single language for that can be used to define infrastructure for Google cloud, AWS, OpenStack and many of the other cloud providers, however I decided to use CloudFormation since it offers better integration with the AWS Managed services and it's a service already provided without any cost and effort of implementation. 

CloudFormation is the AWS infrastructure as code (IaC) tool that provides a common language for to describe and provision all the infrastructure resources in the cloud environment. The [CloudFormation template](ci-cd-codepipeline.cfn.yml) will create the CodeBuildProject, CodeBuildServiceRole, CodePipelineArtifactBucket, CodePipeline pipeline, CodePipelineServiceRole and the EcrDockerRepository.