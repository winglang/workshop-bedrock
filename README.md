# Bedrock + Wing Workshop

_Building a serverless Workflow, Powered by AI_

This repo is a step-by-step guide for an online workshop ([Eventbrite](https://www.eventbrite.com/e/amazon-bedrock-winglang-tickets-769562721817)).

## Any issues? Questions? Let's talk!

Please [join Winglang's Slack](https://t.winglang.io/slack); we have a dedicated channel for this workshop [#workshop-bedrock](https://winglang.slack.com/archives/C06BWT4PC30).

## During the session, you'll learn the following

- Introduction to [Amazon Bedrock](https://aws.amazon.com/bedrock/)
- Use Amazon Bedrock Resource from Wing
- Create a Github App that edits files
- How to produce ready-to-apply Terraform assets

### Some General Resources

- [Learn about Inflights](https://www.winglang.io/docs/concepts/inflights) - Preflight and Inflight are Winglang's most important core concepts.  
- [Cheat Sheet](./cheatsheet.md) - Let's learn Wing in 5 minutes, shall we?
- [Language Reference](https://www.winglang.io/docs/language-reference) - A complete language reference for Winglang. 
- [Winglibs](https://github.com/winglang/winglibs) - Wing Trusted Library Ecosystem
- [Playground](https://www.winglang.io/play/) - Write and test Wing code online.

## The Application 

A Github Application that listens to any incoming pull requests and corrects spelling, grammar & punctuation for any `*.md` files that were changed during this pull request.

### Workshop Sessions 

1. **Setup & Prerequisites** - Tools, setup and getting access to AWS Bedrock Foundation Models ([link](./01-setup.md)) 
2. **Why Winglang** - Problems with serverless, and the Wing approach ([pdf](https://raw.githubusercontent.com/ekeren/react-wing-workshop/main/assets/why.pdf)). 
3. **Introduction to Bedrock** - A short introduction to Amazon Bedrock service by [Arik Porat](https://www.linkedin.com/in/arik-porat-15419426/), Solutions Architect at AWS.  
4. Using Bedrock in Wing - Installing & using `winglibs/bedrock` module ([link](./04-bedrock.md)).
5. Github App DYI - How to build your own Github Bot ([link](./05-github-diy.md))  
6. Wing's Github Module - Using `winglibs/github` instead to make an uppercase bot ([link](./06-github-winglibs.md)).
7. Spell Checking with Bedrock - Modifying the files based on `winglibs/bedrock` ([link]((./07-wrap.md))).
8. Thank You for Flying with Us - Deploy application to AWS ([link]((./08-deploy.md))).

---

### What's Next? 

In the next session, we will be using [LangChain](https://www.langchain.com/) with [AWS Bedrock](https://aws.amazon.com/bedrock/) to embed document into [Momento Vector Index](https://docs.momentohq.com/vector-index)

