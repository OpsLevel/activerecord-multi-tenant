## Description

<!-- Please include a summary of the changes and the related issue. -->

## Type of Change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Other (please describe):

## Checklist

- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] I have added necessary documentation (if appropriate)
- [ ] I have run the test suite and all tests pass
- [ ] **I have added a changelog entry using [changie](https://changie.dev/)** (run `changie new` to create one)

## Additional Context

<!-- Add any other context about the pull request here. -->

---

### How to Add a Changelog Entry

This project uses [changie](https://changie.dev/) for changelog management. To add a changelog entry:

1. Install changie: https://changie.dev/guide/installation/
2. Run `changie new` and follow the prompts
3. Commit the generated file in `.changes/unreleased/`

Alternatively, you can manually create a file in `.changes/unreleased/` with the format:
```yaml
kind: <Added|Changed|Deprecated|Removed|Fixed|Security>
body: <description of your change>
```
