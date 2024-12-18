# GitHub Actions CI/CD Pipeline: Project Overview

This document provides a detailed explanation of the GitHub Actions workflow implemented in this project. The workflow automates the CI/CD (Continuous Integration and Continuous Deployment) process, ensuring efficient builds, testing, code analysis, and deployment.

## Key Features
- **Automatic Workflow Trigger**: Executes on code pushes or pull requests to the `master` branch, as well as manual triggers.
- **Parallel Steps**: SonarCloud and Trivy security scans are run in parallel to optimize execution time.
- **Caching**: Dependencies are cached to speed up builds.

---

## Workflow File Explanation

### Workflow Metadata
```yaml
name: CI/CD Pipeline
```
- **name**: Defines the name of the workflow visible in the GitHub Actions interface.

```yaml
on:
  push:
    branches:
      - master
  pull_request:
    branches:
      - master
  workflow_dispatch:
```
- **on**: Specifies the events that trigger the workflow:
  - **push**: Runs when changes are pushed to the `master` branch.
  - **pull_request**: Runs when a pull request targets the `master` branch.
  - **workflow_dispatch**: Allows manual triggering of the workflow via the GitHub Actions interface.

---

### Jobs Overview

#### 1. **Build Job**
```yaml
build:
  runs-on: ubuntu-latest
```
- **runs-on**: Specifies the environment for the job. Here, `ubuntu-latest` indicates that the job runs on the latest version of Ubuntu.

##### Build Steps
1. **Checkout Code**:
   ```yaml
   - name: Checkout code
     uses: actions/checkout@v3
   ```
   - Fetches the repository code so it’s available in the workflow.

2. **Setup .NET**:
   ```yaml
   - name: Setup .NET
     uses: actions/setup-dotnet@v3
     with:
       dotnet-version: '8.0.x'
   ```
   - Installs the specified version of .NET (version 8.0.x in this case).

3. **Restore Dependencies**:
   ```yaml
   - name: Restore dependencies
     run: dotnet restore
   ```
   - Restores project dependencies defined in the `.csproj` or `.sln` files.

4. **Build Project**:
   ```yaml
   - name: Build
     run: dotnet build --no-restore --configuration Release
   ```
   - Compiles the project in `Release` configuration. The `--no-restore` flag skips restoring dependencies again (already done in the previous step).

---

#### 2. **Code Analysis Job**
```yaml
code_analysis:
  runs-on: ubuntu-latest
  needs: build
```
- **needs: build**: Ensures this job starts only after the `build` job completes successfully.

##### Parallel Steps
This job contains two parallel steps:

1. **SonarCloud Scan**:
   ```yaml
   - name: Run SonarCloud Scan
     uses: sonarsource/sonarcloud-github-action@v2
     with:
       projectBaseDir: .
       args: >
         -Dsonar.organization=ishakuden
         -Dsonar.projectKey=IshaKuden_github-action-sample
         -Dsonar.host.url=https://sonarcloud.io
     env:
       SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
   ```
   - Runs a static code analysis using SonarCloud.
   - **projectBaseDir**: Specifies the project’s root directory.
   - **args**: Passes configuration parameters to the scanner, such as organization and project keys.
   - **SONAR_TOKEN**: Retrieves a secure authentication token from GitHub Secrets.

   **SonarCloud Analysis Completed Successfully**

   The screenshot below shows the result of the SonarCloud analysis performed for this project. This analysis was successfully completed to ensure code quality and security.

   ![SonarQube Analysis Result](https://github.com/IshaKuden/github-action-sample/blob/master/github-action-sample/sonar.PNG)

2. **Trivy Security Scan**:
   ```yaml
   - name: Run Trivy Security Scan
     uses: aquasecurity/trivy-action@0.28.0
     with:
       scan-type: 'fs'
       input: '.'
   ```
   - Performs a file system security scan to detect vulnerabilities in the project’s dependencies and code.
   - **scan-type**: Specifies the scan type (`fs` for file system).
   - **input**: Indicates the directory to scan (`.` for the current directory).

---

#### 3. **Test Job**
```yaml
test:
  runs-on: ubuntu-latest
  needs: code_analysis
```
- **needs: code_analysis**: Ensures the test job runs only after both SonarCloud and Trivy scans complete successfully.

##### Test Steps
1. **Checkout Code**:
   ```yaml
   - name: Checkout code
     uses: actions/checkout@v3
   ```
   - Fetches the code repository.

2. **Setup .NET**:
   ```yaml
   - name: Setup .NET
     uses: actions/setup-dotnet@v3
     with:
       dotnet-version: '8.0.x'
   ```
   - Installs .NET (version 8.0.x).

3. **Run Tests**:
   ```yaml
   - name: Run Tests
     run: dotnet test ./github-action-sample-tests/github-action-sample-tests.csproj --configuration Release
   ```
   - Executes unit tests defined in the specified project file using the `Release` configuration.

---

#### 4. **Deploy Job**
```yaml
deploy:
  runs-on: ubuntu-latest
  needs: [test]
```
- **needs: [test]**: Ensures deployment runs only after the test job completes successfully.

##### Deployment Steps
1. **Checkout Code**:
   ```yaml
   - name: Checkout code
     uses: actions/checkout@v3
   ```
   - Fetches the latest code version.

2. **Simulated Deployment**:
   ```yaml
   - name: Deploy Application (Simulated)
     run: echo "Deploy step placeholder. Customize for your specific deployment process."
   ```
   - Placeholder command simulating deployment. Replace with actual deployment scripts for production use.

---

## Caching for Efficiency
Cache is used to speed up workflow execution:
1. **SonarCloud Scanner Cache**:
   ```yaml
   cache:
     key: sonar-scanner-${{ runner.os }}
     paths:
       - .sonar/cache
   ```
   - Reuses downloaded analysis tools and data.

2. **Trivy Cache**:
   ```yaml
   cache:
     key: cache-trivy-${{ runner.os }}-${{ github.event.date }}
     paths:
       - /path/to/trivy/cache
   ```
   - Reuses security scanning data to reduce runtime.

---

## Parallel Execution
The `code_analysis` job is optimized for performance by splitting it into two parallel steps:
1. **SonarCloud Scan**
2. **Trivy Security Scan**

This ensures the workflow finishes faster without compromising functionality.

---

## Future Improvements
- Automate deployment to production environments.
- Add integration tests for end-to-end validation.
- Enhance caching strategies to include build dependencies.

---

This pipeline demonstrates a comprehensive CI/CD approach with build, analysis, testing, and deployment phases, ensuring high-quality code delivery.

