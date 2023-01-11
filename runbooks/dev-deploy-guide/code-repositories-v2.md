# Code Repositories for v2

The Platform and the environments are built from:
- [tre-terraform-platform](https://github.com/nationalarchives/tre-terraform-platform)
- [tre-terraform-environments](https://github.com/nationalarchives/tre-terraform-environments)

Common build scripts / workflows / actions are in:
- [tre-github-actions](https://github.com/nationalarchives/tre-github-actions)

There environments contain these modules:
- [tre-tf-module-common](https://github.com/nationalarchives/tre-tf-module-common)
- [tre-tf-module-tre-forward](https://github.com/nationalarchives/tre-tf-module-tre-forward)
- [tre-tf-module-validate-bag](https://github.com/nationalarchives/tre-tf-module-validate-bag)
- [tre-tf-module-dpsg](https://github.com/nationalarchives/tre-tf-module-dpsg)

The step functions in these modules use containers for their lambda functions, these functions are in these repos:
- [tre-fn-vb-bag-validation](https://github.com/nationalarchives/tre-fn-vb-bag-validation)
- [tre-fn-vb-bag-files-validation](https://github.com/nationalarchives/tre-fn-vb-bag-files-validation)
- [tre-fn-dpsg-bag-to-dri-sip](https://github.com/nationalarchives/tre-fn-dpsg-bag-to-dri-sip)
- [tre-fn-forward](https://github.com/nationalarchives/tre-fn-forward)
- [tre-fn-sqs-sf-trigger](https://github.com/nationalarchives/tre-fn-sqs-sf-trigger)
- [tre-fn-dlq-slack-alerts](https://github.com/nationalarchives/tre-fn-dlq-slack-alerts)
- [tre-fn-slack-alerts](https://github.com/nationalarchives/tre-fn-slack-alerts)

The functions use some common libraries pulled from CodeArtifact.  The libraries/actions to build/publish them are here:
- [tre-event-library](https://github.com/nationalarchives/tre-event-library)
- [tre-s3-library](https://github.com/nationalarchives/tre-s3-library)

For completeness note that:
- v1 modules are not included above (they are listed elsewhere)
- there is sample data in [da-transform-sample-data](https://github.com/nationalarchives/da-transform-sample-data)
