# Composer-enabled Drupal template

This is Pantheon's recommended starting point for forking new [Drupal](https://www.drupal.org/) upstreams
that work with the Platform's Integrated Composer build process. It is also the
Platform's standard Drupal 9 upstream.

Unlike with earlier Pantheon upstreams, files such as Drupal Core that you are
unlikely to adjust while building sites are not in the main branch of the
repository. Instead, they are referenced as dependencies that are installed by
Composer.

For more information and detailed installation guides, please visit the
Integrated Composer Pantheon documentation: https://pantheon.io/docs/integrated-composer

## Drupal Core Version Management

This upstream currently requires **Drupal 11.3.5 or higher**. The version constraints use the caret (`^`) operator, which allows:
- ✅ Patch updates (e.g., 11.3.6, 11.3.7)
- ✅ Minor updates (e.g., 11.4.0, 11.5.0)
- ❌ Major updates (e.g., 12.0.0)

This ensures automatic security updates while maintaining API compatibility.

### When to Update Drupal Core Versions

Update the minimum Drupal core version when:
- **Security releases** require a minimum version
- **Bug fixes** in newer versions are needed for stability
- **New features** in Drupal core are required by the upstream
- Maintaining compatibility with Pantheon platform updates

### How to Update

1. Update all core packages together in `composer.json`:
   - `drupal/core-composer-scaffold`
   - `drupal/core-project-message`
   - `drupal/core-recommended`
   - `drupal/core-dev`

2. Test the changes (see [TESTING.md](TESTING.md) for detailed procedures)

3. Update [CHANGELOG.md](CHANGELOG.md) with the change

4. Create a pull request with clear rationale for the version update

See [TESTING.md](TESTING.md) for comprehensive validation procedures.

## Contributing

Contributions are welcome in the form of GitHub pull requests. However, the
`pantheon-upstreams/drupal-composer-managed` repository is a mirror that does not
directly accept pull requests.

Instead, to propose a change, please fork [pantheon-systems/drupal-composer-managed](https://github.com/pantheon-systems/drupal-composer-managed)
and submit a PR to that repository.

## Merging updates from your upstream.
To manually apply changes from the external `upstream` repository (branch: `main`):

1. Add the upstream repository as a remote (if not already added):
   ```sh
   git remote add upstream https://github.com/bluecadet/upstream.git
   ```
   Replace `bluecadet` with the correct organization or user name.

2. Fetch the latest changes from upstream:
   ```sh
   git fetch upstream
   ```

3. Merge the upstream main branch into your current branch:
   ```sh
   git merge upstream/main
   ```

4. Resolve any merge conflicts if they occur, then commit the result.

5. Push your changes to your fork or origin as needed.

For more details, see the [GitHub documentation on syncing a fork](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/syncing-a-fork).
