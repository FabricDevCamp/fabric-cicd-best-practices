---
name: create-fabric-cicd-parameter-file
description: >
  Use this skill to create parameter.yml configuration files for fabric-cicd projects. 
  Generates find-and-replace rules to substitute environment-specific values (workspace IDs, 
  lakehouse references, SQL endpoints) across notebooks, semantic models, and other artifacts during deployment.
---

# Create Fabric CICD Parameter File

## Overview

The `parameter.yml` file is the core configuration for the fabric-cicd library's parameterization engine. It defines regex-based find-and-replace rules that automatically substitute environment-specific values (such as workspace IDs and lakehouse names) when deploying artifacts to dev, test, and prod environments.

## Architecture

```
parameter.yml (one per git repo)
├── find_replace[] (array of substitution rules)
│   ├── find_value (regex pattern to match in source code)
│   ├── replace_value (environment-specific replacements)
│   │   ├── dev: "value"
│   │   ├── test: "value"
│   │   └── prod: "value"
│   ├── is_regex (true/false)
│   ├── item_type (Notebook, SemanticModel, etc. - optional)
│   ├── item_name (specific item name - optional)
│   └── file_path (glob pattern - optional)
```

## MUST DO

1. **One parameter.yml per repository** — Store at the repository root or in a designated config folder
2. **Use regex patterns for flexibility** — Match hardcoded values that vary by environment
3. **Test regex patterns** — Validate find_value patterns don't over-match or miss targets
4. **Environment-consistent naming** — Use `dev`, `test`, `prod` (lowercase) or match your deploy.yml environments
5. **Reference substitution syntax** — Use fabric-cicd variables:
   - `$workspace.$id` — Target workspace ID (from deploy.yml)
   - `$items.<ItemType>.<ItemName>.$id` — Item ID by type and name
   - `$items.<ItemType>.<ItemName>.$sqlendpoint` — SQL endpoint for items
6. **Filter by context** — Use `item_type`, `item_name`, and `file_path` to apply rules only where needed

## PREFER

- Start with workspace ID and lakehouse ID substitutions (most common)
- Use `item_type` filters to avoid unintended replacements in unrelated files
- Document the source value (hardcoded ID) in comments or a reference table
- Group related substitution rules together
- Use `is_regex: "true"` when matching flexible patterns; `false` for exact string matching

## AVOID

- Overly broad regex patterns that match unintended code
- Missing environment configurations (use `_ALL_` only if truly identical across environments)
- Regex patterns without proper escaping (`\s`, `\d`, etc.)
- Hardcoded values that don't get replaced (breaks reproducibility)

## Common Parameterization Patterns

### Pattern 1: Workspace ID in Notebook Comments
**Source**: Notebook with hardcoded workspace ID in META comment
```
# META "default_lakehouse_workspace_id": "12345678-1234-1234-1234-123456789012"
```

**Rule**:
```yaml
- find_value: '#\s*META\s+"default_lakehouse_workspace_id":\s+"([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})"'
  replace_value:
    dev: "$workspace.$id"
    test: "$workspace.$id"
    prod: "$workspace.$id"
  is_regex: true
  item_type: ["Notebook"]
```

### Pattern 2: Lakehouse ID with Item-Specific Targeting
**Source**: References to specific lakehouse IDs in notebooks
```
# META "default_lakehouse": "87654321-4321-4321-4321-210987654321"
```

**Rule**:
```yaml
- find_value: '#\s*META\s+"default_lakehouse":\s+"([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})"'
  replace_value:
    dev: "$items.Lakehouse.sales_silver.$id"
    test: "$items.Lakehouse.sales_silver.$id"
    prod: "$items.Lakehouse.sales_silver.$id"
  is_regex: true
  item_type: ["Notebook"]
  item_name: "Build 01 Silver Tables"
```

### Pattern 3: SQL Endpoint in Semantic Model
**Source**: Direct SQL endpoint references in TMDL expressions
```
Sql.Database("Server=fab-dw.datawarehouse.fabric.microsoft.com;Database=...")
```

**Rule**:
```yaml
- find_value: 'Sql\.Database\(\s*"([^"]*datawarehouse\.fabric\.microsoft\.com[^"]*)"\s*,'
  replace_value:
    _ALL_: "$items.Lakehouse.sales.$sqlendpoint"
  is_regex: true
  item_type: ["SemanticModel"]
  file_path: "**/expressions.tmdl"
```

## Step-by-Step: Creating a Parameter File

1. **Collect source values** from your artifact files:
   - Search notebooks for hardcoded workspace/lakehouse IDs
   - Scan semantic models for SQL endpoint references
   - Document which environments have which values

2. **Create regex patterns** for each substitution:
   - Use anchors (`^`, `$`) for precision
   - Escape special regex characters (`.`, `*`, `?`, etc.)
   - Test the pattern against actual code samples

3. **Map environments**:
   - Reference your `deploy.yml` for workspace IDs by environment
   - Use `$items.<Type>.<Name>.$id` syntax for fabric-cicd artifact substitution

4. **Add filters**:
   - Use `item_type` to limit rule scope (e.g., only Notebooks)
   - Use `item_name` for specific artifacts
   - Use `file_path` for TMDL or specific file patterns

5. **Validate**:
   - Ensure all environments (dev, test, prod) have values
   - Test regex on sample code to confirm matches
   - Verify no unintended replacements occur

## Example: Complete Parameter.yml

```yaml
version: "1"
find_replace:
  # Notebook workspace ID references
  - find_value: '#\s*META\s+"default_lakehouse_workspace_id":\s+"([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})"'
    replace_value:
      dev: "$workspace.$id"
      test: "$workspace.$id"
      prod: "$workspace.$id"
    is_regex: true
    item_type: ["Notebook"]

  # Lakehouse ID for silver layer
  - find_value: '#\s*META\s+"default_lakehouse":\s+"([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})"'
    replace_value:
      dev: "$items.Lakehouse.sales_silver.$id"
      test: "$items.Lakehouse.sales_silver.$id"
      prod: "$items.Lakehouse.sales_silver.$id"
    is_regex: true
    item_type: ["Notebook"]
    item_name: "Build 01 Silver Tables"

  # Lakehouse ID for gold layer
  - find_value: '#\s*META\s+"default_lakehouse":\s+"([0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12})"'
    replace_value:
      dev: "$items.Lakehouse.sales_gold.$id"
      test: "$items.Lakehouse.sales_gold.$id"
      prod: "$items.Lakehouse.sales_gold.$id"
    is_regex: true
    item_type: ["Notebook"]
    item_name: "Build 02 Gold Tables"

  # SQL endpoint for semantic model
  - find_value: 'Sql\.Database\(\s*"([^"]*datawarehouse\.fabric\.microsoft\.com[^"]*)"\s*,'
    replace_value:
      _ALL_: "$items.Lakehouse.sales.$sqlendpoint"
    is_regex: true
    item_type: ["SemanticModel"]
    file_path: "**/expressions.tmdl"
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Rule doesn't match | Verify regex pattern with online regex tester; check for escaped special chars |
| Unwanted replacements | Add `item_type` or `item_name` filters; make regex pattern more specific |
| Missing environment | Ensure all target environments (dev/test/prod) have values in `replace_value` |
| Substitution variables not resolved | Confirm item names and types match your `deploy.yml` artifact definitions |

