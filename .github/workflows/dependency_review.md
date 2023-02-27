# dependency_review

A reusasble workflow that generates a depenency graph for Java projects (e.g., Maven and Gradle) so that transitive dependency information is uploaded to the [submission API](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/using-the-dependency-submission-api). Additionally creates a Dependency Review status check on Pull Requests so that branch protection rules can be leveraged.

This is more-or-less a wrapper for these two Java-specific actions. However, this reusable workflow also supports `npm` as an `ecosystem` input.
- [maven-dependency-submission-action](https://github.com/advanced-security/maven-dependency-submission-action)
- [gradle-dependency-submission](https://github.com/mikepenz/gradle-dependency-submission)


# Usage
Simply add a `dependency_review.yml` file similar to the below to the `.github/workflows` directory of a repository.

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
- To ensure that CLI binaries are available (e.g., `mvn` and `gradle`) the [setup-java action](https://github.com/actions/setup-java) is used to prepare the Java environment. This requires inputs related to Java distribution and version, as noted in the Inputs section below. 
- As this is a reusable workflow, there are no configuration options (e.g., Maven settings) currently available. This workflow is meant to target _typical_ Java projects, having a `pom.xml` file in the root of the repository.
- It is still difficult to determine which dependencies that are listed in the Dependency Graph (on the Insights tab of a repository) are direct versus those that are transitive (indirect). This can lead to confusion about how to remediate (i.e. pin) specific transitive dependencies because the dependency chain is not present in the GitHub UI.

## Inputs
| Input     | Required | Description                                                                                |
|-----------|----------|--------------------------------------------------------------------------------------------|
| ecosystem | true     | Package dependency ecosystem that the application is written in (e.g., maven, gradle, npm) |
| java-version | false     | The Java version to set up for Maven and Gradle projects. Takes a whole or semver Java version. |
| java-distribution | false     | The [distribution](https://github.com/actions/setup-java#supported-distributions) of Java to setup for Maven and Gradle projects |


# Examples
The below examples demonstrate the capabilities of this reusable workflow.

## Observing Vulnerable Transitive Dependencies 
In this example, it is shown how a vulnerable transitive dependency is identified by executing the `workflow_dispatch` trigger.

### Default Dependabot Alerts

![image](https://user-images.githubusercontent.com/107562400/221617553-bdc30855-80de-4166-b2da-a0e95195a820.png)


### Additional Dependabot Alerts

### Dependency Graph 

## Pull Request Workflow
In this example, it is shown how developers can prevent directly vulnerable dependencies from being merged as part of a Pull Request.

### Branch Protection Rules
> **Note**
Dependency Review does not yet take into account the results of the Submission API. Said differently, the status check that is published on a Pull Request as part of Dependency Review does not examine transitive dependencies.

Creating a Branch Protection Rule that targets `ma**` includes both `main` and `master` in the scope of protection. Requiring the **Dependency scan results** check to pass on Pull Requests effectively ensures that no vulnerabilities exists in any **direct** dependencies that are added or changed in a given manifest file that has been modified as part of a Pull request.

![image](https://user-images.githubusercontent.com/107562400/221601613-e88aba40-d271-4aac-8031-de8adc35492e.png)

### Status Checks
The way that checks are presented resemble how checks for Code Scanning (CodeQL) are natively reported.


![image](https://user-images.githubusercontent.com/107562400/221601572-10730531-5454-42c2-92e9-60a751266641.png)

### Additional Details
When selecting the details of the check, further details of the Dependency Review are given. Namely, the vulnerability that was identified, as well as the dependencies that were updated can help a Developer understand why a given Pull Request check is failing. 


![image](https://user-images.githubusercontent.com/107562400/221601709-dae321b7-0901-4fb0-9c9b-714aee55afa7.png)




