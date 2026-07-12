# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A **Contao 5.3 / Symfony 6.4 sandbox installation** (based on the official Contao Demo) running in **DDEV**. It is not a product itself — its purpose is to serve as a test bed for developing Heimrich & Hannot Contao extensions, primarily **FLARE** (`heimrichhannot/contao-flare-bundle`).

Most of the tree is generated/ignored (`vendor/`, `public/`, `var/`, `assets/`, `system/`, `contao-manager/`). What's actually versioned: `composer.json`/`composer.lock`, `.ddev/`, `templates/`, `src/` (App\ namespace, currently empty), `files/`.

## The FLARE bundle setup (important)

- The bundle's source lives on the host at `~/dev/contao-flare-bundle` (github.com/heimrichhannot/contao-flare-bundle). **Edit bundle code there, not in `vendor/`.** That repo has its own AGENTS.md with architecture and conventions.
- `.ddev/docker-compose.mounts.yaml` mounts `${HOST_MOUNT_REPOS_PATH}` (set to `${HOME}/dev` in `.ddev/.env`, which is gitignored) to `/mnt/repos/heimrichhannot` inside the web container.
- `composer.json` declares a **path repository** at `/mnt/repos/heimrichhannot/contao-flare-bundle` with `symlink: true`, required as `dev-main`. So `vendor/heimrichhannot/contao-flare-bundle` is a symlink that only resolves **inside the container**.
- Consequence: **always run Composer via `ddev composer`**, never on the host — the path repository doesn't exist on the host, and the vendor symlink is broken there.
- Bundle code changes take effect immediately (symlink + dev mode), but DCA/service/attribute-registration changes may need `ddev exec bin/console cache:clear`.
- Tests, PHPStan, and linting for FLARE run in *that* repo (its `Makefile` wraps Docker: `make phpstan`, etc.), not here. This sandbox has no test suite.

## Common commands

```bash
ddev start                                  # start the environment -- usually already running
ddev describe                               # show environment details
ddev composer install|update|require ...    # Composer (in-container only, see above)
ddev console <command>                      # Symfony/Contao console
ddev console contao:migrate                 # run DB migrations after schema/DCA changes
ddev console cache:clear                    # clear Symfony/Contao cache
ddev console debug:contao-twig              # inspect Contao Twig template hierarchy
ddev phpmyadmin                             # open phpMyAdmin (custom command)
```

- Site: https://contao53.sandbox.localhost (project TLD is `localhost`, not `ddev.site`); backend at `/contao`.
- Stack: PHP 8.3, Apache-FPM, MariaDB 10.11, Composer 2. Mail is caught by Mailpit (`smtp://127.0.0.1:1025` in-container).
- `vendor/bin/contao-setup` runs automatically on every `composer install`/`update` (post-install/update scripts).

## Structure notes

- `templates/` contains custom Twig overrides using Contao 5's modern Twig template hierarchy: e.g. `templates/content_element/text/slider.html.twig` (a named variant of the `text` element) and `templates/frontend_module/feed_reader/items_only.html.twig`. New overrides follow this `templates/<type>/<name>/<variant>.html.twig` scheme.
- `src/` is autoloaded as `App\` for sandbox-local PHP (controllers, DCA adjustments, etc.) if needed.
- Contao DCA/config additions specific to this app would go in `contao/` (Contao convention) or `config/` — neither exists yet; the demo content and configuration live in the database.
