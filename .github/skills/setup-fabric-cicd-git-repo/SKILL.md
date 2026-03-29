---
name: setup-fabric-cicd-git-repo
description: >
  Use this skill to initialize and configure a Git repository for a Fabric CI/CD project. 
  Sets up folder structure, configuration files, GitHub Actions workflows, and best-practice 
  documentation for your fabric-cicd deployment pipeline.
---

# Set Up Git Repository for Fabric CICD

## Overview

This skill guides you through creating a new Git repository configured for the fabric-cicd library. It establishes the proper folder structure, initializes configuration templates, and optionally creates GitHub Actions workflows for automated deployments.

## Prerequisites

- Git installed locally
- GitHub or Azure DevOps account
- Access to at least one Fabric workspace (for workspace ID collection)
- Basic familiarity with Git workflows

## Repository Structure

```
fabric-project-repository/
├── .github/
│   ├── workflows/               # CI/CD automation
│   │   └── deploy.yml           # GitHub Actions deployment workflow
│   ├── skills/                  # Copilot skills (optional)
│   │   ├── create-fabric-cicd-parameter-file/
│   │   └── create-fabric-cicd-deploy-file/
│   ├── copilot-instructions.md  # Project-level copilot guidance
│   └── AGENTS.md                # Custom agent definitions
├── artifacts/                   # All Fabric artifacts organized by type
│   ├── notebooks/               # Fabric notebooks (.ipynb)
│   ├── semantic_models/         # Semantic models (TMDL format)
│   ├── reports/                 # Power BI reports (.pbix)
│   ├── dataflows/               # Dataflow definitions (.json)
│   ├── lakehouses/              # Lakehouse configurations
│   └── pipelines/               # Data factory pipelines
├── docs/                        # Project documentation
│   ├── ARCHITECTURE.md          # System design and data flow
│   ├── SETUP.md                 # Local environment setup
│   └── DEPLOYMENT.md            # Deployment procedures
├── deploy.yml                   # Workspace ID mappings and artifact paths
├── parameter.yml                # Environment-specific substitutions
├── .gitignore                   # Git ignore rules
├── README.md                    # Project overview
└── LICENSE                      # License file
```

## Step 1: Initialize Repository

### Option A: Create New Repository (GitHub)

```bash
# Create locally
mkdir fabric-cicd-project
cd fabric-cicd-project
git init

# Create on GitHub, then add remote
git remote add origin https://github.com/YOUR-ORG/fabric-cicd-project.git
git branch -M main
```

### Option B: Clone Existing Template

```bash
git clone https://github.com/microsoft/fabric-cicd-best-practices.git
cd fabric-cicd-best-practices
git remote set-url origin https://github.com/YOUR-ORG/your-project.git
```

## Step 2: Create Folder Structure

```bash
# Create artifact directories
mkdir -p artifacts/{notebooks,semantic_models,reports,dataflows,lakehouses,pipelines}
mkdir -p docs
mkdir -p .github/workflows
```

## Step 3: Initialize Configuration Files

### Create .gitignore

```
# Python / Jupyter
__pycache__/
*.py[cod]
*$py.class
*.ipynb_checkpoints/
.ipynb
*.egg-info/
dist/
build/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Environment / Secrets
.env
.env.local
.env.*.local
secrets/
credentials/

# OS
.DS_Store
Thumbs.db

# Fabric / Power BI
*.pbix.tmp
.pbix.backup/

# Build artifacts
*.tmp
*.log
```

### Create README.md

```markdown
# Fabric CI/CD Project

Brief project description and business context.

## Quick Start

### Prerequisites
- Git installed
- Fabric workspace access
- GitHub Actions runner configured (if using GitHub)

### Setup

1. Clone this repository:
   \`\`\`bash
   git clone https://github.com/YOUR-ORG/fabric-cicd-project.git
   \`\`\`

2. Install dependencies:
   \`\`\`bash
   pip install pyramid-fabric-cicd
   \`\`\`

3. Configure environments:
   - Copy \`deploy.yml\` template and update workspace IDs
   - Copy \`parameter.yml\` template and adjust for your artifacts

4. Deploy to dev:
   \`\`\`bash
   fabric-cicd deploy -e dev
   \`\`\`

## Project Structure

- **artifacts/** - Fabric notebooks, semantic models, reports
- **deploy.yml** - Environment and workspace mapping
- **parameter.yml** - Environment-specific substitutions
- **.github/workflows/** - CI/CD automation (GitHub Actions)
- **docs/** - Architecture, setup, and deployment guides

## Documentation

- [Architecture](./docs/ARCHITECTURE.md) - System design
- [Setup Guide](./docs/SETUP.md) - Local environment
- [Deployment Guide](./docs/DEPLOYMENT.md) - Release procedures

## Environments

- **dev** - Development and testing
- **test** - User acceptance testing (UAT)
- **prod** - Production deployment

## Contributing

[Your contribution guidelines]

## License

[Your license]
```

## Step 4: Create Configuration Templates

### Create deploy.yml (template)

```yaml
version: "1"

environments:
  - name: dev
    workspace_id: "{{ DEV_WORKSPACE_ID }}"
    workspace_name: "Fabric-Dev"
    
  - name: test
    workspace_id: "{{ TEST_WORKSPACE_ID }}"
    workspace_name: "Fabric-Test"
    
  - name: prod
    workspace_id: "{{ PROD_WORKSPACE_ID }}"
    workspace_name: "Fabric-Prod"

item_types:
  - name: Notebook
    path: "./artifacts/notebooks"
    
  - name: SemanticModel
    path: "./artifacts/semantic_models"
    
  - name: Report
    path: "./artifacts/reports"
```

### Create parameter.yml (template)

```yaml
version: "1"

find_replace:
  # Add your environment-specific substitution rules here
  # Example:
  # - find_value: '#\s*META\s+"default_lakehouse_workspace_id":\s+"([0-9a-fA-F-]{36})"'
  #   replace_value:
  #     dev: "$workspace.$id"
  #     test: "$workspace.$id"
  #     prod: "$workspace.$id"
  #   is_regex: true
  #   item_type: ["Notebook"]
```

## Step 5: Set Up GitHub Actions (Optional)

### Create .github/workflows/deploy.yml

```yaml
name: Deploy to Fabric

on:
  push:
    branches:
      - main
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'dev'
        type: choice
        options:
          - dev
          - test
          - prod

env:
  FABRIC_CLI_VERSION: "1.0.0"

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install fabric-cicd
        run: |
          pip install pyramid-fabric-cicd

      - name: Deploy to Fabric
        env:
          FABRIC_SERVICE_PRINCIPAL_ID: ${{ secrets.FABRIC_SERVICE_PRINCIPAL_ID }}
          FABRIC_SERVICE_PRINCIPAL_SECRET: ${{ secrets.FABRIC_SERVICE_PRINCIPAL_SECRET }}
          FABRIC_TENANT_ID: ${{ secrets.FABRIC_TENANT_ID }}
        run: |
          fabric-cicd deploy \
            -e ${{ github.event.inputs.environment || 'dev' }} \
            --deploy-config deploy.yml \
            --param-config parameter.yml

      - name: Verify deployment
        run: |
          echo "✅ Deployment completed successfully"
```

## Step 6: Configure Secrets (GitHub Actions)

1. Go to repository **Settings** → **Secrets and variables** → **Actions**
2. Add these secrets:

```
FABRIC_SERVICE_PRINCIPAL_ID      # Azure AD app registration ID
FABRIC_SERVICE_PRINCIPAL_SECRET  # Client secret
FABRIC_TENANT_ID                 # Azure tenant ID
FABRIC_DEV_WORKSPACE_ID          # Dev workspace UUID
FABRIC_TEST_WORKSPACE_ID         # Test workspace UUID
FABRIC_PROD_WORKSPACE_ID         # Prod workspace UUID
```

## Step 7: Add Copilot Skills (Optional)

Copy skills to your repository:

```bash
cp -r .github/skills/create-fabric-cicd-parameter-file .github/skills/
cp -r .github/skills/create-fabric-cicd-deploy-file .github/skills/
```

Create `.github/copilot-instructions.md`:

```markdown
# Project-Level Copilot Guidelines

This project uses fabric-cicd for CI/CD deployment to Microsoft Fabric.

## Available Skills

- `/fabric-skills:create-fabric-cicd-parameter-file` - Create parameter.yml configuration
- `/fabric-skills:create-fabric-cicd-deploy-file` - Create deploy.yml configuration

## Key Files

- `deploy.yml` - Workspace ID mappings
- `parameter.yml` - Environment substitution rules
- `.github/workflows/` - Automated deployment pipelines

## Workflows

### Local Development
1. Develop in dev workspace
2. Commit changes to feature branch
3. Open pull request for review

### Deployment
1. Merge to main branch
2. GitHub Actions automatically deploys to dev
3. Manual approval in Actions UI to deploy to test/prod
```

## MUST DO

1. **Never commit workspace IDs to the repository** — Use GitHub Secrets or environment variables
2. **Keep deploy.yml and parameter.yml in sync** — Both must reference the same environments
3. **Test in dev before prod** — Always validate changes in development first
4. **Document artifact structure** — Ensure others understand folder layouts
5. **Set appropriate branch protection** — Require reviews before merging to main

## PREFER

- Use a service principal for CI/CD authentication
- Enable branch protection on main/production branches
- Document workspace purposes and team responsibilities
- Keep configuration templates updated with team changes
- Use semantic versioning for releases

## AVOID

- Storing credentials in code or configuration files
- Deploying directly to production without testing
- Large binary files in Git (use Git LFS for .pbix files if needed)
- Committing temporary or transient artifacts
- Circular dependencies between artifacts

## Verification Checklist

- [ ] Repository initialized with main branch
- [ ] Folder structure created (artifacts/, docs/, .github/)
- [ ] .gitignore configured
- [ ] README.md written with project overview
- [ ] deploy.yml template created and workspace IDs documented externally
- [ ] parameter.yml template created with sample rules
- [ ] GitHub Secrets configured (if using Actions)
- [ ] GitHub Actions workflow tested in dev environment
- [ ] .github/copilot-instructions.md added
- [ ] Created initial .gitignore and committed to main

## Next Steps

1. **Onboard team members**:
   - Share repository access
   - Brief them on deploy.yml and parameter.yml
   - Point them to this setup guide

2. **Start artifact development**:
   - Create notebooks in artifacts/notebooks/
   - Add semantic models to artifacts/semantic_models/
   - Store reports in artifacts/reports/

3. **Run first deployment**:
   - Validate deploy.yml is correct
   - Run fabric-cicd deploy command
   - Verify artifacts appear in target workspace

4. **Iterate and refine**:
   - Update parameter.yml as needed
   - Test deployment pipeline end-to-end
   - Add team documentation

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Authentication fails | Verify service principal secret in GitHub Secrets |
| Deployment paths not found | Check artifact folder paths match deploy.yml |
| Workspace ID mismatch | Confirm workspace IDs from Fabric Settings match deploy.yml |
| GitHub Actions not triggering | Check branch name and workflow trigger conditions |
| Parameter substitution fails | Validate regex patterns in parameter.yml against source code |

## See Also

- [Create Fabric CICD Parameter File skill](./skills/create-fabric-cicd-parameter-file/) — Define substitution rules
- [Create Fabric CICD Deploy File skill](./skills/create-fabric-cicd-deploy-file/) — Configure deployment targets
- [Fabric CICD Documentation](https://github.com/microsoft/Fabric-CICD) — Official library docs
