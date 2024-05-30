# CI

Personal repository, which contains templates for GitLab and GitHub CI/CD pipelines, as well as custom Docker images designed specifically for continuous integration workflows.
  
## Overview

This repository provides a set of tools to facilitate and streamline the CI/CD process for various projects. 

It includes:
- **CI/CD Templates:** Predefined configurations for setting up pipelines in GitLab and GitHub.
- **Docker Images:** Custom-built Docker images tailored for CI/CD environments, including a lighter version of the Haxe 4.2.5 image with NodeJs integration.

## CI/CD Templates
### GitLab CI/CD

The GitLab CI/CD templates are designed to help you quickly set up and configure your pipelines. 
They include predefined stages and jobs that cover common tasks such as building, testing, and deploying your code.

### GitHub Actions

The GitHub Actions templates provide a similar set of configurations for GitHub repositories. 
These templates help you automate your workflows using GitHub's native CI/CD system.

## Custom Docker Images
### Haxe 4.2.5 with NodeJs
- **Description:** A custom Docker image based on the official Haxe image, integrated with NodeJs 20.11.1. This image is optimized for Haxe JavaScript compilation in CI/CD pipelines.
- **Usage:**
```bash
docker pull registry.gitlab.com/bizouarn/ci/haxe:4.2.5
```

### Other Images

Additional custom Docker images are included in this repository, each tailored for specific CI/CD needs. Check the `docker` directory for more details on available images and their configurations.

## How to Use

### GitLab CI/CD

To use the GitLab CI/CD templates in your project, include the desired template in your `.gitlab-ci.yml` file:

```yaml
include:
  - remote: 'https://raw.githubusercontent.com/bizouarn/ci/main/gitlab-ci/template.yml'
```

### GitHub Actions

For GitHub Actions, reference the templates in your workflow files located in the `.github/workflows` directory:\n\n
```yaml
name: CI Pipeline
on: [push, pull_request]
jobs:
  example-job:
    runs-on: ubuntu-latest
    steps:
    - name: Run custom action from template
      uses: owner/repo-template/.github/actions/custom-action@main
```

## Contributing

Contributions are welcome! 
If you have improvements, please fork the repository and submit a pull request.

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.
