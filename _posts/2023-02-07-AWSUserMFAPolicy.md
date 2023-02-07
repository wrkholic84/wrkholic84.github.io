---
title: "AWS IAM 사용자의 MFA 설정을 위해 필요한 정책"
# author: wrkholic84
date: 2023-02-07 00:00:00 +0900
categories: [Cloud, AWS]
tags: [AWS, IAM, MFA, policy]
math: true
mermaid: true
---
## AWS IAM 사용자의 MFA 설정을 위한 필수 정책
아래 내용의 정책을 사용자에게 할당해줘야 사용자는 자신의 계정에 필요한 MFA를 설정할 수 있다.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowManageOwnVirtualMFADevice",
            "Effect": "Allow",
            "Action": [
                "iam:DeleteVirtualMFADevice",
                "iam:CreateVirtualMFADevice"
            ],
            "Resource": "arn:aws:iam::*:mfa/${aws:username}"
        },
        {
            "Sid": "AllowViewAccountInfo",
            "Effect": "Allow",
            "Action": "iam:ListVirtualMFADevices",
            "Resource": "*"
        },
        {
            "Sid": "AllowManageOwnUserMFA",
            "Effect": "Allow",
            "Action": [
                "iam:EnableMFADevice",
                "iam:ListMFADevices"
            ]
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        }
    ]
}
```