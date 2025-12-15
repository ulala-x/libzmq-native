# Contributing to libzmq-native

Thank you for your interest in contributing to libzmq-native! This document provides guidelines and information for contributors.

## How to Contribute

### Reporting Bugs

1. Check if the bug has already been reported in [Issues](https://github.com/ulala-x/libzmq-native/issues)
2. If not, create a new issue using the bug report template
3. Include as much detail as possible:
   - OS and version
   - Build environment details
   - Steps to reproduce
   - Expected vs actual behavior

### Suggesting Features

1. Check existing issues for similar suggestions
2. Create a new issue using the feature request template
3. Describe the use case and expected behavior

### Pull Requests

1. Fork the repository
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. Make your changes
4. Test on target platforms if possible
5. Commit with clear messages:
   ```bash
   git commit -m "feat: add new feature"
   ```
6. Push to your fork and submit a PR

## Development Setup

### Prerequisites

- **Windows**: Visual Studio 2019/2022, CMake 3.15+
- **Linux**: GCC 7+, CMake 3.15+, build-essential
- **macOS**: Xcode Command Line Tools, CMake 3.15+

### Building Locally

```bash
# Windows
.\build-scripts\windows\build.ps1

# Linux
./build-scripts/linux/build.sh

# macOS
./build-scripts/macos/build.sh arm64  # or x86_64
```

## Code Style

- Follow existing code patterns
- Use meaningful variable names
- Add comments for complex logic
- Keep scripts modular and maintainable

## Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `chore:` - Maintenance tasks
- `ci:` - CI/CD changes

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Questions?

Feel free to open an issue for any questions or concerns.
