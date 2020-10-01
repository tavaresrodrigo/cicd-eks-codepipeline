# CI/CD WITH CODEPIPELINE AND EKS

Continuous integration (CI) and continuous delivery (CD) are essential in the world of microservices. Teams are more productive when they can make discrete changes frequently, release those changes programmatically and deliver updates without disruption. This simple web app with 2 microservices architecture follows the recommendations on [The Twelve Factors methodology](https://12factor.net/) and can be easily applied in a production environment with just a few staging adaptations.

![two-tier-application.png](two-tier-application.png)

This CI/CD pipeline using AWS CodePipeline and EKS. The CI/CD pipeline will deploy a tier two Kubernetes service, after making a change to the GitHub repository a new image will be built and the deployment object will be patched rolling out a new version of the application in the Kubernetes cluster running on EKS.

![pipeline.png](pipeline.png)
