# Technical Context

## Technologies

### Go 1.26.4
**Purpose**: Primary programming language
**Version**: 1.26.4 (upgraded from 1.25.3 on 2026-07-06)
**Key Features Used**:
- Standard library only (`encoding/json`, `html/template`, `os`, `fmt`)
- Static compilation with `CGO_ENABLED=0`
- Cross-platform binary generation

**Build Command**:
```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -o terraform-plan-visualizer .
```

### Docker (Multi-Stage Build)
**Purpose**: Containerized distribution
**Base Images**:
- Build stage: `golang:1.26.4-alpine`
- Runtime stage: `alpine:3.18`

**Build Process**:
```dockerfile
FROM golang:1.26.4-alpine AS builder
COPY . /app
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags='-w -s' -a -o terraform-plan-visualizer .

FROM alpine:3.18
COPY --from=builder /app/terraform-plan-visualizer /usr/local/bin/
USER 1000:1000
ENTRYPOINT ["/usr/local/bin/terraform-plan-visualizer"]
```

**Final Image Size**: ~15MB

### GitHub Actions
**Purpose**: CI/CD integration
**Action Definition**: `action.yml`
**Inputs**:
- `plan-file`: Path to Terraform plan JSON
- `output-file`: Path for generated HTML
- `upload-artifact`: Boolean to upload as workflow artifact

**Usage**:
```yaml
- name: Generate Visualization
  uses: cloudvic-org/terraform-plan-visualizer@v1
  with:
    plan-file: terraform/plan.json
    output-file: plan-visualization.html
    upload-artifact: true
```

## Development Setup

### Prerequisites
- Go 1.26.4 or later
- Docker (optional, for container builds)
- Terraform (for generating test plans)

### Build Commands

**Local Development**:
```bash
go build -o terraform-plan-visualizer .
```

**Production Binary**:
```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -o terraform-plan-visualizer .
```

**Docker Image**:
```bash
docker build -t terraform-plan-visualizer .
```

### Testing Workflow

1. **Generate Terraform Plan**:
```bash
cd terraform/
terraform init
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan > plan.json
```

2. **Run Visualizer**:
```bash
../terraform-plan-visualizer -i plan.json -o visualization.html
```

3. **Open in Browser**:
```bash
open visualization.html  # macOS
xdg-open visualization.html  # Linux
start visualization.html  # Windows
```

## Technical Constraints

### 1. Go Version Compatibility
- Minimum: Go 1.26.4
- No backward compatibility guarantees
- Latest stable version recommended

### 2. Terraform Plan Format
- Requires `terraform show -json` output
- Must match Terraform JSON plan format version 1.0+
- Supports all providers using standard format

### 3. File System
- Input/output paths are relative to working directory
- Requires read permission on input file
- Requires write permission on output directory

### 4. Browser Compatibility
- Modern browsers (Chrome, Firefox, Safari, Edge)
- JavaScript required for interactive features
- No IE support

## Dependencies

### Runtime Dependencies
**None** - Static binary with zero external dependencies

### Build Dependencies
- Go 1.26.4+ compiler
- Standard library packages:
  - `encoding/json` - JSON parsing
  - `fmt` - Formatted I/O
  - `io` - I/O utilities
  - `os` - File operations
  - `path/filepath` - Path manipulation
  - `strings` - String manipulation

### Test Dependencies
- Terraform CLI (for generating test plans)
- Web browser (for visual validation)

## Tool Usage Patterns

### CLI Flags
```bash
# Basic usage
./terraform-plan-visualizer -i plan.json -o report.html

# Show version
./terraform-plan-visualizer -v

# Show help
./terraform-plan-visualizer -h
```

### Docker Usage
```bash
docker run --rm -v $(pwd):/workspace \
  ghcr.io/cloudvic-org/terraform-plan-visualizer:latest \
  -i /workspace/plan.json -o /workspace/visualization.html
```

### GitHub Actions Usage
```yaml
- name: Generate Terraform Plan Visualization
  uses: cloudvic-org/terraform-plan-visualizer@v1
  with:
    plan-file: terraform/plan.json
    output-file: plan-visualization.html
    upload-artifact: true
```

## File Structure

```
terraform-plan-visualizer/
├── main.go                    # CLI entry point
├── html_generator.go          # HTML generation logic
├── Dockerfile                 # Multi-stage build
├── entrypoint.sh              # GitHub Action entrypoint
├── action.yml                 # GitHub Action definition
├── go.mod                     # Go module (Go 1.26.4)
├── go.sum                     # Dependency checksums
├── scripts/                   # Build script
│   └── build.sh              # Build script
├── examples/                  # Sample plans and outputs
│   ├── complex-modules-example-plan.json
│   ├── create-and-update-example-plan.json
│   └── replace-example-plan.json
└── .dockerignore              # Agent exclusions
```

## Performance Characteristics

### Binary Size
- Development build: ~20MB
- Production build (stripped): ~15MB
- Docker image: ~18MB

### Execution Time
- Small plans (<50 resources): <100ms
- Medium plans (50-200 resources): 100-500ms
- Large plans (200+ resources): 500ms-2s

### Memory Usage
- Typical: <50MB RAM
- Large plans: <200MB RAM
- No goroutines (single-threaded)

## Security Considerations

### 1. File I/O
- Validates input file exists before reading
- No shell command execution
- No network access

### 2. JSON Parsing
- Uses safe `encoding/json` package
- No dynamic code execution
- Handles malformed JSON gracefully

### 3. Docker Security
- Non-root user (UID 1000)
- Minimal Alpine base image
- No unnecessary packages

### 4. Supply Chain
- Zero external dependencies
- No third-party libraries
- Reproducible builds
