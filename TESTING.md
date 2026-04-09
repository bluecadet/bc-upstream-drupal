# Testing and Validation Guide

This document provides guidance on testing and validating changes to this Drupal upstream, particularly when updating Drupal core versions.

## Validating Drupal Core Version Updates

When updating Drupal core version constraints in `composer.json`, follow these steps to ensure the changes are safe and functional:

### 1. Validate Composer Configuration

```bash
# Check that composer.json syntax is valid
composer validate

# Preview what will be updated without making changes
composer update --dry-run

# Check for any dependency conflicts
composer why-not drupal/core-recommended 11.3.5
```

### 2. Test Installation in a Clean Environment

```bash
# Create a fresh test installation
mkdir /tmp/test-drupal-upstream
cd /tmp/test-drupal-upstream

# Copy composer.json from the updated repository
cp /path/to/bc-upstream-drupal/composer.json .

# Install dependencies
composer install

# Verify the installed Drupal core version
composer show drupal/core-recommended
```

### 3. Verify Drupal Version with Drush

If you have a working Drupal installation:

```bash
# Check Drupal version
drush status

# Look for the "Drupal version" field - it should be 11.3.5 or higher
```

### 4. Check for Security Updates

```bash
# Check if the specified version includes important security updates
# Visit: https://www.drupal.org/project/drupal/releases

# Or use composer to check available updates
composer outdated "drupal/*"
```

### 5. Test on Pantheon

Since this is a Pantheon upstream, test on Pantheon infrastructure:

```bash
# Create a new site from this upstream on Pantheon
# Or update an existing test site

# SSH into the Pantheon environment
# Verify Drupal core version
drush status

# Run database updates if needed
drush updatedb

# Clear caches
drush cr
```

## What to Look For When Updating Drupal Core

### Version Constraint Syntax

- Use `^` (caret) for minor version updates: `^11.3.5` allows 11.3.5, 11.4.0, 11.5.0, etc., but not 12.0.0
- Use `~` (tilde) for patch updates only: `~11.3.5` allows 11.3.5, 11.3.6, etc., but not 11.4.0
- For upstreams, `^` is preferred to allow automatic security updates

### Core Packages to Update Together

When updating Drupal core, always update these packages in sync:
- `drupal/core-composer-scaffold`
- `drupal/core-project-message`
- `drupal/core-recommended`
- `drupal/core-dev` (in require-dev)

### Breaking Changes

Check the Drupal release notes for:
- **Deprecated APIs** being removed
- **Minimum PHP version** requirements
- **Database version** requirements (MySQL/MariaDB/PostgreSQL)
- **Breaking changes** in APIs or behavior

Resources:
- Drupal release notes: https://www.drupal.org/project/drupal/releases
- Drupal 11 change records: https://www.drupal.org/list-changes/drupal/published?version=11.x

### Security Considerations

When updating to a specific version:
1. Check if it's a **security release** (marked with red "Security" badge)
2. Review security advisories: https://www.drupal.org/security
3. Ensure the minimum version includes all relevant security patches

### Pantheon-Specific Considerations

- **PHP Version**: Check that `config.platform.php` in composer.json matches Pantheon's supported versions
- **Pantheon Packages**: Ensure `pantheon-systems/drupal-integrations` is compatible with the Drupal version
- **Scaffold Files**: Some Drupal updates modify scaffold files that may conflict with Pantheon customizations

## Continuous Integration Testing

### Automated Tests to Add

Consider adding these automated checks:

```yaml
# Example GitHub Actions workflow
name: Validate Composer

on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          
      - name: Validate composer.json
        run: composer validate --no-check-all --no-check-publish
        
      - name: Install dependencies
        run: composer install --prefer-dist --no-progress
        
      - name: Check Drupal core version
        run: |
          VERSION=$(composer show drupal/core-recommended --format=json | jq -r '.versions[]' | head -1)
          echo "Installed Drupal core version: $VERSION"
```

## Manual Testing Checklist

When making a Drupal core version update, verify:

- [ ] `composer.json` syntax is valid (`composer validate`)
- [ ] All core packages are updated to the same version
- [ ] Dependencies can be resolved (`composer update --dry-run`)
- [ ] Fresh installation works (`composer install` in clean directory)
- [ ] Drupal version is correct (`drush status` or `composer show`)
- [ ] No security vulnerabilities in dependencies (`composer audit`)
- [ ] PHP version compatibility is maintained
- [ ] CHANGELOG.md is updated with the change
- [ ] PR description explains why this version was chosen

## Common Issues and Solutions

### Issue: Dependency Conflicts

```
Your requirements could not be resolved to an installable set of packages.
```

**Solution**: Check if contrib modules or other dependencies are compatible with the new Drupal version. You may need to:
- Update incompatible modules
- Wait for compatible versions
- Adjust version constraints

### Issue: Scaffold File Conflicts

```
[RuntimeException] Scaffold file conflict on...
```

**Solution**: Review `extra.drupal-scaffold.file-mapping` in composer.json and ensure Pantheon-specific overrides are maintained.

### Issue: PHP Version Incompatibility

```
drupal/core-recommended requires php ^8.x but your PHP version does not satisfy that requirement.
```

**Solution**: Update `config.platform.php` in composer.json to match Drupal's requirements and ensure Pantheon environments support this PHP version.

## Additional Resources

- [Drupal Composer Documentation](https://www.drupal.org/docs/develop/using-composer)
- [Pantheon Composer Documentation](https://pantheon.io/docs/guides/composer)
- [Drupal Security Advisories](https://www.drupal.org/security)
- [Drupal 11 System Requirements](https://www.drupal.org/docs/system-requirements)
