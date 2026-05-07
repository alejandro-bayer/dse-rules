[Image description: Bayer corporate logo/header banner, likely featuring the Bayer cross logo in green and blue, positioned at the top of the document as a title page header.]

## Job Playbook

[Image description: A decorative or title-related graphic/banner associated with the "Job Playbook" heading, possibly featuring an icon representing jobs, workflows, or data processing (such as gears, pipelines, or ETL visual elements).]

Eric Grembocki Ext: Slalom

## What is a Job?

Jobs are intended to be solutions for ETL problems, reading data, performing transformations, etc. DSE Jobs will have access to S3 and CSW via IAM roles and Google Service accounts. The process for providing access to a job is similar to the process for a Sandbox account.

All jobs deployed into a DSE Environment will share the same CSW Service Account. For this reason, we recommend grouping jobs based on workload needs into logical DSE environments.

## Overview: Job Registration and Management

This section explains how to register and manage Jobs in DSE.

## Quick registration steps

1. Register the Job in DSE once (creates the job record).
2. Register the Job's first version .
3. After that, register new versions only when you update the code.

## Lifecycle

- One-time setup

[Image description: A lifecycle or workflow diagram icon, likely a circular or cyclical arrow graphic representing the stages of a job lifecycle (develop, register, deploy, test, maintain).]

[Image description: A secondary workflow or process illustration, possibly showing connected boxes or steps representing the sequential nature of the job management process.]

- Develop
- Register Job
- Deploy
- Test / troubleshoot
- Maintenance
- Grant access
- Update: register new version and replace deployment
- Remove Job (if needed)

## Quick considerations

- Increment the version for every new release.
- Test changes in a dev/local environment before registering a new version.
- Use feature branches and pull requests to avoid introducing breaking changes.
- Use non-prod to test before deploying to prod.
- Use the DS Starter Kit to jumpstart your project with a repository preconfigured perfectly for DSE

## Recommended development workflow

- Make code changes on side/feature branches (for example feature/... or dev/...), not directly on main .
- Test locally and in non-prod.
- Submit a pull request and get a trusted reviewer's approval before merging to main .
- This minimizes regressions and deployment issues.

## I Want To...

 Register a Job

- [x]  Register a Job Version

-  Deploy a Job
-  Troubleshooting
-  Review Job FAQ

[Image description: A navigation or menu-style graphic with clickable options or icons representing the different actions a user can take (register, deploy, troubleshoot, review FAQ).]

[Image description: An additional navigation/interface graphic, possibly a screenshot of a UI panel or sidebar showing job-related options.]

## Jobs Demo (Python)

[Image description: A video thumbnail or embedded demo preview image, likely showing a screenshot of a Python code editor or Jupyter notebook demonstrating the job registration process in DSE.]

##  Register a Job

Before we get started, let's go over some assumptions

1. You have the correct PAPI access required to develop within a given Sandbox environment
2. You have cloned the repository containing your job code down to your space
3. Your repo has the job and all of its dependencies in one folder which we call a subdirectory
4. Your subdirectory has, at minimum, a main.py or main.R script which the job runs when it executes
5. Your main file has a main()function. This is what is called by DSE to run your job.
6. Your git working tree is clean; commit or stash any changes before registering a job

In order to deploy a Job within DSE, it must first be registered from within a Sandbox Environment. If you need a refresher on how to navigate a Sandbox environment, click here

This process can be done from anywhere you can interact with the DSE SDK typically the terminal or a jupyter notebook. For context, all reference material in this playbook was executed within a notebook in Jupyterlab. Also - these commands may slightly change syntactically between Python and R, but they work the exact same.

Import the DSE SDK with import dse_sdk

Importing the SDK

2. Register your job using the register_job function, making sure to fill in the display_name and description fields. You will get a console message showing whether registration was successful or not.

- display_name: An identifiable, findable, and understandable job_display_name.
- description: A clear job description that allows users to understand the purpose of the jobs
- custom_identifier (OPTIONAL): For advanced usage. Used to support multiple jobs within the same repository, y, adding an additional identifier to differentiate between the jobs. See screenshot for more details

SDK Command for Registering Job

3. You can now navigate to the DSE  and see that the profile page for your Job shows up under the Jobs tab.

A view of the jobs "Owned" by me filtered by name "My Job"

Job Profile Page for "My Job"

It's important to think of this page as a "profile" for your job which doesn't actually represent any deployed or working code. In the next steps, we will populate this profile page with our job code by registering a job version and deploying it

##  Register a Job Version

A Job Version is a fully contained, fully functional, packaged subdirectory containing our working job code, notably the main.py or main.R file. We specify a subdirectory to package which contains all of the necessary resources (scripts, dependencies, helper functions, etc.) for our job to run.

As stated before, the only mandatory file is our main.py or main.R

When you are confident in your job code and are ready to deploy your job in DSE, follow these steps

1. If you haven't already, import the DSE SDK with import dse_sdk as you did when registering the job
2. Register your job version using the register_job_version function, making sure to fill in the version_id and subdirectory fields
3. version_id: a version number which identifies the state of this job version at the time of registration
4. subdirectory: a path to the subdirectory which contains all necessary files for your job
5. custom_identifier (OPTIONAL): For advanced usage. Used to support multiple jobs within the same repository, y, adding an

additional identifier to differentiate between the jobs. See screenshot for more details

NOTE: If you have multiple jobs housed within your repository (each in its own subdirectory), you will need to add a custom_identifier field to differentiate between the different jobs. This job version, in particular, coincides with the job registered in the above "How to Register a Job".

SDK Command for Registering Job Version

3. You can now navigate to the DSE  and see that the Job Version for your Job has been updated on the profile page. You're ready to deploy!

Profile Page Sidebar Updated with Latest Job Version

##  Deploy a Job

From here, the process becomes pretty easy and is done completely through the DSE UI.

1. Decide whether you want to deploy this job to NonProd or Prod. In most situations, DSE recommends deploying to NonProd first. Once the Job is confirmed to be working as intended, it can be deployed to Prod.

Job Profile Page featuring Buttons for Deploying

2. After selecting your environment, you will get a pop up allowing you to configure your job as you see fit

Job Deployment Me

Some important things to consider when requesting a job deployment

- a. While it may be tempting to pick the largest instance size available, remember that bigger is more expensive. You should choose the smallest instance that allows your job to run as efficiently as needed.
- b. Consider your Schedule type. On Demand jobs can be run ad-hoc right from the Job profile page at any time; Scheduled jobs run on a set schedule and are completely hands off. Don't stress too much, this can be easily undone by removing or replacing the Job later.
3. Once you are sure of your configuration, select:
4. You will now be shown that your Job is awaiting review. To view the deployment, select View deployment run or navigate to the Job Deployments tab

5. On the Job Deployment page, you will see several pieces of useful information, let's go over a few
2. An approve/decline option will appear once the build succeeds; this is where a Target Environment owner can approve or decline a job deployment (this can also be done from within the associated Velocity notification)

- Image Build Status: If this is unsuccessful, it will give you logs to help troubleshoot what's wrong with your Job.
- Allowed Approvers: These individuals are the only people that can approve the deployment of your Job into the Target Environment; they are typically the owners of the Target Environment. If a Job has not been approved, it is because one of them has not done so.
6. Once your Job Deployment has been approved, you will see the job switch from a yellow icon to green, showing it has been deployed successfully!

Job Profile Page Showing Successful Deployment

## 7. Your job is now ready to go!

- If it is a "Scheduled" job, it will run when the next scheduled execution time passes.
- If it is "On Demand" as the one shown above, you can run the job at any time by selecting Run Now, shown as a green "play" button

## Additional Functionality

Once a Job has been deployed, the process of replacing (updating) or removing it is very easy.

From the Job Profile page, navigate to the Deployment and select the ellipsis menu. You will see a few options

1. View Logs: Allows you to view logs output by your job script, allowing for useful feedback or troubleshooting

2. Replace: Allows you to re-deploy the Job in place by updating it's version, instance size or schedule type. This is how Jobs are upgraded after a new Job Version is registered
3. Remove: Removes a Job Version Deployment from DSE.

## Troubleshooting

What happens if I don't have a main() and get an error like this?

Main file /home/sagemakeruser/<path>/<job> missing required pattern: ^(def main\(\):|main <function\(\) \{). Please visit the dsesdk docs site for more info

DSE requires your Job to have 2 things:

- main.R or main.py file

## callable main() function

Visit the DSE SDK documentation for further information

## DSE is telling me I don't have a ' clean' working directory - what does that mean and how do I fix it?

DseValidationError: Git working directory is not clean. Please commit changes and try again.

When I go to register_job_version, DSE is giving me this error, how do I fix it?
