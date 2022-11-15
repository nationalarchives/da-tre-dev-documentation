# GitHub actions

Date: 10-11-2022

## Status

In progress

## Context

The CI/CD pipeline to deploy changes to TRE v1 was implemented using AWS CodePipeline and AWS CodeBuild. In order to align with the TNA standards and to be able to support the TRE v2 project internally, it is necessary to implement the CI/CD pipeline using the [GitHub Actions platform](https://docs.github.com/en/actions).

TDR team has already got a mature implementation of CI/CD pipeline using the GitHub Actions platform, with some examples and reusable workflows available [here](https://github.com/nationalarchives/tdr-github-actions). 

TRE team will run a SPIKE to evaluate the GitHub Actions platform using the GitHub repo [tre-blueprint-test-repository](https://github.com/nationalarchives/tre-blueprint-test-repository) and record the decisions in this document.


## Decision

> What is the change that we're proposing and/or doing?

## Consequences

> What becomes easier or more difficult to do because of this change?