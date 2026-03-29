---
name: create-fabric-cicd-deploy-file
description: >
  Use this skill to create deploy.yml configuration files for fabric-cicd projects. 
  Defines environment-specific workspace IDs and item definition file locations used 
  for deploying Fabric artifacts to dev, test, and prod.
---

# Create Fabric CICD Deploy File

## Overview

The `deploy.yml` file configures the fabric-cicd library's deployment targets. It establishes a one-to-one mapping between deployment environments and Fabric workspace IDs, and specifies where artifact definitions (notebooks, semantic models, etc.) are located in your repository.

## Architecture

```
deploy.yml (one per git repo)
├── environments[]:
│   └── name: "dev" / "test" / "prod"
│       ├── workspace_id: "uuid"
│       └── artifact_path: "./path/to/items"
├── item_types[] (definitions of artifacts to deploy)
│   └── name: "Notebook", "SemanticModel", etc.
│       └── path: "./notebooks", "./semantic_models", etc.
```

## MUST DO

1. **One deploy.yml per repository** — Store at the repository root
2. **Define all target environments** — Include dev, test, and prod (or your CI/CD stages)
3. **Use correct Fabric workspace IDs** — Format: UUID (8-4-4-4-12 hex digits)
4. **Consistent environment names** — Match names used in parameter.yml (lowercase: dev, test, prod)
5. **Specify artifact locations** — Path to notebooks, semantic models, dataflows, etc.
6. **Version your configuration** — Include a `version` field for compatibility tracking

## PREFER

- Document where workspace IDs come from (e.g., "Fabric workspace settings" or "Azure Portal")
- Use relative paths for artifact locations (e.g., `./artifacts/notebooks`)
- Order environments logically (dev → test → prod)
- Include comments explaining each workspace's purpose
- Use consistent naming for item types across all environments

## AVOID

- Hardcoding production workspace IDs in unprotected repositories
- Mixing environment names (don't use both `Dev` and `dev`)
- Paths that don't match your repository structure
- Missing or incomplete workspace ID mappings
- Deploying to the wrong workspace due to configuration errors

## Environment Workspace ID Setup

Before creating `deploy.yml`, obtain workspace IDs:

1. **In Fabric UI**:
   - Open Fabric workspace
   - Go to **Settings** → **General** → **Workspace Settings**
   - Copy **Display name** and **Workspace ID** (UUID format)

2. **Store securely**:
   - For CI/CD: Use GitHub Secrets or Azure DevOps Variable Groups
   - For local development: Use environment variables or `.env` (add to `.gitignore`)

3. **Format verification**:
   ```
   Valid UUID: 12345678-1234-1234-1234-123456789012
   ```

## Artifact File Structure

Organize your repository so deploy.yml can locate all artifacts:

```
repository-root/
├── deploy.yml
├── parameter.yml
├── artifacts/
│   ├── notebooks/
│   │   ├── Build 01 Silver Tables.ipynb
│   │   └── Build 02 Gold Tables.ipynb
│   ├── semantic_models/
│   │   └── Sales Analytics/
│   │       └── model.tmdl
│   ├── dataflows/
│   │   └── daily-refresh.json
│   └── reports/
│       ├── sales_dashboard.pbix
│       └── inventory_dashboard.pbix
```

## Example Deploy Files

### Minimal deploy.yml (3 environments)

```yaml
version: "1"
environments:
  - name: dev
    workspace_id: "abcd1234-ab12-cd34-ef56-abcdef123456"
    workspace_name: "Fabric-Dev"
    
  - name: test
    workspace_id: "efgh5678-ef56-gh78-ij90-efghij567890"
    workspace_name: "Fabric-Test"
    
  - name: prod
    workspace_id: "ijkl9012-ij90-kl12-mn34-ijklmn901234"
    workspace_name: "Fabric-Prod"

item_types:
  - name: Notebook
    path: "./artifacts/notebooks"
    
  - name: SemanticModel
    path: "./artifacts/semantic_models"
    
  - name: Report
    path: "./artifacts/reports"
```

### Full deploy.yml (with artifact details)

```yaml
version: "1"

# Deployment environments and target workspace IDs
environments:
  - name: dev
    workspace_id: "abcd1234-ab12-cd34-ef56-abcdef123456"
    workspace_name: "Fabric-Development"
    description: "Development environment for active development and testing"
    
  - name: test
    workspace_id: "efgh5678-ef56-gh78-ij90-efghij567890"
    workspace_name: "Fabric-Testing"
    description: "Testing environment for UAT and integration testing"
    
  - name: prod
    workspace_id: "ijkl9012-ij90-kl12-mn34-ijklmn901234"
    workspace_name: "Fabric-Production"
    description: "Production environment for live data pipelines and reports"

# Item type definitions and their repository paths
item_types:
  - name: Notebook
    display_name: "Notebooks"
    path: "./artifacts/notebooks"
    description: "Fabric notebooks for data transformation and analysis"
    
  - name: SemanticModel
    display_name: "Semantic Models"
    path: "./artifacts/semantic_models"
    description: "DAX semantic models for analysis services"
    
  - name: Report
    display_name: "Reports"
    path: "./artifacts/reports"
    description: "Power BI reports and dashboards"
    
  - name: Dataflow
    display_name: "Dataflows"
    path: "./artifacts/dataflows"
    description: "Data refresh pipelines and transformations"
    
  - name: Lakehouse
    display_name: "Lakehouses"
    path: "./artifacts/lakehouses"
    description: "Data lake house definitions"
    
  - name: Pipeline
    display_name: "Pipelines"
    path: "./artifacts/pipelines"
    description: "Data factory pipelines"

# Global deployment settings (optional)
deployment_settings:
  overwrite_existing: true
  validate_before_deploy: true
  backup_on_deploy: false
  timeout_minutes: 30
```

## Creating Your Deploy File: Step-by-Step

1. **Identify your environments**:
   - Which deployment stages do you have? (dev, test, prod, staging, etc.)
   - Are all workspace IDs already created in Fabric?

2. **Collect workspace IDs**:
   - Access each Fabric workspace
   - Note the UUID from Settings → General

3. **Plan artifact organization**:
   - Where will notebooks live in the repo?
   - Where will semantic models and reports go?
   - Mirror your production structure from day one

4. **Create the YAML structure**:
   - Define each environment with workspace ID
   - List all item types your project uses
   - Point to correct repository paths

5. **Validate paths**:
   - Ensure artifact paths exist and contain your items
   - Test that paths match your `.gitignore` patterns (if applicable)
   - Confirm naming consistency with parameter.yml

6. **Secure sensitive data**:
   - Never commit workspace IDs to public repositories
   - Use GitHub Secrets or Azure DevOps for CI/CD
   - Use environment variables for local development

## Environment Variables (for CI/CD)

Example GitHub Actions workflow integration:

```yaml
# In .github/workflows/deploy.yml
env:
  FABRIC_DEV_WORKSPACE_ID: ${{ secrets.FABRIC_DEV_WORKSPACE_ID }}
  FABRIC_TEST_WORKSPACE_ID: ${{ secrets.FABRIC_TEST_WORKSPACE_ID }}
  FABRIC_PROD_WORKSPACE_ID: ${{ secrets.FABRIC_PROD_WORKSPACE_ID }}

# Then reference in deploy.yml:
# workspace_id: ${{ env.FABRIC_DEV_WORKSPACE_ID }}
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Workspace ID not found | Verify the UUID format and that you have access to the workspace |
| Artifact path doesn't exist | Check path relative to repository root; ensure formatting is correct |
| Environment not recognized | Confirm environment name matches parameter.yml exactly (case-sensitive) |
| Deploy fails to find items | Verify item types in deploy.yml match actual artifact folders |
| Credentials not working | Ensure CI/CD secrets are configured with correct workspace IDs |

## See Also

- [Create Fabric CICD Parameter File skill](/skills/create-fabric-cicd-parameter-file/) — Define environment-specific substitutions
- [Set Up Git Repository for Fabric CICD](/skills/setup-fabric-cicd-git-repo/) — Initialize a new Git repository for fabric-cicd
