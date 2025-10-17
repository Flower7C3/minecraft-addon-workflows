# Minecraft Addon Workflows

Reusable GitHub Actions workflows for building and releasing Minecraft Bedrock Edition addons.

## Features

- **Automated Building**: Uses local `minecraft-pack-build` script for consistent builds
- **Version Management**: Automatic version bumping and tagging
- **Release Generation**: Automated GitHub releases with customizable templates
- **Package Integrity**: Built-in testing of .mcaddon and .mcpack files
- **Flexible Configuration**: Support for projects with and without test workflows

## Workflows

### Build and Release (`build-and-release.yml`)

Main workflow that handles the complete build and release process.

**Inputs:**
- `project_name` (required): Name of the project for display purposes
- `has_test_workflow` (optional): Whether the project has a test workflow (default: false)
- `total_blocks` (optional): Whether to include total blocks count in statistics (default: false)

**Outputs:**
- `version`: The incremented version number
- `tag_name`: The tag name for the release

## Usage

### 1. Create a release template

Create `.github/templates/release.md` in your project with placeholders:

```markdown
A Minecraft Bedrock Edition addon that adds [description] to your world.

## Files
- BP files: {{ bp_files }}
- RP files: {{ rp_files }}
- Total blocks: {{ total_blocks }}

## Installation
1. Download the `.mcaddon` file from the assets below
2. Open Minecraft Bedrock Edition
3. Go to Settings > Global Resources > My Packs
4. Click "Import" and select the downloaded file
5. Activate the pack in your world

## Features
- Feature 1
- Feature 2
- Feature 3

## Changes
{{ commits }}
```

### 2. Create your project workflow

Create `.github/workflows/build.yml` in your project:

```yaml
name: Build & Release Workflow

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build-and-release:
    name: Build and release
    uses: flower7c3/minecraft-addon-github-workflows/.github/workflows/build-and-release.yml@main
    with:
      project_name: "Your Project Name"
      has_test_workflow: true  # Set to false if no test workflow
      total_blocks: true       # Set to false if no blocks
    secrets: inherit
```

### 3. Required project structure

Your project must have:
- `BP/` directory with `manifest.json`
- `RP/` directory with `manifest.json`
- `version.json` with version array (e.g., `{"version": [1, 0, 0]}`)
- `.github/templates/release.md` template file
- `minecraft-pack-build` script (executable)

## Template Variables

The following variables are available in your release template:

- `{{ version }}` - Version number (e.g., "1.2.3")
- `{{ bp_files }}` - Number of files in BP directory
- `{{ rp_files }}` - Number of files in RP directory
- `{{ total_blocks }}` - Number of block files (only if total_blocks is true)
- `{{ commits }}` - List of commits since last tag

## Examples

### Project with test workflow and blocks
```yaml
jobs:
  build-and-release:
    name: Build and release
    uses: flower7c3/minecraft-addon-github-workflows/.github/workflows/build-and-release.yml@main
    with:
      project_name: "Polish road signs"
      has_test_workflow: true
      total_blocks: true
    secrets: inherit
```

### Project without test workflow
```yaml
jobs:
  build-and-release:
    name: Build and release
    uses: flower7c3/minecraft-addon-github-workflows/.github/workflows/build-and-release.yml@main
    with:
      project_name: "Simple money"
      has_test_workflow: false
      total_blocks: false
    secrets: inherit
```

## Standalone usage

You can use script for Your local builds. Download it and install locally.

```shell
curl -fsSL https://raw.githubusercontent.com/flower7c3/minecraft-addon-workflows/main/minecraft-pack-build -o ~/.local/bin/minecraft-pack-build
chmod +x ~/.local/bin/minecraft-pack-build
minecraft-pack-build
```

## Dependencies

- `minecraft-pack-build` - Local script for building Minecraft addons (included in repository)
- [actions/checkout](https://github.com/actions/checkout) - For checking out code
- [actions/upload-artifact](https://github.com/actions/upload-artifact) - For uploading build artifacts
- [actions/download-artifact](https://github.com/actions/download-artifact) - For downloading artifacts
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release) - For creating GitHub releases

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test your changes thoroughly
5. Submit a pull request

## Support

If you encounter any issues or have questions, please open an issue on the [GitHub repository](https://github.com/flower7c3/minecraft-addon-workflows).