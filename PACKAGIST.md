# Publishing MicroPHP Core and the Application Skeleton

MicroPHP is now designed as two Composer packages:

| Package | Composer type | Repository purpose |
| --- | --- | --- |
| `yacho/microphp-core` | `library` | Framework classes and helper functions |
| `yacho/microphp` | `project` | Ready-to-run application skeleton |

The package names use the existing Composer vendor `yacho`; the GitHub account
is `y4ch0`. Those names may differ. If `yacho` is not the Packagist vendor you
intend to own, change both package names and the skeleton dependency before the
first Packagist submission. Composer package names cannot be cleanly renamed
after users begin depending on them.

## 1. Create the core repository

Create an empty public GitHub repository named `microphp-core`. Copy the
*contents* of `PreparationForDivide/` into the repository root, so its
`composer.json`, `src/`, `tests/`, and `README.md` are at the top level. Do not
commit the wrapping `PreparationForDivide` directory.

The resulting repository should begin like this:

```text
microphp-core/
├── .github/workflows/tests.yml
├── src/
├── tests/
├── composer.json
├── LICENSE
├── phpunit.xml
└── README.md
```

Check that the `support` URLs in `composer.json` match the final public GitHub
URL. Then, from the new repository, run:

```bash
composer validate --strict
composer install
composer dump-autoload --optimize --strict-psr
composer test
git add .
git commit -m "Prepare MicroPHP core package"
git branch -M main
git remote add origin https://github.com/y4ch0/microphp-core.git
git push -u origin main
git tag -a v1.0.0 -m "MicroPHP Core 1.0.0"
git push origin v1.0.0
```

Do not add a `version` property to `composer.json`. Packagist derives versions
from Git tags. Use semantic versions: patch releases for fixes, minor releases
for backward-compatible features, and a new major version for breaking API
changes.

## 2. Publish the core package on Packagist

1. Sign in at [Packagist](https://packagist.org/) with GitHub.
2. Choose **Submit**, enter `https://github.com/y4ch0/microphp-core`, and submit.
3. Confirm that Packagist shows `yacho/microphp-core` version `1.0.0` and that
   its dependency list includes PHP, JSON, PDO, and Session.
4. Enable the GitHub/Packagist update hook offered by Packagist so new tags are
   imported automatically.

Packagist must know about the stable core release before Composer can resolve
the skeleton's `"yacho/microphp-core": "^1.0"` requirement.

## 3. Finalize the skeleton repository

Return to this repository only after the core package is visible on Packagist.
Remove `PreparationForDivide/` from the skeleton repository: it is staging
material and must not ship inside generated applications. Then run:

```bash
composer update
composer validate --strict
composer dump-autoload --optimize --strict-psr
composer test
```

`composer update` creates the skeleton's new `composer.lock` and records the
released core dependency. Commit that lock file: application/project packages
should provide reproducible dependency versions. Never copy back the old lock
file from before the split.

Also confirm that `.env`, `vendor/`, `.phpunit.cache/`, and writable files below
`var/` are not tracked. Keep `.env.example`, `public/assets/.gitkeep`, and the
empty runtime directory creation performed by `bin/setup-project.php`.

Commit and tag the skeleton:

```bash
git add .
git commit -m "Use the published MicroPHP core package"
git push origin main
git tag -a v1.0.0 -m "MicroPHP Skeleton 1.0.0"
git push origin v1.0.0
```

If `v1.0.0` already exists in this repository, do not move or overwrite that
tag. Choose the next correct semantic version instead.

## 4. Publish and verify the skeleton

Submit `https://github.com/y4ch0/microphp-composer` to Packagist and enable its
automatic GitHub updates. Packagist should show it as `yacho/microphp` with
type `project` and a dependency on `yacho/microphp-core`.

Finally, test the same installation path a user will run, in a directory
outside both repositories:

```bash
composer create-project yacho/microphp microphp-test "^1.0"
cd microphp-test
composer show yacho/microphp-core
composer test
php -S localhost:8000 -t public public/index.php
```

Open `http://localhost:8000`, exercise one page and one API route, and delete
the temporary test project afterward.

## Release order for future versions

When a change touches framework code, release and tag `microphp-core` first.
Then update the skeleton, run `composer update yacho/microphp-core`, test it,
and tag the skeleton. A `^1.0` constraint accepts backward-compatible `1.x`
core releases but will not silently install a breaking `2.0` release. When a
new core major version is ready, change the skeleton constraint deliberately
and publish a corresponding tested skeleton release.
