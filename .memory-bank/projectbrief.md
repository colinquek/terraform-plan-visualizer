## Project Brief

### Project Name
Terraform Plan Visualizer

### Purpose
Convert Terraform plan JSON files into interactive, self-contained HTML visualizations for CI/CD pipelines, pull requests, and team collaboration.

### Core Requirements
- Parse `terraform show -json` output format
- Generate standalone HTML with embedded CSS/JavaScript
- Visualize resource changes: create, update, delete, replace
- Display infrastructure drift with before/after diffs
- Support multiple distribution methods: binary, Docker, GitHub Action

### Distribution Targets
1. **Static Binary**: Go-based, no runtime dependencies
2. **Docker Image**: Multi-stage build (Alpine-based)
3. **GitHub Action**: Reusable workflow action

### Technical Stack
- **Language**: Go (currently 1.26.4, upgrading from 1.25.3)
- **Build**: Static compilation with CGO disabled
- **Base Image**: Alpine 3.18
- **Module**: cloudvic-tf-plan-viz

### Key Files
- `main.go`: CLI entry point, flag parsing, validation
- `html_generator.go`: HTML generation, resource extraction
- `Dockerfile`: Multi-stage container build
- `action.yml`: GitHub Action definition
- `entrypoint.sh`: GitHub Action entrypoint

### Current Status
- Version: v1.0.2.06jul
- Go version being upgraded to 1.26.4
- Docker and build files already updated to 1.26.4

### Goals
- Maintain backward compatibility with existing Terraform plan JSON formats
- Keep binary size minimal through static compilation flags
- Support all major Terraform providers
- Enable easy CI/CD integration
