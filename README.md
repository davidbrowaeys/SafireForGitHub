<!-- Header Banner -->
<div align="center">
  <img src="https://img.shields.io/badge/Salesforce-00A1E0?style=for-the-badge&logo=salesforce&logoColor=white" alt="Salesforce"/>
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="GitHub Actions"/>
  <img src="https://img.shields.io/badge/CI%2FCD-Pipeline-success?style=for-the-badge" alt="CI/CD"/>
</div>

<h1 align="center">🔥 Safire for GitHub</h1>

<p align="center">
  <strong>A powerful, reusable CI/CD pipeline for Salesforce projects on GitHub Actions</strong>
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-configuration">Configuration</a> •
  <a href="#-workflow-inputs">Inputs</a> •
  <a href="#-secrets--variables">Secrets</a> •
  <a href="#-examples">Examples</a>
</p>

---

## 📖 Overview

**Safire for GitHub** is a production-ready, reusable GitHub Actions workflow designed specifically for Salesforce development teams. It provides a complete CI/CD solution that handles everything from delta deployments to code analysis, test execution, and automated community publishing.

Built on top of the powerful [DXB CLI](https://github.com/davidbrowaeys/DXB), Safire streamlines your Salesforce DevOps process with intelligent delta detection, comprehensive test coverage reporting, and seamless multi-environment deployments.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔄 **Delta Deployments** | Smart change detection - deploy only what changed |
| 🧪 **Automated Testing** | Apex tests with coverage reports + LWC Jest tests |
| 🛡️ **Code Analysis** | Integrated Salesforce Code Analyzer |
| 📊 **Rich Reporting** | Detailed deployment reports in GitHub Summary |
| 🌐 **Community Publishing** | Auto-publish Experience Cloud sites |
| 📥 **Data Import** | Automated data seeding post-deployment |
| ⚙️ **Org Pre-Configuration** | Environment-specific metadata setup |
| 🎯 **Multi-Environment** | Support for QA, SIT, UAT, and Production |

---

## 🚀 Quick Start

### Prerequisites

1. **Salesforce CLI Auth URL** - Generate using:
   ```bash
   sf org display --target-org <your-org-alias> --verbose --json
   ```
   Copy the `sfdxAuthUrl` value from the output.

2. **GitHub Environment Setup** - Create environments in your repository settings (e.g., `QA`, `SIT`, `UAT`, `PROD`)

3. **Secrets Configuration** - Add `AUTH_URL` secret to each environment

### Basic Setup

Create a workflow file in your Salesforce project repository:

```yaml
# .github/workflows/salesforce-cicd.yml
name: Salesforce CI/CD

on:
  pull_request:
    branches: [develop]
  push:
    branches: [develop]

jobs:
  deploy:
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: QA
      deployment-mode: DELTA
      check-only: ${{ github.event_name == 'pull_request' }}
    secrets: inherit
```

That's it! Your Salesforce project now has automated CI/CD. 🎉

---

## ⚙️ Configuration

### GitHub Environment Variables

Configure these variables in **Settings → Environments → [Your Environment] → Environment Variables**:

| Variable | Description | Example |
|----------|-------------|---------|
| `VAL_COMPARE_WITH` | Git reference for validation delta comparison | `origin/develop` |
| `DEP_COMPARE_WITH` | Git reference for deployment delta comparison | `HEAD~1` |
| `VAL_DELTA_MODE` | Delta mode for validations | `branch` |
| `DEP_DELTA_MODE` | Delta mode for deployments | `commit` |
| `VAL_TEST_LEVEL` | Test level for validations | `RunLocalTests` |
| `DEP_TEST_LEVEL` | Test level for deployments | `RunSpecifiedTests` |
| `MIN_CODE_COVERAGE` | Minimum required code coverage (%) | `75` |
| `CHECKPERFORMANCE` | Max test execution time (seconds) | `3` |
| `PRE_DESTRUCTIVE_XML_PATH` | Path to pre-destructive changes XML | `destructive/pre.xml` |
| `POST_DESTRUCTIVE_XML_PATH` | Path to post-destructive changes XML | `destructive/post.xml` |
| `DATA_DEF_FILE` | Data import definition file path | `config/data-def.json` |
| `ORG_SPEC_METADATA_DEF` | Org-specific metadata config path | `config/org-config.json` |

### GitHub Secrets

Configure in **Settings → Environments → [Your Environment] → Environment Secrets**:

| Secret | Description | Required |
|--------|-------------|----------|
| `AUTH_URL` | Salesforce SFDX Auth URL | ✅ Yes |

---

## 📋 Workflow Inputs

### Required Inputs

| Input | Type | Description |
|-------|------|-------------|
| `target-env` | `string` | Salesforce target environment alias (must match GitHub Environment name) |
| `deployment-mode` | `string` | `DELTA` for incremental or `FULL` for complete deployment |
| `check-only` | `boolean` | `true` for validation only, `false` for actual deployment |

### Optional Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `test-level` | `string` | *from env vars* | `NoTestRun`, `RunLocalTests`, or `RunSpecifiedTests` |
| `outputdir` | `string` | `results` | Output directory for deployment artifacts |
| `change-detection-type` | `string` | `classes` | Metadata type for test class detection |
| `test-class-regex` | `string` | `.*Test.*$` | Regex pattern for matching test class names |
| `org-preconfig` | `boolean` | `false` | Run pre-deployment org configuration |
| `run-code-scanner` | `boolean` | `false` | Run Salesforce Code Analyzer |
| `run-data-import` | `boolean` | `false` | Run data import after deployment |
| `auto-publish-communities` | `boolean` | `false` | Auto-publish Experience Cloud communities |
| `runjest` | `boolean` | `false` | Run LWC Jest tests |

---

## 📚 Examples

### Example 1: Simple PR Validation + Deploy on Merge

```yaml
name: Salesforce CI/CD

on:
  pull_request:
    branches: [develop]
  push:
    branches: [develop]

jobs:
  salesforce:
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: QA
      deployment-mode: DELTA
      check-only: ${{ github.event_name == 'pull_request' }}
      run-code-scanner: ${{ github.event_name == 'pull_request' }}
      runjest: true
    secrets: inherit
```

### Example 2: Multi-Environment Pipeline (QA → SIT → UAT)

```yaml
name: Salesforce Project CI/CD

on:
  pull_request:
    branches:
      - develop
      - 'release/**'
  push:
    branches:
      - develop
      - 'release/**'

jobs:
  # ═══════════════════════════════════════════════════════════════════
  # STEP 1: Determine Target Environment
  # ═══════════════════════════════════════════════════════════════════
  determine_targets:
    runs-on: ubuntu-latest
    outputs:
      TARGET_ENV: ${{ steps.set_targets.outputs.TARGET_ENV }}
      CHECK_ONLY: ${{ steps.set_targets.outputs.CHECK_ONLY }}
      
    steps:
      - name: 🔍 Determine Target Environment & Mode
        id: set_targets
        run: |
          EVENT_NAME="${{ github.event_name }}"
          BASE_REF="${{ github.base_ref }}"
          REF="${{ github.ref }}"
          
          echo "Event: $EVENT_NAME | Base: $BASE_REF | Ref: $REF"
          
          if [ "$EVENT_NAME" = "pull_request" ]; then
            CHECK_ONLY_MODE="true"
            
            if [ "$BASE_REF" = "develop" ]; then 
              echo "TARGET_ENV=QA" >> $GITHUB_OUTPUT
            elif [[ "$BASE_REF" =~ ^release/ ]]; then
              echo "TARGET_ENV=SIT" >> $GITHUB_OUTPUT
            fi
            echo "CHECK_ONLY=$CHECK_ONLY_MODE" >> $GITHUB_OUTPUT
          else
            CHECK_ONLY_MODE="false"
            
            if [[ "$REF" =~ ^refs/heads/release/ ]]; then
              echo "TARGET_ENV=SIT" >> $GITHUB_OUTPUT
            elif [ "$REF" = "refs/heads/develop" ]; then
              echo "TARGET_ENV=QA" >> $GITHUB_OUTPUT
            fi
            echo "CHECK_ONLY=$CHECK_ONLY_MODE" >> $GITHUB_OUTPUT
          fi
        shell: bash

  # ═══════════════════════════════════════════════════════════════════
  # STEP 2: Primary CI/CD (QA or SIT)
  # ═══════════════════════════════════════════════════════════════════
  continuous_integration:
    name: 🚀 Deploy to ${{ needs.determine_targets.outputs.TARGET_ENV || 'N/A' }}
    needs: determine_targets
    if: ${{ needs.determine_targets.outputs.TARGET_ENV != '' }}
    
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: ${{ needs.determine_targets.outputs.TARGET_ENV }}
      deployment-mode: DELTA
      check-only: ${{ needs.determine_targets.outputs.CHECK_ONLY == 'true' }}
      run-code-scanner: ${{ needs.determine_targets.outputs.CHECK_ONLY == 'true' }}
      run-data-import: false
      runjest: true
    secrets: inherit

  # ═══════════════════════════════════════════════════════════════════
  # STEP 3: Release Stage - Validate UAT
  # ═══════════════════════════════════════════════════════════════════
  release_validate_uat:
    name: ✅ Validate UAT
    needs: continuous_integration
    if: ${{ github.event_name == 'push' && startsWith(github.ref_name, 'release/') }}
    
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: UAT
      deployment-mode: DELTA
      check-only: true
      run-code-scanner: false
      run-data-import: false
    secrets: inherit

  # ═══════════════════════════════════════════════════════════════════
  # STEP 4: Release Stage - Deploy UAT
  # ═══════════════════════════════════════════════════════════════════
  release_deploy_uat:
    name: 🚀 Deploy to UAT
    needs: release_validate_uat
    if: ${{ github.event_name == 'push' && startsWith(github.ref_name, 'release/') }}
    
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: UAT
      deployment-mode: DELTA
      check-only: false
      run-code-scanner: false
      run-data-import: false
    secrets: inherit
```

### Example 3: Full Deployment with All Features

```yaml
name: Full Feature Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target Environment'
        required: true
        type: choice
        options: [QA, SIT, UAT, PROD]

jobs:
  deploy:
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: ${{ inputs.environment }}
      deployment-mode: FULL
      check-only: false
      run-code-scanner: true
      run-data-import: true
      org-preconfig: true
      auto-publish-communities: true
      runjest: true
    secrets: inherit
```

### Example 4: Scheduled Validation

```yaml
name: Nightly Validation

on:
  schedule:
    - cron: '0 2 * * *'  # Every day at 2 AM

jobs:
  validate:
    uses: davidbrowaeys/SafireForGitHub/.github/workflows/continuous-integration.yml@main
    with:
      target-env: QA
      deployment-mode: FULL
      check-only: true
      run-code-scanner: true
      runjest: true
    secrets: inherit
```

---

## 🔄 Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SAFIRE PIPELINE FLOW                             │
└─────────────────────────────────────────────────────────────────────────┘

  ┌──────────────┐
  │  PR Created  │
  └──────┬───────┘
         │
         ▼
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │   Checkout   │────▶│  SF Login    │────▶│ Delta Calc   │
  └──────────────┘     └──────────────┘     └──────┬───────┘
                                                   │
                       ┌───────────────────────────┼───────────────────────┐
                       │                           │                       │
                       ▼                           ▼                       ▼
                ┌──────────────┐           ┌──────────────┐         ┌──────────────┐
                │ Code Scanner │           │  Org Config  │         │   Deploy/    │
                │  (optional)  │           │  (optional)  │         │   Validate   │
                └──────────────┘           └──────────────┘         └──────┬───────┘
                                                                          │
                       ┌──────────────────────────────────────────────────┤
                       │                           │                      │
                       ▼                           ▼                      ▼
                ┌──────────────┐           ┌──────────────┐         ┌──────────────┐
                │ Jest Tests   │           │ Data Import  │         │  Community   │
                │  (optional)  │           │  (optional)  │         │   Publish    │
                └──────────────┘           └──────────────┘         └──────────────┘
                       │                           │                      │
                       └───────────────────────────┼──────────────────────┘
                                                   │
                                                   ▼
                                           ┌──────────────┐
                                           │   Reports    │
                                           │  & Summary   │
                                           └──────────────┘
```

---

## 📊 GitHub Summary Reports

Safire automatically generates rich deployment reports in your GitHub Actions Summary:

- ⚙️ **Configuration Summary** - All deployment settings at a glance
- 📦 **Package Contents** - What's being deployed
- 🧪 **Test Results** - Apex test outcomes with coverage
- 🛡️ **Code Analysis** - Security and quality findings
- ✅ **Deployment Status** - Success/failure details

---

## 🛠️ Included Actions

### LWC Jest Test Runner

Safire includes a reusable action for running LWC Jest tests:

```yaml
- uses: davidbrowaeys/SafireForGitHub/.github/actions/sfdx-lwc-test-run@main
  with:
    runDeltaTest: true              # Only test changed components
    deltaManifest: results/package.xml
    failTaskOnFailedTests: true     # Fail pipeline on test failures
    checkPerformance: true          # Verify performance thresholds
```

---

## 🐳 Docker Container

Safire runs on the `brovasi/dxb` Docker container, which includes:

- Salesforce CLI (sf)
- DXB Plugin
- Node.js & npm
- Git
- All required dependencies

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Built with [DXB CLI](https://github.com/davidbrowaeys/DXB)
- Powered by [GitHub Actions](https://github.com/features/actions)
- Made for the [Salesforce](https://www.salesforce.com) community

---

<div align="center">
  <p>
    <strong>Made with ❤️ for Salesforce Developers</strong>
  </p>
  <p>
    <a href="https://github.com/davidbrowaeys/SafireForGitHub/issues">Report Bug</a>
    •
    <a href="https://github.com/davidbrowaeys/SafireForGitHub/issues">Request Feature</a>
  </p>
</div>
