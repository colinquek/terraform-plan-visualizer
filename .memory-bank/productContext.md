## Product Context

### Problem Statement
Terraform plan output in CI/CD pipelines is difficult to read and share:
- Raw JSON is verbose and hard to parse visually
- Terminal output gets truncated in PR comments
- Team members need clear visibility into infrastructure changes
- Security/compliance teams require audit-friendly formats

### Solution
Generate interactive HTML visualizations that:
- Show resource changes at a glance with color coding
- Provide collapsible sections for detailed attribute diffs
- Work as standalone files (no external dependencies)
- Integrate seamlessly into existing CI/CD workflows

### Target Users
1. **DevOps Engineers**: Sharing plan results in pull requests
2. **Security Teams**: Reviewing infrastructure changes
3. **Platform Teams**: Building internal Terraform workflows
4. **Consultants**: Delivering client-ready plan documentation

### User Experience Goals
- **Zero Configuration**: Run with single command, get working HTML
- **Self-Contained**: No external CSS/JS dependencies
- **Fast**: Minimal processing overhead
- **Clear Visual Hierarchy**: Color-coded actions, collapsible details
- **CI/CD Ready**: Artifact-friendly output format

### Success Metrics
- Time to generate visualization < 5 seconds for typical plans
- HTML file size reasonable for artifact storage
- All resource change types accurately represented
- Works with standard Terraform JSON output (no custom formatting)

### Design Principles
- **Simplicity**: Single binary, minimal flags
- **Portability**: Works across platforms via Docker/binary
- **Clarity**: Visual design prioritizes readability
- **Open Source**: Free to use, modify, distribute
