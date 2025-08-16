# GitHub Actions Build Pipeline

This directory contains GitHub Actions workflows for building and releasing the OpenFIRE-App project.

## Workflows Overview

### 1. `build.yml` - Main Build Pipeline
**Triggers:** Push to main/master, Pull Requests, Release creation
**Purpose:** Builds the application for all supported platforms and creates release assets

**Features:**
- **Linux (Qt6):** Builds for x86_64 and aarch64 architectures, creates AppImage for x86_64
- **Linux (Qt5):** Builds for x86_64 with Qt5 compatibility
- **Windows:** Builds with Qt6, creates basic package
- **macOS:** Builds with Qt6, creates basic .app package
- **Release:** Automatically uploads assets to GitHub releases when a release is published

### 2. `build-windows-enhanced.yml` - Enhanced Windows Build
**Triggers:** Push to main/master, Pull Requests, Release creation
**Purpose:** Creates a complete Windows package with all Qt dependencies

**Features:**
- Uses `windeployqt` to include all required Qt libraries
- Creates a self-contained Windows package
- Includes README.txt with usage instructions

### 3. `pr-check.yml` - Pull Request Quality Checks
**Triggers:** Pull Requests to main/master
**Purpose:** Ensures code quality and buildability before merging

**Features:**
- Builds on Linux, Windows, and macOS
- Performs basic build verification
- Checks CMake configuration validity
- Scans for TODO/FIXME comments
- Verifies essential files exist

### 4. `nightly.yml` - Nightly Development Builds
**Triggers:** Daily at 2 AM UTC, Manual dispatch
**Purpose:** Creates development builds for testing

**Features:**
- Daily automated builds
- Manual trigger option
- Creates development artifacts with date/hash naming
- Optional GitHub release creation (manual trigger only)
- 30-day artifact retention

## Usage

### For Contributors

1. **Pull Requests:** The `pr-check.yml` workflow will automatically run on your PRs to ensure they build correctly on all platforms.

2. **Local Testing:** You can test the build process locally using the same commands:
   ```bash
   mkdir build && cd build
   cmake .. -DCMAKE_BUILD_TYPE=Release
   make -j$(nproc)  # Linux/macOS
   # or
   cmake --build . --config Release --parallel  # Windows
   ```

### For Maintainers

1. **Creating Releases:**
   - Create a new release on GitHub
   - The `build.yml` workflow will automatically build and upload assets
   - Assets will include AppImage for Linux, ZIP for Windows, and ZIP for macOS

2. **Nightly Builds:**
   - Run automatically every day at 2 AM UTC
   - Can be manually triggered from the Actions tab
   - Use "Create nightly release" job to create a GitHub release with development builds

3. **Manual Builds:**
   - Go to Actions tab in GitHub
   - Select any workflow
   - Click "Run workflow" to manually trigger builds

## Configuration

### Environment Variables

The workflows use these environment variables:
- `QT_VERSION`: Qt6 version to use (default: "6.5.2")
- `QT_VERSION_5`: Qt5 version to use (default: "5.15.2")

### Build Options

The CMake configuration includes:
- `-DCMAKE_BUILD_TYPE=Release`: Release build configuration
- `-DOFAPP_GITHASH=<hash>`: Embeds git hash in the build
- `-DOFAPP_QT_VERSION=Qt5`: Forces Qt5 build (when needed)

### Platform Support

| Platform | Qt6 | Qt5 | Package Format |
|----------|-----|-----|----------------|
| Linux x86_64 | ✅ | ✅ | AppImage, Binary |
| Linux aarch64 | ✅ | ❌ | Binary |
| Windows x86_64 | ✅ | ❌ | ZIP |
| macOS x86_64 | ✅ | ❌ | ZIP |

## Troubleshooting

### Common Issues

1. **Build Failures:**
   - Check the Actions logs for specific error messages
   - Ensure all required Qt modules are available
   - Verify CMake configuration is valid

2. **Missing Dependencies:**
   - Linux: Ensure `build-essential`, `cmake`, `libgl1-mesa-dev` are installed
   - Windows: Qt dependencies are handled by `windeployqt`
   - macOS: Dependencies are included in the build

3. **AppImage Creation:**
   - Only works on x86_64 Linux builds
   - Requires `linuxdeployqt` tool
   - May fail if Qt modules are missing

### Debugging

1. **Local Reproduction:**
   ```bash
   # Install Qt6
   # Configure with same options as CI
   cmake .. -DCMAKE_BUILD_TYPE=Release -DOFAPP_GITHASH=$(git rev-parse --short HEAD)
   ```

2. **Check Qt Installation:**
   ```bash
   # Verify Qt installation
   qmake --version
   # Check available modules
   ls $QT_DIR/lib/cmake/
   ```

## Customization

### Adding New Platforms

1. Add a new job to the workflow
2. Configure the appropriate runner (`ubuntu-latest`, `windows-latest`, `macos-latest`)
3. Install required dependencies
4. Configure CMake with appropriate options
5. Build and package the application

### Modifying Build Options

1. Update the CMake configuration step
2. Add new environment variables if needed
3. Modify the build commands as required

### Adding New Qt Modules

1. Update the `modules` parameter in the Qt setup step
2. Ensure the module is available in the specified Qt version
3. Update the CMake configuration if needed

## Security

- Workflows use `GITHUB_TOKEN` for authentication
- No sensitive data is exposed in logs
- Artifacts are automatically cleaned up after retention period
- Dependencies are pinned to specific versions for reproducibility

## Performance

- Builds run in parallel across platforms
- Uses parallel compilation (`make -j$(nproc)`)
- Caches Qt installation between runs
- Optimized for speed while maintaining reliability
