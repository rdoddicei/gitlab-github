# GitHub Actions Reusable Templates

This repository contains reusable GitHub Actions workflow templates converted from GitLab CI/CD pipeline configurations. These templates maintain the same variable names and logic structure for easy migration.

## 📁 Repository Structure

```
.github/
├── workflows/
│   ├── reusable-cron-dotnet.yml       # .NET cron job pipeline
│   ├── reusable-nuget-dotnet.yml      # .NET NuGet package pipeline
│   ├── reusable-processor-dotnet.yml  # .NET processor application pipeline
│   ├── reusable-service-dotnet.yml    # .NET service application pipeline
│   ├── reusable-build.yml             # Generic build workflow
│   ├── reusable-deploy.yml            # Deployment workflow
│   ├── reusable-multi-cluster-deploy.yml # Multi-cluster deployment workflow
│   ├── reusable-quality-gates.yml     # Quality gates (tests, SonarQube)
│   └── main-workflow.yml              # Example main workflow using all templates
└── workflow-templates/
    ├── build-image.yml                 # Build image template (existing)
    ├── deploy.yml                      # Deploy template (existing)
    ├── multi-cluster-deploy.yml        # Multi-cluster deploy template (existing)
    └── quality-gates.yml              # Quality gates template (existing)
```

## 🚀 Available Reusable Templates

### 1. .NET Application Templates

#### `reusable-cron-dotnet.yml`
**Purpose**: CI/CD pipeline for .NET cron job applications
**Original**: `v1/.net/cron.gitlab-ci.yml`

**Usage**:
```yaml
jobs:
  cron-pipeline:
    uses: ./.github/workflows/reusable-cron-dotnet.yml
    with:
      TMPL_IS_CRON: "true"
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:7.0"
      DEVOPS_QG_OVERRIDE_SMOKE_TESTS: "1"
      DEVOPS_QG_OVERRIDE_REGRESSION_TESTS: "1"
      CI_COMMIT_BRANCH: ${{ github.ref_name }}
      CI_DEFAULT_BRANCH: "main"
      CI_MERGE_REQUEST_ID: ${{ github.event.number }}
    secrets: inherit
```

#### `reusable-nuget-dotnet.yml`
**Purpose**: CI/CD pipeline for .NET NuGet package projects
**Original**: `v1/.net/nuget.gitlab-ci.yml`

**Usage**:
```yaml
jobs:
  nuget-pipeline:
    uses: ./.github/workflows/reusable-nuget-dotnet.yml
    with:
      SOLUTION_NAME: "MySolution"
      PROJ_NUGET_PACKAGE_NAME: "MyPackage"
      CI_COMMIT_BRANCH: ${{ github.ref_name }}
      CI_DEFAULT_BRANCH: "main"
      CI_MERGE_REQUEST_ID: ${{ github.event.number }}
    secrets: inherit
```

#### `reusable-processor-dotnet.yml`
**Purpose**: CI/CD pipeline for .NET processor applications
**Original**: `v1/.net/processor.gitlab-ci.yml`

**Usage**:
```yaml
jobs:
  processor-pipeline:
    uses: ./.github/workflows/reusable-processor-dotnet.yml
    with:
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:7.0"
      TMPL_SONAR_IMAGE: "harbor.use.ucdp.net/utp_common/upr-dotnet-sonarqube-image:dotnet-sdk-7.0"
      TMPL_HELM_ARGS: "--set serviceMonitor.enabled=false"
      DEVOPS_QG_OVERRIDE_SMOKE_TESTS: "1"
      DEVOPS_QG_OVERRIDE_REGRESSION_TESTS: "1"
    secrets: inherit
```

#### `reusable-service-dotnet.yml`
**Purpose**: CI/CD pipeline for .NET service applications
**Original**: `v1/.net/service.gitlab-ci.yml`

**Usage**:
```yaml
jobs:
  service-pipeline:
    uses: ./.github/workflows/reusable-service-dotnet.yml
    with:
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:7.0"
      TMPL_SONAR_IMAGE: "harbor.use.ucdp.net/utp_common/upr-dotnet-sonarqube-image:dotnet-sdk-7.0"
    secrets: inherit
```

### 2. Core Infrastructure Templates

#### `reusable-build.yml`
**Purpose**: Generic build workflow with Docker image creation
**Original**: `v1/common/jobs.build.yml`

**Features**:
- Semantic versioning
- Docker image building
- Conditional builds for MR/branches
- Support for different .NET SDK versions

#### `reusable-deploy.yml`
**Purpose**: Multi-environment deployment workflow
**Original**: `v1/common/jobs.deploy.yml`

**Features**:
- Feature branch deployments to DEV
- Environment-specific deployments (DEV, QA, UAT, STG, PROD)
- Multi-datacenter support (UO, USH, USJ, Azure, AWS)
- PCI compliance support
- RKE2 cluster support

#### `reusable-multi-cluster-deploy.yml`
**Purpose**: Multi-cluster deployment workflow
**Original**: `v1/common/jobs.multiClusterDeploy.yml`

**Features**:
- Deployment to multiple Kubernetes clusters simultaneously
- Support for B1A and B1S3 clusters
- RKE2 and traditional Kubernetes support

#### `reusable-quality-gates.yml`
**Purpose**: Quality assurance workflow with testing and code analysis
**Original**: `v1/common/jobs.qualityGates.yml`

**Features**:
- .NET unit testing with coverage reports
- NodeJS unit testing
- SonarQube code quality analysis
- JUnit test result reporting
- Coverage artifact uploads

## 🔧 Key Features Preserved

### Variable Naming Convention
All original variable names from GitLab CI are preserved:
- `TMPL_*` prefixed template variables
- `DEVOPS_QG_*` prefixed quality gate overrides
- `SCRT_*` prefixed secrets
- `PROJ_*` prefixed project-specific variables

### Environment Support
- **Non-Production**: DEV, QA, UAT, STG
- **Production**: PROD
- **Datacenters**: UO, USH, USJ, Azure (EASTUS), AWS (EASTUS)
- **Special**: PCI compliance environments
- **Kubernetes**: Traditional and RKE2 clusters

### Quality Gates
- Unit testing for .NET and NodeJS
- SonarQube integration for code quality
- Test coverage reporting
- Quality gate override mechanisms

## 🎯 Usage Examples

### Simple Project Example
```yaml
name: My Project Pipeline
on: [push, pull_request]

jobs:
  build-and-deploy:
    uses: ./.github/workflows/reusable-service-dotnet.yml
    with:
      CI_COMMIT_BRANCH: ${{ github.ref_name }}
      CI_DEFAULT_BRANCH: "main"
    secrets: inherit
```

### Complex Multi-Pipeline Example
See `main-workflow.yml` for a comprehensive example that:
- Automatically detects project type
- Runs appropriate pipeline based on detection
- Provides pipeline execution summary
- Supports multiple .NET project types

### Custom Configuration Example
```yaml
jobs:
  custom-processor:
    uses: ./.github/workflows/reusable-processor-dotnet.yml
    with:
      TMPL_UNIT_TEST_IMAGE: "mcr.microsoft.com/dotnet/sdk:8.0"
      TMPL_HELM_ARGS: "--set resources.requests.memory=512Mi"
      DEVOPS_QG_OVERRIDE_SMOKE_TESTS: "0"  # Enable smoke tests
      PROJ_SOLUTION_NAME: "MyCustomSolution"
      PROJ_TEST_DIRECTORY: "tests"
    secrets: inherit
```

## 🔐 Required Secrets

Configure these secrets in your GitHub repository settings:

### Kubernetes Secrets
- `SCRT_NONPROD_KUBE_CONFIG_*` - Non-production cluster configs
- `SCRT_PROD_KUBE_CONFIG_*` - Production cluster configs

### Quality Gates Secrets
- `SCRT_SONAR_LOGIN_INBCU` - SonarQube authentication token
- `SCRT_NPMRC_TOKEN` - NPM registry token
- `UNIT_TEST_DOTENV` - Test environment variables

### Build Secrets
- `SCRT_GITLAB_API_PRIVATE_TOKEN` - For semantic versioning (if used)

## 🔄 Migration from GitLab CI

1. **Copy Templates**: Copy all `.github/workflows/reusable-*.yml` files to your repository
2. **Configure Secrets**: Set up required secrets in GitHub repository settings
3. **Update Main Workflow**: Use `main-workflow.yml` as a starting point
4. **Test Pipeline**: Run the pipeline to verify functionality
5. **Customize**: Adjust inputs and conditions based on your specific needs

## 🛠️ Customization Guide

### Adding New Environments
To add a new deployment environment, edit the respective deployment workflow and add new jobs following the existing pattern:

```yaml
deploy_new_env:
  runs-on: ubuntu-latest
  if: github.ref == format('refs/heads/{0}', inputs.CI_DEFAULT_BRANCH)
  environment: new-env
  steps:
    - name: Deploy to New Environment
      run: |
        echo "Deploying to new environment"
      env:
        TMPL_KUBE_CONFIG: ${{ secrets.SCRT_NEW_ENV_KUBE_CONFIG }}
        TMPL_DATACENTER: new-datacenter
        TMPL_ENVIRONMENT_NAME: new-env
```

### Overriding Quality Gates
Use the override variables to skip specific quality checks:

```yaml
with:
  DEVOPS_QG_OVERRIDE_UNIT_TESTS_DOTNET: "1"  # Skip .NET unit tests
  DEVOPS_QG_OVERRIDE_SONARQUBE: "1"          # Skip SonarQube analysis
  DEVOPS_QG_OVERRIDE: "1"                    # Skip all quality gates
```

### Custom Docker Images
Specify custom Docker images for different stages:

```yaml
with:
  TMPL_UNIT_TEST_IMAGE: "your-registry/custom-dotnet:8.0"
  TMPL_SONAR_IMAGE: "your-registry/custom-sonar:latest"
```

## 📋 Validation Checklist

- [ ] All variable names match original GitLab CI configuration
- [ ] Environment-specific deployments work correctly
- [ ] Quality gates execute with proper test detection
- [ ] Docker images build successfully
- [ ] Secrets are properly configured and used
- [ ] Multi-cluster deployments target correct clusters
- [ ] Pipeline triggers work for all branch types
- [ ] Artifact uploads and test reporting function correctly

## 🤝 Contributing

When adding new templates or modifying existing ones:
1. Maintain variable naming conventions
2. Preserve original logic flow
3. Add comprehensive input validation
4. Update this documentation
5. Test with various project types

## 📞 Support

For issues or questions regarding these templates:
1. Check the variable names match your original GitLab configuration
2. Verify all required secrets are configured
3. Review the pipeline execution logs for specific errors
4. Compare with the original GitLab CI files in the `v1/` directory
