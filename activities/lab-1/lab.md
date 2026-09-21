---
week: 8
graded: true
counts_toward: Midterm Class Standing — Lab Activities (30%)
duration: 45 minutes
mode: Team, one IAM user per team. Evidence goes to the private class form or TBL Hub, not to this repository.
coverage: IAM users, writing and attaching a policy, permissions boundary, least privilege, CloudTrail
---

# Lab 1: Write and Attach an IAM Policy

> This lab does not use a Pull Request. Do not post screenshots or evidence in this repository. Screenshots show the account ID. Submit your evidence to the private class form or TBL Hub.

## Team rules

- Teams have 3 or 4 people and sit in one pod. One IAM user belongs to the whole team. Every member can sign in on their own PC. Only the Driver clicks Create and Launch, so two people never change the same resource at once.
- Roles rotate at each lab part (Lab 1 Parts A to B, C, D to E, then Lab 2 steps 1 to 2, 3 to 4, 5 to 7): Driver (types), Navigator (reads the error aloud), Recorder (writes the team notes), Reviewer (checks each step against the lab).
- The password emailed to you belongs to your team. Do not paste it into any chat, form, or repository. It stops working when the lab access window ends.
- Never create anything outside the Singapore Region (Asia Pacific, ap-southeast-1). The account blocks it.
- Every resource you create carries the tag `team` with your IAM user name as the value. Some steps fail without it. That is on purpose.

## What the account does automatically

| Control | Rule |
| --- | --- |
| Region lock | Only ap-southeast-1 works |
| Instance type lock | Only t3.micro can be launched |
| Disk size lock | Volumes over 8 GiB are denied |
| Maximum group size | Auto Scaling groups are capped at 2 instances |
| Automatic cutoff | 90 minutes after an instance launches, the account ends it or sets its group to 0 |
| Ownership by tag | You can change only resources tagged `team=<your user name>` |
| Time window | Launch and scaling actions are denied after the announced end time |
| Logging | Every call is recorded in CloudTrail |

**Goal:** Write an IAM policy, attach it to your team's IAM user, and use it to launch one t3.micro instance. Then attach a policy that is too wide, and see the permissions boundary stop it.

**Prerequisites:** Your team's IAM user name and password. A browser. The starter policy in this folder, [`lab1-policy-starter.json`](lab1-policy-starter.json).

**Duration:** 45 minutes. Answer the questions only after Part E.

**Taught in the lecture:** the console layout, Regions, the EC2 launch page, IAM users, groups, and policies, the permissions boundary, how AWS decides, reading a denial, and CloudTrail Event history.

In every step, replace `<user>` with your IAM user name, for example `acsad-g03`.

## Part A. Sign in and find your account ID (5 minutes)

1. Open the sign-in URL from your email. Enter the account ID or alias, your IAM user name, and the password. Set a new password if the page asks.
2. In the top bar, open the Region menu and choose Asia Pacific (Singapore) `ap-southeast-1`.
3. In the top bar, click your user name. Copy the 12-digit Account ID from the menu.
4. Open the section group link on the board (for example `acsad-class`). The User groups list page is blocked for your user, so use the link. Open the Permissions tab. Open the policy `umak-lab-t0-observe` and choose the JSON tab. Notice that no statement allows `ec2:RunInstances`.
5. Open the user link on the board. Under Permissions boundary, open `umak-lab-boundary`. Find the statement `DenyAnyInstanceTypeButT3Micro`.

## Part B. Try to launch, and read the denial (5 minutes)

1. Open EC2, then Instances, then Launch instances.
2. Name: `<user>-test`. Application and OS Image: Amazon Linux 2023. Instance type: `t3.micro`. Key pair: Proceed without a key pair. Leave the other settings.
3. Click Launch instance. The console shows an error.
4. Copy the full error text into your team notes. Underline the action name after "not authorized to perform".

## Part C. Write your policy (12 minutes)

1. Open IAM, then Policies, then Create policy.
2. Choose the JSON editor. Delete the text in the editor. Paste the starter policy.
3. Replace every `<ACCOUNT_ID>` with your account ID from Part A. Use Find and replace if the editor offers it.
4. Fill the three blanks in the first statement, `RunOnlyT3MicroInstances`:
   - `"Action"`: the action from your Part B error.
   - `"Resource"`: the resource type from your error. It replaces `____` before `/*`.
   - `"ec2:InstanceType"`: the instance type you tried to launch.
5. Confirm the editor shows no red errors. Security warnings and suggestions are expected. A notice that you cannot validate the policy is also expected, because your user cannot call the policy validator. Click Next.
6. Policy name: `<user>-launch`. The name must start with your user name and a hyphen. Any other name is denied. Click Create policy.

## Part D. Attach the policy and launch (10 minutes)

1. Open the user link from Part A. Choose the Permissions tab, then Add permissions, then Attach policies directly.
2. In the search box, type `<user>-launch`. Tick the box next to your policy. Click Next, then Add permissions.
3. Wait 15 seconds. Refresh the page. The policy `<user>-launch` now appears under Permissions policies.
4. Open EC2, then Security Groups, then Create security group. Name: `<user>-web`. Description: `Lab 1`. VPC: the default VPC. Inbound rules: Add rule, Type HTTP, Source Anywhere-IPv4. Do not add a tag yet. Click Create security group. Copy the error text into your team notes.
5. Repeat step 4. This time, under Tags, click Add new tag. Key: `team`. Value: `<user>`. Click Create security group. It succeeds.
6. Open EC2, then Instances, then Launch instances. Use the settings from Part B. Under Network settings, choose Select existing security group and pick `<user>-web`. Under Advanced or Resource tags, add tag Key `team`, Value `<user>`, Resource types Instances. Click Launch instance. Do not skip the tag. An instance without the `team` tag launches, but your user cannot terminate it, even with `ec2:*` attached. Only the 90-minute cutoff ends it.
7. Open the instance list. Wait until Instance state is Running. Write the time in your team notes. The 90-minute cutoff clock starts now.

If step 6 fails, copy the error text. Read the action and the resource in it. Find the statement in your policy that allows it, and fix the policy. To edit: IAM, Policies, `<user>-launch`, Edit, JSON tab. Save changes, wait 15 seconds, and retry.

## Part E. Test the boundary (8 minutes)

1. Launch a second instance as in Part D, but choose instance type `t3.small`. Copy the error. It says explicit deny in a permissions boundary.
2. Open IAM, then Policies, then Create policy, then the JSON editor. Paste this policy:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{ "Effect": "Allow", "Action": "ec2:*", "Resource": "*" }]
   }
   ```

   Name it `<user>-too-wide`. Create it. Attach it to your user as in Part D steps 1 to 3.
3. Wait 15 seconds. Try the `t3.small` launch again. It is still denied.
4. Switch the Region to Asia Pacific (Tokyo). Try to launch any instance. Copy the error. Switch back to Singapore.
5. Open your user, then the Permissions tab. Select `<user>-too-wide`, click Remove, and confirm. Then open IAM, Policies, select `<user>-too-wide`, and choose Delete.
6. Open EC2, Instances. Select the instance from Part D. Choose Instance state, then Terminate (delete) instance. This works because the instance carries your `team` tag.
7. Open CloudTrail, then Event history. Set Lookup attributes to User name and enter `<user>`. Open one `RunInstances` event whose Error code is `Client.UnauthorizedOperation`. Events with `Client.DryRunOperation` are the console's own permission check, so skip them. Find the field `errorMessage`. New events can take several minutes to appear. If the list is empty, do Part F first and come back.

## Part F. Questions and evidence (5 minutes)

Answer in your team notes.

1. Which action did the Part B error name?
2. In your policy, which condition limits `ec2:RunInstances`?
3. After you attached `ec2:*` on `*`, why was `t3.small` still denied? Name the boundary statement.
4. Why is `ec2:*` on `*` a poor policy even with a boundary?
5. In two sentences: what does the boundary control that your policy cannot?

Submit to the private form or TBL Hub:

1. Screenshot of the Part B error with your user name visible.
2. Screenshot of your user's Permissions tab that lists `<user>-launch`.
3. Screenshot of the instance in the Running state.
4. Screenshot of the `t3.small` or Tokyo denial.
5. Screenshot of the CloudTrail event with `errorMessage`.
6. The three filled blanks and your answers to the five questions.

Every screenshot needs one sentence that says what it proves. Screenshots show the account ID. Never post them publicly.

**Expected output:** One denied launch before your policy. One created and attached policy. One Running t3.micro. Two boundary denials that stay denied after `ec2:*` is attached. One CloudTrail event.

**Limitation and next step:** This lab writes an identity policy for a user. Week 12 gives an instance an IAM role instead of keys.

**No-account alternative:** Use the paper set below, `Lab 1 paper set`.

### Lab 1 paper set (no AWS account)

Write the four blanks in the starter policy. Then evaluate each request against the starter policy and the boundary. Write Allow or Deny and name the deciding statement.

| Request | Decision | Deciding statement |
| --- | --- | --- |
| Launch a t3.micro in Singapore | | |
| Launch an m5.large in Singapore | | |
| Launch a t3.micro in Tokyo | | |
| Create a security group without the `team` tag | | |
| Terminate an instance tagged `team=acsad-g02` while signed in as `acsad-g03` | | |
| Create an IAM user | | |
