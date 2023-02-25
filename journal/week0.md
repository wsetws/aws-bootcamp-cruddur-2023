# Week 0 — Billing and Architecture

# Required Homework

## Diagrams:

I made the following diagrams in Lucid charts, they can be viewed here: [Lucid Charts](https://lucid.app/lucidchart/036d3fc3-5a8b-4e1b-a44e-3241bfb914cc/edit?view_items=3swA5.gq4n1c%2CytwAES9DRCm7%2COuwAfipBd8vn%2CuAwAtd5aveKt%2CeAwAh7HtzP86%2CMxwACab9Dx3l%2CJAwA6Ho6R44m%2CfywAOG.CTRA2%2C3swAq4gdxaZo%2C3FwAq.PRZnWN%2CwAwAhQ.NfVJg%2CcwwAOwLJpZqB%2CSAwAJe8uy7Ed%2CdEwAQnBnyShZ%2C3swA8kiK-Mea%2C3swA6J9YYWJi%2CmGwAu1JgYirJ%2C3swALQ6Lr_6z%2C3swAhaotwEk4%2C3swAfwLzDTHB%2C3swA3uHuPLUy%2C3swAsTz9TEnV%2C3swArGqclgly%2CUDwA6e6Nww1~&invitationId=inv_2a3b46c4-7c1b-47bd-812f-123b1409f5ef)

### Recreate Conceptual Diagram in Lucid Charts or on a Napkin:

![imgur](https://i.imgur.com/ZzCA5sf.png)

### Recreate Logical Architectural Diagram in Lucid Charts:

![imgur](https://i.imgur.com/RnS8kQn.png)

## Create an Admin User

I created an admin user to be able to not use the root account anymore

![imgur](https://i.imgur.com/ZDnM3Xk.png)

I had to add the user to an admin group I had created before:

![imgur](https://i.imgur.com/4bbdENt.png)

![imgur](https://i.imgur.com/jRn9uHC.png)

This is the result:

![imgur](https://i.imgur.com/dayPoLX.png)

## Generate AWS Credentials

After that, I created an access key to be able to use the cli console:

![imgur](https://i.imgur.com/855AzR9.png)

![imgur](https://i.imgur.com/ayVxS2s.png)

![imgur](https://i.imgur.com/3enJNzD.png)

![imgur](https://i.imgur.com/P14q3LN.png)

## Use CloudShell

To use CloudShell, I logged in and clicked the icon in the top center

![imgur](https://i.imgur.com/2B3YTJd.png)

After the welcome message, a prompt appears

![imgur](https://i.imgur.com/9xetjuC.png)

![imgur](https://i.imgur.com/lDUV6Wl.png)

To test this function I asked it to provide information of the logged user, and it worked

![imgur](https://i.imgur.com/a6Sewbu.png)

I could do any task from Cloudshell but I think its better to learn how to use the AWS CLI on gitpod since we will be working on that environment.

## Install AWS CLI (on gitpod)

![imgur](https://i.imgur.com/xOtW8zw.png)

Change gitpod to dark mode

![imgur](https://i.imgur.com/lCBOQso.png)

Commands used to install aws cli :

```bash
mkdir /workspace/downloads
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/workspace/downloads/awscliv2.zip"
unzip /workspace/downloads/awscliv2.zip
cd /workspace/downloads/aws
sudo ./install
aws --version
```

![imgur](https://i.imgur.com/Xwc1IfK.png)

Configure credential for AWS cli using environmental variables

```Bash
export AWS_ACCESS_KEY_ID="*****************"
export AWS_SECRET_ACCESS_KEY="*******************"
export AWS_DEFAULT_REGION="us-east-1"
env | grep "AWS"
```

![imgur](https://i.imgur.com/hlI3Bmd.png)

AWS-CLI working:

![imgur](https://i.imgur.com/LZWPKrF.png)

Now, all I did will get deleted if I log out from gitpod so, we need a better way to set this up, so we will modify the gitpod configuration file adding the following to gitpod yml:

```yml
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

This task will always install the aws cli when we access gitpod.

One caveat I encounter is that I needed to provide my gitpod account access to commiting changes to github

![imgur](https://i.imgur.com/zA9FF7y.png)

After that, I commit the change to the main branch

![imgur](https://i.imgur.com/6muSI8W.png)

Also, gitpod has a way to save environmental variables between sessions. For that I run the following commands:

```bash
gp env AWS_ACCESS_KEY_ID="*****************"
gp env AWS_SECRET_ACCESS_KEY="*******************"
gp env AWS_DEFAULT_REGION="us-east-1"
```

After that, I tested a new environment, and it installed AWS-cli automatically

![imgur](https://i.imgur.com/7pkAhAM.png)

Lastly I tests that my credentials were actually read by gitpod and it worked!

![imgur](https://i.imgur.com/ZexQrh5.png)

## Setting a billing alarm

Now that I have access to AWS CLI on gitpod I will try to create a billing alarm using it.

After reading the documentation I figured I need to create an SNS topic first and then create the alarm. Also we have to use the cloudwatch API to create the alarm, is not a different service

Documentation I reviewed on the topic:

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor\_estimated\_charges\_with\_cloudwatch.html

https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/put-metric-alarm.html

Steps to create the SNS topic:

![imgur](https://i.imgur.com/jd3hMy2.png)

![imgur](https://i.imgur.com/CwHhpHc.png)

![imgur](https://i.imgur.com/VWMbAvx.png)

![imgur](https://i.imgur.com/1oYjhu9.png)

SNS subscription 

![imgur](https://i.imgur.com/VWMbAvx.png)

Now that I have created the Billing_Alarms topic I will open my gitpod env and will use the cli to create the Billing Alarm

```Bash
aws cloudwatch put-metric-alarm \
--alarm-name "20 dollars alarm" \
--metric-name "TotalEstimatedCharge" \
--statistic "Maximum" \
--period 21600 \
--threshold 20 \
--comparison-operator "GreaterThanThreshold" \
--datapoints-to-alarm 1 \
--evaluation-periods 1 \
--treat-missing-data "missing" \
--alarm-actions arn:aws:sns:********************** \
--namespace Billing \
--region us-east-1
```

After tinkering with the required parameters I got it to work, no success or error message appeard

![imgur](https://i.imgur.com/2euz3Iv.png)

I could not figure a way to make it appear with the list-metrics command in the cli but it showed up on the Web Console

![imgur](https://i.imgur.com/WtCKIUN.png)

## Setting a budget

Since I have an old account and I expect to spend on the bootcamp I set up a top monthly spend of 50 dollars.

![imgur](https://i.imgur.com/A0JViS2.png)

![imgur](https://i.imgur.com/9FNcBUf.png)

* * *

## TODO:

- Create a budget that tracks all bootcamp resources
- Security considerations for ROOT account
- Security considerations of new user group permissions.
- Change language to English (currently AWS console shows a mix of spanish and english)
- Monitor the billing alarm to see if it works

## Lessons Learned:

- Work directly on the repository even for the journal, it will speed things up and help avoid rework.