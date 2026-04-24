# Bluecadet Base Drupal Site

Forked from [Pantheon Drupal 11 Upstream](pantheon-upstreams/drupal-composer-managed)

This is Bluecadet's recommended starting point for forking new [Drupal](https://www.drupal.org/) upstreams
that work on Pantheon Platform's Integrated Composer build process.

For more information, please visit the Integrated Composer Pantheon
documentation: https://pantheon.io/docs/integrated-composer

## Getting Started

- [Bluecadet Base Drupal Site](#bluecadet-base-drupal-site)
  - [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Drupal Core Version Management](#drupal-core-version-management)
    - [When to Update Drupal Core Versions](#when-to-update-drupal-core-versions)
    - [How to Update](#how-to-update)
  - [Setup \& Installing Dependencies](#setup--installing-dependencies)
  - [Developing Locally](#developing-locally)
    - [Adjusting PHP Settings Per-Project](#adjusting-php-settings-per-project)
  - [Pantheon Environments](#pantheon-environments)
  - [Branching Strategy \& CI/CD](#branching-strategy--cicd)
    - [Branch Types](#branch-types)
    - [Workflow](#workflow)
      - [Typical Feature Development](#typical-feature-development)
      - [Persistent Branches](#persistent-branches)
      - [Hotfixes](#hotfixes)
    - [CI/CD Process ](#cicd-process-)
  - [Commit Guidelines](#commit-guidelines)
    - [Commit Message Format](#commit-message-format)
    - [Types](#types)
    - [Scopes](#scopes)
    - [Breaking Changes](#breaking-changes)
    - [Examples](#examples)
  - [Code Quality Checks](#code-quality-checks)
    - [Quick Start](#quick-start)
    - [Available Checks](#available-checks)
    - [Output Modes](#output-modes)
    - [Git Hooks](#git-hooks)
    - [Documentation](#documentation)
    - [Examples](#examples-1)
  - [Drupal Services (and local development)](#drupal-services-and-local-development)
  - [New Relic](#new-relic)
  - [Starting a New Project ](#starting-a-new-project-)
  - [Transitioning a Project between "development", "pre-launch", and "post-launch"](#transitioning-a-project-between-development-pre-launch-and-post-launch)
  - [Merging updates from your upstream.](#merging-updates-from-your-upstream)

## Prerequisites

Before you begin, ensure you have the following installed:

- [DDEV](https://ddev.readthedocs.io/en/stable/) + Docker Desktop or [Orbstack](https://orbstack.dev/)

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

2. Test the changes by running the project's standard validation steps locally, including dependency installation, Drupal updates as needed, and a smoke test of key site functionality

3. Update [CHANGELOG.md](CHANGELOG.md) with the change

4. Create a pull request with clear rationale for the version update

Before opening a pull request, confirm the updated core version installs cleanly and does not break local development workflows.

## Setup & Installing Dependencies

1. Clone the repository:
   ```sh
   git clone [repository-url]
   cd [project-directory]
   ```

2. Start the DDEV environment:
   ```sh
   ddev start
   ```

3. Install Composer dependencies:
   ```sh
   ddev composer install
   ```

4. Import the database and files (download from Pantheon dev environment, latest backup):
   ```sh
   ddev pull pantheon --environment=PANTHEON_SITE=bc-base-drupal,PANTHEON_ENVIRONMENT=live
   ```

5. Import configuration:
   ```sh
   ddev drush cim
   ```

6. Clear the cache:
   ```sh
   ddev drush cr
   ```

7. Add Storybook environment variable to point to local Drupal instance:
   ```sh
   cp .env.example .env
   ```

## Developing Locally

1. Start the DDEV environment:
   ```sh
   ddev start
   ```

2. Run asset compilation in development mode from the project root:
   ```sh
   ddev npm run dev         # builds CSS via PostCSS, JS
   ddev npm run watch       # builds stories.twig into stories.json
   ddev npm run storybook   # starts Storybook on port 3000, must have Drupal running
   ```
   **Note:** When you run `ddev npm run dev`:
   - Asset generation tasks will automatically run.
   - BrowserSync will be available, enabling automatic page reloads when changes are made.
   - You'll see output similar to this:
     ```
     [bldr] Proxying: https://bc-base-drupal.ddev.site
     [bldr] Access URLs:
           Local: https://localhost:3000
           External: https://192.168.4.30:3000
     ```
   - Use these URLs to access your site with live reloading.

3. Access your local site at `https://[project-name].ddev.site`

4. To build for production:
   ```sh
   ddev npm run build
   ddev npm run build-storybook
   ```

5. Run code quality checks before committing:
   ```sh
   .ci/scripts/quality/run-all-checks.sh
   ```

   See [Code Quality Scripts](.ci/scripts/quality/README.md) for detailed documentation.

  TODO: see ()[]
6. Disable Drupal caches for local development:

### Adjusting PHP Settings Per-Project

The default DDEV configuration includes PHP settings in `.ddev/php/pantheon.ini` that are aligned with Pantheon hosting environments to ensure parity between local development and production.

**Default settings:**
- `memory_limit = 1024M` - Matches Pantheon Performance Medium+ plans
- `max_execution_time = 120` - Matches Pantheon's fixed execution time limit
- `upload_max_filesize = 100M` - Reasonable limit for local development
- `post_max_size = 100M` - Must be >= upload_max_filesize
- `max_input_vars = 1000` - Matches Pantheon defaults

**To adjust settings for your project:**

1. Edit `.ddev/php/pantheon.ini` or create additional `.ini` files in `.ddev/php/`
2. Adjust values to match your Pantheon plan or project requirements:
   ```ini
   ; Example: Match Pantheon Basic plan (256M memory)
   memory_limit = 256M

   ; Example: Increase upload limits for media-heavy projects
   upload_max_filesize = 256M
   post_max_size = 256M
   ```
3. Restart DDEV to apply changes:
   ```sh
   ddev restart
   ```
4. Verify settings:
   ```sh
   ddev ssh
   php -i | grep memory_limit
   php -i | grep upload_max_filesize
   php -i | grep max_execution_time
   ```

**Important Notes:**
- These settings only affect your local DDEV environment
- On Pantheon, PHP settings are managed by the platform and based on your plan:
  - **Memory limit**: 256M (Basic), 512M (Performance Small), 1024M (Performance Medium+)
  - **Max execution time**: 120 seconds (fixed, cannot be changed)
  - **Upload limits**: Platform-managed for security and resource management
- Always ensure local settings match or are more restrictive than your production Pantheon environment
- Reference: [Pantheon PHP Platform Support](https://docs.pantheon.io/guides/platform-considerations/php-platform)

## Pantheon Environments

| Pantheon | Description |
| --- | --- |
| [`dev`](https://dev-bc-base-drupal.pantheonsite.io/) | Development environment where new features and bug fixes are initially deployed and tested. This environment is used for ongoing development work and is not stable for client review. |
| [`test`](https://test-bc-base-drupal.pantheonsite.io/) | Staging environment used for quality assurance and user acceptance testing. It can be for temporarily testing content but should not used for staging content that will be deployed to live. It is important that PMs and the client understand that the environment can be wiped and reset at any time. (See Persistent Branches below if a more stale environment is needed.) |
| [`live`](https://live-bc-base-drupal.pantheonsite.io/) | Production environment that hosts the public-facing website. This is the stable, live version of the site accessible to all users. |

We use Pantheon's multidev environments to preview changes from pull requests and sprint release branches before they are merged into the main development workflow.

Persistent branches: If a client needs a stable environment for testing content, a persistent branch can be used to create a multidev that will not get deleted automatically. The database and files on this branch should not be re-synced from live without explicit approval from the client.

## Branching Strategy & CI/CD

Our project follows a structured branching strategy to manage development, releases, and deployments. Here's an overview of our workflow:

### Branch Types

- `master`: The main branch representing the production-ready state.
- `develop`: The primary development branch where features are integrated.
- `release/[release name]`: (Optional) Sprint release branches for integrating features before merging to `develop`.
- `feature/[ticket#]-[label]`: Feature branches for individual tickets or user stories. Our CI/CD process will automatically create a new Pantheon multidev when a PR is created, and will automatically delete the multidev when the PR is merged.
- `persist/[name]`: Persistent branches for long-running features or experiments. Our CI/CD process will automatically create a new Pantheon multidev when a PR is created, but will not delete it when the PR is merged. You will have to manually delete the multidev when you are done with it.


### Workflow

<!-- TODO: write this out in mermaid.js -->

<!-- [![](https://mermaid.ink/img/pako:eNqtUj1vgzAQ_SvWzZCCAZMwtpUydWm2isWFC1gBG5mjbRrlv9chqZSkoepQT6d79z58uh0UpkTIwPf9XJOiBjO2VMTurdRFrXTFVmQlYbVl74pqMxBbdVZpYs_YoOyRPQyU65FeKVpa2dW5Zu4Vpm0VHevXUY2V-IaN6U54jcXmoHfZ_clao6TB4h2pYoPkh1f0CfhM56K-6dqirXBC6XYK_nsK_m8pro1aqfT5-B92evpBh7ZXPU3tcQIeU4MHzs1Zl-5UdgckB6qxxRwyVzaqqimHXO_doBzIrLa6gIzsgB4MXenu51HJysr2u9lJDdkOPiCLhJjx-TyNkjCNRRALD7aQhUk6S4VIozgSMeeLmO89-DTGCQSzNOJBEgQi5DyZL8LQAywVGft0POXC6LWqYDR5GSkHz_0Xmq72CA?type=png)](https://mermaid.live/edit#pako:eNqtUj1vgzAQ_SvWzZCCAZMwtpUydWm2isWFC1gBG5mjbRrlv9chqZSkoepQT6d79z58uh0UpkTIwPf9XJOiBjO2VMTurdRFrXTFVmQlYbVl74pqMxBbdVZpYs_YoOyRPQyU65FeKVpa2dW5Zu4Vpm0VHevXUY2V-IaN6U54jcXmoHfZ_clao6TB4h2pYoPkh1f0CfhM56K-6dqirXBC6XYK_nsK_m8pro1aqfT5-B92evpBh7ZXPU3tcQIeU4MHzs1Zl-5UdgckB6qxxRwyVzaqqimHXO_doBzIrLa6gIzsgB4MXenu51HJysr2u9lJDdkOPiCLhJjx-TyNkjCNRRALD7aQhUk6S4VIozgSMeeLmO89-DTGCQSzNOJBEgQi5DyZL8LQAywVGft0POXC6LWqYDR5GSkHz_0Xmq72CA) -->

#### Typical Feature Development

1. Development starts on the `develop` branch.
2. Feature branches (`feature/ticket-X`) are created from the `develop` branch.
   - A pull request (PR) created from a feature branch automatically creates a new Pantheon multidev, which will be used for internal QA and client UAT.
3. As work on feature branches is approved, the branches are merged into `develop`.
4. `develop` is then merged into `master` (or `main`) for deployment to Pantheon `dev` environment.

#### Persistent Branches

Persistent branches (`persist/[name]`) can be used for long-term development that spans multiple sprints; the associated Pantheon multidev environment will not be auto-cleaned up.

#### Hotfixes

1. Create a feature branch off of `master`, to test the hotfix
2. After approval, merge the feature branch into `master` and deploy.
3. Then merge `master` back into develop.

### CI/CD Process <!-- Needs to be cleaned up when fixing up our CI process -->

Our CI/CD pipeline is triggered by GitHub Actions and deploys to Pantheon:

- Push to `master`: Triggers the "Deploy to Pantheon" workflow, deploying to the Pantheon `dev` environment.
- Pull Request to `develop`: Triggers a workflow that creates a multidev for the branch that is being compared against `develop`.

<!-- TODO Needs CI documentation (ML-155) -->
TODO: For more information refer to our CI [documentation]().

## Commit Guidelines

We follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for our commit messages. This format improves readability and enables automated versioning and changelog generation.

### Commit Message Format

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `chore`: Updating build tasks, package manager configs, etc; no production code change
- `perf`: Changes that bring measurable performance improvements
- `docs`: Documentation changes
- `revert`: Roll back a previous commit
- `style`: Formatting, missing semicolons, etc; no code change
- `refactor`: Refactoring production code
- `ci`: Changes to CI/CD configurations or workflows
- `test`: Adding or refactoring tests; no production code change

Added by BC:
- `wip`: Work in progress; code is potentially unstable (used in a non-merge squashing context)

### Scopes

The scope provides additional contextual information.

- The scope is an optional part
- Allowed scopes vary and are typically defined by the specific project
- Do not use issue identifiers as scopes

| Area                      | Example Scopes                                                                            |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| **Drupal Core / Backend** | `module`, `theme`, `config`, `entity`, `hook`, `service`, `routing`, `migration`, `drush` |
| **Frontend / Theme**      | `css`, `js`, `template`, `component`, `accessibility`, `navigation`                       |
| **Tooling & DevOps**      | `build`, `deps`, `ddev`, `docker`, `env`                                                  |
| **Project / Process**     | `security`, `release`                                                                     |

### Breaking Changes

A commit that has a footer "BREAKING CHANGE:", or appends a "!" after the type/scope, introduces a breaking API change (correlating with MAJOR in Semantic Versioning).

### Examples

```sh
feat(navigation): add new dropdown menu
```

```sh
fix(form): resolve Safari submission issue
```

```sh
fix(#123): resolve Safari submission issue
```

```sh
fix(ML-123): resolve Safari submission issue
```

For more detailed guidelines, refer to the [Conventional Commits specification](https://www.conventionalcommits.org/) and [Conventional Commits cheatsheet](https://www.bavaga.com/blog/2025/01/27/my-ultimate-conventional-commit-types-cheatsheet/).

## Code Quality Checks

This repository includes automated code quality check scripts that can be run locally, in git hooks, or in CI/CD pipelines.

### Quick Start

Run all quality checks:
```sh
.ci/scripts/quality/run-all-checks.sh
```

Run individual checks:
```sh
.ci/scripts/quality/check-npm-lint.sh      # JavaScript/TypeScript linting (Biome)
.ci/scripts/quality/check-phpcs.sh         # PHP coding standards
.ci/scripts/quality/check-phpstan.sh       # Static analysis
.ci/scripts/quality/check-drupal-check.sh  # Deprecated code detection
```

### Available Checks

- **JavaScript/TypeScript Linting** - Uses Biome via `npm run lint`
- **PHPCS** - Drupal and DrupalPractice coding standards
- **PHPStan** - Static analysis for type errors and bugs
- **drupal-check** - Detects deprecated Drupal API usage

### Output Modes

Scripts support multiple output formats for different environments:

- `--output terminal` - Colored output for local development (default)
- `--output github` - GitHub Actions formatted output with step summaries
- `--output json` - JSON output for programmatic parsing
- `--output plain` - Plain text without colors

### Git Hooks

Install a pre-push hook to automatically run checks:

```sh
cp .ci/scripts/quality/examples/pre-push.sh .git/hooks/pre-push
chmod +x .git/hooks/pre-push
```

### Documentation

- [📖 Full Documentation](.ci/scripts/quality/README.md) - Complete guide with all options and examples
- [🚀 Quick Start Guide](.ci/scripts/quality/QUICKSTART.md) - Get started quickly
- [📋 Examples](.ci/scripts/quality/examples/README.md) - Git hooks and GitHub Actions examples
- [🎯 Decision Document](.ci/scripts/quality/DECISION.md) - Why we chose shell scripts

### Examples

```sh
# Run all checks, continue even if one fails (see all issues)
.ci/scripts/quality/run-all-checks.sh --continue

# Run checks for git hook (warn but don't block)
.ci/scripts/quality/run-all-checks.sh --no-fail

# Run only PHP checks
.ci/scripts/quality/run-all-checks.sh --skip-npm

# Check only custom modules
.ci/scripts/quality/check-phpcs.sh --modules-only

# Verbose output for debugging
.ci/scripts/quality/check-phpstan.sh --verbose
```

## Drupal Services (and local development)

Please refer to [web/sites/SERVICES_README.md](web/sites/SERVICES_README.md).

## New Relic

New Relic should be enabled on all Pantheon sites.

- Go to the LIVE env in the Pantheon Dashboard.
- On the left side, there should be a tab for "New Relic". Currently it is last in the list.
- Click "Activate New Relic Pro" button.

Create a New Relic API key and connect it to the project:

1. [Activate New Relic Pro](https://pantheon.io/docs/new-relic/#activate-new-relic-pro) within your site Dashboard.
2. Get a [New Relic User Key](https://one.newrelic.com/launcher/api-keys-ui.api-keys-launcher)
    - This key should be saved in 1Password
3. Using [Terminus Secrets Manager Plugin](https://github.com/pantheon-systems/terminus-secrets-manager-plugin), set a site secret for the API key just created (e.g. `new_relic_api_key`, if you name it something else, make sure to update in the Terminus command below, and within the `new_relic_deploy.php` script in Step 4). Make sure type is `runtime` and scope contains `web`.

  ```sh
    terminus secret:site:set mysite new_relic_api_key --scope=web --type=runtime MY_API_KEY_HERE
  ```

4. Add the example `new_relic_deploy.php` script to the `/private/scripts/` directory of your code repository.
5. Add a Quicksilver operation to your `pantheon.yml` to fire the script after a deploy.

  ```yaml
  ...

    sync_code:
      after:
        -
          type: webphp
          description: Log to New Relic
          script: private/new_relic_deploy.php

  ...
  ```

6. Test a deploy out!

References:

- https://docs.pantheon.io/guides/new-relic/new-relic-quicksilver
- https://github.com/pantheon-systems/quicksilver-examples/tree/main/new_relic_deploy

## Starting a New Project <!-- Needs Verification on next project Start -->

The steps to take when creating a new web project at Bluecadet

- Decide on a project machine name. For ease we will use this across all entities, e.g. `project-name-example`.
- Create a new repo on github.com, using the project name.
  - TODO: detail directions.
  - Setup Repo Secrets
    - https://github.com/bluecadet/[PROJECT NAME]/settings/secrets/actions
    - Add `PANTHEON_REPO` -> TODO: document these with CI updates
    - Add `PROJECT_PAT` -> TODO: document these with CI updates
- Create a Pantheon Project, using the project name.
  - TODO: detail directions.
  - Start DEV site
  - Init TEST site
  - Init LIVE site
- Create a clean code base from bluecadet/bc-base-drupal repo by cloning.
  - Clone repo
  ```bash
  # Clone last commit of latest from the develop branch.
  git clone -b develop --single-branch --depth 1 git@github.com:bluecadet/bc-base-drupal.git [PROJECT NAME]

  # Remove ./.git to remove all history and connection to bluecadet/bc-base-drupal.
  rm -rf [PROJECT NAME]/.git/
  ```
  - Update project specific references (should be able to do a quick search on 'bc-base-drupal')
    - ./.projectconfig.cjs
    - ./bldr.local.config.js
    - ./package.json
    - ./README.md
    - ./.ddev/config.yml
  - Setup new repo
  ```
  cd [PROJECT NAME]
  git init
  git commit .
  git branch -M develop
  git remote add origin git@github.com:bluecadet/[PROJECT NAME].git
  git push -u origin develop
  ```
  - Check default branch for the repo is "develop"
    - If following these directions, it should be.
    - Goto https://github.com/bluecadet/[PROJECT NAME]/settings
    - There should be a section labeled "Default branch"
  - TODO: Check ddev is working...
    - Pull LIVE
  - TODO: Check PR/GHA are firing correctly.
    - ...
    - Verify your changes are on panth dev

## Transitioning a Project between "development", "pre-launch", and "post-launch"

We should be treating a few things differently after development of a site is complete and we have launched.

1. In pre-launch or post-launch we should update quicksilver hooks.
   1. in `./pantheon.yml, remove the "Run update database" and "Import configuration from .yml files" entries so these have to be manually done on production sites.

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
