# Install RPM from GitHub Release

A reusable GitHub Action to download and install an RPM package from a GitHub release, with automatic RHEL version detection.

## Features

- 🎯 **Auto-detection** - Automatically detects your system's RHEL version (8, 9, or 10)
- 🔍 **Smart matching** - Finds the correct RPM for your system from release assets
- 📦 **One-liner integration** - Easy to use in your GitHub Actions workflows
- 🛡️ **Secure** - Uses GitHub tokens for secure API access
- 📊 **Output information** - Returns the downloaded filename and URL for further use

## Usage

### Basic Usage

Use this action in your GitHub Actions workflow:

```yaml
name: Install Package
on: [push, pull_request]

jobs:
  install:
    runs-on: ubuntu-latest
    steps:
      - name: Install RPM from Release
        uses: NSLS2/install-rpm-from-gh-release@v1
        with:
          repository: 'owner/package-repo'
          release_tag: 'v1.0.0'`
```

### Example with RHEL Version Specification

```yaml
- name: Install specific version
  uses: NSLS2/install-rpm-from-gh-release@v1
  with:
    repository: 'orgs/my-package'
    release_tag: 'v2.3.1'
    rhel_version: '9'
```

### Using Outputs

```yaml
- name: Install RPM
  id: install
  uses: NSLS2/install-rpm-from-gh-release@v1
  with:
    repository: 'owner/package'
    release_tag: 'v1.0.0'

- name: Log Results
  run: |
    echo "Installed: ${{ steps.install.outputs.rpm_filename }}"
    echo "From: ${{ steps.install.outputs.rpm_url }}"
    echo "Detected RHEL: ${{ steps.install.outputs.detected_rhel_version }}"
```

### Installing Development Package

To also install the `-devel` package variant (useful for building against the installed package):

```yaml
- name: Install Package and Devel Variant
  uses: NSLS2/install-rpm-from-gh-release@v1
  with:
    repository: 'owner/package'
    release_tag: 'v1.0.0'
    install_devel: 'true'
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | ✓ | - | Target repository in `owner/repo` format |
| `release_tag` | ✓ | - | Release tag to download from (e.g., `v1.0.0`) |
| `rhel_version` | ✗ | Auto-detect | RHEL version (8, 9, or 10). If empty, auto-detects. |
| `github_token` | ✗ | `${{ github.token }}` | GitHub token for API access |
| `install_devel` | ✗ | `'false'` | Also install the -devel package variant if available |

## Outputs

| Output | Description |
|--------|-------------|
| `rpm_filename` | The filename of the downloaded RPM |
| `rpm_url` | The download URL of the RPM |
| `devel_rpm_filename` | The filename of the downloaded -devel RPM (if `install_devel: 'true'`) |
| `devel_rpm_url` | The download URL of the -devel RPM (if `install_devel: 'true'`) |
| `detected_rhel_version` | The RHEL version used for matching |

## How It Works

1. **Version Detection** - Detects the system's RHEL version or uses the provided `rhel_version`
2. **Release Fetch** - Queries the GitHub API to get release information
3. **RPM Matching** - Searches release assets for an RPM matching the detected/specified RHEL version
4. **Devel Package Search** (optional) - If `install_devel: 'true'`, searches for a matching `-devel` variant package
5. **Download** - Downloads the matched RPM(s)
6. **Installation** - Installs the RPM(s) using `dnf`

### RPM Naming Patterns Supported

The action recognizes RPMs with these naming patterns:
- `package-1.0-1.el8.x86_64.rpm`
- `package-1.0-1.rhel-8.x86_64.rpm`
- `package-1.0-1.fc8.x86_64.rpm`
- `package-1.0.el8.x86_64.rpm`
- And other variations with `el{version}`, `rhel-{version}`, or `fc{version}`

## Requirements

- A GitHub Actions runner with a RHEL-like OS installed (RHEL, AlmaLinux, Rocky Linux, CentOS) version 8, 9, or 10
- `curl` and `jq` installed (typically pre-installed)

## Example Workflow

Here's a complete workflow example:

```yaml
name: Build and Install

on:
  workflow_dispatch:
    inputs:
      repo:
        description: 'Repository to install from'
        required: true
      tag:
        description: 'Release tag'
        required: true

jobs:
  install-rpm:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install Package
        uses: NSLS2/install-rpm-from-gh-release@v1
        with:
          repository: ${{ github.event.inputs.repo }}
          release_tag: ${{ github.event.inputs.tag }}

      - name: Verify Installation
        run: |
          # Replace 'package-name' with your actual package command
          package-name --version
```

## Troubleshooting

### "No RPM found for RHEL version X"

The action couldn't find an RPM matching your RHEL version in the release. Check that:
1. The release tag exists
2. The repository contains RPM assets
3. The RPM filename contains an RHEL version identifier (e.g., `.el8`, `rhel-9`)
4. You're specifying the correct `rhel_version` if not using auto-detection

### "Failed to fetch release"

This often means:
1. The repository doesn't exist or is private (use a valid token)
2. The release tag doesn't exist
3. GitHub API rate limits are exceeded

Ensure your `github_token` has appropriate permissions.

## License

BSD 3-Clause License - See LICENSE file for details

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues, questions, or suggestions, please open an issue in the repository.
