In this section, we will ensure that:
- Wing is installed and functioning properly on your machine.
- An AWS account is configured and CLI installed with the required credentials.
- Access is granted for Amazon Bedrock foundation models.
- ngrok is installed and working.

## Wing 

### Prerequisites

* [Node.js](https://nodejs.org/en/) (version 18.13.0 or higher)
* [VSCode](https://code.visualstudio.com/download)

### Wing Toolchain

The Wing Toolchain is distributed via [npm](https://www.npmjs.com/):

```sh
npm install -g winglang
```

Verify your installation:

```sh
wing --version
```

### Wing VSCode Extension

The Wing VSCode Extension adds syntax highlighting and other conveniences for the Wing Programming Language in [VSCode](https://code.visualstudio.com/).

To install the Wing VSCode extension, [download it from the VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=Monada.vscode-wing).

### Wing it

1. Create a new directory on your filesystem (e.g., `/tmp/wing-workshop`).
2. Start VSCode from this directory.
3. Create `main.w` with the following content:
```ts
bring cloud;

let queue = new cloud.Queue(timeout: 2m);
let bucket = new cloud.Bucket();
let counter = new cloud.Counter();

queue.setConsumer(inflight (body: str): str => {
  let next = counter.inc();
  let key = "key-{next}";
  bucket.put(key, body);
});
```

Verify that the Wing toolchain is working as expected:
```sh
wing run main.w
```

🚀 In the Wing Console, you can push messages to the Queue and observe the files created in the Bucket. 🚀

## AWS CLI, Terraform and Required Credentials

We will be using AWS Bedrock.
- [AWS account](https://aws.amazon.com/free) 
- [Associated credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) that allow you to create resources.
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) with configured credentials. See [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) for more information.
- [Terraform](https://terraform.io/downloads) - will be required for deploying your application to your own account.

## ngrok

We will be using ngrok for tunneling GitHub webhooks into our localhost 
from a known and accessible address. 

- [Install ngrok](https://ngrok.com/docs/getting-started/)


## Amazon Bedrock Foundation Model Access

AWS Bedrock can only be accessed from the us-east-1 region. Make sure all your credentials and applications reside in this region, so you don’t encounter a permission denied exception.

1. Log into the AWS Console and navigate to Amazon Bedrock.
2. In the Bedrock screen, at the bottom left-hand corner, click on the model access menu and request access to Anthropic Claude (& Claude Instant) foundation Models.
3. You should receive an email about your request and also a confirmation of access email after a few minutes.
