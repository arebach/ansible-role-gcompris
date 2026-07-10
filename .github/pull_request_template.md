## Summary

Brief description of what this PR does and why.

## Type of change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing behavior to not work as expected)
- [ ] Documentation update
- [ ] CI/CD improvement
- [ ] Refactor (no functional changes)

## Related issues

Fixes #(issue number)
Closes #(issue number)

## Changes made

- Change 1
- Change 2
- Change 3

## Testing

- [ ] `ansible-lint --profile production` passes clean
- [ ] `molecule test` passes (Docker-based static config verification)
- [ ] Tested against a real Raspberry Pi (if applicable)

### Test output

```
Paste ansible-lint or molecule output here
```

## Checklist

- [ ] My code follows the project's coding style (ansible-lint production profile)
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation (README.md, CHANGELOG.md)
- [ ] I have added variables to `defaults/main.yml` with documentation comments
- [ ] My changes generate no new ansible-lint warnings
- [ ] I have tested the role against a real Raspberry Pi or noted why this was not possible
- [ ] Any dependent changes have been merged and published

## Breaking changes

If this is a breaking change, describe:
- What breaks
- How users should migrate
- Whether a version bump (major/minor) is needed

## Additional notes

Any other information relevant to this PR.
