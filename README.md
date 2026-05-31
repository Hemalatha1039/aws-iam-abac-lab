# AWS IAM ABAC Lab - Tag-Based Access Control for EC2

## Problem Statement

How do you prevent one team from accidentally managing another team's AWS resources when both teams require the same permissions?

Consider the following scenario:

* Pegasus Team manages Project Pegasus resources.
* Unicorn Team manages Project Unicorn resources.

Both teams need the ability to manage Amazon EC2 instances.

A traditional RBAC approach often requires separate IAM policies for each team or project. As organizations grow, this leads to:

* Increased policy sprawl
* Higher administrative overhead
* More complex permission management
* Increased risk of misconfigured access

This lab demonstrates how AWS IAM Attribute-Based Access Control (ABAC) can solve this problem using tags.

---

# Solution Overview

Instead of creating separate policies for each project, access decisions are driven by tags attached to:

* IAM Users (Principal Tags)
* EC2 Instances (Resource Tags)

A single IAM policy is shared by all users and dynamically evaluates whether the user's tags match the target resource's tags.

---

# Architecture

```text
                    ┌─────────────────────┐
                    │  EC2ABACUsers Group │
                    └──────────┬──────────┘
                               │
                               ▼
                EC2-ABAC-Team-Project-Policy
                               │
          ┌────────────────────┴────────────────────┐
          ▼                                         ▼

    PegasusUser                               UnicornUser

 Team=Engineering                         Team=Engineering
 Project=Pegasus                          Project=Unicorn

          │                                         │
          └─────────────────┬───────────────────────┘
                            ▼

                 AWS ABAC Policy Evaluation

                            │

          ┌─────────────────┴────────────────────┐
          ▼                                      ▼

      Pegasus EC2                           Unicorn EC2

 Team=Engineering                      Team=Engineering
 Project=Pegasus                       Project=Unicorn
```

---

# Services Used

* AWS Identity and Access Management (IAM)
* Amazon EC2

---

# IAM Design

## IAM User Group

Created a user group:

```text
EC2ABACUsers
```

Purpose:

* Centralized permission management
* Single policy attachment
* Easier onboarding of new users

---

## IAM Users

### PegasusUser

| Key     | Value       |
| ------- | ----------- |
| Team    | Engineering |
| Project | Pegasus     |

### UnicornUser

| Key     | Value       |
| ------- | ----------- |
| Team    | Engineering |
| Project | Unicorn     |

---

## EC2 Resource Tags

### Pegasus EC2

| Key     | Value       |
| ------- | ----------- |
| Team    | Engineering |
| Project | Pegasus     |

### Unicorn EC2

| Key     | Value       |
| ------- | ----------- |
| Team    | Engineering |
| Project | Unicorn     |

---

# Custom ABAC Policy

Policy Name:

```text
EC2-ABAC-Team-Project-Policy
```

Policy Document:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDescribeInstances",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeTags"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowManageOwnProjectInstances",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Team": "${aws:PrincipalTag/Team}",
          "ec2:ResourceTag/Project": "${aws:PrincipalTag/Project}"
        }
      }
    }
  ]
}
```

---

# Implementation Steps

## Step 1

Create IAM Users:

* PegasusUser
* UnicornUser

---

## Step 2

Create IAM User Group:

```text
EC2ABACUsers
```

---

## Step 3

Create the custom ABAC policy:

```text
EC2-ABAC-Team-Project-Policy
```

---

## Step 4

Attach the custom policy to:

```text
EC2ABACUsers
```

---

## Step 5

Add users to the group:

```text
PegasusUser
UnicornUser
```

---

## Step 6

Apply Principal Tags to users.

---

## Step 7

Launch and tag EC2 instances.

---

# Validation Testing

## Test Case 1

User:

```text
PegasusUser
```

Action:

```text
Stop Pegasus EC2
```

Result:

✅ Allowed

---

## Test Case 2

User:

```text
PegasusUser
```

Action:

```text
Stop Unicorn EC2
```

Result:

❌ Access Denied

---

## Test Case 3

User:

```text
UnicornUser
```

Action:

```text
Reboot Unicorn EC2
```

Result:

✅ Allowed

---

## Test Case 4

User:

```text
UnicornUser
```

Action:

```text
Stop Pegasus EC2
```

Result:

❌ Access Denied

---

# Screenshots

## IAM User Group

<img width="922" height="176" alt="image" src="https://github.com/user-attachments/assets/7584572e-2f2a-4ade-88ec-31d5e567ec78" />


---

## IAM User Tags

<img width="919" height="329" alt="image" src="https://github.com/user-attachments/assets/41f1029b-2c69-465a-91bc-763b5e0aae60" />


---

## EC2 Resource Tags

<img width="728" height="371" alt="image" src="https://github.com/user-attachments/assets/f90eef24-d98d-47c3-9cf0-8c56c7223b35" />


---

## Successful Access

<img width="799" height="387" alt="image" src="https://github.com/user-attachments/assets/2babab3b-826a-48af-9057-50316ef9bae5" />

<img width="806" height="352" alt="image" src="https://github.com/user-attachments/assets/2eeaa2d2-c6c9-42e7-82e9-6dfd6b32252d" />



---

## Access Denied Error

<img width="929" height="416" alt="image" src="https://github.com/user-attachments/assets/fc917228-3054-41a8-8016-b4e10d482e85" />

<img width="924" height="300" alt="image" src="https://github.com/user-attachments/assets/cd6e3895-1c97-4dab-b784-f7adbbf97b99" />



---

# Key Learning

The biggest takeaway from this lab:

### RBAC

Answers:

> What actions can I perform?

### ABAC

Answers:

> Which resources can I perform those actions on?

ABAC enables a single policy to scale across multiple teams while maintaining strong security boundaries.

---

# Benefits of ABAC

* Reduces policy sprawl
* Simplifies access management
* Supports least-privilege access
* Scales across teams and projects
* Improves security in shared AWS accounts
