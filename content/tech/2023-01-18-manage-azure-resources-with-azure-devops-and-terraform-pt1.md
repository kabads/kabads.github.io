---
layout: post
title: Managing Azure Resources with Azure DevOps and Terraform - pt. 1
date: 2023-01-18
categories:
- tech
giscus: true
---

This post will show you how to create a simple Azure DevOps pipeline that can use the Azure CLI command within a Microsoft hosted agents on Azure DevOps. 
<!--more-->
It contains two main parts:

 - Setting up Authentication for Azure 
 - Setting up a Pipeline with a Microsoft hosted agent to run Azure CLI 

### Setting up Authentication for AZ


Within your Azure account, go to https://dev.azure.com. This is where you manage repositories and pipelines. 

Firstly, you need to create a connection between your Azure DevOps account and your Azure portal account. This means that pipelines have access using the `az` command. This is needed for terraform. 

Create a service connection by clicking `Service connections`, then `Azure Resource Manager`: 

{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mgRltTc4JkF1zSK5g4602-ilL-vJuVAUYHC-rNnwXCJYzqRN7PDRQMRxF5rpKiSZV9L7V8vWhbKGmN7r0Bd5IWYtSmZTRyBH0VX3dMP-J8ynSc8IZno5UsaGBrJIWfUwBlqqU96Nsr-Vj8fiJr_FBoEWMM4iK1XsmPzyP1irnf1AkjPuDY7laObUba9rGOETq?width=1427&height=774&cropmode=none" />
<p>
{{< /rawhtml >}}

Then choose the automatic one - you will be prompted in a pop-up window to login to your Azure account:

{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mZx8Q1O5PaH6GOrnJ3Z5I07Tz7UrueEVQUcdf0qwWw7iBLmi7SueNNUJ8rY303k55Q9XCZiSQicB_X58w0lOPRyE_DPJSdBTF8ExgrHevwysuezfmfE8MnsQGdcmJv9aWuqKvA9C3_V3i_ftlzWBwSpv1lpDddAnrWnibgvZdWskmo_zXl1aGr7VJDWjVK304?width=420&height=308&cropmode=none" />
<p>
{{< /rawhtml >}}



Then, Name the Service connection in the Service connection name. You will use this in a moment in your script. 

{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mSrhG6pWYBsLZ6XJhpuuGg_xzMRKJeZ6aIAjZwX8nrw5UAHl4Drw-jPd9R9ejcgBxGdjRA6q_8eyqQOzK7d-6fZ2AfX9Dnfypp7M8oOqyeheLhaE9u2hemtD-2BxxCkJieRWggETbmH6RKCY_npH1tOK3RoEJ7CisZSSWBn-wgJP1bD-wdFwXiyfcdSQapCsL?width=356&height=568&cropmode=none" />
<p>
{{< /rawhtml >}}


This now means you can attach your pipeline to this Service connection and your `az` command will be automatically authenticated. This authentication lasts for two years, within Active Directory. 

You can test your az and whether it will login with the script in an Azure DevOps pipeline (we haven't yet got to where we create a pipeline, but if you feel confident, you can try this now). Otherwise, follow the next step below.  

      - task: AzureCLI@2
        displayName: Azure CLI
        inputs:
          azureSubscription: AZURE
          scriptType: bash
          scriptLocation: inlineScript
          inlineScript: |
            az --version
            az account show


If everything works, you will have a representation of the account connection returned in the terminal in a JSON format. 

### Setting up a pipeline 

Now navigate to the repositories part of https://dev.azure.com - this repository will be where the YAML code will reside, for your pipeline. 

Create a project to hold all the aspects of your deployment: 

{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mBR67LXuIDoJ5H1VH0HECL5bJhicSH8bVVj8Gbgpfcjfe9DbjSJ0Ufp5do5n6nSs5sdmFgxqRNkZHKyNMW6axDrUUG_9JCCPT0YIl1H6iknQX33sJoqRFv6p7ewttcjUHRBFYdOQCefi-CdowfyA1Pd4dx8jCXaD8nGQ0l0RfD1ePTrgES4jMgfOkvL6JWQeb?width=1149&height=328&cropmode=none"
<p>
{{< /rawhtml >}}

Give the project a name and decide whether it is public or private. 

Then, click in to the project. There won't be anything here right now, but that will change - we will add a repository. Click on the Repos link on the navigation. You will be presented with a window that lets you configure your computer to connect to the repository. The best method for authentication is ssh whereby you upload your public key (the default is $HOME/.ssh/id_rsa.pub), to Azure DevOps - this means you can use the repository without a password. 

{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mBR67LXuIDoJ5H1VH0HECL5bJhicSH8bVVj8Gbgpfcjfe9DbjSJ0Ufp5do5n6nSs5sdmFgxqRNkZHKyNMW6axDrUUG_9JCCPT0YIl1H6iknQX33sJoqRFv6p7ewttcjUHRBFYdOQCefi-CdowfyA1Pd4dx8jCXaD8nGQ0l0RfD1ePTrgES4jMgfOkvL6JWQeb?width=1149&height=328&cropmode=none"
<p>
{{< /rawhtml >}}

The command for cloning the repository, via ssh, to your machine: 

    git clone git@ssh.dev.azure.com:v3/adam0690/Test/Test

This will download the repository to your local machine. Now `cd` to that directory.

Now, within that directory create a file called `azure-pipeline.yml`. However, this can have any name, but this is a defacto standard (where you only have one pipeline with this project). 

To test the `az` command line works, create the file below: 

    ---
    trigger: none
    pr: none
    
    pool:
      vmImage: ubuntu-latest
    
    jobs:
      - job: InstallTerraformAndRun
        displayName: "Install Terraform And Run"
        steps:
          - task: AzureCLI@2
            displayName: Azure CLI
            inputs:
              azureSubscription: AZURE
              scriptType: bash
              scriptLocation: inlineScript
              inlineScript: |
                az --version
                az account show

The `trigger: none` defines that this pipeline shouldn't be run by any automoted means. This is what tells the pipeline to run - in this case, we don't want the pipeline to run by itself. The `pr: none` is related to the pull-requests on the repository. This can be used to trigger on a pull-request being created. `pool:` is related to the pool of virtual machines that Microsoft provide - you choose the type of machine you would like to use here.  

`jobs:` is a list of jobs that you want to run. Then the first job is called `InstallTerraformAndRun` - and also has a `displayName:` attribute, that will show up in a user friendly format when the pipeline is running. The `- task: AzureCLI@2` is a task provided by Azure DevOps marketplace, that means you can use the azure service connection that you created earlier. The specified part here is `azureSubscription: AZURE` which must match the name of the Service connection. The `inlineScript: |` allows for a bash script to be placed here - and we test the `az` commands. 

Save the file, then on the command line type

    git add azure-pipeline.yml
    git commit -m"Initial commit"
    git push

This pushes the file to Azure Repos, and allows you to create a simple pipeline from it. 

To do that, navigate to the Pipelines and create a new pipeline and you will be asked where your pipeline is hosted (it doesn't have to be in Azure Repos - if you create a repo in anothor service, you can create a Service connection as before, to connect to that service):

{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mt0ZMa0gysYnWGXt1VJk9EReZv1CpaX3_pp-DNBmWsYGLSp61LlwBnXRDKBRYgxqzOP8F9dVxfWvbLgY1KDflf3xBUbkV1TP_EHrPw4OHnVYdp1rjLRh9bsjK5L2re785L4TF18RI6gm1gNCTx8Q9NzfU20PNE6NCD-dOEtVGHxfQBJy0NkFPCja7611e_C6I?width=637&height=461&cropmode=none"
<p>
{{< /rawhtml >}}

Once you click in to the Repo that you create you will have to select the repository, and then choose the YAML file that you created earlier for this pipeline). 


{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mzNNcnVXfLOJliAY4ROuhaoBx879xeazHCapd-clc2RJMHjbXRSPDIIejGGjCD2i_1je8GtVGAB2tP2RUCNtfPPYlFDHJISk8qvdCC43tC0SyNPQYZje0Q_wKrxTUA-bGxK5_fHUYW-WNEHhn1YmUpHKUzQvq8chY6I90BODOuaqvBLC_DpHjPhfxncQxYTyG?width=558&height=343&cropmode=none"
<p>
{{< /rawhtml >}}

This will create a modal pop-up that will ask for the branch and path the the yaml file. Once you have selected those, you will be able to run the pipeline: 


{{< rawhtml >}}
<img src="https://phx02pap002files.storage.live.com/y4mGaVo3p9mkoRX8PCHuqcHYVMMYWJdQMcYO8s7hNBtSRWjZ3FPTv7dSdoMcbewOojZPairtp1xehyj57ELC3P57eBeqxBCRAGQlQ5T5TKKvCLyn_wyPzktkfFy1PGW4muDFBe337jYwO665uo8bGmTYm0R1GJorVX5He701RYCYet5iBaJ0oEXNw6QBWY3LN0r?width=1129&height=148&cropmode=none"
<p>
{{< /rawhtml >}}
