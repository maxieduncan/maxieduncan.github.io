---
layout: post
category: aws
title: "AWS Credentials, CLI and MFA"
tags: [aws, cli, mfa, security]
---

{% include JB/setup %}

We recently started [enforcing MFA on our AWS accounts](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_users-self-manage-mfa-and-creds.html) which for the most part has been seamless, the exception being the requirement for [new credentials to be generated](https://aws.amazon.com/premiumsupport/knowledge-center/authenticate-mfa-cli/) every day (or rather for each session) when using the CLI. In order to do this I've ended up creating a simple script based on an [example created by DevNambi](https://devnambi.com/2020/set-aws-session.html) and maintaining two AWS profiles for each account. The first AWS profile is for my standard user credentials that allow me to request more privileges using my MFA device; the second AWS profile is for the session credentials generated using my MFA token to gain more privileges.

To start my `.aws/credentials` looks like:

```
[production-mfa]
aws_secret_access_key = ***
aws_access_key_id = ***
```

After I run my credentials script it looks like:

```
[production-mfa]
aws_secret_access_key = ***
aws_access_key_id = ***

[production]
aws_secret_access_key = ***
aws_access_key_id = ***
aws_session_token = ***
```

The session based credentials get updated every time I run the script using the current MFA token and give me full access via the AWS cli.

The script:

```
#!/bin/bash

MFA_PROFILE=$1
PROFILE=$2
SERIAL_NUMBER=$3
TOKEN=$4

if [[ -z $MFA_PROFILE || -z $PROFILE || -z $SERIAL_NUMBER || -z $TOKEN ]]; then
  echo "Usage: aws-creds.sh <mfa profile> <profile> <serial number> <token>"
  exit -1
fi

read -p "You are about to request session creditionals for profile: [$PROFILE] using MFA profile: [$MFA_PROFILE] for device: [$SERIAL_NUMBER] with token: [$TOKEN]. Are you sure (y/n)? " -n 1 -r
echo    # move to a new line
if [[ $REPLY =~ ^[Yy]$ ]]
then

  CREDJSON=`aws sts get-session-token --profile $MFA_PROFILE --serial-number $SERIAL_NUMBER --token-code $TOKEN`

  ACCESSKEY="$(echo $CREDJSON | jq '.Credentials.AccessKeyId' | sed 's/"//g')"
  SECRETKEY="$(echo $CREDJSON | jq '.Credentials.SecretAccessKey' | sed 's/"//g')"
  SESSIONTOKEN="$(echo $CREDJSON | jq '.Credentials.SessionToken' | sed 's/"//g')"

  aws configure set aws_access_key_id $ACCESSKEY --profile $PROFILE
  aws configure set aws_secret_access_key $SECRETKEY --profile $PROFILE
  aws configure set aws_session_token $SESSIONTOKEN --profile $PROFILE

fi
```

The script requires 4 arguments:

1. the AWS profile that will be used to request a session token
2. the AWS profile that the session token will stored under
3. the serial number of the MFA device
4. the MFA token

I have a few aliases set up in my `.profile` that allows me to easily update the credentials for each account:

```
alias aws-creds-test="$SCRIPTS_PATH/aws-creds.sh ecogy-test-mfa ecogy-test arn:aws:iam::***:mfa/*** $1"
```

Now I just call `aws-creds-test <current token>` with the current MFA token and the `ecogy-test` profile is available to use with full user privileges.
