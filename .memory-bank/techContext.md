## Technical Context

### Technologies

#### Go (Golang)
- **Current Version**: 1.26.4 (upgrading from 1.25.3)
- **Module**: `cloudvic-tf-plan-viz`
- **Key Packages**:
  - `encoding/json`: JSON parsing
  - `flag`: CLI flag parsing
  - `os`: File I/O
  - `fmt`: Formatted I/O
  - `runtime`: Version/platform info

#### Docker
- **Builder Image**: `golang:1.26.4-alpine`
- **Runtime Image**: `alpine:3.18`
- **Build Strategy**: Multi-stage for minimal image size
- **Security**: Non-root user (UID 1001, GID 1001)

#### Terraform
- **Expected Input**: `terraform show -json` output
- **Format Version**: 1.2 (current standard)
- **Compatibility**: All providers using standard JSON format

### Development Setup

#### Prerequisites
- Go 1.26.4 or later
- Git (for version info)
- Docker (optional, for container builds)

#### Build Commands
```bash
# Local development
go build -o terraform-plan-visualizer .

# Production static binary
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -o terraform-plan-visualizer .

# Docker image
docker build -t terraform-plan-visualizer .
```

#### Testing Workflow
```bash
# Generate plan JSON
cd terraform/
terraform init
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan > plan.json

# Run visualizer
../terraform-plan-visualizer -i plan.json -o visualization.html

# Open in browser
xdg-open visualization.html  # Linux
open visualization.html      # macOS
```

### Technical Constraints

#### Go Version
- **Minimum**: Go 1.26.4 (after upgrade)
- **Reason**: Latest features, security patches, performance improvements
- **Impact**: Docker build, local development, CI/CD runners

#### Static Compilation
- **CGO Disabled**: No C library dependencies
- **Static Linking**: Binary must be fully static
- **Platform**: Linux amd64 primary target

#### Docker
- **Base Image**: Alpine 3.18 (small footprint)
- **User**: Non-root required for security
- **Entrypoint**: Single binary with flags

### Dependencies

#### Go Modules
- Check `go.mod` and `go.sum` for current dependencies
- Minimal external dependencies (stdlib-focused)

#### Runtime Dependencies
- **Binary**: None (fully static)
- **Docker**: ca-certificates, tzdata (included in image)
- **GitHub Action**: Docker or Node.js runner

### Tool Usage Patterns

#### CLI Flags
- `-i` / `-input`: Input JSON file (required)
- `-o` / `-output`: Output HTML file (default: index.html)
- `--output-html-path`: Alternative output flag
- `-v` / `-version`: Show version info
- `-h` / `-help`: Show help

#### Version Management
- Version string: `v1.0.2.06jul`
- Set via ldflags during build:
  ```bash
  -ldflags="-X main.Version=v1.0.2.06jul -X main.BuildTime=... -X main.GitCommit=..."
  ```

### CI/CD Integration

#### GitHub Actions
- Action: `cloudvic-org/terraform-plan-visualizer@v1`
- Inputs: `plan-file`, `output-file`, `upload-artifact`
- Runner: Ubuntu latest

#### Docker Registry
- Image: `ghcr.io/cloudvic-org/terraform-plan-visualizer:latest`
- Tagging: Semantic versioning + `latest`
