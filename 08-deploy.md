In this section, we will deploy the application into our own AWS account.

## The Code

Here is the complete code; feel free to use or modify it.

```ts
bring cloud;
bring expect;
bring util;
bring github;
bring bedrock;
bring ex;

let claude = new bedrock.Model("anthropic.claude-v2:1") as "claude";

let correct = inflight (s: str):str => {
  log("invoking {claude.modelId}");
  let sysPrompt = "Correct the English including typos grammar and punctuation in the following markdown text, reply with corrected markdown text delimited the text with a <body> tag.";
  let res = claude.invoke({
    "prompt": "Human:{sysPrompt}\n{s}nAssistant:",
    "max_tokens_to_sample": 2048,
  });
  log("{res}"); 
  let r = res.get("completion").asStr().split("<body>").at(1).split("</body>").at(0);
  log(r);
  return r;
};

let handler = inflight (ctx) => {
  let repo = ctx.payload.repository;

  // find all changed mdfiles by comparing the commits of the PR
  let compare = ctx.octokit.repos.compareCommits(
    owner: repo.owner.login,
    repo: repo.name,
    base: ctx.payload.pull_request.base.sha,
    head: ctx.payload.pull_request.head.sha,
  );

  log("{Json.stringify(compare.data.commits)}");
  
  let mdFiles = MutMap<str>{};
  for commit in compare.data.commits {
    let commitContent = ctx.octokit.repos.getCommit(
      owner: repo.owner.login,
      repo: repo.name,
      ref: ctx.payload.pull_request.head.ref,
    );
    log("{Json.stringify(commitContent)}");
    if let files = commitContent.data.files {
      for file in files {
        log("{Json.stringify(file)}");
        if file.filename.endsWith(".md") && (file.status == "modified" || file.status == "added" || file.status == "changed") {
          mdFiles.set(file.filename, file.sha);
        }
      }
    }
  }

  // list over mdfiles and update them
  for filename in mdFiles.keys() {
    let contents = ctx.octokit.repos.getContent(
      owner: repo.owner.login,
      repo: repo.name,
      path: filename,
      ref: ctx.payload.pull_request.head.sha);
    
    let fileContents = util.base64Decode("{contents.data.content}");
      
      
    ctx.octokit.repos.createOrUpdateFileContents(
      owner: repo.owner.login,
      repo: repo.name,
      branch: ctx.payload.pull_request.head.ref,
      sha: contents.data.sha,
      path: filename,
      message: "chore: Correct the spelling, grammar & punctuation in '{filename}'",
      content: util.base64Encode(correct(fileContents))
    );
  }
};


class SecretCredentialsProvider impl github.IProbotAppCredentialsSupplier{
  privateKey: cloud.Secret;
  id: cloud.Secret;
  webhookSecret: cloud.Secret;
  new() {
    this.privateKey = new cloud.Secret(name: "gitub.app_private_key") as "gitub.app_private_key";
    this.id = new cloud.Secret(name: "github.app_id") as "github.app_id";
    this.webhookSecret = new cloud.Secret(name: "github.webhook_secret") as "github.webhook_secret";
  }
  pub inflight getId(): str {
    return this.id.value();
  }
  pub inflight getWebhookSecret(): str {
    return this.webhookSecret.value();
  }
  pub inflight getPrivateKey(): str {
    return this.privateKey.value();
  }
}

let credentialsProvider = new SecretCredentialsProvider();

let markdown = new github.ProbotApp(
  credentialsSupplier: credentialsProvider,
  onPullRequestOpened: handler,
  onPullRequestReopened: handler
 );    

```

## Create a New GitHub App

We want to have different GitHub apps for Development and Production. 
Please go to the [Create GitHub App Instructions](./05-create-github-app.md) and create another app with the same name but without the "(dev)" postfix (e.g., "My English Editor") and a different webhook secret.

## Store Secrets in AWS Secret Manager

We want our application to read the Secrets from AWS Secret Manager. 
The simplest way is to use the AWS CLI to generate these secrets.

```sh
aws secretsmanager create-secret --name github.app_id --secret-string "APP-ID"
aws secretsmanager create-secret --name github.webhook_secret --secret-string "some-good-secret"
aws secretsmanager create-secret --name github.app_private_key --secret-string "$(cat '/path/to/private-key.pem')"
```

## Create a New App

Follow the instructions on how to [Deploy to AWS](https://www.winglang.io/docs/start-here/aws).

**Notice**: The webhook URL of your new GitHub app should be modified to an AWS public URL once `terraform apply` is completed.

🚀 You now have your own grammar fixer. Hope you had fun!!! 🚀

