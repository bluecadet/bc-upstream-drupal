# Quick Validation Checklist for Drupal Core Updates

Use this checklist when updating Drupal core versions in this upstream.

## Pre-Update Checklist

- [ ] Check [Drupal.org releases](https://www.drupal.org/project/drupal/releases) for the target version
- [ ] Review release notes for breaking changes
- [ ] Check if it's a security release
- [ ] Verify PHP version compatibility
- [ ] Ensure Pantheon supports the required PHP version

## Update Process

- [ ] Update all four core packages in `composer.json` to the same version:
  - `drupal/core-composer-scaffold`
  - `drupal/core-project-message`
  - `drupal/core-recommended`
  - `drupal/core-dev`

## Validation Steps

### Local Validation (Quick)
```bash
# 1. Validate syntax
composer validate

# 2. Check what would be updated (if you have network access)
composer update --dry-run

# 3. Check for security issues (if composer audit is available)
composer audit
```

### Test Installation (Recommended)
```bash
# In a clean directory
composer create-project bluecadet/bc-upstream-drupal:dev-your-branch test-install
cd test-install
drush status  # Verify Drupal version is 11.3.5+
```

### Pantheon Testing (Comprehensive)
```bash
# Create a test site on Pantheon from this upstream
# Or update an existing test site and verify:
terminus drush <site>.<env> -- status
terminus drush <site>.<env> -- updatedb
terminus drush <site>.<env> -- cr
```

## Documentation Updates

- [ ] Update `CHANGELOG.md` with the version change and date
- [ ] Update PR description with rationale for the version update
- [ ] Reference any relevant security advisories or Drupal.org issues

## Post-Update Verification

- [ ] Composer validation passes
- [ ] No dependency conflicts
- [ ] Fresh install completes successfully
- [ ] Code review passes (if automated)
- [ ] Security scan passes (if automated)

## Common Gotchas

- ❌ Don't update only some core packages - update all four together
- ❌ Don't skip testing - even "minor" updates can have issues
- ❌ Don't forget to update CHANGELOG.md
- ✅ Do use the same version constraint for all core packages
- ✅ Do test on Pantheon infrastructure if possible
- ✅ Do check for security advisories first

## Need Help?

See [TESTING.md](TESTING.md) for comprehensive testing procedures and troubleshooting.
