---
week: 8
graded: true
counts_toward: Midterm Class Standing — Lab Activities (30%)
duration: 45 minutes
mode: Team, one IAM user per team. Evidence goes to the private class form or TBL Hub, not to this repository.
coverage: EC2 launch templates, security groups, Auto Scaling, CloudWatch
---

# Lab 2: Build an EC2 Auto Scaling Group

> This lab does not use a Pull Request. Do not post screenshots or evidence in this repository. Screenshots show the account ID. Submit your evidence to the private class form or TBL Hub.

**Goal:** Build a group of servers that adds a server when load rises, and watch it happen.

**Prerequisites:** Lab 1 finished. The instructor has unlocked Auto Scaling for your section. The User-Data text in this folder, [`lab2-user-data.sh`](lab2-user-data.sh).

**Duration:** 45 minutes. Answer the questions only after Step 7.

**Taught in the lecture:** launch templates, security groups, Auto Scaling groups and target tracking, the group page tabs, and how to read a scale-out.

## How the lab works

```
You (browser)
   |  http://<public-ip>/burn
   v
Instance 1 (t3.micro) ----- CPU rises above 40 percent
   |                            |
   |                    CloudWatch alarm (created by the scaling policy)
   |                            |
   |                    Auto Scaling group: desired 1 -> 2 (maximum is 2)
   |                            |
   |                            v
   |                     Instance 2 launches in another Availability Zone
   +--> Each page shows its instance ID and Availability Zone
```

There is no load balancer this week. You open each instance by its public IP.

## Steps

In every step, replace `<user>` with your IAM user name.

**1. Confirm the Region and the security group (3 minutes).**

1. Check that the top bar shows Asia Pacific (Singapore).
2. Open EC2, then Security Groups. If `<user>-web` exists from Lab 1, keep it. Otherwise create it as in Lab 1 Part D step 5.
3. Select `<user>-web`, open the Inbound rules tab, and confirm one rule: HTTP, port 80, source `0.0.0.0/0`. If it is missing, click Edit inbound rules, Add rule, then Save rules.

**2. Create the launch template (10 minutes).**

1. Open EC2, then Launch Templates, then Create launch template.
2. Launch template name: `<user>-lt`. Template version description: `Lab 2`.
3. Tick Provide guidance to help me set up a template that I can use with EC2 Auto Scaling.
4. Under Template tags, add Key `team`, Value `<user>`.
5. Application and OS Images: choose Quick Start, then Amazon Linux 2023 AMI.
6. Instance type: `t3.micro`.
7. Key pair: Don't include in launch template.
8. Network settings: leave Subnet as Don't include in launch template. Under Security groups, choose `<user>-web`.
9. Under Resource tags, click Add tag three times. Use Key `team` Value `<user>`, Key `lab` Value `umak`, and Key `Name` Value `<user>-web`. Set Resource type to Instances for each.
10. Open Advanced details. Set Detailed CloudWatch monitoring to Enable.
11. Scroll to User data. Paste the full text of `lab2-user-data.sh`.
12. Click Create launch template. Confirm the success message.

**3. Create the Auto Scaling group (12 minutes).**

1. Open EC2, then Auto Scaling Groups, then Create Auto Scaling group.
2. Step 1: Name `<user>-asg`. Launch template: `<user>-lt`. Version: Latest. Click Next.
3. Step 2: VPC: the default VPC. Availability Zones and subnets: select two subnets in different Availability Zones. Click Next.
4. Step 3: Load balancing: No load balancer. Health checks: leave EC2. Under Additional settings, tick Enable group metrics collection within CloudWatch. Click Next.
5. Step 4: Desired capacity `1`, Minimum capacity `1`, Maximum capacity `2`. Under Scaling policies, choose Target tracking scaling policy. Metric type: Average CPU utilization. Target value: `40`. Instance warmup: `60` seconds. Click Next.
6. Step 5: Add notifications. Click Next without changes.
7. Step 6: Add tags. Click Add tag. Key `team`, Value `<user>`, and tick Tag new instances. Add a second tag: Key `lab`, Value `umak`, and tick Tag new instances. Click Next.
8. Step 7: Review. Click Create Auto Scaling group.

If creation fails, copy the full error text. Read the action and the resource in it. Tell the instructor the action name.

**4. Check the first instance (4 minutes).**

1. Open the group `<user>-asg`, then the Instance management tab.
2. Wait until one instance shows Lifecycle InService.
3. Click the instance ID. Copy the Public IPv4 address.
4. Open `http://<public-ip>` in a new browser tab. Use http, not https.
5. Write down the instance ID and the Availability Zone shown on the page.

**5. Trigger the load (10 minutes).**

1. Open `http://<public-ip>/burn`. The page says it is burning both vCPUs.
2. On the group page, open the Monitoring tab, then EC2. Watch Average CPU utilization. It rises in one to three minutes.
3. Open the Activity tab. Wait for a line that says a new instance is launching. Expect this in three to six minutes.
4. On the Instance management tab, wait until two instances are InService.
5. Open the second instance's public IP in a new tab. Write down its instance ID and Availability Zone.
6. Open `http://<first-public-ip>/stop` to end the load on the first instance.

The group will not shrink during class. Scale-in waits about fifteen minutes of low CPU. The automatic cutoff ends the group before then if you do nothing.

**6. Replace an instance by hand (3 minutes).**

1. Open EC2, then Instances. Select one of the two group instances. Choose Instance state, then Terminate (delete) instance.
2. Open the group's Activity tab. Watch the group launch a replacement.

**7. Clean up (3 minutes).**

1. Open EC2, Auto Scaling Groups. Select `<user>-asg`, choose Delete, type `delete`, and confirm.
2. Open EC2, Launch Templates. Select `<user>-lt`, choose Actions, then Delete template, and confirm.
3. Open EC2, Security Groups. Select `<user>-web`, choose Actions, then Delete security group. It can take a minute to delete while instances shut down. Retry after one minute.

## Questions and evidence

Answer after Step 7.

1. Why did the group stop at 2 instances?
2. Why did terminating an instance by hand not remove the cost?
3. Why is the target value 40 percent and not 90 percent?
4. What did the automatic cutoff protect us from?
5. What changes when a load balancer sits in front of the group?

Submit to the private form or TBL Hub:

1. Screenshot of the Activity tab that shows the scale-out.
2. Screenshot of the Monitoring tab CPU chart.
3. Two screenshots of the page, one from each instance, that show different instance IDs.
4. Screenshot of the launch template summary.
5. Screenshot of the group's Details tab with desired 1, minimum 1, maximum 2.
6. Your answers to the five questions.

Every screenshot needs one sentence that says what it proves.

**Expected output:** A group that grows from 1 to 2 instances within about 6 minutes of `/burn`. Two different instance IDs. One replacement instance after you terminate one by hand.

**Limitation and next step:** With no load balancer, users must know each address. This lab does not show scale-in, and it does not use an ELB health check. Week 10 adds a shared load balancer and target group.

**No-account alternative:** Use the paper timeline. The handout shows a CPU line over 20 minutes and a policy with target 40 and maximum 2. Mark on the timeline when the group adds an instance, and explain why it stops adding at 2.

## Take-home window (fallback only)

Use this only if a section runs out of class time.

- Window: from the end of class until the announced end time (the instructor posts a date and time in Manila time). After the time, launch and scaling actions are denied and the account ends what runs.
- Do the lab in one sitting of about 45 minutes. The automatic cutoff ends any instance 90 minutes after it launches.
- If the console denies an action with a message about time, the window closed. Do not retry. Submit what you have with a note.
- If you finish early, delete the Auto Scaling group, the launch template, and the security group.
- Support: post the exact error text in the section channel. Do not post screenshots that show your password or the account ID.
