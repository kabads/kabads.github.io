---
categories:
- devops
date: "2022-02-14T15:00:00Z"
title: Azure DevOps Variables
---

Azure DevOps has a funny kind of virtualization. When you run your pipelines, the jobs can be on different hosts. This creates a head-scratching moment for people like me who have been working on linux bare-metal for quite a while and thinking that variables are just inherited. In fact, in other pipelines, we have a problem where variables leak all over the place and overwrite each other. We have bash scripts that do this.

So the problem statement is thus: we have a login process that is used in lots of places. This login process returns an `ACCESS_TOKEN`. This access token help with API calls. The ideal is that we don't copy this ACCESS_TOKEN code in to all the pipelines, but instead create a template that returns the `ACCESS_TOKEN`. However, the problem with this on Azure DevOps is that each job may be processed on a different agent. It seems that there is a pool of agents whereby you can run jobs in parallel, if needed. This document describes it far better than I am here: (https://docs.microsoft.com/en-us/azure/devops/pipelines/process/runs?view=azure-devops).

### Variables Between Tasks

This means you need to think carefully about how to pass variables from one step to another. Passing variables between steps in the same job is by far the easiest thing to do. 

Consider this code: 

    steps:
    - task: Bash@3
    displayName: Get access token, sandbox ID, then refresh
    name: GetToken
    inputs:
      targetType: inline
      workingDirectory: $(System.DefaultWorkingDirectory)
      script: |
        # Let's set the access token
        ACCESS_TOKEN=$(sfdx force:org:display -u ${{ parameters.sf_user }} --json | jq '.result.accessToken' | sed 's/\"//g')
        echo "##vso[task.setvariable variable=ACCESS_TOKEN; isOutput=true]$ACCESS_TOKEN" 

The last line: 

    echo "##vso[task.setvariable variable=ACCESS_TOKEN; isOutput=true]$ACCESS_TOKEN"

of this task is the one that will make it available to other tasks in the same job.

This is where the weirdness begins (for me). In a traditional environment, $ACCESS_TOKEN will be available to any other processes after this one. However, in Azure DevOps, $ACCESS_TOKEN was on a different Azure DevOps agent, and therefore is lost. The `echo "##vso[task.setvariable` makes this variable available to any subsequent tasks afterwards. Therefore, you can call this variable in another `task` with the following code:

      - task: Bash@3
      displayName: Check Variables exists and run Python script create_user.py
      inputs:
        targetType: inline
        workingDirectory: $(System.DefaultWorkingDirectory)
        script: |
          if [ -n $(ACCESS_TOKEN) ]; then echo "ACCESS_TOKEN is set. "; else echo "ACCESS_TOKEN is not set."; fi

Note the parentheses. 

### Variables from Variable Groups

If you are used to Classic Pipelines, the variables become implicitly available via `$(myvar)`. However, this variable is really a template, and not a bash variable. If you need your bash environment to use this variable in a traditional sense you will find problems. For example, the following code will not work (on Azure DevOps yaml pipeline). 

    echo $(ACCESS_TOKEN)
    python run_code.py

where `run_code.py` has the following script:

    import os
    token = (os.environ.get("ACCESS_TOKEN))

`token` will remain `NULL`. To get around this problem, in the bash call, you should have `export ACCESS_TOKEN=$(ACCESS_TOKEN)` and then the above python code will work. The ACCESS_TOKEN assignment is a traditional bash one, and one that will get around a lot of other environment problems you may find in other scripts. 
    
