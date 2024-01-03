In this section, we will create a new GitHub app with the 
required permissions to commit changes to an open pull request.

## Create a GitHub App

Go to https://github.com/settings/apps

Click "New GitHub App" and complete the form:
- GitHub App Name: my English editor (dev)
- Homepage URL: e.g., https://winglang.io
- Webhook:
  - Active: ✅
  - URL: http://this-will-change.com
  - Webhook secret: "this-is-a-bad-secret"
- Permissions -> Repository permissions:
  - Contents: Read and write 
  - Pull requests: Read and write
- Subscribe to events:
   - Pull request
   - Push
- Save
- Notice the app id and save it.
- Generate a private key for the app - The generated key will be downloaded to your computer.

## Install the App

Create a new repository in GitHub; we will use this repository for 
local development and testing.

Install the app by going to https://github.com/settings/apps.

Look for "my English editor" application and click **Edit**.

On the left side, click "Install App" and select the new repository as the installation target.

🚀 We will next use the app id, private key, and webhook secret to develop the app. 🚀
