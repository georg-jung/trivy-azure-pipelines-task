# trivy-contrib

An Azure DevOps Pipelines Task for [Trivy](https://github.com/aquasecurity/trivy), with an integrated UI.

This fork of Aqua Security's [official extension](https://marketplace.visualstudio.com/items?itemName=AquaSecurityOfficial.trivy-official) includes some improvements:

* [Mount docker.sock](https://github.com/aquasecurity/trivy-azure-pipelines-task/pull/57) to scan docker images from a containerized trivy instance.
* [Update obsolete usage of --security-checks to --scanners](https://github.com/aquasecurity/trivy-azure-pipelines-task/pull/47).
* [Mount a consistent cache dir](https://aquasecurity.github.io/trivy/v0.48/getting-started/installation/#use-container-image) so that multiple runs using docker only download the vulnerability db once.
* Specify which scanners should be used. You might want to skip e.g. secret scanning in certain scenarios to reduce the scanning time.
* Use a recent version of trivy if not using the trivy docker image.
  * Due to the other changes above it should be possible to just use the docker-based execution in most cases, which always automatically uses the latest trivy version and does not require updates to this extension.
* It can be installed in parallel to the official trivy extension.
* This is a drop-in replacement, just change `- task: trivy@1` to `- task: trivy-contrib@1` after installing this extension.

You're welcome to star this fork on GitHub or [contribute](https://github.com/georg-jung/trivy-azure-pipelines-task) if you need further improvements.

![Screenshot showing the trivy extension in the Azure Devops UI](screenshot.png)

## Installation

1. Install the Trivy task in your Azure DevOps organization (hit the `Get it free` button above).

2. Add the task to your `azure-pipelines.yml` in a project where you'd like to run trivy:

```yaml
- task: trivy-contrib@1
```

## Configuration

You can supply several inputs to customise the task.

| Input        | Description                                                                                                                          |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------|
| `version`    | The version of Trivy to use. Currently defaults to `latest`.                                                                         |
| `docker`     | Run Trivy using the aquasec/trivy docker image. Alternatively the Trivy binary will be run natively. Defaults to `true`.             |
| `loginDockerConfig` | Set this to true if the `Docker login` task is used to access private repositories. Defaults to `false`.                      |
| `debug`      | Enable debug logging in the build output.                                                                                            |
| `path`       | The path to scan relative to the root of the repository being scanned, if an `fs` scan is required. Cannot be set if `image` is set. |
| `severities` | The severities (`CRITICAL,HIGH,MEDIUM,LOW,UNKNOWN`) to include in the scan (comma sperated). Defaults to `CRITICAL,HIGH,MEDIUM,LOW,UNKNOWN`. |
| `ignoreUnfixed` | When set to `true` all unfixed vulnerabilities will be skipped. Defaults to `false`.                                               |
| `image`      | The image to scan if an `image` scan is required. Cannot be set if `path` is set.                                                    |
| `exitCode`   | The exit-code to use when Trivy detects issues. Set to `0` to prevent the build failing when Trivy finds issues. Defaults to `1`.    |
| `scanners`   | Which scanners to use. Defaults to `vuln,misconfig,secret`.                                                                          |
| `aquaKey`    | The Aqua API Key to use to link scan results to your Aqua Security account _(not required)_.                                         |
| `aquaSecret` | The Aqua API Secret to use to link scan results to your Aqua Security account _(not required)_.                                      |
| `options`    | Additional flags to pass to trivy. Example: `--timeout 10m0s` _(not required)_.                                                      |

### Example of scanning multiple targets

```yaml
trigger:
- main

pool:
  vmImage: ubuntu-latest

jobs:
- job: Scan the local project
  steps:
  - task: trivy-contrib@1
    inputs:
      path: .
- job: Scan the ubuntu image
  steps:
  - task: trivy-contrib@1
    inputs:
      image: ubuntu
```

## Scanning Images in Private Registries

You can scan images in private registries by using the `image` input after completing a `docker login`. For example:

```yaml
steps:
- task: Docker@2
  displayName: Login to ACR
  inputs:
    command: login
    containerRegistry: dockerRegistryServiceConnection1
- task: trivy-contrib@1
  inputs:
    image: my.registry/org/my-image:latest
```
