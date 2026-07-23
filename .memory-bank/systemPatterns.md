## System Patterns

### Architecture Overview
```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Terraform CLI  │────▶│  Plan Visualizer  │────▶│  Interactive    │
│  (show -json)   │     │  (Go Binary)      │     │  HTML Output    │
└─────────────────┘     └──────────────────┘     └─────────────────┘
       JSON                    Processing               CSS/JS
```

### Core Components

#### 1. CLI Entry Point (`main.go`)
- **Responsibility**: Flag parsing, input validation, orchestration
- **Key Functions**:
  - `main()`: CLI flag handling
  - `validateInput()`: File existence and readability checks
  - `processPlanFile()`: End-to-end processing workflow
  - `readJSONFile()` / `writeHtmlFile()`: File I/O
  - `showVersionInfo()` / `showHelpInfo()`: User information

#### 2. HTML Generator (`html_generator.go`)
- **Responsibility**: Parse Terraform plan JSON, generate HTML visualization
- **Key Functions**:
  - `generateHtml()`: Main HTML template with embedded CSS/JS
  - `extractResourceChanges()`: Parse `resource_changes` array from JSON
  - `countDriftChanges()`: Identify resources with drift (excludes replace)
  - `generateResourceChangesHtml()`: Render change sections
  - `generateDriftHtml()`: Render drift detection with diffs
  - `getDriftDetails()` / `getChangeDetails()`: Before/after attribute diffs
  - `getActionClass()`: Map actions to CSS classes

### Data Flow
1. **Input**: Terraform plan JSON (`terraform show -json`)
2. **Parsing**: Go `encoding/json` unmarshals to `interface{}`
3. **Extraction**: `extractResourceChanges()` processes `resource_changes` array
4. **Classification**: Group by action type (create/update/delete/replace)
5. **Rendering**: HTML template with embedded styling
6. **Output**: Self-contained HTML file

### Key Terraform JSON Structure
```json
{
  "format_version": "1.2",
  "terraform_version": "1.5.7",
  "planned_values": { "root_module": { "resources": [...] } },
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

### Build Patterns

#### Static Binary Build
```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -o terraform-plan-visualizer .
```
- **CGO_ENABLED=0**: No C dependencies
- **-ldflags='-w -s'**: Strip debug info and symbol table
- **-extldflags "-static"**: Static linking
- **-a**: Force rebuilding of packages

#### Docker Multi-Stage Build
- **Stage 1 (builder)**: Go 1.26.4 Alpine, compiles binary
- **Stage 2 (final)**: Alpine 3.18, copies binary, sets non-root user
- **Result**: Minimal runtime image (~15-20MB)

### Change Type Visualization
- **Create** (green): New resources
- **Update** (orange): Modified resources
- **Delete** (red): Removed resources
- **Replace** (gradient red-green): Recreated resources
- **Drift** (orange): Infrastructure changed outside Terraform

### HTML Output Structure
- Embedded CSS in `<style>` block
- Embedded JavaScript in `<script>` block
- Collapsible sections with toggle buttons
- Side-by-side diffs for before/after comparisons
- No external dependencies
