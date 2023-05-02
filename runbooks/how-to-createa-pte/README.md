# Personal Test Environment

A developer can create a temporary TRE Environement by running the [Personal Test Environment Deployment](https://github.com/nationalarchives/tre-terraform-environments/actions/workflows/personal-test-environment.yml)

## Workflow
Enter initials once asked which will be used as AWS Resource name_prefix. When run by an user for the first time it will
1. createa new parameter with tfvars values in AWS Parameter Store

    > pte-{github-username}-tfvars
2. createa new terraform workspace

    > pte-{github-username}
3. use [tf-plan-approve-apply.yml](https://github.com/nationalarchives/tre-github-actions/blob/main/.github/workflows/tf-plan-approve-apply.yml) workflow to deploy PTE

## Destroy PTE

To destroy PTE, run [Destroy Personal Test Environment Deployment](https://github.com/nationalarchives/tre-terraform-environments/actions/workflows/destroy-personal-test-environment.yml) workflow. This will delete all resources in AWS and terraform workspace
