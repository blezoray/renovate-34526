# 34526

First, read the [Renovate minimal reproduction instructions](https://github.com/renovatebot/renovate/blob/main/docs/development/minimal-reproductions.md).

Then replace the current `h1` with the Renovate Issue/Discussion number.

## Current behavior

In my `openshift-deploy/base/helmRequirements.yml` file, I list the helm charts I want to deploy.

With jsonata parser, it doesn't detect updates.
With regex parser, it detects only the last entry.

Execution:

```shell
$ docker run --rm -v "${PWD}:/usr/src/app" -e LOG_LEVEL=debug renovate/renovate --platform=local --repository-cache=reset 
DEBUG: Using RE2 regex engine
DEBUG: Parsing configs
DEBUG: No config file found on disk - skipping
DEBUG: File config
       "config": {}
DEBUG: CLI config
       "config": {"repositoryCache": "reset", "platform": "local"}
DEBUG: Env config
       "config": {"hostRules": []}
DEBUG: Combined config
       "config": {"hostRules": [], "repositoryCache": "reset", "platform": "local"}
DEBUG: Enabling forkProcessing while in non-autodiscover mode
DEBUG: Enabling onboardingNoDeps while in non-autodiscover mode
DEBUG: Found valid git version: 2.48.1
DEBUG: Setting global hostRules
DEBUG: Using baseDir: /tmp/renovate
DEBUG: Using cacheDir: /tmp/renovate/cache
DEBUG: Using containerbaseDir: /tmp/renovate/cache/containerbase
DEBUG: Initializing Renovate internal cache into /tmp/renovate/cache/renovate/renovate-cache-v1
DEBUG: Commits limit = null
DEBUG: Setting global hostRules
DEBUG: validatePresets()
DEBUG: Reinitializing hostRules for repo
DEBUG: Clearing hostRules
 INFO: Repository started (repository=local)
       "renovateVersion": "39.190.1"
DEBUG: Using localDir: /usr/src/app (repository=local)
...
```

## Expected behavior

It should detect updates for both vector and loki helm charts.

## Link to the Renovate issue or Discussion

See [the discussion](https://github.com/renovatebot/renovate/discussions/34526).
