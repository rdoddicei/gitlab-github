# Migration Steps from GitLab CI to GitHub Actions

This document outlines the steps taken to convert the CI/CD configurations from GitLab to GitHub Actions, along with the logic behind these conversions and instructions for execution.

## Conversion Steps

1. **Analyze Repository Structure**:
   - Identified the location of `.gitlab-ci.yml` files in the `v1` folder.
   - Evaluated the structure of these files to understand their configurations.

2. **Convert GitLab CI Jobs to GitHub Actions Workflows**:
   - For each GitLab CI file in the `.net` and `common` directories, a corresponding GitHub Actions workflow was created.
   - Preserved variable names and logic as much as possible to ease the conversion.
   
3. **Create Reusable Templates**:
   - **Templates converted**:
     - `.net/cron.gitlab-ci.yml` to `reusable-cron-dotnet.yml`
     - `.net/nuget.gitlab-ci.yml` to `reusable-nuget-dotnet.yml`
     - `.net/processor.gitlab-ci.yml` to `reusable-processor-dotnet.yml`
     - `.net/service.gitlab-ci.yml` to `reusable-service-dotnet.yml`
     - `common/jobs.build.yml` to `reusable-build.yml`
     - `common/jobs.deploy.yml` to `reusable-deploy.yml`
     - `common/jobs.multiClusterDeploy.yml` to `reusable-multi-cluster-deploy.yml`
     - `common/jobs.qualityGates.yml` to `reusable-quality-gates.yml`
   
4. **Design Main Workflow**:
   - Created `main-workflow.yml` as a central workflow to invoke the reusable templates.
   - Integrated automated project type detection.

5. **Documentation**:
   - Developed `GITHUB_ACTIONS_TEMPLATES.md` to provide comprehensive guidance on template usage.
   - Added this migration steps document to capture the entire process.

## Logic Conversion Details

### GitLab to GitHub Mapping
- **Stages vs Jobs**:
  - GitLab stages are mapped to GitHub job sequences.
  - Conditional logic using `when:` is translated to `if:` in GitHub.

- **Variables**:
  - GitLab environment variables are preserved as inputs or secrets in GitHub Actions.

- **Includes**:
  - External templates or includes are replaced with reusable workflows.

### Example Translations
- **GitLab Script Execution**:
  ```yaml
  script:
    - dotnet build
  ```
  **GitHub Equivalent**:
  ```yaml
  steps:
    - name: Build
      run: dotnet build
  ```

- **Conditional Job Execution**:
  **GitLab**:
  ```yaml
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  ```
  **GitHub**:
  ```yaml
  if: github.ref == 'refs/heads/main'
  ```

## Execution Instructions

1. **Set Up Secrets**:
   - Configure required secrets in GitHub repository settings.
   - Ensure all environment-specific and build secrets are properly set.

2. **Copy Templates**:
   - Ensure all workflow templates are present in the `.github/workflows/` directory.

3. **Execution**:
   - Trigger the workflows by pushing changes to the repository or making pull requests.
   - Monitor the execution via the GitHub Actions tab in your repository.

4. **Review and Customize**:
   - Inspect pipeline execution under the Actions tab.
   - Customize workflows as necessary to better fit your project requirements.

## Conclusion

This migration ensures that the CI/CD pipeline from GitLab is successfully translated to GitHub Actions while maintaining the same functional workflow integrity. For any adjustments, follow the reusable template guidelines in `GITHUB_ACTIONS_TEMPLATES.md`.
