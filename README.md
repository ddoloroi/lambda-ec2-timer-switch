# AWS EC2 timer switch
Automatically stop and start Amazon EC2 instances using [AWS Lambda](https://aws.amazon.com/lambda/).

The official way to schedule start and stop times for EC2 instances is using AWS Data Pipeline, as described [here](https://aws.amazon.com/premiumsupport/knowledge-center/stop-start-ec2-instances/).
This is an alternative way using AWS Lambda.

## Prerequisites

Node.js, Npm and Grunt.

## HOWTO

### Create an IAM User (Optional)

  More info about creating IAM users:

  http://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html
  
  http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html
  
  If you are creating a new user, you need to have at least the following permissions:
  
  ```javascript
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "ec2:Describe*",
                  "ec2:Start*",
                  "ec2:RunInstances",
                  "ec2:Stop*",
                  "iam:CreateRole",
                  "iam:ListRoles",
                  "iam:PassRole",
                  "iam:PutRolePolicy",
                  "lambda:AddPermission",
                  "lambda:CreateFunction",
                  "lambda:GetFunction",
                  "lambda:GetPolicy",
                  "lambda:ListFunctions",
                  "lambda:UpdateFunctionCode"
              ],
              "Resource": "*"
          }
      ]
  }
  ```

### Build the zip archives

  1 - Clone this project.

  2 - In the project root directory run `npm install`.

  3 - Run `grunt build --id=INSTANCE_ID`. Where INSTANCE_ID is the id of your EC2 machine, or a comma separated list of ids.
  
  This will create a `starter.zip` file and `stopper.zip` in the `target` folder.

  To avoid typing your instance id in the grunt build param you can put it on `config.json` file. 
  
### Create an execution role and policy

  Open the IAM console: https://console.aws.amazon.com/iam/.
  
  In the Details area, choose Policies.
  
  Choose Create Policy.
  
  Select Create Your Own Policy.
  
  Choose a name (e.g. `LambdaEC2SwitcherPolicy`) and a description.
  
  Paste the JSON below in the Policy Document text area and then click on Create Policy.
  
  ```javascript
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:Describe*",
          "ec2:Start*",
          "ec2:RunInstances",
          "ec2:Stop*",
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ],
        "Resource": ["*"]
      }
    ]
  }
  ```
  
  In the Details area, choose Roles.
  
  Choose Create New Role.
  
  Use `LambdaEC2SwitcherRole` or some similar name to Role Name.
  
  Select AWS Lambda.
  
  Check the `LambdaEC2SwitcherPolicy` box in the policies list (use the filter to find it).
  
  Choose Next Step and then Create Role.

### Create a new Lambda function

  Open the AWS Lambda console at https://console.aws.amazon.com/lambda/.
  
  If a welcome page appears, choose Get Started Now. Else, choose Create a Lambda function.
  
  Choose Skip at the bottom of the Select blueprint pane.
  
  Choose a name (e.g. ec2-starter) and a description.
  
  For Runtime, choose Node.js.
  
  In Code entry type choose Upload a .ZIP file.
  
  Upload your starter.zip file.
  
  For Role, choose the Lambda execution role you created earlier (e.g. `LambdaEC2SwitcherRole`).
  
  Choose Next and then Create function.
  
  Repeat all the steps above to create a new Lambda function for your stopper.zip file.
  
### Append an event source

  Open your Lambda function (`ec2-starter`) at https://console.aws.amazon.com/lambda/.
  
  Choose the Event sources tab.
  
  Click to Add event source.
  
  Choose CloudWatch Events - Schedule for Event source type.
  
  Choose Rule name and description.
  
  Use a cron expression for the Schedule expression, e.g., `cron(0 10 ? * MON-FRI *)` to start your machine every week day at 10am (UTC).
  
  Pick the enable now radio button and Submit.
  
  Repeat the above steps for the `ec2-stopper` function, and choose a shutdown time.
