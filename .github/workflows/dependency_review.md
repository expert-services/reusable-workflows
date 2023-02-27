# dependency_review

A reusasble workflow that generates a depenency graph for Java projects (e.g., Maven and Gradle) so that transitive dependency information is uploaded to the [submission API](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/using-the-dependency-submission-api). Additionally creates a Dependency Review status on Pull Requests so that branch protection rules can be leverages.

This is more-or-less a wrapper for these two Java-specific actions. However, this reusable workflow also supports JavaScript as an `ecosystem` input.
- [maven-dependency-submission-action](https://github.com/advanced-security/maven-dependency-submission-action)
- [gradle-dependency-submission](https://github.com/mikepenz/gradle-dependency-submission)


# Usage

```yaml
name: Dependency Review
on:
  pull_request:
  workflow_dispatch:
jobs:
  dependency_review:
    uses: oodles-noodles/reusable-workflows/.github/workflows/dependency_review.yml@main
    with:
      ecosystem: maven
      java-version: 11
      java-distribution: microsoft
```

## Prerequisites


## Inputs
| Input     | Required | Description                                                                                |
|-----------|----------|--------------------------------------------------------------------------------------------|
| ecosystem | true     | Package dependency ecosystem that the application is written in (e.g., maven, gradle, npm) |
| java-version | false     | The Java version to set up for Maven and Gradle projects. Takes a whole or semver Java version. |
| java-distribution | false     | The [distribution](https://github.com/actions/setup-java#supported-distributions) of Java to setup for Maven and Gradle projects |


# Examples

## Workflow Dispatch

## Observing Vulnerable Transitive Dependencies 

## Pull Request

### Branch Protection Rules
Creating a Branch Protection Rule that targets `ma**` includes both `main` and `master` in the scope of protection. Requiring the **Dependency scan results** check to pass on Pull Requests effectively ensures that no vulnerabilities exists in any **direct** dependencies that are added or changed in a given manifest file that has been modified as part of a Pull request.

> **Note**
Dependency Review does not yet take into account the results of the Submission API. Said differently, the status check that is published on a Pull Request as part of Dependency Review does not examine transitive dependencies.

<img width="709" alt="image" src="https://user-images.githubusercontent.com/107562400/221440010-d9880034-3557-4e2a-893a-e86459b4611e.png">


The way that checks are presented resemble how checks for Code Scanning (CodeQL) are natively reported.


<img width="816" alt="image" src="https://user-images.githubusercontent.com/107562400/221440147-93f35c80-3e41-429f-b5ff-adfca50ba2f1.png">


When selecting the details of the check, further details of the Dependency Review are given. Namely, the vulnerability that was identified, as well as the dependencies that were updated can help a Developer understand why a given Pull Request check is failing. 


<img width="1374" alt="image" src="https://user-images.githubusercontent.com/107562400/221440200-b4c227c9-4125-45e4-829a-187f34dcd668.png">




