# Project Brief: Terraform Plan Visualizer

## Project Overview
A Go-based CLI tool that converts Terraform plan JSON files into interactive HTML visualizations for CI/CD pipelines and team collaboration.

## Core Purpose
Transform complex Terraform plan output into human-readable, interactive HTML reports that show:
- Resource changes (create, update, delete)
- Drift detection
- Before/after attribute comparisons

## Distribution Methods
1. **Static Binary** - Cross-platform Go binary (no runtime dependencies)
2. **Docker Image** - Multi-stage build (~15MB final image)
3. **GitHub Action** - `cloudvic-org/terraform-plan-visualizer@v1`

## Technical Foundation
- **Language**: Go 1.26.4 (upgraded from 1.25.3)
- **Build**: Static compilation with CGO disabled
- **Dependencies**: Zero external dependencies (standard library only)
- **License**: Open source

## Key Files
- `main.go` - CLI entry point, flag parsing, validation
- `html_generator.go` - Terraform plan JSON parsing and HTML generation
- `Dockerfile` - Multi-stage Docker build
- `action.yml` - GitHub Action definition
- `go.mod` - Go module definition (Go 1.26.4)

## Project Goals
1. Generate clear, interactive visualizations from Terraform plans
2. Support all Terraform providers and plan structures
3. Integrate seamlessly into CI/CD pipelines
4. Maintain zero external dependencies for reliability

## Current Status
- ✅ Go version upgraded to 1.26.4
- ⏳ Code understanding phase for recoding
- 📝 Memory Bank documentation being created

## Agent Files Configuration
Agent directories (`.agents/`, `.claude/`, `.memory-bank/`, etc.) are:
- ✅ Tracked in git for team collaboration
- ✅ Excluded from Docker builds via `.dockerignore`
- ✅ NOT in `.gitignore` (must stay in git)
