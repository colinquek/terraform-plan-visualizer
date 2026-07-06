# System Patterns

## Architecture Overview

```
┌─────────────────┐
│ Terraform Plan  │
│   (JSON file)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   main.go       │
│ - Parse flags   │
│ - Validate input│
│ - Orchestrate   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ html_generator  │
│ - Parse JSON    │
│ - Extract data  │
│ - Generate HTML │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Interactive HTML│
│ - Collapsible   │
│ - Color-coded   │
│ - Searchable    │
└─────────────────┘
```

## Core Components

### 1. main.go - CLI Entry Point
**Purpose**: Command-line interface and orchestration

**Key Functions**:
- `main()` - Parse CLI flags (`-i`, `-o`, `-v`, `-h`)
- `validateInput()` - Verify input file exists and is readable
- `processPlanFile()` - Read JSON, call generator, write HTML
- `readJSONFile()` - File I/O for JSON input
- `writeHtmlFile()` - File I/O for HTML output

**Flow**:
```
User runs CLI → Parse flags → Validate input → Read JSON → Generate HTML → Write output
```

### 2. html_generator.go - Core Logic
**Purpose**: Transform Terraform plan JSON into interactive HTML

**Key Functions**:
- `generateHtml()` - Main HTML template with embedded CSS/JS
- `extractResourceChanges()` - Parse `resource_changes` array from JSON
- `countDriftChanges()` - Detect resources with drift metadata
- `generateResourceChangesHtml()` - Render each resource change section
- `getActionClass()` - Map actions to CSS classes (create/update/delete)
- `formatChangedFields()` - Show before/after attribute differences

**Terraform Plan JSON Structure**:
```json
{
  "format_version": "1.2",
  "terraform_version": "1.5.7",
  "planned_values": {
    "root_module": {
      "resources": [...]
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

## Design Patterns

### 1. Pipeline Pattern
```
Input → Validation → Processing → Output
```
Each stage is independent and testable.

### 2. Template Pattern
HTML generation uses a single template function with embedded CSS/JS for portability.

### 3. Strategy Pattern
Different distribution methods (binary, Docker, GitHub Action) use the same core logic.

### 4. Fail-Fast Pattern
Validate input early, fail with clear error messages before processing.

## Component Relationships

```
main.go
  ├── Validates user input
  ├── Reads JSON file
  └── Calls html_generator.go

html_generator.go
  ├── Parses Terraform JSON
  ├── Extracts resource changes
  ├── Generates HTML structure
  └── Embeds CSS/JS for interactivity
```

## Critical Implementation Paths

### 1. JSON Parsing Path
```
terraform show -json → Read file → Parse JSON → Extract resource_changes → Map to structs
```

### 2. HTML Generation Path
```
Resource structs → Generate HTML sections → Embed CSS classes → Add JavaScript → Write file
```

### 3. Error Handling Path
```
File not found → Clear error message → Exit code 1
Invalid JSON → Parse error details → Exit code 1
Write failure → Permission error → Exit code 1
```

## Key Technical Decisions

### 1. Zero External Dependencies
**Decision**: Use only Go standard library
**Rationale**: 
- Simplifies distribution (no `go mod download`)
- Reduces security vulnerabilities
- Improves build reproducibility
- Faster compilation

### 2. Static Binary Compilation
**Decision**: Compile with `CGO_ENABLED=0` and `-extldflags "-static"`
**Rationale**:
- No runtime dependencies
- Works on any Linux system
- Easy to distribute via GitHub releases
- ~15MB binary size acceptable

### 3. Embedded CSS/JS
**Decision**: Single HTML file with embedded styles and scripts
**Rationale**:
- No external file dependencies
- Easy to share and archive
- Works offline
- Simpler deployment

### 4. Color-Coded Actions
**Decision**: Green (create), Orange (update), Red (delete)
**Rationale**:
- Universal visual language
- Quick scanning
- Accessibility (color + icons)
- Industry standard (Terraform uses same)

## Code Flow Example

**User Command**:
```bash
./terraform-plan-visualizer -i plan.json -o report.html
```

**Execution Flow**:
1. `main()` parses `-i` and `-o` flags
2. `validateInput("plan.json")` checks file exists
3. `readJSONFile("plan.json")` loads JSON into memory
4. `generateHtml(jsonData)` processes plan:
   - Extract `resource_changes` array
   - Group by action type (create/update/delete)
   - Generate HTML sections for each resource
   - Add CSS classes based on action
   - Embed interactive JavaScript
5. `writeHtmlFile("report.html", html)` writes output
6. User opens `report.html` in browser
