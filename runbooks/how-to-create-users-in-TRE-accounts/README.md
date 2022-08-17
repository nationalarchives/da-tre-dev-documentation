# How to create users for TRE AWS Accounts

### Creating users in AWS Console

**Note: Users should only ever be created by running the _terraform-common_ pipeline**

- Add new user details in **codepipeline.tfvars** in  **TRE Management** account. (User should be added to the correct group(s). Group permission details can be found in [here](https://github.com/nationalarchives/da-transform-terraform-environments/tree/common/common/templates) )

![Screenshot](images/adding-users-in-tfvars.png)

- Run **terraform-common** pipeline in **TRE Management** account. Check the plan to make sure no unexpected changes are being made.

### Creating login profiles using AWS CLI 

**Note: Only users with** `iam:CreateLoginProfile` **permission can create a login profile.**

-  Run  
```bash

aws iam create-login-profile \
--user-name <value> \
--password <value> \
--password-reset-required
```

- This will create a loging profile in **TRE User** account for the user with a password which they have to reset on first sign-in. 
- Users then can assume role(s) that is associated with their respective group(s) in other TRE accounts. (Instructions on how to assume roles can be found in [**here**](https://github.com/nationalarchives/da-transform-dev-documentation/tree/master/runbooks/how-to-assume-roles-using-AWS-CLI))