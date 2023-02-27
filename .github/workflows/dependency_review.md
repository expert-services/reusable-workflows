# dependency_review

A reusasble workflow that generates a depenency graph for Java projects (e.g., Maven and Gradle) so that transitive dependency information is uploaded to the [submission API](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/using-the-dependency-submission-api). Additionally creates a Dependency Review status on Pull Requests so that branch protection rules can be leverages.

This is more-or-less a wrapper for these two Java-specific actions. However, this reusable workflow also supports JavaScript as a `language` input.
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
| Input     | Required | Default | Description                                                                                |
|-----------|----------|---------|--------------------------------------------------------------------------------------------|
| ecosystem | true     |         | Package dependency ecosystem that the application is written in (e.g., maven, gradle, npm) |
| java-version | false     |         | The Java version to set up for Maven and Gradle projects. Takes a whole or semver Java version. |
| java-distribution | false     |         | The [distribution(https://github.com/actions/setup-java#supported-distributions) of Java to setup for Maven and Gradle projects |


# Examples


## Pull Request

### Branch Protection Rules

<img width="709" alt="image" src="https://user-images.githubusercontent.com/107562400/221440010-d9880034-3557-4e2a-893a-e86459b4611e.png">


<img width="816" alt="image" src="https://user-images.githubusercontent.com/107562400/221440147-93f35c80-3e41-429f-b5ff-adfca50ba2f1.png">


<img width="1374" alt="image" src="https://user-images.githubusercontent.com/107562400/221440200-b4c227c9-4125-45e4-829a-187f34dcd668.png">




