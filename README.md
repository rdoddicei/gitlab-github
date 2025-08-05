# Centralized GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows for .NET applications, converted from GitLab CI/CD pipelines. These workflows provide a centralized approach to CI/CD that can be shared across multiple repositories.

## 🚀 Available Workflows

### Main Orchestrator Workflow
- **`dotnet-service-pipeline.yml`** - Complete CI/CD pipeline for .NET service applications
  - Orchestrates build, quality gates, and deployment
  - Supports conditional deployment (skips PRs)
  - Cross-repository reusable workflow calls

### Core Workflows
- **`dotnet-build-pipeline.yml`** - Build and Docker image creation
  - Semantic versioning with branch-based strategies
  - Docker image building and registry pushing
  - Harbor registry integration
  - Build caching and optimization

- **`dotnet-quality-gates.yml`** - Quality assurance and testing
  - Unit tests with coverage reporting
  - SonarQube integration and quality gates
  - Optional smoke and regression tests
  - Comprehensive test reporting

- **`dotnet-multi-cluster-deploy.yml`** - Multi-cluster deployment
  - Multiple environment support (DEV, QA, UAT, PROD)
  - Kubernetes deployment across clusters
  - Multi-datacenter capabilities

## 📋 Usage Examples

### Basic Usage (Recommended)
```yaml
name: .NET CI/CD Pipeline

on:
  push:
    branches: [main, develop, 'feature/**']
  pull_request:
    branches: [main, develop]

jobs:
  ci-cd:
    uses: rdoddicei/gitlab-github/.github/workflows/dotnet-service-pipeline.yml@main
    with:
      PROJ_SERVICE_NAME: "my-service"
      PROJ_SOLUTION_NAME: "MySolution"
      PROJ_IMAGE_NAME: "my-company/my-service"
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:8.0"
      CI_COMMIT_BRANCH: ${{ github.ref_name }}
      CI_DEFAULT_BRANCH: "main"
      CI_MERGE_REQUEST_ID: ${{ github.event.number }}
    secrets: inherit
```

### Advanced Individual Workflow Usage
```yaml
jobs:
  build:
    uses: rdoddicei/gitlab-github/.github/workflows/dotnet-build-pipeline.yml@main
    with:
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:8.0"
      PROJ_SOLUTION_NAME: "MySolution"
      PROJ_IMAGE_NAME: "my-company/my-service"
      CI_COMMIT_BRANCH: ${{ github.ref_name }}
    secrets: inherit

  quality-gates:
    needs: build
    uses: rdoddicei/gitlab-github/.github/workflows/dotnet-quality-gates.yml@main
    with:
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:8.0"
      TMPL_SONAR_IMAGE: "sonarqube/sonar-scanner-cli:latest"
      PROJ_SOLUTION_NAME: "MySolution"
    secrets: inherit
```

## 🔧 Required Secrets

Configure these secrets in your repository settings:

```yaml
HARBOR_USERNAME          # Container registry username
HARBOR_PASSWORD          # Container registry password
SONAR_TOKEN             # SonarQube authentication token
GITLAB_API_TOKEN        # GitLab API token (if needed)
```

## 📊 Input Parameters

### Common Parameters
| Parameter | Description | Required | Default |
|-----------|-------------|----------|----------|
| `TMPL_UNIT_TEST_IMAGE` | .NET SDK Docker image | No | `mcr.microsoft.com/dotnet/sdk:8.0` |
| `TMPL_SONAR_IMAGE` | SonarQube scanner image | No | `harbor.use.ucdp.net/...` |
| `CI_COMMIT_BRANCH` | Current branch name | No | - |
| `CI_DEFAULT_BRANCH` | Default branch name | No | `main` |
| `CI_MERGE_REQUEST_ID` | Pull request ID | No | - |

### Project-Specific Parameters
| Parameter | Description | Required | Default |
|-----------|-------------|----------|----------|
| `PROJ_SERVICE_NAME` | Service name | No | - |
| `PROJ_SOLUTION_NAME` | .NET solution name | No | - |
| `PROJ_IMAGE_NAME` | Docker image name | No | - |
| `PROJ_K8S_NAMESPACE_BASE` | Kubernetes namespace | No | - |
| `PROJ_HARBOR_DIRECTORY` | Harbor directory | No | - |

### Quality Gate Overrides
| Parameter | Description | Default |
|-----------|-------------|----------|
| `DEVOPS_QG_OVERRIDE_UNIT_TESTS` | Skip unit tests (1=skip, 0=run) | `0` |
| `DEVOPS_QG_OVERRIDE_SONAR_ANALYSIS` | Skip SonarQube (1=skip, 0=run) | `0` |
| `DEVOPS_QG_OVERRIDE_SMOKE_TESTS` | Skip smoke tests (1=skip, 0=run) | `1` |
| `DEVOPS_QG_OVERRIDE_REGRESSION_TESTS` | Skip regression tests (1=skip, 0=run) | `1` |

## 🏗️ Repository Structure

```
.github/
└── workflows/
    ├── dotnet-service-pipeline.yml      # Main orchestrator workflow
    ├── dotnet-build-pipeline.yml        # Build and Docker pipeline
    ├── dotnet-quality-gates.yml         # Quality assurance pipeline
    └── dotnet-multi-cluster-deploy.yml  # Multi-cluster deployment
```

## 🎯 Key Features

- ✅ **Cross-Repository Reusability** - Use from any repository
- ✅ **Conditional Logic** - Smart branching and PR handling
- ✅ **Quality Gates** - Comprehensive testing and analysis
- ✅ **Multi-Environment** - Support for multiple deployment targets
- ✅ **Docker Integration** - Container building and registry pushing
- ✅ **Semantic Versioning** - Automatic version generation
- ✅ **Override Support** - Flexible quality gate controls
- ✅ **Comprehensive Logging** - Detailed workflow summaries

## 🔄 Migration from GitLab CI

This repository provides a direct migration path from GitLab CI to GitHub Actions while preserving:
- Original variable names and logic
- Quality gate configurations
- Deployment strategies
- Build and test processes

## 📞 Support

For questions or issues with these workflows, please create an issue in this repository or contact the DevOps team.

---

**Last Updated**: January 2025  
**Maintained By**: Infrastructure Team
