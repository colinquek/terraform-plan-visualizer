# Product Context

## Problem Statement
Terraform plan output is verbose and difficult to review, especially in CI/CD pipelines where stakeholders need to quickly understand infrastructure changes.

## Solution
An automated visualization tool that:
- Parses Terraform plan JSON (from `terraform show -json`)
- Generates interactive HTML reports
- Highlights changes with color coding (green=create, orange=update, red=delete)
- Provides collapsible sections for easy navigation

## User Experience Goals
1. **Clarity**: Anyone can understand what changes Terraform will make
2. **Speed**: Quick visual scan to identify critical changes
3. **Collaboration**: Shareable HTML reports for team review
4. **Integration**: Seamless CI/CD pipeline integration

## Target Users
- DevOps engineers reviewing infrastructure changes
- Security teams auditing Terraform plans
- Management stakeholders approving deployments
- CI/CD pipelines generating artifacts

## Use Cases
1. **CI/CD Pipeline Artifact**: Generate HTML report on every PR
2. **Manual Review**: Convert saved plan files to visual format
3. **Audit Trail**: Archive HTML reports for compliance
4. **Drift Detection**: Visualize resources changed outside Terraform

## Success Metrics
- Time to review Terraform plans reduced by 50%
- Increased stakeholder confidence in infrastructure changes
- Reduced deployment errors from misunderstood plans
- Easy integration with existing GitLab/GitHub workflows

## Design Principles
- **Simplicity**: Zero external dependencies
- **Portability**: Static binary, Docker, GitHub Action
- **Compatibility**: Handle all Terraform providers and plan structures
- **Performance**: Fast generation even for large plans
