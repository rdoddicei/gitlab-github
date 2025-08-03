# Comprehensive Migration Guide: GitLab CI to GitHub Actions

## Table of Contents
1. [Overview](#overview)
2. [What Was Done](#what-was-done)
3. [Logic Conversion Details](#logic-conversion-details)
4. [Step-by-Step Migration Process](#step-by-step-migration-process)
5. [How to Execute](#how-to-execute)
6. [Validation and Testing](#validation-and-testing)
7. [Troubleshooting](#troubleshooting)

## Overview

This document provides a comprehensive guide for migrating from GitLab CI/CD pipelines to GitHub Actions. The migration converts all existing GitLab CI configurations from the `v1` folder into reusable GitHub Actions workflows while preserving the original logic, variable names, and functionality.

## What Was Done

### 1. Repository Analysis
```bash
# Files analyzed and converted:
v1/.net/cron.gitlab-ci.yml              → .github/workflows/reusable-cron-dotnet.yml
v1/.net/nuget.gitlab-ci.yml             → .github/workflows/reusable-nuget-dotnet.yml  
v1/.net/processor.gitlab-ci.yml         → .github/workflows/reusable-processor-dotnet.yml
v1/.net/service.gitlab-ci.yml           → .github/workflows/reusable-service-dotnet.yml
v1/common/jobs.build.yml                → .github/workflows/reusable-build.yml
v1/common/jobs.deploy.yml               → .github/workflows/reusable-deploy.yml
v1/common/jobs.multiClusterDeploy.yml   → .github/workflows/reusable-multi-cluster-deploy.yml
v1/common/jobs.qualityGates.yml         → .github/workflows/reusable-quality-gates.yml
```

### 2. Created Files
- **8 Reusable Workflow Templates** in `.github/workflows/`
- **1 Main Workflow** (`main-workflow.yml`) demonstrating usage
- **2 Documentation Files** (`GITHUB_ACTIONS_TEMPLATES.md`, `MIGRATION_STEPS.md`)

### 3. Key Conversions Performed

#### A. GitLab `.net/cron.gitlab-ci.yml` → `reusable-cron-dotnet.yml`
**Original GitLab Structure:**
```yaml
include:
  - local: 'v1/common/templates.yml'
  - local: 'v1/common/jobs.build.yml'
  - local: 'v1/common/jobs.deploy.yml'
  - local: 'v1/common/jobs.qualityGates.yml'

variables:
  TMPL_IS_CRON: "true"
  TMPL_UNIT_TEST_IMAGE: mcr.microsoft.com/dotnet/sdk:7.0
  DEVOPS_QG_OVERRIDE_SMOKE_TESTS: 1
  DEVOPS_QG_OVERRIDE_REGRESSION_TESTS: 1
```

**Converted GitHub Actions:**
```yaml
name: cron-dotnet
on:
  workflow_call:
    inputs:
      TMPL_IS_CRON:
        required: false
        type: string
        default: "true"
      TMPL_UNIT_TEST_IMAGE:
        required: false
        type: string
        default: "mcr.microsoft.com/dotnet/sdk:7.0"
      DEVOPS_QG_OVERRIDE_SMOKE_TESTS:
        required: false
        type: string
        default: "1"

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      TMPL_UNIT_TEST_IMAGE: ${{ inputs.TMPL_UNIT_TEST_IMAGE }}
    secrets: inherit
```

#### B. GitLab `common/jobs.build.yml` → `reusable-build.yml`
**Original GitLab Structure:**
```yaml
Semver:
  extends: .semver_template

Build_Image:
  extends: .build_image_template

Build_Image[MR]:
  extends: .build_image_no_push_template
  rules:
    - if: $CI_MERGE_REQUEST_ID
```

**Converted GitHub Actions:**
```yaml
jobs:
  semver:
    runs-on: ubuntu-latest
    container:
      image: ${{ inputs.TMPL_UNIT_TEST_IMAGE }}
    outputs:
      version: ${{ steps.version.outputs.version }}
    steps:
      - name: Generate semantic version
        id: version
        run: |
          VERSION="1.0.${{ github.run_number }}"
          echo "version=$VERSION" >> $GITHUB_OUTPUT

  build_image_mr:
    runs-on: ubuntu-latest
    if: ${{ inputs.CI_MERGE_REQUEST_ID != '' }}
    steps:
      - name: Build Docker image (MR - no push)
        uses: docker/build-push-action@v5
        with:
          push: false
```

## Logic Conversion Details

### 1. Conditional Logic Conversion

| GitLab CI | GitHub Actions |
|-----------|----------------|
| `if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH` | `if: github.ref == format('refs/heads/{0}', inputs.CI_DEFAULT_BRANCH)` |
| `if: $CI_MERGE_REQUEST_ID` | `if: ${{ inputs.CI_MERGE_REQUEST_ID != '' }}` |
| `when: manual` | `environment: manual-approval` |
| `when: never` | `if: false` |

### 2. Variable Mapping

| GitLab CI Variable | GitHub Actions Equivalent |
|-------------------|---------------------------|
| `$CI_COMMIT_BRANCH` | `${{ github.ref_name }}` |
| `$CI_DEFAULT_BRANCH` | `${{ inputs.CI_DEFAULT_BRANCH }}` |
| `$CI_MERGE_REQUEST_ID` | `${{ github.event.number }}` |
| `$CI_PIPELINE_ID` | `${{ github.run_number }}` |
| `$SCRT_*` | `${{ secrets.SCRT_* }}` |

### 3. Job Dependencies

**GitLab CI:**
```yaml
job_b:
  needs:
    - job: job_a
  stage: deploy
```

**GitHub Actions:**
```yaml
job_b:
  needs: job_a
  runs-on: ubuntu-latest
```

### 4. Artifacts and Caching

**GitLab CI:**
```yaml
artifacts:
  reports:
    junit: '**/*unitTest-results.xml'
    coverage_report:
      coverage_format: cobertura
      path: '**/coverage.cobertura.xml'
```

**GitHub Actions:**
```yaml
- name: Upload test results
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      **/*unitTest-results.xml
      **/coverage.cobertura.xml
```

## Step-by-Step Migration Process

### Phase 1: Environment Setup
1. **Create GitHub Repository Structure**
   ```bash
   mkdir -p .github/workflows
   mkdir -p .github/workflow-templates
   ```

2. **Copy Reusable Templates**
   ```bash
   # All reusable workflow files are already created in .github/workflows/
   ls .github/workflows/reusable-*.yml
   ```

### Phase 2: Secret Configuration
1. **Navigate to Repository Settings**
   - Go to your GitHub repository
   - Click on "Settings" → "Secrets and variables" → "Actions"

2. **Add Required Secrets**
   ```bash
   # Kubernetes Configuration Secrets
   SCRT_NONPROD_KUBE_CONFIG_B1A
   SCRT_NONPROD_KUBE_CONFIG_B1A_PCI
   SCRT_NONPROD_KUBE_CONFIG_USH
   SCRT_NONPROD_KUBE_CONFIG_B1360_PCI
   SCRT_NONPROD_KUBE_CONFIG_B551
   SCRT_NONPROD_KUBE_CONFIG_B551_PCI
   SCRT_NONPROD_KUBE_CONFIG_EASTUS_AKS
   SCRT_NONPROD_KUBE_CONFIG_JAPANEAST
   SCRT_NONPROD_KUBE_CONFIG_EASTUS_AWS
   
   # Production Kubernetes Secrets
   SCRT_PROD_KUBE_CONFIG_B1A
   SCRT_PROD_KUBE_CONFIG_B1A_PCI
   SCRT_PROD_KUBE_CONFIG_USH
   SCRT_PROD_KUBE_CONFIG_B1360_PCI
   SCRT_PROD_KUBE_CONFIG_B551
   SCRT_PROD_KUBE_CONFIG_B551_PCI
   SCRT_PROD_KUBE_CONFIG_JAPANEAST
   SCRT_PROD_KUBE_CONFIG_EASTUS_AKS
   
   # Quality Gates Secrets
   SCRT_SONAR_LOGIN_INBCU
   SCRT_NPMRC_TOKEN
   UNIT_TEST_DOTENV
   
   # Build Secrets
   SCRT_GITLAB_API_PRIVATE_TOKEN
   ```

### Phase 3: Workflow Implementation
1. **Use the Main Workflow (Recommended)**
   ```yaml
   # Copy main-workflow.yml to your repository
   cp main-workflow.yml .github/workflows/
   ```

2. **Or Create Custom Workflows**
   ```yaml
   # Example: Custom .NET service workflow
   name: My Service Pipeline
   on:
     push:
       branches: [main, develop]
     pull_request:
       branches: [main]
   
   jobs:
     service-pipeline:
       uses: ./.github/workflows/reusable-service-dotnet.yml
       with:
         CI_COMMIT_BRANCH: ${{ github.ref_name }}
         CI_DEFAULT_BRANCH: "main"
         CI_MERGE_REQUEST_ID: ${{ github.event.number }}
       secrets: inherit
   ```

## How to Execute

### 1. Automatic Execution (Recommended)
The main workflow automatically detects your project type and runs the appropriate pipeline:

```bash
# Push code to trigger the workflow
git add .
git commit -m "Add GitHub Actions workflows"
git push origin main
```

### 2. Manual Workflow Execution
1. Go to GitHub repository → "Actions" tab
2. Select the workflow you want to run
3. Click "Run workflow"
4. Select branch and provide inputs if required

### 3. Triggered Execution
Workflows automatically trigger on:
- **Push** to main, develop, feature/*, hotfix/*, release/* branches
- **Pull Requests** to main, develop branches
- **Schedule** (daily at 2 AM UTC for cron jobs)

### 4. Project-Specific Execution Examples

#### For .NET Service Projects:
```bash
# The main workflow will detect AspNetCore references and run:
# → reusable-service-dotnet.yml
# → → reusable-build.yml
# → → reusable-multi-cluster-deploy.yml  
# → → reusable-quality-gates.yml
```

#### For .NET NuGet Projects:
```bash
# Detection: *.nupkg or *.nuspec files
# Executes: reusable-nuget-dotnet.yml
```

#### For .NET Processor Projects:
```bash
# Detection: Microsoft.Extensions.Hosting + not scheduled
# Executes: reusable-processor-dotnet.yml
```

#### For .NET Cron Projects:
```bash
# Detection: Microsoft.Extensions.Hosting + scheduled trigger
# Executes: reusable-cron-dotnet.yml
```

## Validation and Testing

### 1. Pre-Migration Checklist
- [ ] All GitLab CI variables documented
- [ ] All secrets identified and available
- [ ] Environment-specific configurations noted
- [ ] Deployment targets documented

### 2. Post-Migration Validation
```bash
# Test each workflow component:

# 1. Test build workflow
git push origin feature/test-build

# 2. Test deployment (to dev environment)
git push origin develop  

# 3. Test quality gates
# Ensure test projects exist with naming: *Tests.csproj or *Test.ts

# 4. Test full pipeline
git push origin main
```

### 3. Monitoring and Verification
1. **Check Workflow Execution**
   - Navigate to Actions tab in GitHub
   - Verify all jobs complete successfully
   - Review logs for any errors

2. **Validate Deployments**
   ```bash
   # Verify deployments reached target environments
   kubectl get pods -n your-namespace
   ```

3. **Verify Artifacts**
   - Check test results are uploaded
   - Verify coverage reports are generated
   - Confirm Docker images are built

## Troubleshooting

### Common Issues and Solutions

#### 1. Secret Not Found Errors
```yaml
Error: Secret SCRT_NONPROD_KUBE_CONFIG_B1A not found
```
**Solution:**
- Verify secret is configured in repository settings
- Check secret name spelling matches exactly
- Ensure secret has appropriate permissions

#### 2. Workflow Permission Errors
```yaml
Error: Resource not accessible by integration
```
**Solution:**
```yaml
# Add to workflow file:
permissions:
  contents: read
  actions: read
  checks: write
```

#### 3. Docker Image Build Failures
```yaml
Error: Cannot connect to the Docker daemon
```
**Solution:**
```yaml
# Ensure Docker setup in workflow:
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3
```

#### 4. Test Detection Issues
```yaml
No tests found matching pattern
```
**Solution:**
- Verify test project naming conventions: `*Tests.csproj`, `*Test.ts`, `*Spec.ts`
- Check file paths in repository structure
- Review test discovery patterns in workflow

#### 5. Environment Deployment Failures
```yaml
Error: Environment 'prod-uo' protection rules not satisfied
```
**Solution:**
- Configure environment protection rules in repository settings
- Set up required reviewers for production environments
- Verify branch protection rules

### Debug Mode
Enable debug logging by setting repository secret:
```bash
ACTIONS_STEP_DEBUG = true
ACTIONS_RUNNER_DEBUG = true
```

### Performance Optimization
1. **Parallel Job Execution**
   ```yaml
   # Jobs run in parallel by default unless dependencies exist
   job1:
     runs-on: ubuntu-latest
   job2:
     runs-on: ubuntu-latest  # Runs parallel to job1
   job3:
     needs: [job1, job2]     # Waits for both job1 and job2
   ```

2. **Conditional Job Execution**
   ```yaml
   # Skip unnecessary jobs based on file changes
   if: contains(github.event.head_commit.modified, '.cs') || contains(github.event.head_commit.added, '.cs')
   ```

## Migration Validation Checklist

- [ ] All original GitLab CI files analyzed
- [ ] Variable names preserved in GitHub Actions
- [ ] All deployment environments configured
- [ ] Quality gates properly implemented
- [ ] Secret management configured
- [ ] Workflow triggers match original pipeline
- [ ] Test execution and reporting functional
- [ ] Docker image building operational
- [ ] Multi-cluster deployments working
- [ ] Environment-specific configurations applied
- [ ] Production deployment protection enabled
- [ ] Pipeline execution monitored and validated

## Next Steps

1. **Customize for Your Environment**
   - Adjust environment names and configurations
   - Modify deployment targets as needed
   - Update Docker image repositories

2. **Enhance Security**
   - Implement environment-specific approvals
   - Add branch protection rules
   - Configure security scanning

3. **Optimize Performance**
   - Implement workflow caching
   - Optimize Docker builds
   - Configure matrix builds for multiple environments

4. **Monitor and Maintain**
   - Set up workflow notifications
   - Implement metric collection
   - Regular review and updates

This comprehensive migration ensures your CI/CD pipeline functionality is preserved while taking advantage of GitHub Actions' native features and capabilities.
