---
layout: post
title: "From simple to sophisticated: Terraform infrastructure evolution"
description: "How I transformed a basic Terraform configuration into a sophisticated infrastructure-as-code solution with modules, testing, CI/CD, and governance."
date: 2025-06-16 08:00:00 +0300
author: hidde
image: '/images/terraform-evolution.png'
tags: [Terraform, GitHub, CI/CD, Azure, IaC]
featured: false
toc: true
---

When preparing samples for a Terraform training I was giving, I wanted to demonstrate how infrastructure code can evolve from simple beginnings through incremental improvements. Rather than starting with a complex example, I decided to build a project that shows the natural progression most teams experience, beginning with basic functionality and gradually adding sophisticated features like testing, automation, and governance.

This post shares the insights and lessons learned from creating those training samples. It's designed to help teams understand not just the "what" but the "why" behind each improvement, providing a roadmap for elevating your own Terraform practices. Whether you're just starting with Infrastructure as Code or looking to mature your existing setup, this evolution demonstrates practical steps you can take to build more reliable, maintainable infrastructure.

## The starting point: simple but limited

My initial Terraform configuration was straightforward but had significant limitations that became apparent as the needs grew:

### Version 0.1.0: The basic foundation

I started with a monolithic `main.tf` file containing all the Azure resources:

- Resource Group
- Virtual Network and Subnet  
- Network Security Group
- Storage Account and Container
- App Service Plan and Linux Web App
- Key Vault

**The pain points:**
- Single massive file with all resources
- No reusability across environments
- Manual deployment process prone to errors
- No standardized naming conventions
- Limited documentation and change tracking

While this got me started quickly, I knew it wouldn't scale as the infrastructure requirements grew.

## Evolution phase 1: breaking down the monolith

### Version 0.2.0: Modular architecture

The first major change was breaking the monolithic configuration into logical, reusable modules:

```
modules/
├── network/     # VNet, subnet, NSG
├── storage/     # Storage account and containers
├── webapp/      # App Service Plan and Web App
└── keyvault/    # Key Vault resources
```

**Key improvements:**
- **Reusability**: Modules could be used across multiple environments
- **Maintainability**: Isolated components for easier debugging
- **Collaboration**: Teams could work on different modules simultaneously
- **Testing**: Individual modules could be tested in isolation

**Lesson learned:** Modularization from the start saves significant refactoring time later. Even if you're starting small, think about logical boundaries for your resources.

## Evolution phase 2: standardization and governance

### Version 0.3.0: Naming conventions and environment separation

As the infrastructure grew, inconsistent naming became a problem. I implemented:

- **Naming module**: Standardized patterns for all Azure resources
- **Environment separation**: Dedicated `dev.tfvars` and `prod.tfvars` files
- **Terraform workspaces**: Proper state separation between environments

Here's how the naming module evolved to handle Azure's complex naming requirements:

```hcl
# modules/naming/main.tf
locals {
  # Resource abbreviations following Azure CAF
  resource_type_abbreviations = {
    resource_group         = "rg"
    virtual_network        = "vnet"
    subnet                 = "snet"
    network_security_group = "nsg"
    storage_account        = "st"
    storage_container      = "stcont"
    app_service_plan       = "asp"
    web_app                = "app"
    key_vault              = "kv"
  }

  # Standard naming pattern: prefix-abbreviation-environment-suffix
  resource_group_name = var.resource_group != "" ? 
    var.resource_group : 
    "${var.prefix}-${local.resource_type_abbreviations.resource_group}-${var.environment}-${var.suffix}"
}

# Special naming for storage accounts (no dashes, lowercase only)
resource "null_resource" "storage_account_name" {
  triggers = {
    name = "${var.prefix}st${var.environment}${var.suffix}"
  }
}

# Special naming for containers (lowercase with hyphens allowed)
resource "null_resource" "storage_container_name" {
  triggers = {
    name = lower("${var.prefix}-stcont-${var.environment}-${var.suffix}")
  }
}
```

Example usage in main configuration:
```hcl
# main.tf
module "naming" {
  source = "./modules/naming"

  prefix       = var.prefix
  environment  = local.environment
  suffix       = var.suffix
  project_name = var.project_name
}

resource "azurerm_resource_group" "main" {
  name     = module.naming.resource_group_name
  location = var.location
  tags     = module.naming.common_tags
}
```

### Version 0.4.0: Validation and comprehensive tagging

I added two critical governance modules:

**Validation module:**
- Azure resource name compliance checks
- Length constraints and character restrictions
- Environment-specific validation rules

```hcl
# modules/validation/main.tf
locals {
  # Maximum length validation for Azure resources
  max_length = {
    resource_group_name    = 90
    storage_account_name   = 24
    key_vault_name         = 24
    web_app_name           = 60
    virtual_network_name   = 64
    subnet_name            = 80
    nsg_name               = 80
    storage_container_name = 63
    app_service_plan_name  = 40
  }

  # Storage account specific validation (lowercase letters and numbers only)
  validate_storage_account_chars = can(regex("^[a-z0-9]+$", var.storage_account_name))

  # Overall validation result
  is_valid = local.validate_resource_group_name && 
             local.validate_storage_account_name && 
             local.validate_storage_account_chars &&
             # ... other validations
}
```

**Tagging module:**
- Standardized tags across all resources (Environment, Owner, Cost Center)
- Automatic timestamp and Terraform version tracking
- Compliance with organizational tagging policies

```hcl
# modules/tagging/main.tf
locals {
  # Standard tags applied to all resources
  common_tags = merge(
    var.custom_tags,
    {
      Environment    = var.environment
      ManagedBy      = "terraform"
      Project        = var.project_name
      CostCenter     = var.cost_center
      Owner          = var.owner
      CreatedDate    = formatdate("YYYY-MM-DD", timestamp())
      TerraformVersion = "1.12.1"
    }
  )
}

output "tags" {
  description = "Common tags to be applied to all resources"
  value       = local.common_tags
}
```

**Impact:** These changes transformed the infrastructure from ad-hoc deployments to a governed, auditable system that met enterprise requirements.

## Workspace vs environment separation strategies

One of the critical decisions during the evolution was how to handle environment separation. I experimented with different approaches and learned valuable lessons about their trade-offs:

### Strategy 1: Terraform workspaces (my initial approach)

```bash
# Environment switching with workspaces
terraform workspace new dev
terraform workspace new prod

# Deploy to development
terraform workspace select dev
terraform apply -var-file="environments/dev.tfvars"

# Deploy to production  
terraform workspace select prod
terraform apply -var-file="environments/prod.tfvars"
```

**Pros:**
- Simple to implement initially
- Single codebase for all environments
- Built-in Terraform feature

**Cons:**
- Shared state file increases blast radius
- Human error risk when switching workspaces
- Difficult to implement different approval workflows per environment
- Limited isolation for security and compliance

### Strategy 2: Separate directories (current recommendation)

```
environments/
├── dev/
│   ├── main.tf
│   ├── variables.tf
│   ├── terraform.tfvars
│   └── backend.tf
└── prod/
    ├── main.tf
    ├── variables.tf
    ├── terraform.tfvars
    └── backend.tf
```

**Pros:**
- Complete state isolation
- Environment-specific configurations possible
- Clear separation for CI/CD pipelines
- Better security and access control

**Cons:**
- Code duplication between environments
- More complex maintenance
- Requires discipline to keep environments in sync

**Key lesson:** Start with workspaces for simplicity, but plan migration to separate backends as security and compliance requirements grow.

## Evolution phase 3: automation and CI/CD

### Version 0.5.0: GitHub Actions integration

Manual deployment was becoming a bottleneck and risk. I implemented comprehensive CI/CD:

**Branch-based strategy:**
- `develop` branch → Development environment
- `main` branch → Production environment  
- Feature branches → PR validation only

**Security features:**
- Azure Service Principal authentication
- GitHub environment protection rules
- Automated plan generation and review
- Manual workflow dispatch for emergency operations

Here's the GitHub Actions workflow structure:

```yaml
# .github/workflows/terraform-deploy.yml
name: 'Terraform Deploy'

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'dev'
        type: choice
        options: [dev, prod]
      action:
        description: 'Terraform action to perform'
        required: true
        default: 'plan'
        type: choice
        options: [plan, apply, destroy]

env:
  TF_VERSION: '1.12.1'
  ARM_CLIENT_ID: {% raw %}${{ secrets.AZURE_CLIENT_ID }}{% endraw %}
  ARM_CLIENT_SECRET: {% raw %}${{ secrets.AZURE_CLIENT_SECRET }}{% endraw %}
  ARM_SUBSCRIPTION_ID: {% raw %}${{ secrets.AZURE_SUBSCRIPTION_ID }}{% endraw %}
  ARM_TENANT_ID: {% raw %}${{ secrets.AZURE_TENANT_ID }}{% endraw %}

jobs:
  terraform-check:
    name: 'Terraform Check'
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: {% raw %}${{ env.TF_VERSION }}{% endraw %}
    
    - name: Terraform Format Check
      run: terraform fmt -check -recursive
    
    - name: Terraform Validation
      run: |
        terraform init -backend=false
        terraform validate
```

**Before vs. after:**

*Manual process:*
```bash
terraform workspace select dev
terraform init
terraform plan -var-file="environments/dev.tfvars"
terraform apply -var-file="environments/dev.tfvars"
```

*Automated process:*
```bash
git checkout develop
git add .
git commit -m "Update infrastructure configuration"
git push origin develop  # Automatically deploys to dev
```

**Results:** Deployment time reduced from 20+ minutes of manual work to seconds of automated execution, with built-in approval workflows for production.

## Evolution phase 4: comprehensive testing

### Version 0.6.0: Terratest implementation

Testing infrastructure code was the final major hurdle. I implemented a comprehensive testing framework using Terratest:

**Test categories implemented:**

1. **Validation tests** (< 1 second): Fast syntax and configuration validation
2. **Module tests** (< 1 minute): Individual component isolation testing  
3. **Infrastructure tests** (< 30 minutes): Full end-to-end deployment verification
4. **Naming convention tests** (< 1 second): Resource naming standard compliance

**Development workflow enhancement:**

I created a Makefile for standardized development:

```bash
# Makefile targets for development workflow
make test           # Quick validation (recommended for TDD)
make test-all       # Complete test suite
make test-modules   # Individual module tests
make test-infrastructure  # Full deployment tests
```

Here's an example of the Terratest implementation:

```go
// test/terraform_modules_test.go
func TestNamingConventions(t *testing.T) {
    t.Parallel()

    terraformOptions := &terraform.Options{
        TerraformDir: "../modules/naming",
        Vars: map[string]interface{}{
            "prefix":       "test",
            "environment":  "dev",
            "suffix":       "001",
            "project_name": "terraform-course",
        },
    }

    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)

    // Test resource group naming convention
    resourceGroupName := terraform.Output(t, terraformOptions, "resource_group_name")
    assert.Contains(t, resourceGroupName, "test-rg-dev-001")

    // Test storage account naming (no dashes, lowercase)
    storageAccountName := terraform.Output(t, terraformOptions, "storage_account_name")
    assert.Regexp(t, "^[a-z0-9]+$", storageAccountName)
}

func TestValidationModule(t *testing.T) {
    terraformOptions := &terraform.Options{
        TerraformDir: "../modules/validation",
        Vars: map[string]interface{}{
            "storage_account_name": "invalidSTORAGEname123!", // Should fail
        },
    }

    // Test that validation catches invalid names
    _, err := terraform.InitAndPlanE(t, terraformOptions)
    assert.Error(t, err, "Expected validation to fail for invalid storage account name")
}
```

**Real-world impact:** During testing implementation, we discovered and fixed Azure storage container naming compliance issues that would have caused production failures.

## The advanced features: beyond the basics

### Current state: enterprise-ready infrastructure

The latest iteration includes advanced features that provide a foundation for enterprise-level operations:

**Policy as code:**
- OPA policy definitions for security and tagging compliance
- Python script for validating Terraform plans against policies
- Security policies for storage accounts and key vaults

```hcl
# Example from policies/security.rego
package terraform.security

# Ensure storage accounts use secure transfer
deny[msg] {
    resource := tfplan.root_module.resources[_]
    resource.type == "azurerm_storage_account"
    not resource.values.enable_https_traffic_only
    
    msg := sprintf(
        "Storage account %s must have enable_https_traffic_only set to true",
        [resource.address]
    )
}
```

**Cost management foundation:**
- Python script for generating cost estimates using Infracost integration
- Framework for cost reporting and historical tracking
- Baseline for optimization recommendations

**Infrastructure monitoring tools:**
- Drift detection script that compares Terraform state with actual Azure resources
- Email notification framework for configuration changes
- Reports generated in markdown format for easy review

```python
# Example from scripts/drift_detection.py
def detect_drift(terraform_dir, output_dir):
    """Detect drift between Terraform state and actual infrastructure."""
    
    # Run terraform plan to detect drift
    subprocess.run(
        ["terraform", "-chdir=" + terraform_dir, "plan", "-detailed-exitcode"],
        check=False  # Don't fail on drift detection
    )
```

### Alternative approaches: staying within your existing tool stack

While the Python scripts provide comprehensive functionality, they do introduce another tool stack. Here are alternatives that use tools you likely already have:

**Policy validation alternatives:**
```bash
# Use native Terraform validation
terraform plan -detailed-exitcode
terraform validate

# Use Checkov for policy scanning (single binary)
checkov -f main.tf --framework terraform
```

**Cost estimation without Python:**
```bash
# Use Infracost CLI directly in CI/CD
infracost breakdown --path . --format json > cost-estimate.json

# Or use Azure CLI for basic cost queries
az consumption usage list --top 10
```

**Drift detection with shell scripts:**
```bash
#!/bin/bash
# Simple drift detection using Terraform exit codes
terraform plan -detailed-exitcode -no-color > drift-report.txt
exit_code=$?

if [ $exit_code -eq 2 ]; then
    echo "⚠️  Drift detected - see drift-report.txt"
    # Send notification using existing tools (curl, mail, etc.)
fi
```

**GitHub Actions for automation:**
```yaml
# Policy validation in CI without Python
- name: Run Checkov
  uses: bridgecrewio/checkov-action@master
  with:
    framework: terraform
    
- name: Cost estimation
  uses: infracost/infracost-gh-action@master
  with:
    api-key: {% raw %}${{ secrets.INFRACOST_API_KEY }}{% endraw %}
```

**Key consideration:** Start with the simplest approach that meets your needs. You can always evolve to more sophisticated tooling as requirements grow.

**Documentation:**
- Architecture Decision Records (ADRs) for design decisions
- Automated diagram generation from Terraform code
- Comprehensive module documentation with examples

## Key lessons learned

### 1. Start simple, evolve systematically

You don't need to implement everything at once. The phased approach allowed me to:
- Learn and adapt at each stage
- Maintain working infrastructure throughout the evolution
- Build team expertise gradually

### 2. Governance is not optional

What started as "nice to have" features like naming conventions and tagging became essential as I scaled:
- Prevented configuration drift
- Enabled cost tracking and optimization
- Simplified troubleshooting and auditing

### 3. Testing infrastructure code is critical

Infrastructure failures are expensive and disruptive. My testing framework:
- Catches issues before they reach production
- Validates complex module interactions
- Provides confidence for infrastructure changes

### 4. Automation pays dividends

The initial investment in CI/CD automation provided:
- Reduced deployment-related errors significantly
- Major time savings on infrastructure operations
- Improved security through consistent processes

### 5. Documentation drives adoption

Comprehensive documentation, including upgrade guides and ADRs, provided:
- Faster onboarding of new team members
- Better decision-making through recorded rationale
- Smoother handoffs and knowledge transfer

## What's next: the roadmap ahead

My infrastructure evolution continues with planned enhancements:

**Short-term goals:**
- Automate the policy validation in CI/CD pipeline
- Integrate cost estimation with pull request workflows  
- Set up automated drift detection scheduling
- Enhanced monitoring and alerting integration

**Long-term vision:**
- Fully automated policy enforcement with blocking rules
- Real-time cost alerts and optimization recommendations
- Self-healing infrastructure with automated drift remediation
- Advanced security scanning integration (Checkov, tfsec)

## Getting Started: Your evolution path

If you're starting your own Terraform evolution, consider this roadmap:

1. **Phase 1**: Start with basic functionality, but plan for modules
2. **Phase 2**: Implement naming conventions and basic governance
3. **Phase 3**: Add validation and comprehensive tagging
4. **Phase 4**: Introduce CI/CD automation with branch protection
5. **Phase 5**: Implement comprehensive testing with Terratest
6. **Phase 6**: Add advanced features like policy enforcement and cost monitoring

## Conclusion: evolution over revolution

My transformation from a simple Terraform script to a sophisticated infrastructure platform demonstrates that evolution often trumps revolution. By taking a systematic, phased approach, I:

- Maintained working infrastructure throughout the process
- Built team expertise and confidence gradually  
- Added value at each iteration
- Created a foundation for future growth

The key is starting with your current needs while planning for future requirements. Your infrastructure code is a living system, expect it to evolve.




