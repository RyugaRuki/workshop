---
title : "Build and synthesize locally"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### Step 1: Install dependencies

```bash
cd ~/Code/Technology-News-Collection-and-Summarization-System
bun install
```

#### Step 2: Run the production build

```bash
bun run build
```

![CloudBrief build succeeded](/images/5-Workshop/cloudbrief-evidence/build-ok.png)

#### Step 3: Run regression tests

```bash
bun run test
```

![CloudBrief tests passed](/images/5-Workshop/cloudbrief-evidence/tests-ok.png)

The suite covers route validation, API-key protection, SSRF-safe fetching, collection, extraction, image processing, social actions, and infrastructure assertions.

#### Step 4: Synthesize CloudFormation

```bash
bun run cdk synth
```

![CDK synthesis succeeded](/images/5-Workshop/cloudbrief-evidence/cdk-synth-ok.png)

Synthesis is the final local gate before deployment. It confirms the CDK app can produce a CloudFormation template without changing AWS resources.

#### Step 5: Review security-sensitive values

Before deploy, verify that the following values come from local configuration, Keychain, or approved AWS stores and never from Git:

- Admin API key and its hash.
- CloudFront origin header secret.
- Google OAuth client configuration.
- Notification email.
- AWS access keys.
