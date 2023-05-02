# TRE-TERRAFORM-BACKEND

[tre-terraform-backend](https://github.com/nationalarchives/tre-terraform-platform/tree/main/tf-backend) is used to manage tre-roles which are used by GitHub Actions and terfaform to deploy TRE.

## New Role Creation

To create a new backend role, terraform needs runnig by an user who has IAM Admin Access from their local machine

1. Set up AWS profiles by running `aws configure`

    > [How to assume roles using the AWS CLI](../how-to-assume-roles-using-AWS-CLI/README.md)

2. Add terraform code for role creation in [tre-terraform-platform/modules](https://github.com/nationalarchives/tre-terraform-platform/tree/main/modules)
3. Export new role and policy ARNs with [terraform output](https://developer.hashicorp.com/terraform/language/values/outputs)
4. Update [tre-terraform-platform/tf-backend/main.tf](https://github.com/nationalarchives/tre-terraform-platform/blob/main/tf-backend/main.tf)
5. Add the output values in [tre-terraform-platform/tf-backend/locals.tf](https://github.com/nationalarchives/tre-terraform-platform/blob/main/tf-backend/locals.tf)
5. Run `terraform apply`

Once the role and the polices are created, any further modifications can be done by running the [terraform platform deployment](https://github.com/nationalarchives/tre-terraform-platform/actions/workflows/tre-platform-deployment.yml) workflow with tf_backend as input.
