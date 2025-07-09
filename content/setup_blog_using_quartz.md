---
lastSync: Sat May 03 2025 15:05:01 GMT+0530 (India Standard Time)
title: Setup Blog using Quartz
draft: true
---

### Update Deployment Script
To set up a blog using Quartz, you need to update the deployment script in your GitHub Actions workflow. This involves modifying the `deploy.yml` file to ensure that it uses the correct versions of actions and configurations.

For details please refer to the [Custom github action worflow](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow) documentation.

### Pushing changes
After updating the deployment script, you need to push the changes to your GitHub repository. This will trigger the GitHub Actions workflow to run and deploy your blog.
using the following command:

```bash
npx quartz sync
```
This command will synchronize your local setup with the remote repository, ensuring that all changes are pushed correctly.

### Syncing newer Quartz version
If you are using an older version of Quartz, you may need to update it to the latest version. You can do this by running the following command:
```bash
npx quartz update
```
This command will update your Quartz installation to the latest version, ensuring that you have all the latest features and fixes.
You may encounter merge conflicts if you have made local changes that conflict with the updates. In such cases, you will need to resolve the conflicts manually before proceeding.

Also, I usually tend to delete all the workflow file synced incase of quartz update other than the `deploy.yml` file to keep things simple.

### Verify Deployment
Once the changes are pushed, you can verify the deployment by checking the GitHub workflow status. Constituting of two steps `build` and `deploy`, the workflow should complete successfully without any errors. You can view the status of the workflow in the "Actions" tab of your GitHub repository.
