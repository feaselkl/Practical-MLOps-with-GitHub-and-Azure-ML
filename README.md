# Practical MLOps with GitHub and Azure ML

This repository provides the supporting code for my presentation entitled [Practical MLOps with GitHub and Azure ML](https://www.catallaxyservices.com/presentations/practical-mlops-with-github-and-azure-ml/).

## Generating Data

This data comes from the [Chicago Parking Ticket database, courtesy of Daniel Hutmacher](https://sqlsunday.com/2022/12/05/new-demo-database/).  I sampled 1,000,000 records from it and [the file I used is available in CSV format](https://cspolybasepublic.blob.core.windows.net/cstrainingpublicdata/ChicagoParkingTickets.txt).

Import this into Azure ML using the Dataset name `ChicagoParkingTicketsFolder`.  Be sure to upload this as a `uri_folder` instead of an `MLtable` or `uri_file`!

## Running the Code

### ML Pipeline

In order to run the ML pipeline notebooks and jobs locally, you will need to have the following installed on your machine:

* Python (preferably the [Anaconda distribution](https://www.anaconda.com/download#downloads)), with `pip` installed:  `conda install -c anaconda pip`
* [The Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
* The Azure ML Azure CLI extension:  `az extension add -n ml`
* Pip packages:  `pip install azure-ai-ml`, `pip install azure-identity`
* [Visual Studio Code](https://code.visualstudio.com/download)
* The [Azure ML Visual Studio Code extension](https://code.visualstudio.com/docs/datascience/azure-machine-learning)

Before you run the code, make sure your console has you logged into Azure via CLI:

```cmd
az login
```

Then, create a folder called `.azureml` and a file named `config.json`.  The file should look like the following structure:

```json
{
    "subscription_id": "YOUR SUBSCRIPTION ID",
    "resource_group": "YOUR RESOURCE GROUP",
    "workspace_name": "YOUR WORKSPACE NAME"
}
```

Note that you must be logged into `az cli` with an account which has access to the subscription, resource group, and workspace.

From there, run the training code:

```python
python deploy-train.py
```

You can see the job in action by going to [Azure ML Studio](https://ml.azure.com) and viewing the "Chicago_Parking_Tickets_Code-First" experiment.  There will be a new "train_pipeline" job.

For scoring, run the following code:

```python
python deploy-score.py
```

This will create a batch endpoint and deployment, upload data to a Datastore in Azure ML, create a job to generate predictions, and downloads the resulting predictions to a local file called `predictions.csv`.

**IMPORTANT NOTE** -- You must *explicitly* grant rights to the account running `deploy-score.py` against the Azure ML workspace.  I granted Owner because I was running this personally, but it must be explicitly granted and not just have ownership as a side effect of subscription-level or resource group-level rights.

If you do not do this, you will likely get a strange `BY_POLICY` error message when running this script.

### Linking Azure Machine Learning to GitHub via OpenID Connect

Before we can use GitHub Action workflows to execute Azure Machine Learning pipelines, we need to grant appropriate permissions.  The steps for this come from [Microsoft Learn](https://learn.microsoft.com/azure/machine-learning/how-to-github-actions-machine-learning), specifically the options for OpenID Connect.

Follow the instructions in the Cheat Sheets folder, [02 - Application Security Configuration.txt](Cheat%20Sheets/02%20-%20Application%20Security%20Configuration.txt).  The individual commands to run are in [02b - az cli commands.txt](Cheat%20Sheets/02b%20-%20az%20cli%20commands.txt).  Note that this is NOT an automated script!

### Running the GitHub Actions

Each GitHub Action is in the .github/workflows folder.  There are two workflows for training AML pipelines and two for scoring.  The prior section on linking AML to GitHub **must** be completed before you can successfully run a pipeline.

For a review of what each workflow is doing, refer to the Cheat Sheets folder, specifically [03 - GitHub Actions Review.txt](Cheat%20Sheets/03%20-%20GitHub%20Actions%20Review.txt)

Note that the GitHub Action workflow will kick off an Azure Machine Learning pipeline but it will not wait for that pipeline to complete, so in order to see if the pipeline run was successful, you will need to review those results in the Azure ML Studio or via CLI.
