# Contributing to activerecord-multi-tenant

Thank you for your interest in contributing to activerecord-multi-tenant! We appreciate your help in making this project better.

## Getting Started

1. Fork the repository
2. Clone your fork: `git clone https://github.com/YOUR-USERNAME/activerecord-multi-tenant.git`
3. Create a new branch for your feature or bug fix: `git checkout -b my-feature-branch`
4. Make your changes
5. Run the test suite to ensure your changes don't break existing functionality
6. Commit your changes
7. Push to your fork
8. Open a Pull Request

## Running Tests

```bash
bundle install
bundle exec rspec
```

## Changelog Management

This project uses [Changie](https://changie.dev/) for changelog management. When you make a contribution, please add a changelog entry:

### Adding a Changelog Entry

**Option 1: Using Changie CLI (Recommended)**

1. Install changie: https://changie.dev/guide/installation/
2. Run `changie new` from the repository root
3. Follow the prompts to select a change type and describe your change
4. Commit the generated file in `.changes/unreleased/`

**Option 2: Manual Creation**

Create a new YAML file in `.changes/unreleased/` with this format:

```yaml
kind: <Added|Changed|Deprecated|Removed|Fixed|Security>
body: <description of your change>
```

Example:
```yaml
kind: Fixed
body: Fix query building for models with mismatched partition_keys
```

### Change Types

- **Added**: New features
- **Changed**: Changes in existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Removed features
- **Fixed**: Bug fixes
- **Security**: Security-related changes

## Code Style

Please follow the existing code style in the project. We use RuboCop for Ruby code style enforcement.

## Pull Request Guidelines

- Include tests for any new functionality or bug fixes
- Update documentation as needed
- Add a changelog entry (see above)
- Ensure all tests pass
- Keep pull requests focused on a single feature or bug fix
- Write clear, descriptive commit messages

## Questions?

If you have questions or need help, feel free to:
- Open an issue for discussion
- Check existing issues and pull requests
- Review the project documentation

Thank you for contributing!
