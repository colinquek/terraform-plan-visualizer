# AI Agent Guide for terraform-plan-visualizer

This repository contains a **Go-based Terraform plan visualization tool** that converts Terraform plan JSON files into interactive HTML pages for CI/CD pipelines and team collaboration.

## Project Overview

**Purpose:** Generate interactive HTML visualizations from Terraform plan JSON output
**Language:** Go 1.25.3
**Distribution:** Binary, Docker image, GitHub Action
**License:** Open source (see LICENSE)

## Repository Structure

```
terraform-plan-visualizer/
├── main.go                    # Entry point, CLI flag parsing, validation
├── html_generator.go          # HTML generation logic, resource change extraction
├── Dockerfile                 # Multi-stage build (Go 1.25.3 → Alpine 3.18)
├── entrypoint.sh              # GitHub Action entrypoint script
├── action.yml                 # GitHub Action definition
├── go.mod / go.sum           # Go module dependencies
├── scripts/
│   └── build.sh              # Build script
└── examples/                  # Sample plan JSON files and generated HTML
    ├── complex-modules-example-plan.json
    ├── create-and-update-example-plan.json
    └── replace-example-plan.json
```

## Core Concepts

### 1. Input/Output Flow
```
Terraform Plan JSON → [main.go] → [html_generator.go] → Interactive HTML
     (plan.json)      (validation)   (resource extraction)  (visualization)
```

### 2. Change Types Visualized
- **Create** (green) - New resources being added
- **Update** (orange) - Existing resources being modified
- **Delete** (red) - Resources being removed
- **Replace** (gradient red-green) - Resources being recreated
- **Drift Detection** (orange) - Resources that changed outside Terraform, shown with before/after diffs

### 3. Distribution Methods
- **Binary:** Static Go binary (no runtime dependencies)
- **Docker:** `ghcr.io/cloudvic-org/terraform-plan-visualizer:latest`
- **GitHub Action:** `cloudvic-org/terraform-plan-visualizer@v1`

## Development Workflow

### Build Commands

```bash
# Local build
go build -o terraform-plan-visualizer .

# Cross-platform static binary
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -o terraform-plan-visualizer .

# Docker build
docker build -t terraform-plan-visualizer .

# Run tests (if any)
go test ./...
```

### Testing Workflow

1. **Generate a Terraform plan JSON:**
   ```bash
   cd terraform/
   terraform init
   terraform plan -out=plan.tfplan
   terraform show -json plan.tfplan > plan.json
   ```

2. **Run the visualizer:**
   ```bash
   ../terraform-plan-visualizer -i plan.json -o visualization.html
   ```

3. **Open in browser:**
   ```bash
   open visualization.html  # macOS
   xdg-open visualization.html  # Linux
   start visualization.html  # Windows
   ```

### Debugging

```bash
# Enable verbose output (if implemented)
./terraform-plan-visualizer -i plan.json -v

# Check version
./terraform-plan-visualizer -v

# Show help
./terraform-plan-visualizer -h
```

## Key Functions

### main.go
- `main()` - CLI flag parsing, validation, orchestration
- `validateInput()` - Validates input file exists and is readable
- `processPlanFile()` - Reads JSON, parses, generates HTML
- `readJSONFile()` - File I/O for JSON input
- `writeHtmlFile()` - File I/O for HTML output
- `showVersionInfo()` - Displays version, build time, git commit
- `showHelpInfo()` - Shows usage help and examples

### html_generator.go
- `generateHtml()` - Main HTML generation with embedded CSS/JS
- `extractResourceChanges()` - Parses Terraform plan JSON structure
- `countDriftChanges()` - Detects resources with drift (excludes replace operations)
- `generateResourceChangesHtml()` - Renders resource change sections
- `generateDriftHtml()` - Renders drift detection section with before/after diffs
- `getDriftDetails()` - Generates side-by-side diff for drifted resources
- `getChangeDetails()` - Shows before/after attribute changes for updates
- `getActionClass()` - Maps actions (create/update/delete/replace) to CSS classes
- `formatChangedFields()` - Shows before/after attribute changes

## Terraform Plan JSON Structure

The tool expects the standard `terraform show -json` output format:

```json
{
  "format_version": "1.2",
  "terraform_version": "1.5.7",
  "planned_values": {
    "root_module": {
      "resources": [
        {
          "address": "aws_instance.web",
          "mode": "managed",
          "type": "aws_instance",
          "name": "web",
          "provider_name": "registry.terraform.io/hashicorp/aws",
          "values": { ... }
        }
      ]
    }
  },
  "resource_changes": [
    {
      "address": "aws_instance.web",
      "mode": "managed",
      "type": "aws_instance",
      "name": "web",
      "provider_name": "registry.terraform.io/hashicorp/aws",
      "change": {
        "actions": ["create"],
        "before": null,
        "after": { ... }
      }
    }
  ]
}
```

## Common Development Tasks

### Adding Support for New Resource Types

1. **Update `extractResourceChanges()`** in `html_generator.go`
2. **Add CSS styling** for new action types (if needed)
3. **Test with example JSON** in `examples/` directory
4. **Update README.md** with new features

### Modifying HTML Output

1. **Edit the HTML template** in `generateHtml()` function
2. **Update embedded CSS** in the `<style>` block
3. **Test in multiple browsers** for compatibility
4. **Regenerate examples** in `examples/` directory

### Adding CLI Flags

1. **Add flag definition** in `main()`
2. **Update validation** in `validateInput()`
3. **Update help text** in `showHelpInfo()`
4. **Update README.md** usage section

## CI/CD Integration

### GitHub Actions Workflow

```yaml
- name: Generate Visualization
  uses: cloudvic-org/terraform-plan-visualizer@v1
  with:
    plan-file: terraform/plan.json
    output-file: plan-visualization.html
    upload-artifact: true
```

### Docker Usage

```bash
docker run --rm -v $(pwd):/workspace \
  ghcr.io/cloudvic-org/terraform-plan-visualizer:latest \
  -i /workspace/plan.json -o /workspace/visualization.html
```

## Gotchas and Pitfalls

1. **JSON Format:** Must use `terraform show -json`, not raw `terraform plan` output
2. **File Paths:** Input/output paths are relative to working directory
3. **Permissions:** Binary needs execute permissions (`chmod +x`)
4. **Docker Volumes:** Mount the directory containing plan.json when using Docker
5. **Version Compatibility:** Built with Go 1.25.3, may not work with older versions

## Testing Strategy

### Unit Tests (if implemented)
- Test `extractResourceChanges()` with sample JSON
- Test `countDriftChanges()` for drift detection
- Test HTML generation for valid output

### Integration Tests
- Use example JSON files in `examples/`
- Compare generated HTML against known-good output
- Test with real Terraform plans from various providers

### Manual Testing
1. Generate plan from complex Terraform module
2. Verify all resource changes are displayed correctly
3. Check interactive elements (collapsible sections, etc.)
4. Test in multiple browsers

## Documentation Links

- **README.md:** User-facing documentation, installation, usage examples
- **examples/:** Sample plan JSON files and generated HTML outputs
- **Dockerfile:** Build process and runtime configuration
- **action.yml:** GitHub Action interface definition

## Questions to Ask Before Implementing

1. **Does this change the HTML output structure?** → Update examples/
2. **Does this add new CLI flags?** → Update main.go, README.md, action.yml
3. **Does this change JSON parsing?** → Test with all example JSON files
4. **Does this affect Docker build?** → Verify multi-stage build still works
5. **Is this backward compatible?** → Ensure existing users aren't broken

## AI Agent Behavior Guidelines

**When modifying Go code:**
1. Check if changes affect CLI interface (flags, validation)
2. Verify HTML generation produces valid output
3. Test with example JSON files in `examples/`
4. Update README.md if user-facing behavior changes
5. No emojis at all.

**When adding features:**
1. Start with html_generator.go (core logic)
2. Add CLI flags in main.go if needed
3. Update action.yml for GitHub Action support
4. Add example to `examples/` directory
5. Document in README.md

**When debugging:**
1. Check JSON input format matches expected structure
2. Verify file paths are correct (input/output)
3. Look for parsing errors in `extractResourceChanges()`
4. Test with simple plan JSON first, then complex

---

**Last Updated:** July 2026
**Project Type:** Go CLI tool + Docker + GitHub Action
**Key Constraint:** Must handle all Terraform provider types and plan structures
