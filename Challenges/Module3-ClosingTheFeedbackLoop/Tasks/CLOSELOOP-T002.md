# Setup Continuous Deployment with GitHub Actions

After setting up monitoring, nothing happened. After some investigation, it showed that the new container was not used by the Azure Web App, because all deployments are done manually. There are only a few people in the team that are comfortable with doing a new release of the application. You and your team advise that is time to level up the Continuous Integration pipeline to a Continuous Deployment(CD) pipeline. This should take care of the deployment of the infrastructure and a new version of the containers when they have been built and pushed to the container registry. The team also finds that instead of creating 3 different pipelines for all containers, the docker-compose files can also be used to build and push the containers all at once.

## Challenge 

In this challenge, you are going to create a continuous deployment pipeline that triggers after the Continuous Integration build has been completed. The CD Pipeline performs the following steps
* Build a new version of the containers using docker-compose
* Push the latest version of the containers using docker-compose
* Execute the deploy-infrastructure.ps1 and setup

## Validation

* Added a build.docker-compose.yml that can build the containers with docker-compose
* GitHub Action added that uses docker-compose to build and publish the containers to the Container Registry
* GitHub secret `AZURE_CREDENTIALS` added that contains the credentials of a Service Principal to deploy application and infrastructure to Azure
* Application deployed to Azure with `deploy-infrastructure.ps1` script



