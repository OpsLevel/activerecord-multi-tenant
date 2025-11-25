# Changelog Management with Changie

This directory contains changelog entries managed by [changie](https://changie.dev/).

## Adding a New Changelog Entry

When submitting a pull request, you should add a changelog entry describing your changes:

### Using Changie CLI (Recommended)

1. Install changie: https://changie.dev/guide/installation/
2. Run `changie new` from the repository root
3. Follow the prompts to select a change type and describe your change
4. Commit the generated file in `.changes/unreleased/`

### Manual Creation

Alternatively, create a new YAML file in `.changes/unreleased/` with the following format:

```yaml
kind: <Added|Changed|Deprecated|Removed|Fixed|Security>
body: <description of your change>
```

Example:
```yaml
kind: Fixed
body: Fix query building for models with mismatched partition_keys
```

## Change Types

- **Added**: New features
- **Changed**: Changes in existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Removed features
- **Fixed**: Bug fixes
- **Security**: Security-related changes

## Releasing

When preparing a release, maintainers will run:
```bash
changie batch <version>
changie merge
```

This consolidates all unreleased changes into the CHANGELOG.md file.
