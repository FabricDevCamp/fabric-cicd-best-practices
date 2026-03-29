---
name: fabric-cicd-workspace-instructions
applyTo: "**"
---

# Fabric CICD Best Practices — Project Guide

This workspace teaches best practices for implementing CI/CD pipelines with Microsoft Fabric and the fabric-cicd library. It includes skills, documentation, and examples to help you set up, configure, and deploy Fabric workspaces programmatically.

## 🎯 Project Purpose

**Goal**: Help teams implement enterprise-grade CI/CD workflows for Microsoft Fabric projects using Git-based version control and automated deployments.

**Audience**: Intermediate to advanced Fabric users, data engineers, and DevOps professionals.

## 📚 Available Skills

Use these Copilot skills to accelerate your setup:

### 1. **Set Up Git Repository for Fabric CICD**
- **Use when**: Starting a new Fabric CI/CD project from scratch
- **Creates**: Initial repository structure, workflows, and configuration templates
- **Command**: `/fabric-skills:setup-fabric-cicd-git-repo`

### 2. **Create Fabric CICD Deploy File**
- **Use when**: Defining which workspaces to deploy to (dev, test, prod)
- **Creates**: `deploy.yml` file with environment mappings and workspace IDs
- **Command**: `/fabric-skills:create-fabric-cicd-deploy-file`

### 3. **Create Fabric CICD Parameter File**
- **Use when**: Setting up environment-specific substitutions (workspace IDs, lakehouse names, etc.)
- **Creates**: `parameter.yml` file with find-and-replace rules
- **Command**: `/fabric-skills:create-fabric-cicd-parameter-file`

## 📁 Key Files in This Project

```
.github/
├── skills/                                  # Copilot skills for templates and guidance
│   ├── setup-fabric-cicd-git-repo/SKILL.md
│   ├── create-fabric-cicd-deploy-file/SKILL.md
│   └── create-fabric-cicd-parameter-file/SKILL.md
├── workflows/                               # GitHub Actions automation (examples)
└── copilot-instructions.md                  # This file

deploy.yml                                   # Template: Environment and workspace mapping
parameter.yml                                # Template: Environment-specific substitutions
README.md                                    # Project overview
```

## 🚀 Quick Start (New Project)

1. **Initialize repository**: `/fabric-skills:setup-fabric-cicd-git-repo`
   - Sets up folder structure and templates

2. **Create deploy.yml**: `/fabric-skills:create-fabric-cicd-deploy-file`
   - Define your dev, test, prod workspaces

3. **Create parameter.yml**: `/fabric-skills:create-fabric-cicd-parameter-file`
   - Add substitution rules for environment-specific values

4. **Commit to Git**: Push files to your repository

5. **Deploy**: Run `fabric-cicd deploy -e dev` to test

## 🔑 Core Concepts

### deploy.yml
Maps deployment environments to Fabric workspace IDs. Answers: *Where* do we deploy?

```yaml
environments:
  - name: dev
    workspace_id: "abc123..."
  - name: prod
    workspace_id: "def456..."
```

### parameter.yml
Defines regex-based substitution rules for environment-specific values. Answers: *What* values change per environment?

```yaml
find_replace:
  - find_value: "hardcoded_workspace_id_xxx"
    replace_value:
      dev: "$workspace.$id"
      prod: "$workspace.$id"
```

### Artifacts
Your Fabric items (notebooks, semantic models, reports) organized in Git:

```
artifacts/
├── notebooks/
├── semantic_models/
└── reports/
```

## 💡 Common Workflows

### Deploy to Dev
```bash
fabric-cicd deploy -e dev
```

### Deploy to Production
```bash
fabric-cicd deploy -e prod --validate
```

### Test Parameter Substitutions
```bash
fabric-cicd validate --param-config parameter.yml
```

### View Environment Mappings
```bash
fabric-cicd show-environments --deploy-config deploy.yml
```

## ❓ When to Use Each Skill

| Scenario | Use This Skill |
|----------|----------------|
| Starting a brand-new Fabric CI/CD project | `setup-fabric-cicd-git-repo` |
| You have notebooks/models but no CI/CD yet | `setup-fabric-cicd-git-repo` → `create-fabric-cicd-deploy-file` |
| Adding new environments (staging, pre-prod) | `create-fabric-cicd-deploy-file` |
| Notebooks have hardcoded workspace IDs | `create-fabric-cicd-parameter-file` |
| Semantic models reference specific lakehouses | `create-fabric-cicd-parameter-file` |
| Setting up GitHub Actions automation | `setup-fabric-cicd-git-repo` (includes workflow examples) |

## 📖 Documentation

- **[Fabric-CICD Library Docs](https://github.com/microsoft/Fabric-CICD)** — Official fabric-cicd repository
- **[Parameter.yml Deep Dive](../skills/create-fabric-cicd-parameter-file/SKILL.md)** — Detailed reference for substitution rules
- **[Deploy.yml Reference](../skills/create-fabric-cicd-deploy-file/SKILL.md)** — Complete deploy configuration guide
- **[Repository Setup Guide](../skills/setup-fabric-cicd-git-repo/SKILL.md)** — Step-by-step project initialization

## 🔒 Security Best Practices

### Never commit workspace IDs to public repositories
- Use GitHub Secrets to store workspace IDs
- Use Azure DevOps Variable Groups for pipelines
- Use environment variables for local development

### Protect production deployments
- Require approvals before deploying to prod
- Use branch protection on main branch
- Audit and log all deployments

### Service Principal Setup
- Create Azure AD app registration for CI/CD
- Store client secret in GitHub Secrets securely
- Rotate secrets quarterly

Example GitHub Secret names:
```
FABRIC_SERVICE_PRINCIPAL_ID
FABRIC_SERVICE_PRINCIPAL_SECRET
FABRIC_TENANT_ID
FABRIC_DEV_WORKSPACE_ID
FABRIC_TEST_WORKSPACE_ID
FABRIC_PROD_WORKSPACE_ID
```

## ❓ Frequently Asked Questions

**Q: Can I use this with Azure DevOps instead of GitHub?**

A: Yes! The core concepts (deploy.yml, parameter.yml, fabric-cicd library) are platform-agnostic. Adapt the workflow examples for Azure Pipelines syntax.

**Q: How do I handle secrets in parameter.yml?**

A: Use environment variables or GitHub Secrets directly in CI/CD. Don't store secrets in parameter.yml; use substitution variables like `${{ env.SECRET_NAME }}`.

**Q: What if my workspace IDs change?**

A: Update deploy.yml with new IDs, commit to Git, and re-run deployments. CI/CD will use the updated IDs.

**Q: Can I deploy a single notebook instead of everything?**

A: Yes! Use fabric-cicd's filtering options: `fabric-cicd deploy -e dev --filter "item_type:Notebook" --filter "item_name:MyNotebook"`

**Q: How do I test parameter substitutions locally?**

A: Run `fabric-cicd validate --param-config parameter.yml` to test regex patterns before committing.

## 🤝 Contributing

To improve these skills or documentation:

1. Fork this repository
2. Create a feature branch
3. Submit a pull request with improvements
4. Include examples or test cases

## 📞 Support

- **fabric-cicd Issues**: [GitHub Issues](https://github.com/microsoft/Fabric-CICD/issues)
- **Fabric Docs**: [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
- **Copilot Skills**: Ask in VS Code with `@workspace` or `/`

## 🎓 Learning Path

1. **Beginner**: Read the README, watch fabric-cicd demo
2. **Intermediate**: Set up a dev project using skills, commit to Git
3. **Advanced**: Implement multi-environment deployments, GitHub Actions automation
4. **Expert**: Customize parameter rules, build monitoring/validation

---

**Last Updated**: March 2026  
**fabric-cicd Version**: 1.0.0+  
**License**: [Your License Here]
