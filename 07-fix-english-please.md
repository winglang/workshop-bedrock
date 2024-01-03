In this section we will be using `anthropic.claude-v2:1` Foundation Model
to correct the english for every `*.md` file that we find

## Challenge

Based on `@winglibs/github` [README.md](https://www.npmjs.com/package/@winglibs/github) and the work that we've done in the [bedrock session](./04-bedrock.md) try to have a commit that spell checks all markdown files

<details>
<summary>Solution</summary>
     
 
    bring cloud;
    bring util;
    bring github;
    bring bedrock;
    bring fs;

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


    class SimpleCredentialsSupplier impl github.IProbotAppCredentialsSupplier {
      
      pub inflight getId(): str {
      return "772448";
      }

      pub inflight getWebhookSecret(): str {
      return "this-is-a-bad-secret";
      }

      pub inflight getPrivateKey(): str {
      return fs.readFile("/Users/eyalkeren/Downloads/english-grammar-fixer.2024-01-02.private-key.pem");
      }
    }


    let credentialsProvider = new SimpleCredentialsSupplier();

    let markdown = new github.ProbotApp(
      credentialsSupplier: credentialsProvider,
      onPullRequestOpened: handler,
      onPullRequestReopened: handler
    );    

     
     
</details>

## Not Ready for AWS yet - Lets use Secrets

Currently the credentials are known at build time, which means that they
are included in the application bundle (js and terraform). We should
move to using `cloud.Secret` for each of these values instead

### Challenge

Can you implement the `github.IProbotAppCredentialsSupplier` interface using the `cloud.Secret` primitives?

<details>
<summary>Solution</summary>


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
</details>     

### How to use Secrets on Sim

To make the above `SecretCredentialsProvider` work on the simulator you
need to create a JSON file under `~/.wing/secret.json` with the following keys:
- `gitub.app_private_key`
- `github.webhook_secret`
- `github.app_id`

Storing the pem file in a json format is a bit tricky, because of the sensitivity to any white space chars and its structure.

You can use `jq` ([Install jq](https://jqlang.github.io/jq/)) to generate your `~/.wing/secret.json` with the following command:

```sh
jq --null-input \
  --arg secret "this-is-a-bad-secret" \
  --arg app_id "some-app-id" \
  --arg private_key "$(cat '/path/to/private-key.pem')" \
  '{"github.webhook_secret": $secret, "github.app_id": $app_id, "gitub.app_private_key": $private_key}' > ~/.wing/secrets.json
```

### How to create Secrets on AWS

The simplest way is to use AWS CLI in order to generate these secrets.

```sh
aws secretsmanager create-secret --name gitub.app_id --secret-string "APP-ID"

aws secretsmanager create-secret --name gitub.webhook_secret --secret-string "this-is-a-bad-secret"

aws secretsmanager create-secret --name gitub.app_private_key --secret-string "$(cat '/path/to/english-grammar-fixer.private-key.pem')"
```
