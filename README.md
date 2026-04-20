https://learn.nareshit.com/courses/take/new-course-795/multimedia/70560082-session-8-24rth-nov


Good call—this is the right direction. Moving to a passwordless setup with AWS Systems Manager + OIDC will make your pipeline significantly safer and cleaner.

I’ll walk you through a production-ready setup step by step.

🔐 Goal

GitHub Actions → (OIDC) → AWS IAM Role → SSM → Windows EC2
✅ No passwords
✅ No open ports
✅ Fully auditable

🧱 1. Prepare your EC2 instance

On your Windows instance in Amazon EC2:

✅ Ensure SSM is working

Most modern AMIs already have the SSM agent installed.

Check:

Get-Service AmazonSSMAgent

If not installed, install the SSM agent.

✅ Attach IAM role to EC2

Attach a role with this policy:

AmazonSSMManagedInstanceCore

This allows SSM to control the instance.

🔑 2. Create IAM Role for GitHub Actions (OIDC)

In AWS IAM:

➤ Add Identity Provider
Provider: token.actions.githubusercontent.com
Audience: sts.amazonaws.com
➤ Create IAM Role

Trust policy (replace repo details):

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<AWS_ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
➤ Attach permissions policy

Minimum required:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:SendCommand",
        "ssm:GetCommandInvocation"
      ],
      "Resource": "*"
    }
  ]
}

(You can scope this down later to specific instance IDs.)

⚙️ 3. GitHub Actions Workflow

Here’s a clean working example using GitHub Actions:

name: Deploy via SSM

on:
  push:
    branches: [main]

permissions:
  id-token: write   # required for OIDC
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::<AWS_ACCOUNT_ID>:role/GitHubActionsSSMRole
          aws-region: ap-south-1

      - name: Run command on EC2 via SSM
        run: |
          aws ssm send-command \
            --instance-ids "i-1234567890abcdef0" \
            --document-name "AWS-RunPowerShellScript" \
            --parameters commands=["hostname","Get-Date"] \
            --output text
🧪 4. (Optional) Wait for command output

To fetch results:

COMMAND_ID=$(aws ssm send-command \
  --instance-ids "i-1234567890abcdef0" \
  --document-name "AWS-RunPowerShellScript" \
  --parameters commands=["hostname"] \
  --query "Command.CommandId" \
  --output text)

aws ssm get-command-invocation \
  --command-id $COMMAND_ID \
  --instance-id "i-1234567890abcdef0"
🔒 5. Security wins you just achieved

Compared to your old setup:

❌ No EC2 password stored anywhere
❌ No RDP (3389) exposed
❌ No WinRM ports exposed
❌ No long-lived AWS keys
✅ Temporary credentials via OIDC
✅ IAM-based access control
✅ Full audit logs in CloudTrail
✅ Works inside private VPC (no public IP needed)
⚠️ Common mistakes to avoid
Forgetting id-token: write → OIDC won’t work
Using Resource: * forever → restrict later
Not attaching IAM role to EC2 → SSM fails silently
Instance not appearing in SSM → agent or IAM issue
🧠 Final takeaway

This setup is:

Industry standard
Used in production by most AWS teams
Far more secure than direct RDP/WinRM from CI/CD
