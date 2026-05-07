[Image description: Bayer corporate logo/header banner at the top of the document, likely featuring the Bayer cross logo in corporate colors as a title page header.]

## App Playbook

[Image description: A decorative graphic or banner associated with the "App Playbook" heading, possibly featuring an icon representing applications, user interfaces, or interactive dashboards (such as browser windows, UI components, or app-related imagery).]

## What is an App?

An App is a program that combines a user interface, business logic, data handling, and output so users can interact with data, run computations, and see results in real time.

## When to use an App

App are best for recurring problems or repetitive tasks that benefit from an automated, user-friendly interface. They let users avoid manual steps and standardize workflows.

## What types of Apps does DSE support?

- Python: Dash and Streamlit

- R: Shiny

## Access and integrations

Registered Apps can securely interact with infrastructure and services, including:

- Amazon S3 (via IAM roles)
- CSW (via Google service accounts)

- Registered models and scheduled jobs within the DSE

This access enables data retrieval, model execution, scheduled processing, and observability of usage and costs.

## I Want To...

-  Register an App

-  Deploy an App

-  Launch an App

-  Grant App View Access

- [x]  View App Logs

- [x]  Replace or Remove an App

 Additional App Resources

- [x]  Review App FAQ

- [ ]  View a Full App Deployment Demo

## Register an App

This section explains how to register and manage Apps in DSE.

## Registration steps

1. Register the App in DSE once (creates the App record).
2. Immediately register the App's first Version .
3. After that, register new Versions only when you update the code.

## Lifecycle (high level)

- One-time setup
- Develop
- Register App → register first Version
- Deploy
- Test / troubleshoot
- Maintenance
- Grant access
- Update: register new Version and replace deployment
- Remove App (if needed)

## Quick considerations

- Increment the Version for every release.
- Test changes in a dev/local environment before registering a new Version.
- Use feature branches and pull requests to avoid introducing breaking changes.
- use non-prod to test before deploying to prod.

## Important details

- One App per repository: only a single App can be registered per repository. The App name you choose becomes permanently associated with that repository—pick a clear, stable name.
- Updating an App: bump the App Version for each release and re-register that Version.

## App registration demonstration

View an App registration demo on the right. Continue below for step-bystep instructions for App registration.

## App registration basics

- Develop anywhere: Build and test your app in a local IDE or the DSE SageMaker space.
- Test in Sandbox:
- Code Editor: Launch a VS Code session via the Code Editor's Remote Access toggle.
- RStudio: Select your ui.R or server.R file and click Run App in RStudio.
- Register in Sandbox: You must be in the Sandbox Environment SageMaker space to register your app with DSE.
- Repository layout: Keep your app and all dependencies in a single folder. DSE recursively scans only this specified directory during registration.
- Visibility:
- PUBLIC: The App is visible to All DSE users.
- PRIVATE: Access to the App is managed via PAPI Application Entitlements.

## Required Folder Contents

- Main script: app.py or app.R or ui.R and server.R
- Dependency list: requirements.txt , pyproject.toml , poetry.lock , requirements.R, or renv.lock .
- Start command: app.sh. Ensure your app binds to host 0.0.0.0 and port 8080 (e.g., 0.0.0.0:8080), as this is required for DSE deployment.

- Supporting files: Include any necessary static assets or configuration files.

Important: Ensure your workspace is clean by committing or stashing all changes before registering.

Tip: Use the Data Science Starter Kit for pre-configured templates to accelerate your deployment.

## Code snippets

## Python
