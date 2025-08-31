# Run Laravel Pint (GitHub Action)

A simple composite action to run [Laravel Pint](https://laravel.com/docs/pint) with caching and flexible configuration.

It will:
- Check out your repository (actions/checkout@v4) with fetch-depth: 2.
- Set up PHP (default 8.3) and Composer v2.
- Install dependencies using `ramsey/composer-install` with cache.
- Cache Pint's `.pint.cache` file keyed by OS, `pint.json`, and `composer.lock`.
- Show the Pint version and run Pint in check mode by default.

## Usage

Basic usage (checks style, fails the job if there are violations):

```yaml
name: Lint (Pint)

on:
  push:
  pull_request:

jobs:
  pint:
    runs-on: ubuntu-latest
    steps:
      - name: Run Laravel Pint
        uses: carvemerson/pint-action@v1 
```

Using inputs and running from a subdirectory:

```yaml
jobs:
  pint:
    runs-on: ubuntu-latest
    steps:
      - name: Run Laravel Pint
        uses: carvemerson/pint-action@v1
        with:
          php-version: '8.3'
          working-directory: ./backend
          pint-args: '--test --ansi'   # default; change to e.g. "--dirty" to only scan changed files
          composer-options: '--no-scripts'
          continue-on-error: 'false'   # set to 'true' to not fail the Pint step on style violations
```

## Inputs

- php-version
  - Description: PHP version used to run Pint
  - Required: false
  - Default: 8.3

- working-directory
  - Description: Directory containing composer.json and Pint config
  - Required: false
  - Default: .

- pint-args
  - Description: Extra arguments passed to Pint (e.g., --dirty). By default the action runs in check mode without modifying files.
  - Required: false
  - Default: `--test --ansi`

- composer-options
  - Description: Extra options for `composer install` (e.g., `--no-scripts`)
  - Required: false
  - Default: empty

- continue-on-error
  - Description: Whether to continue on error for the Pint run step in this action.
  - Required: false
  - Default: false


## Notes

- You do not need to add a separate actions/checkout step in your workflow; this action already checks out your repository (fetch-depth: 2).
- Requirements: your project must include Laravel Pint as a dependency, typically via `composer require laravel/pint --dev`. The action expects `vendor/bin/pint` to exist within the working directory.
- The cache step stores `.pint.cache` to speed up subsequent runs; it is keyed by OS and the hashes of `pint.json` and `composer.lock`.
- Input continue-on-error controls whether the Pint step continues on error within this action. Alternatively, you can use GitHub Actions' step-level `continue-on-error` on the calling workflow step as shown below.
- If you want Pint to fix files (not just check), override `pint-args`, e.g.: `pint-args: '--ansi'`.
- Supported runners: Linux, macOS, and Windows should work; examples use `ubuntu-latest`.

## Example: run only on pull requests and only for changed files

```yaml
on:
  pull_request:

jobs:
  pint:
    runs-on: ubuntu-latest
    steps:
      - uses: carvemerson/pint-action@v1
        with:
          pint-args: '--dirty --ansi'
```

## Example: allow Pint failures without failing the job

Use GitHub Actions' standard continue-on-error at the step level:

```yaml
jobs:
  pint:
    runs-on: ubuntu-latest
    steps:
      - uses: carvemerson/pint-action@v1
        continue-on-error: true  # do not fail the step if Pint finds issues
```

## Versioning

Pin to a specific version/tag to ensure stability, for example:

```yaml
uses: carvemerson/pint-action@v1
```
