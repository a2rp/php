# Deployment environments and release workflows — Example

This example deploys one prebuilt PHP application artifact into a versioned Linux release directory. It keeps runtime storage outside the release, switches the live application with a same-filesystem symlink rename, and checks readiness after the switch.

The sample assumes a conventional PHP web app with **public/index.php** and Composer dependencies. It assumes this release does not require a database schema change, so the previous code release remains compatible if a rollback is needed. A real app must adapt its paths, permissions, health URL, and migration plan.

## Question

How can a team promote the same tested PHP artifact to staging and production, switch traffic to it safely, and respond if the new release fails its readiness check?

## Build and verify the artifact in CI

Install the committed dependency versions from the lockfile. Run the project's tests and quality checks in CI. Then prepare a production artifact that contains the app's runtime files and production dependencies, but no secrets, logs, local uploads, or development-only dependencies.

For Composer, the production dependency step can use:

~~~sh
composer install --no-dev --optimize-autoloader --no-interaction --prefer-dist
composer check-platform-reqs --no-dev
~~~

The first command installs production dependencies and creates an optimized autoloader. The second checks the actual PHP and extension requirements. Build the artifact once, give it an immutable identifier such as the tested commit ID, and promote that same artifact to staging and production. Keep secrets and environment configuration outside the artifact.

## Example release layout

~~~text
/srv/catalog/
    current -> /srv/catalog/releases/8f4c2d1
    releases/
        8f4c2d1/
            public/index.php
            vendor/autoload.php
            src/
    shared/
        storage/
~~~

The web server serves the public directory through the current pointer. The deployment user prepares a new release directory, while runtime configuration is supplied by the process environment or a protected configuration service. The shared storage directory holds files that should survive release replacement.

## Deployment script

Save this script as **deploy-release.sh** on the Linux deployment host. The artifact path, release ID, application root, and readiness URL are supplied by the deployment platform.

~~~bash
#!/usr/bin/env bash
set -euo pipefail

: "$APP_ROOT"
: "$RELEASE_ID"
: "$ARTIFACT"
: "$READINESS_URL"

release_dir="$APP_ROOT/releases/$RELEASE_ID"
next_link="$APP_ROOT/current.next.$RELEASE_ID"
previous_release="$(readlink -f "$APP_ROOT/current" 2>/dev/null || true)"

if [ -e "$release_dir" ] || [ -L "$release_dir" ]; then
    echo "Release directory already exists: $release_dir" >&2
    exit 1
fi

mkdir -p "$APP_ROOT/releases" "$APP_ROOT/shared/storage"
mkdir "$release_dir"

tar -xzf "$ARTIFACT" -C "$release_dir"

test -f "$release_dir/public/index.php"
test -f "$release_dir/vendor/autoload.php"

# The artifact must not contain these runtime-specific paths.
ln -s "$APP_ROOT/shared/storage" "$release_dir/storage"

# Apply the site's required ownership and permissions here.
# Supply production configuration through the host or service manager.
# This example has no database migration.

echo "Activating release $RELEASE_ID"
ln -s "$release_dir" "$next_link"
mv -Tf "$next_link" "$APP_ROOT/current"

healthy=0

for attempt in 1 2 3 4 5; do
    if curl --fail --silent --show-error "$READINESS_URL" >/dev/null; then
        healthy=1
        break
    fi

    sleep 2
done

if [ "$healthy" -ne 1 ]; then
    echo "Readiness check failed for $RELEASE_ID" >&2

    # This example assumes the previous release is compatible with the
    # database and application state. Otherwise use the recovery plan.
    if [ -n "$previous_release" ]; then
        rollback_link="$APP_ROOT/current.rollback.$RELEASE_ID"
        ln -s "$previous_release" "$rollback_link"
        mv -Tf "$rollback_link" "$APP_ROOT/current"
        echo "Restored previous release: $previous_release" >&2
    else
        echo "No previous release; operator recovery is required" >&2
    fi

    exit 1
fi

echo "Readiness check passed"
echo "Release active: $release_dir"
~~~

The **mv -T** option is provided by GNU coreutils. The temporary link and current link must be on the same filesystem for the rename to be atomic. Use the equivalent safe switch mechanism for the platform if it differs.

Run the script with the deployment platform's protected variables set:

~~~sh
APP_ROOT=/srv/catalog RELEASE_ID=8f4c2d1 ARTIFACT=/tmp/catalog-8f4c2d1.tar.gz READINESS_URL=https://catalog.example.test/ready ./deploy-release.sh
~~~

The example URL is a placeholder. Configure the real readiness URL in the deployment environment.

## Expected output

When extraction and readiness succeed, the script prints:

~~~text
Activating release 8f4c2d1
Readiness check passed
Release active: /srv/catalog/releases/8f4c2d1
~~~

The current pointer now refers to the new release. The readiness request is retried for a short period to allow the application to start. The script exits with a failure status if readiness never succeeds. When a previous release is available and compatible, it switches the pointer back before exiting.

## Rollback after a successful deployment

A later application issue may be found after the readiness check has passed. A code-only rollback can switch the pointer to a known previous release:

~~~bash
previous_release=/srv/catalog/releases/previous-commit
rollback_link=/srv/catalog/current.manual-rollback

ln -s "$previous_release" "$rollback_link"
mv -Tf "$rollback_link" /srv/catalog/current
~~~

Use this only after checking that the previous application version supports the current database schema and data. If the release performed an incompatible migration or changed persisted data, follow the application's recovery plan instead of blindly switching code.

## Database changes

This example has no schema migration. If a release needs one, first design it so old and new application versions can coexist. Add compatible schema, deploy compatible code, migrate data in observable and repeatable batches, switch application usage, and remove old schema only in a later release.

Do not put a destructive migration into an automatic rollback assumption. Database rollback, application rollback, and data recovery are distinct operations. Verify backups and the recovery procedure before a high-risk production change.

## Staging and production workflow

1. CI installs dependencies from the committed lockfile and runs the project's automated checks.
2. CI builds one artifact with a revision identifier and production dependencies.
3. The artifact is deployed to staging with staging configuration and secrets.
4. The team checks the key user flows, logs, readiness, and any affected integrations.
5. The same artifact is promoted to production with production configuration.
6. The deployment records its revision and outcome, switches traffic, and checks readiness and critical behavior.
7. If verification fails, stop rollout and use the planned compatible rollback or recovery path.

## Important limits

- The script is an example for a GNU/Linux host, not a universal deployment tool.
- It assumes a trusted artifact; production should receive artifacts only from the controlled build pipeline.
- It does not set ownership, filesystem permissions, PHP-FPM settings, or web-server configuration.
- It does not run a framework-specific database migration command.
- It does not automatically roll back a database or undo user data changes.
- The readiness URL must check an appropriate application condition without exposing secrets.
- The old release should be retained only according to the site's safe cleanup and recovery policy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [140 - Background jobs, queues, and workers](../lessons/140-background-jobs-queues-and-workers.md) | [Lesson](../lessons/141-deployment-environments-and-release-workflows.md) | [Q&A](../qnas/141-deployment-environments-and-release-workflows.md) | [142 - Apache, Nginx, and PHP-FPM integration](../lessons/142-apache-nginx-and-php-fpm-integration.md) |