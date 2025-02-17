# Step by Step CLOSELOOP-T002

If you rather watch a video with step by step instructions, you can do that here
   [![Step by Step Video](https://img.youtube.com/vi/KeucYraZ5Qo/0.jpg)](https://www.youtube.com/watch?v=KeucYraZ5Qo)

In this task, you are going to create a continuous deployment pipeline in GitHub Actions that triggers after the Continuous Integration build has been completed. 


1. In your GitHub Codespace, open a PowerShell Terminal and run the starter solution. A Pull Request with a new `deploy-infrastructure.ps1` will be created

      ```
      Workshop-Step Start "CLOSELOOP-T002"
      ```

2. In your GitHub repository, navigate to the Tab Pull Requests and open the Pull Request with CLOSELOOP-T002 in the title

3. In the Pull Request, check the conversation, Commits, Checks, and Files Changed Tabs and got through the instructions and changes.

4. On the Conversation Tab, press the Merge Pull Request Button, to merge the files into the main branch. Link the Pull Request to your Azure Boards Work item for Module 3 by typing AB#Module1WorkItemID in the title or description of the Pull Request Commit Message. 

   ![Shows the button for merging a Pull Request in GitHub](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/mergePullRequest.png)

Now your repository contains the new file.

6. In your GitHub Codespace, update your files to the latest version by pulling them.

   ![](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/2020-10-05-12-10-11.png)

## Create a new GitHub Action Workflow that builds and pushes with Docker Compose

When we start to use docker-compose as our mechanism to build and push containers, we need to give docker-compose instructions where to find the Docker files that can be used to build the images. We already have a `docker-compose.yml` file in our repository, but this contains the name of the images and not the instructions to build the containers. 

Before performing the next steps, you will need to enable the Improved Container Support to use GitHub Container Registry.
      - In the upper-right corner of any page, click your profile photo, then click Feature preview.
      - On the left, select "Improved container support", then click Enable.
      
   ![](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/enable-container-support.gif)


1. Navigate to the codespace and add a `build.docker-compose.yml` file to the root of your repository and add the following contents. And then commit and push the file.

      ```YAML
      version: "3.4"
      services:
        api:
          build: ./content-api

        web:
          build: ./content-web
      ```

2. To use docker-compose in our build, we are going to add a new GitHub Action workflow. In your GitHub repository, open the [Actions] Tab and create a new GitHub Action.

   ![](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/NewGHAction.png)

3. Search for **Publish Docker container** workflow  and call the new YAML file docker-publish.yml

   ![](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/simplewf.png)

  > Note: If you don't see **Publish Docker container** workflow in the page, click on **More continuous integration workflows...** and you will be able to view the Workflow.

4. Change the `name` property to [Docker Compose Build and Deploy]. And add the following code snippet below the  `- uses: actions/checkout@v2` step

      ```YAML
        - name: Log into GitHub Container Registry
          run: echo "${{ secrets.CR_PAT }}" | docker login https://ghcr.io -u ${{ github.actor }} --password-stdin
      ```

  This step logs in to your GitHub Container Registry with the CR_PAT secret that you created earlier

5. Add a script step below the previous one that uses `docker-compose` to build and push the images to the repository

            ```YAML
        - name: Build and Push image
          run: |  
            docker-compose -f docker-compose.yml -f build.docker-compose.yml build
            docker-compose -f docker-compose.yml -f build.docker-compose.yml push
            ```
  This step uses the docker-compose.yml and build.docker-compose.yml to build the containers with the Docker files and push them to the GitHub Container Registry

6. Now that the containers have been built and pushed, the Azure Web Application needs to be updated. Before you can interact with Azure, you need to have access to the Azure API with the Azure CLI. Using the Azure Login Task, we can login securely in Azure using a GitHub secret.

7. Go to Environment details click on **Service principle Credentials** copy **Application id(clientId)** , **clientSecret** , **subscriptionId** and **tenantId** 

   ![](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/sp-creds-auth.png)

Replace the values that you copied in below Json
      ```JSON
      {
        "clientId": "...",
        "clientSecret": "...",
        "subscriptionId": "...",
        "tenantId": "...",
        "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
        "resourceManagerEndpointUrl": "https://management.azure.com/",
        "activeDirectoryGraphResourceId": "https://graph.windows.net/",
        "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
        "galleryEndpointUrl": "https://gallery.azure.com/",
        "managementEndpointUrl": "https://management.core.windows.net/"
      }
      ```

  Copy the complete JSON output to your clipboard.

8. In your repository settings, navigate to [Secrets] and create a new secret called [AZURE_CREDENTIALS]. Paste the copied value from your clipboard to the value of the secret and save it.

   ![](https://raw.githubusercontent.com/CloudLabsAI-Azure/AIW-DevOps/main/Assets/secretAZCRED.png)

9. Back in the GitHub Action workflow, add a new step that uses the [AZURE_CREDENTIALS] secret to login to Azure
      
      ```
        - name: Login on Azure CLI
          uses: azure/login@v1.1
          with:
            creds: ${{secrets.AZURE_CREDENTIALS}}
      ```          

10. Add a script task to the workflow, that executes the `deploy-infrastrucuture.ps1` file. Take note of the ENV variable you need to add to the stage to get access to the secret variable.

      ```YAML
            - name: Deploy Infrastructure
              shell: pwsh
              env:
                  CR_PAT: ${{ secrets.CR_PAT }}        
              run: |
                .\infrastructure\deploy-infrastructure.ps1 -studentprefix <your abbreviation here>
      ```

11. Commit the workflow file. The GitHub Action will trigger.

