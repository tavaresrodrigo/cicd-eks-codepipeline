# CI/CD WITH CODEPIPELINE AND EKS
A CI/CD pipeline using AWS CodePipeline and EKS. The CI/CD pipeline will deploy a tier two Kubernetes service, after making a change to the GitHub repository a new image will be built and the deployment object will be patched rolling out a new version of the applicaiton in the Kubernetes cluster running on EKS.
