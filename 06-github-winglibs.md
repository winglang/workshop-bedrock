In this section we will implement a github app that listen to PR
and modifies the changed markdown files.

## Prerequisites

Please make sure you have ngrok installed.

[Install ngrok](https://ngrok.com/docs/getting-started/)

## The GitHub App

We will use `@winglibs/github` module that uses an API gateway and 
[probot/probot](https://github.com/probot/probot) framework to create 
an easy way to develop, test locally and deploy GitHub applications 
to production. 

In order to configure the app, you will need the webhook secret, app id, and private key pem file path available.

But first, lets start by installing it:
```sh
npm i -s @winglibs/github
```

Now, lets create a `github-app.main.w` file

```js
bring github;
bring fs;

class SimpleCredentialsSupplier impl github.IProbotAppCredentialsSupplier {
   
   pub inflight getId(): str {
    // TODO place app id
    return "app id";
   }

   pub inflight getWebhookSecret(): str {
    // TODO place webhook secret
    return "some-webhook-secret";
   }

   pub inflight getPrivateKey(): str {
    // TODO place path to private key
    return fs.readFile("/path/to/private-key.pem");
   }
}

let credentialsSupplier = new SimpleCredentialsSupplier();
let markdown = new github.ProbotApp(
  credentialsSupplier: credentialsSupplier,
  onPullRequestOpened: inflight () => {
    log("PR was Opened");
  },
  onPullRequestReopened: inflight () => {
    log("PR was Re-Opened");
  }
);
```

Once you run `wing run github-app.main.w` you can notice that the 
Webhook URL that we initially setup to be "http://this-will-change.com" 
has changed to a ngrok generate URL. You can observe this under your application configuration (see https://github.com/settings/apps)

This URL will constantly change on any new hot reload of the localhost application. 

### Lets create a PR

You should see the log notifying you that a PR was created?

### Challenge

Follow @winglibs/github [README.md](https://www.npmjs.com/package/@winglibs/github) file to see how to replace all 
changed markdown files with a uppercase version of them.

🚀 Great, next we will use `anthropic.claude-v2:1` foundation model to make the correct commit 🚀
