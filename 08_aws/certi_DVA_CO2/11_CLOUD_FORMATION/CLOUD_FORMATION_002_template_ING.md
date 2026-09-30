---
CLOUD_FORMATION_000_plan.md
---

CLOUD_FORMATION_002_template.md
[image]

---

---

CLOUD_FORMATION_004_how_works.md

How CloudFormation Works

• Templates must be uploaded in S3 and then referenced in
CloudFormation
• To update a template, we can’t edit previous ones. We have to re-
upload a new version of the template to AWS
• Stacks are identified by a name
• Deleting a stack deletes every single artifact that was created by
CloudFormation.

[image]

---

CLOUD_FORMATION_004_deploying_template.md

Deploying CloudFormation Templates

• Manual way
• Editing templates in Infrastructure Composer or code editor
• Using the console to input parameters, etc…
• We’ll mostly do this way in the course for learning
purposes

• Automated way
• Editing templates in a YAML file
• Using the AWS CLI (Command Line Interface) to deploy
the templates, or using a Continuous Delivery (CD) tool
• Recommended way when you fully want to automate
your flow
