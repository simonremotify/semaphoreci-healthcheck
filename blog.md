# How to Run a Cloud-based `cron` Job Every 5 Minutes with Semaphore

[] Create the repo fork that has only the script in
[] Open Log-in link in a new tab?

### Introduction

`cron` is a Unix job scheduler that runs commands periodically according to a schedule given by the user, known as the 'crontab'.  Scheduled jobs can be used to automate routine, repetitive tasks and receive notifications or tasks that you don't want to run for every push.

This tutorial shows how to use Semaphore's workflow-scheduler/`cron` feature. The step-by-step example creates a project with a super-simple pipeline that runs a `bash` script to check that a website is up (returning a 200 HTTP response) every 5 minutes, and notify a Slack channel if the website gives an unexpected response.

### Prerequisites

To follow this tutorial, you will need:

* Git, a GitHub account and a working knowledge of forking, branches, committing and pushing.
* A Semaphore account. You can get one for free at semaphoreci.com.

## Create your Website Healthcheck Repository in GitHub

To get started quickly, you can fork the demo repository.  This branch contains just the health-check script and a README, so you can follow along to create the Semaphore configuration files.

The `run_healthcheck.sh` `bash`-script works as follows:

* send an HTTP request to an `ENDPOINT` (Google in the example) using `curl`
* store the HTTP response code in `status_code`
* check the response code and set the script's `exit` code as follows:
 * 1 if the response code is not `200` (signifying a status of `OK`)
 * 0 if the response code is `200`

The script's exit code is arguably the most important detail, as this is what Semaphore will check to determine success or failure when it runs the script.

``` bash
#!/bin/bash

# run_healthcheck.sh

ENDPOINT=https://www.google.com

status_code=$(curl --write-out %{http_code} --silent --output /dev/null $ENDPOINT)

if [[ "$status_code" -ne 200 ]] ; then
  echo "$ENDPOINT status $status_code"
  exit 1
else
  echo "$ENDPOINT OK"
  exit 0
fi

```

## Create your Semaphore project and choose a starter Workflow

* [Log in to Semapahore](https://id.semaphoreci.com/login)
* Use the **+** button next to the Projects heading in the left side bar to add a new project to your account
![New Project Button](img/add_project_button.png)
* Select your repository from the list
![Repository chooser](img/choose_repo.png)
* Wait for Semaphore to set up the project and connect it to GitLab
* When presented with the **Invite People** page, skip straight to the **Workflow Builder**
![Go to workflow builder button](img/go_to_workflow_builder.png)
* Choose **Single job** as your starter workflow
![Choose workflow pane](img/choose_workflow.png)
* In the workflow configuration pane, choose to **Customize it first**
![Single job configuration pane](img/customise_single_job_workflow.png)

## Customize your Semaphore Workflow

Here's a very quick overview of the main modelling concepts used in Semaphore Workflows:

* Ultimately, work done by running **Commands**.
* Commands are grouped into sequences within **Jobs**.
* Jobs are collected into **Blocks**, where the Jobs can run in parallel.
* Finally, Blocks are arranged into a **Pipeline**, which sequences and triggers inter-related Blocks.

You can find more detail on the [concepts in the Workflow model in the docs](https://docs.semaphoreci.com/guided-tour/concepts/).

For this tutorial, we want the pipeline to run a command that executes our `run_healthcheck.sh` script.  For that, we need to create a single Job in a single Block.

* Add a command to execute our script to the existing Job
 * append `./run_healthcheck.sh` to Job

 ![Add command to job](img/add_command_to_job.png)

 * Optionally, rename the Job to something more meaningful

 ![Rename Job](img/rename_job.png)

 * You'll see that, as you rename the Job, the Pipeline diagram is instantly updated to help you understand the entirety of the configuration

 ![Pipeline showing new job name](img/pipeline_shows_job_name.png)

The curious can learn more about the [`checkout` command that appears in the job in the docs](https://docs.semaphoreci.com/reference/toolbox-reference/#checkout).  In short, the `checkout` command makes the code in your `git` repository available in the environment where the job's commands are executed.

## Run the Workflow and check the output

* Click **Run the workflow**.  All worklfows execute from version-controlled configuration stored in your repository, so you can always reproduce the result of a workflow.  The dialog that pops up allows you customise the branch and commit message that will be pushed to your repository.

![Run the workflow dialog](img/run_the_workflow.png)

* Click **Start**

Initially goes to set-up-semaphore branch

Check it works

Schedule your pipeline to run every 5 minutes

Tada!

How to take this further (e.g. command line tools)

Create the healthcheck Bash script and get it into a repo (here's my (draft) repo: https://github.com/simonremotify/semaphoreci-healthcheck)
Create the Semaphore pipeline in the UI and check the output is as expected
Schedule the pipeline through the UI
Set up notifications in Slack to alert the appropriate team if the check fails
Voila!
