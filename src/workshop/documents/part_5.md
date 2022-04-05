# Part 5: Continuous deployment (CD)

## Goal 
After a successful run of the CI pipeline, your team is looking to complete the process with a CD pipeline that will handle the deployment of the model without introducing any downtime in production (hot swap).

## Pre-requisites
- Complete parts 0, 1, 2, 3 and 4

## Tasks

- Located the CD pipeline template named my_workshop_cd.yml under .guthub/workflows

### High Level Goals

- The goal of this section is to get a fully functional CD pipeline that will:
    
    1. Trigger based on creation of a Pull Request (PR) to main
    2. Login to Azure using a Service Principal to be able to leverage the Azure ML CLI commands in your workflow
    3. Create a model API endpoint (webservice) using an Azure ML Managed Endpoint and deploy the model to the endpoint into one of the 2 deployment slots (blue/green slots, which will switch staging/production roles)
    4. Test the deployment to the endpoint of the new model
    5. On success of test, swap the deployment to accept 100% of the service endpoint traffic (and therefore become 'production')
    6. Add a Branch Protection rule in GitHub
       

### Implementation Details and Instructions

Locate the #setup sections in the CD pipeline file (my_workshop_cd.yml) and fill in with the proper values.

1. Setup your CD pipeline yaml file triggers:
    - A trigger has already been setup to enable to run the CD pipeline on demand from the GitHub UI as this will greatly facilitate testing. See 'workflow_dispatch'.
    - Define the rest of the triggers section to trigger on Pull Request to 'main', with the help of this documentation: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows.


2. Setup your Azure Credentials to enable the workflow to login to Azure and run Azure CLI commands.

    See 'creds: ${{ secrets.MY_AZURE_CREDENTIALS }}' in your workflow. This is the name of your secret in GitHub, make sure you have such secret or rename the secret key in the workflow yaml file to match the secret key defined in GitHub.

    Please refer to [Use the Azure login action with a service principal secret](https://docs.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows#use-the-azure-login-action-with-a-service-principal-secret) to create the proper Azure Credentials if you haven't done so already (you should have already defined such secret to complete the CI part of the workshop).

3. Configure your endpoint/deployment files:
    - Edit src/workshop/scoring/my_endpoint.yml to define a name for your endpoint (see #setup in that file).
    - Edit src/workshop/scoring/my_deployment.yml to set the name of the endpoint to the one you defined just above.

    Use the .github/actions/aml-endpoint-deploy provided custom action in your CD yaml pipeline:
    - Configure the custom action parameters to match your Azure environment.
    - Have a look at the code inside the custom action (action.yaml) to see how simple it is to build custom actions leveraging any scripting packages, libraries, and the Azure CLI (az) which gives you full access to anything available in AML. Please notice the section of codes relevant to: a) creating then endpoint if it doesn't exist, b) reading the endpoint details to check its traffic, and identify which deployment slot to target to stage the new model, c) deployment of the model.

    Test your CD pipeline by checking your code into your own development branch, and going to the GitHub UI under 'Actions', and select 'my_workshhop_CD', and trigger it on your own branch.

    Troubleshoot and debug until the first step of the CD pipeline (deployment) is functional.

4. Use the .github/actions/aml-endpoint-test provided custom action in your CD yaml pipeline.
    - Configure the custom action parameters to match your Azure environment.
    - Have a look at the custom action, and feel free to modify the test code to your liking. One could consider building a python test script instead of using the az ml command to test the endpoint more thoroughly. Please note that the az ml commands would enable you to retrieve any required metadata about the endpoint for further testing (for instance its URI) and that you can customize targetting a specific deployment of the endpoint URI (via a header hint named 'azureml-model-deployment'). See https://docs.microsoft.com/en-us/azure/machine-learning/how-to-safely-rollout-managed-endpoints#test-the-new-deployment 

    Test your CD pipeline by checking your code into your own development branch, and going to the GitHub UI under 'Actions', and select 'my_workshhop_cd', and trigger it on your own branch.

    Troubleshoot and debug until the first two steps of the CD pipeline (deployment + test) are functional.

5. Use the .github/actions/aml-endpoint-swap provided custom action in your CD yaml pipeline.
    - Configure the custom action parameters to match your Azure environment.
    - Review the custom action code to see how the custom action reads the endpoint metadata, to identify which endpoint is currently live (100% traffic), to proceed with the proper traffic re-assignments of the deployments of this endpoint.

    Test your CD pipeline by checking your code into your own development branch, and going to the GitHub UI under 'Actions', and select 'my_workshhop_cd', and trigger it on your own branch.

    Troubleshoot and debug until all 3 steps of this CD pipeline are green. Validate that each time you run the CD pipeline, the deployment goes to the 0% endpoint, tests it, and then switches the traffic around.

6. Setup a GitHub branch protection rule to require a succesful CD run to be able to merge any code into 'main'

    - Go to your github repo, and click on 'Settings'
    - Click on 'Branches' under 'Code and automation'
    - Click on 'Add rule' next to the 'Branch protection rules' to create a new rule, keep all defaults and set the following:
        - Branch name pattern: main
        - Require a pull request before merging: CHECK
        - Require status checks to pass before merging: CHECK
            - Require branches to be up to date before merging CHECK
            - Status checkes that are required: 'Workshop-Deployment' (that's the name of your CD workflow job as defined in the yaml file)

    Click Save Changes to enable this rule on your repo.


## Success criteria

- The CD pipeline runs sucessfully each time a PR request to 'main' is opened. Please test this by triggering a new CI run (which on success should generate a PR to main), or creating your own PR to main.

## Reference materials

- [GitHub Actions](https://github.com/features/actions)
- [GitHub Actions: Workflow Triggers](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
- [Github Actions: Use the Azure login action with a service principal secret](https://docs.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Cwindows#use-the-azure-login-action-with-a-service-principal-secret)
- [Azure ML CLI v2](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-train-cli)
- [Azure ML CLI v2 Examples](https://github.com/Azure/azureml-examples/tree/main/cli)
- [Azure ML Managed Endpoints](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-deploy-managed-online-endpoints)
- [Azure ML Safe Rollout of Managed Endpoints](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-safely-rollout-managed-endpoints)