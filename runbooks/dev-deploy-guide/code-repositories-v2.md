# Code Repositories for v2

The platform and the environments are built from:
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

# notes on other/related/v1 repos 

For completeness note that:
- v1 modules are not included above (they are explained elsewhere, but a list is provided below)
- there is sample data relevant to v1 and v2 in [da-transform-sample-data](https://github.com/nationalarchives/da-transform-sample-data)
- there is this repo (just to get it all on this list) = [da-transform-dev-documentation](https://github.com/nationalarchives/da-transform-dev-documentation)
- some v2 prototype work is in [tre-blueprint-test-repository](https://github.com/nationalarchives/tre-blueprint-test-repository)

For even more completeness there are 5 other repos that relate to v1
- [da-transform-terraform-environments](https://github.com/nationalarchives/da-transform-terraform-environments)
- [da-transform-terraform-modules](https://github.com/nationalarchives/da-transform-terraform-modules)
- [da-transform-judgments-pipeline](https://github.com/nationalarchives/da-transform-judgments-pipeline)
- [da-transform-schemas](https://github.com/nationalarchives/da-transform-schemas)

And v1 itself has an empty repo at: [transformation-alpha](https://github.com/nationalarchives/transformation-alpha) to be deleted

And, related but not "owned by tre" is the judgement parser itself which is at: 
- [tna-judgments-parser](https://github.com/nationalarchives/tna-judgments-parser)
