# How to assume roles using the AWS CLI

## AWS Command Line Interface (CLI)

The [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) (AWS CLI) is an open source tool that enables you to interact with AWS services using commands in your command-line shell. With minimal configuration, the AWS CLI enables you to start running commands that implement functionality equivalent to that provided by the browser-based AWS Management Console from the command prompt in your terminal program:

- **Linux shells** – Use common shell programs such as bash, zsh, and tcsh to run commands in Linux or macOS.
- **Windows command line** – On Windows, run commands at the Windows command prompt or in PowerShell.
- **Remotely** – Run commands on Amazon Elastic Compute Cloud (Amazon EC2) instances through a remote terminal program such as PuTTY or SSH, or with AWS Systems Manager.

### [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)

The AWS CLI stores sensitive credential information that you specify with `aws configure` in a local file named `credentials`, in a folder named `.aws` in your home directory. The less sensitive configuration options that you specify with `aws configure` are stored in a local file named `config`, also stored in the `.aws` folder in your home directory.

For example, the files generated by the AWS CLI for a default profile configured with aws configure looks similar to the following.

`~/.aws/credentials`

```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

## Assuming roles

### AWS Console

![image-1](./images/image-1.png)

You should always login to the Users account with the credentials provided and using your MFA token.  Once authenticated, you should select your name from the top-right of the console page.

```
**Note**

Some roles will be cached inside your browser - generally only the most recently used 5 will be shown.  You should ensure you keep a record of any assume-role details you will need.
```

You may select a role you have previously used directly from the list, or choose "Switch role" to enter details of one not listed

![image-2](./images/image-2.png)

Enter the details and choose a colour to associate with the role in the history display and when using it in the console.  Once you have assumed the role, you will be able to see what role you are using by clicking the descriptive name you assigned it in the top-right of the console pages:

![image-3](./images/image-3.png)

You can choose "Switch back" to return to the users account, or select any other role from the history if you need to switch back and forth between them.

### AWS CLI and SDKs

The simplest option here, is to add your user credentials - you can create these in the "Users" account once you have enabled (and logged back in using) your MFA token.  First you should create access keys - select "Security Credentials" from the menu by clicking on your username in the AWS console:

![image-4](./images/image-4.png)

First, on the "AWS IAM credentials" tab you should take a note of your MFA device ARN - this is listed under "Multi-factor authentication (MFA)" and will show you the unique identifier for your user:

![image-5](./images/image-5.png)

Next, select the "Create access key" button under "Access keys for CLI, SDK, & API access":

![image-6](./images/image-6.png)

```
**Warning**

The Secret Access Key  shown in the popup will never be available again, do not close the popup window until you have recorded it or you will be unable to use the credentials you have created
```

From a command-line, you can now add these details to your CLI and SDK configuration as a named profile by running the following command - we have named it `tna-temporary` in the example here:

```
aws configure --profile tna-temporary
AWS Access Key ID [None]: <PASTE ACCESS KEY ID FROM CONSOLE>
AWS Secret Access Key [None]: <PASTE SECRET ACCESS KEY FROM CONSOLE>
Default region name [None]: eu-west-2
Default output format [None]: json
```

If you already have a profile set, this same command can be used to update the credentials, region or output format of the profile.

Next, you can configure the assume-role configuration - this needs to be done by editing the file `~/.aws/config`  - the `tna-temporary`  profile from above is shown here, along with an assume-role profile for `tna-temporary-nonprod-admin`:

```
[profile tna-temporary]
region = eu-west-2
output = json
 
[profile tna-temporary-nonprod-admin]
region = eu-west-2
output = json
source_profile=tna-temporary
role_arn = arn:aws:iam::<xxxxxxxx>:role/admin_<xxxxxxxxxxx>
mfa_serial = arn:aws:iam::<xxxxxx>:mfa/<xxxxx>
```

This pattern can be repeated in that file for all Normandy accounts by altering the profile name and role_arn as appropriate.

To use the new profile, you can now set an environment variable to point at it:

```
export AWS_PROFILE=tna-temporary-nonprod-admin
aws sts get-caller-identity
```

Alternatively, you can specify the profile on your command-line when calling AWS:

```
aws --profile=tna-temporary-nonprod-admin sts get-caller-identity
```

### Using environment variables

An alternative to using named profiles is to configure environment variables, which can be achieved by using the `aws sts get-session-token` and `aws sts assume-role` commands.  This process assumes you have not configured your AWS CLI following the instructions above - you will need to know your access credentials and MFA Token Code as described previously.

#### To authenticate directly into an account with an MFA token:

```
export AWS_ACCESS_KEY_ID=<access key saved from AWS console>
export AWS_SECRET_ACCESS_KEY=<secret access key saved from AWS console>
export AWS_REGION=eu-west-2
aws sts get-session-token --duration-seconds 3600 --serial-number <MFA token serial number saved from AWS console> --token-code <value from your MFA token>
```

This will give you session credentials you can use to perform tasks that require you to have authenticated with the provided token.  You will need to set your environment variables with the values shown as in this example (credentials have been simplified):

```
➜  ~ export AWS_ACCESS_KEY_ID=<access_key_from_AWS_console>
➜  ~ export AWS_SECRET_ACCESS_KEY=<secret_key_from_AWS_console>
➜  ~ export AWS_REGION=eu-west-2
➜  ~ aws sts get-session-token --duration-seconds 3600 --serial-number <MFA_token_code_from_AWS_console> --token-code <value_from_authenticator_app>
{
    "Credentials": {
        "AccessKeyId": "NEWMFAKEYFROMCOMMAND",
        "SecretAccessKey": "NEWMFASECRETFROMCOMMAND",
        "SessionToken": "NEWMFASESSIONFROMCOMMAND=",
        "Expiration": "2022-06-22T10:47:59+00:00"
    }
}
➜  ~ export AWS_ACCESS_KEY_ID="NEWMFAKEYFROMCOMMAND"
➜  ~ export AWS_SECRET_ACCESS_KEY="NEWMFASECRETFROMCOMMAND"
➜  ~ export AWS_SESSION_TOKEN="NEWMFASESSIONFROMCOMMAND="
➜  ~ aws sts get-caller-identity
{
    "UserId": "YOURUSERIDFROMAWS",
    "Account": "012345678901",
    "Arn": "arn:aws:iam::012345678901:user/youruser"
}
➜  ~
```

#### To assume a role directly with the AWS CLI

```
➜  ~ export AWS_ACCESS_KEY_ID=<access_key_from_AWS_console>
➜  ~ export AWS_SECRET_ACCESS_KEY=<secret_key_from_AWS_console>
 
➜  ~ export AWS_REGION=eu-west-2
➜  ~ aws sts assume-role --role-arn <target_role_arn> --role-session-name <example-session-name> --duration-seconds 3600 --serial-number <MFA_token_code_from_AWS_console> --token-code <value_from_authenticator_app>
{
    "Credentials": {
        "AccessKeyId": "NEWROLEKEY",
        "SecretAccessKey": "NEWROLESECRET",
        "SessionToken": "NEWROLESESSION=",
        "Expiration": "2022-06-22T10:54:42+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "YOURUSERIDFROMAWS:example-session-name",
        "Arn": "arn:aws:sts::<acount id>:assumed-role/role_name/example-session-name"
    }
}
➜  ~ export AWS_ACCESS_KEY_ID="NEWROLEKEY"
➜  ~ export AWS_SECRET_ACCESS_KEY="NEWROLESECRET"
➜  ~ export AWS_SESSION_TOKEN="NEWROLESESSION="
➜  ~ aws sts get-caller-identity
{
    "UserId": "YOURUSERIDFROMAWS:example-session-name",
    "Account": "<acount id>",
    "Arn": "arn:aws:sts::<acount id>:assumed-role/role_name/example-session-name"
}
➜  ~
```