# pipelines-templates


Common Azure Pipelines [YAML templates] for Scala / JVM based projects.

## Overview
Currently, only sbt projects are supported, with publishing templates for both
Maven (via sbt-aether-deploy) and Bintray (via sbt-bintray) repositories selectable 
based on parameter input.

## Usage
See Azure Pipelines [template documentation] for comprehensive template usage documentation.

#### Required Parameters for all projects
| Parameter Name | Description | Default Value *(if present)* |
| -------------- | ----------- | ---------------------------- |
| `publishStyle` | Defines which publishing template to use. Accepted values are: `maven`, `bintray`, or `none` to disable publishing. | `none` |
| `jdk8Container` | Defines docker container used for compiling and running Java 8 integration tests. | `duchessa/el8-jdk:8` |
| `jdk11Container` | Defines docker container used for compiling and running Java 11 integration tests. | `duchessa/el8-jdk:11` |
| `javaOptions` | Java options passed to sbt during execution of all tasks. | `-Xmx4G -XX:MaxMetaspaceSize=1G` |

#### Required Parameters for projects published to Maven Repositories (in addition to parameters required for all projects above.)
| Parameter Name | Description | Default Value *(if present)* |
| -------------- | ----------- | ---------------------------- |
| `publishContainer` | Defines docker container used for compiling published artefacts. | `duchessa/el8-jdk:8` |
| `sbtPgpVersion` | Version of `sbt-pgp` plugin used when publishing. | `2.1.1` |
| `sbtAetherVersion` | Version of `sbt-aether-deploy-signed` plugin used when publishing. | `0.26.0` |
| `credentialsGroup` | Name of variable group containing credentials used for publishing. Variable group must contain `maven-user`, `maven-key`, `maven-host`, and `maven-realm` secure variables providing Maven repository credentials. | |
| `mavenReleaseRepository` | Maven repository to publish release artifacts to (as defined by sbt `isSnapshot` setting) | |
| `mavenSnapshotRepository` | Maven repository to publish snapshot artifacts to (as defined by sbt `isSnapshot` setting) | |

#### Required Parameters for projects published to Bintray (in addition to parameters required for all projects above.)
| Parameter Name | Description | Default Value *(if present)* |
| -------------- | ----------- | ---------------------------- |
| `publishContainer` | Defines docker container used for compiling published artefacts. | `duchessa/el8-jdk:8` |
| `sbtBintrayVersion` | Version of sbt-bintray plugin used when publishing. | `0.6.1` |
| `credentialsGroup` | Name of [pipelines variable group] containing credentials used for publishing. Variable group must contain `bintray-user` and `bintray-key` secure variables providing bintray credentials. | |
| `bintrayOrganisation` | Bintray organisation to publish to. | |
| `bintrayRepository` | Bintray repository to publish to. | |


_NB: For `credentialsGroup` only a secure variable group **[backed by secrets]** in Azure Key Vault should be used._

Example usage below:
```yaml
trigger:
  branches:
    include:
      - master
      - refs/tags/*
  paths:
    exclude:
      - README.md
      - NOTICE
      - LICENSE

pr:
  branches:
    include:
      - master
  paths:
    exclude:
      - azure-pipelines.yml
      - README.md
      - NOTICE
      - LICENSE

resources:
  repositories:
    - repository: templates
      type: github
      name: duchessa/azure-pipelines-templates
      endpoint: github.com_endpoint

extends:
  template: sbt-pipeline.yml@templates
  parameters:
    publishStyle: 'maven'
    credentialsGroup: 'publish-secret-vars'
    mavenReleaseRepository: 'https://maven-url/releases'
    mavenSnapshotRepository: 'https://maven-url/snapshots'
```

[YAML templates]: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates
[template documentation]: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates
[pipelines variable group]: https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups
[backed by secrets]: https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml#link-secrets-from-an-azure-key-vault
