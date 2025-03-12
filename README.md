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

## Solution

Finally, I found the good configuration that works fine:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    ":dependencyDashboard"
  ],
  "labels": [
    "dependencies"
  ],
  "customManagers": [
    {
      "description": "helmRequirements file",
      "customType": "jsonata",
      "fileMatch": [
        "openshift-deploy/.*/helmRequirements\\.y[a]*ml$"
      ],
      "fileFormat": "yaml",
      "matchStringsStrategy": "recursive",
      "matchStrings": [
        "$each(*, function($v, $n) { { 'depName': $v.chart, 'currentValue': $v.version, 'registryUrl': $v.repo } })"
      ],
      "datasourceTemplate": "helm",
      "versioningTemplate": "semver"
    }
  ]
}
```

Traces:

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
DEBUG: PackageFiles.clear() - Package files deleted (repository=local)
DEBUG: Resetting npmrc (repository=local)
DEBUG: Resetting npmrc (repository=local)
DEBUG: checkOnboarding() (repository=local)
DEBUG: isOnboarded() (repository=local)
DEBUG: findFile(renovate.json) (repository=local)
fatal: detected dubious ownership in repository at '/usr/src/app'
To add an exception for this directory, call:

	git config --global --add safe.directory /usr/src/app
DEBUG: Could not get file list using git, using glob instead (repository=local)
DEBUG: Config file exists, fileName: renovate.json (repository=local)
DEBUG: Repo is onboarded (repository=local)
fatal: detected dubious ownership in repository at '/usr/src/app'
To add an exception for this directory, call:

	git config --global --add safe.directory /usr/src/app
DEBUG: Could not get file list using git, using glob instead (repository=local)
DEBUG: Found renovate.json config file (repository=local)
DEBUG: Repository config (repository=local)
       "fileName": "renovate.json",
       "config": {
         "$schema": "https://docs.renovatebot.com/renovate-schema.json",
         "extends": ["config:recommended", ":dependencyDashboard"],
         "labels": ["dependencies"],
         "customManagers": [
           {
             "description": "helmRequirements file",
             "customType": "jsonata",
             "fileMatch": ["openshift-deploy/.*/helmRequirements\\.y[a]*ml$"],
             "fileFormat": "yaml",
             "matchStringsStrategy": "recursive",
             "matchStrings": [
               "$each(*, function($v, $n) { { 'depName': $v.chart, 'currentValue': $v.version, 'registryUrl': $v.repo } })"
             ],
             "datasourceTemplate": "helm",
             "versioningTemplate": "semver"
           }
         ]
       }
DEBUG: migrateAndValidate() (repository=local)
DEBUG: No config migration necessary (repository=local)
DEBUG: Post-massage config (repository=local)
       "config": {
         "$schema": "https://docs.renovatebot.com/renovate-schema.json",
         "extends": ["config:recommended", ":dependencyDashboard"],
         "labels": ["dependencies"],
         "customManagers": [
           {
             "description": ["helmRequirements file"],
             "customType": "jsonata",
             "fileMatch": ["openshift-deploy/.*/helmRequirements\\.y[a]*ml$"],
             "fileFormat": "yaml",
             "matchStringsStrategy": "recursive",
             "matchStrings": [
               "$each(*, function($v, $n) { { 'depName': $v.chart, 'currentValue': $v.version, 'registryUrl': $v.repo } })"
             ],
             "datasourceTemplate": "helm",
             "versioningTemplate": "semver"
           }
         ]
       }
DEBUG: Found repo ignorePaths (repository=local)
       "ignorePaths": [
         "**/node_modules/**",
         "**/bower_components/**",
         "**/vendor/**",
         "**/examples/**",
         "**/__tests__/**",
         "**/test/**",
         "**/tests/**",
         "**/__fixtures__/**"
       ]
DEBUG: No vulnerability alerts found (repository=local)
DEBUG: No baseBranches (repository=local)
DEBUG: extract() (repository=local)
fatal: detected dubious ownership in repository at '/usr/src/app'
To add an exception for this directory, call:

	git config --global --add safe.directory /usr/src/app
DEBUG: Could not get file list using git, using glob instead (repository=local)
DEBUG: Using file match: (^|/)tasks/[^/]+\.ya?ml$ for manager ansible (repository=local)
DEBUG: Using file match: (^|/)(galaxy|requirements)(\.ansible)?\.ya?ml$ for manager ansible-galaxy (repository=local)
DEBUG: Using file match: (^|/)\.tool-versions$ for manager asdf (repository=local)
DEBUG: Using file match: (^|/).azuredevops/.+\.ya?ml$ for manager azure-pipelines (repository=local)
DEBUG: Using file match: azure.*pipelines?.*\.ya?ml$ for manager azure-pipelines (repository=local)
DEBUG: Using file match: (^|/)batect(-bundle)?\.ya?ml$ for manager batect (repository=local)
DEBUG: Using file match: (^|/)batect$ for manager batect-wrapper (repository=local)
DEBUG: Using file match: (^|/)WORKSPACE(|\.bazel|\.bzlmod)$ for manager bazel (repository=local)
DEBUG: Using file match: \.WORKSPACE\.bazel$ for manager bazel (repository=local)
DEBUG: Using file match: \.bzl$ for manager bazel (repository=local)
DEBUG: Using file match: (^|/|\.)MODULE\.bazel$ for manager bazel-module (repository=local)
DEBUG: Using file match: (^|/)\.bazelversion$ for manager bazelisk (repository=local)
DEBUG: Using file match: \.bicep$ for manager bicep (repository=local)
DEBUG: Using file match: (^|/)\.?bitbucket-pipelines\.ya?ml$ for manager bitbucket-pipelines (repository=local)
DEBUG: Using file match: (^|/)bitrise\.ya?ml$ for manager bitrise (repository=local)
DEBUG: Using file match: buildkite\.ya?ml for manager buildkite (repository=local)
DEBUG: Using file match: \.buildkite/.+\.ya?ml$ for manager buildkite (repository=local)
DEBUG: Using file match: (^|/)project\.toml$ for manager buildpacks (repository=local)
DEBUG: Using file match: (^|/)bun\.lockb?$ for manager bun (repository=local)
DEBUG: Using file match: (^|/)\.bun-version$ for manager bun-version (repository=local)
DEBUG: Using file match: (^|/)Gemfile$ for manager bundler (repository=local)
DEBUG: Using file match: \.cake$ for manager cake (repository=local)
DEBUG: Using file match: (^|/)Cargo\.toml$ for manager cargo (repository=local)
DEBUG: Using file match: (^|/)\.circleci/.+\.ya?ml$ for manager circleci (repository=local)
DEBUG: Using file match: (^|/)cloudbuild\.ya?ml for manager cloudbuild (repository=local)
DEBUG: Using file match: (^|/)Podfile$ for manager cocoapods (repository=local)
DEBUG: Using file match: (^|/)([\w-]*)composer\.json$ for manager composer (repository=local)
DEBUG: Using file match: (^|/)conanfile\.(txt|py)$ for manager conan (repository=local)
DEBUG: Using file match: (^|/)\.copier-answers(\..+)?\.ya?ml for manager copier (repository=local)
DEBUG: Using file match: (^|/)cpanfile$ for manager cpanfile (repository=local)
DEBUG: Using file match: (^|/)(?:deps|bb)\.edn$ for manager deps-edn (repository=local)
DEBUG: Using file match: (^|/)devbox\.json$ for manager devbox (repository=local)
DEBUG: Using file match: ^.devcontainer/devcontainer.json$ for manager devcontainer (repository=local)
DEBUG: Using file match: ^.devcontainer.json$ for manager devcontainer (repository=local)
DEBUG: Using file match: (^|/)(?:docker-)?compose[^/]*\.ya?ml$ for manager docker-compose (repository=local)
DEBUG: Using file match: (^|/|\.)([Dd]ocker|[Cc]ontainer)file$ for manager dockerfile (repository=local)
DEBUG: Using file match: (^|/)([Dd]ocker|[Cc]ontainer)file[^/]*$ for manager dockerfile (repository=local)
DEBUG: Using file match: (^|/)\.drone\.yml$ for manager droneci (repository=local)
DEBUG: Using file match: (^|/)fleet\.ya?ml for manager fleet (repository=local)
DEBUG: Using file match: (?:^|/)gotk-components\.ya?ml$ for manager flux (repository=local)
DEBUG: Using file match: (^|/)\.fvm/fvm_config\.json$ for manager fvm (repository=local)
DEBUG: Using file match: (^|/)\.fvmrc$ for manager fvm (repository=local)
DEBUG: Using file match: (^|/)\.gitmodules$ for manager git-submodules (repository=local)
DEBUG: Using file match: (^|/)(workflow-templates|\.(?:github|gitea|forgejo)/(?:workflows|actions))/.+\.ya?ml$ for manager github-actions (repository=local)
DEBUG: Using file match: (^|/)action\.ya?ml$ for manager github-actions (repository=local)
DEBUG: Using file match: \.gitlab-ci\.ya?ml$ for manager gitlabci (repository=local)
DEBUG: Using file match: \.gitlab-ci\.ya?ml$ for manager gitlabci-include (repository=local)
DEBUG: Using file match: (^|/)gleam.toml$ for manager gleam (repository=local)
DEBUG: Using file match: (^|/)go\.mod$ for manager gomod (repository=local)
DEBUG: Using file match: \.gradle(\.kts)?$ for manager gradle (repository=local)
DEBUG: Using file match: (^|/)gradle\.properties$ for manager gradle (repository=local)
DEBUG: Using file match: (^|/)gradle/.+\.toml$ for manager gradle (repository=local)
DEBUG: Using file match: (^|/)buildSrc/.+\.kt$ for manager gradle (repository=local)
DEBUG: Using file match: \.versions\.toml$ for manager gradle (repository=local)
DEBUG: Using file match: (^|/)versions.props$ for manager gradle (repository=local)
DEBUG: Using file match: (^|/)versions.lock$ for manager gradle (repository=local)
DEBUG: Using file match: (^|/)gradle/wrapper/gradle-wrapper\.properties$ for manager gradle-wrapper (repository=local)
DEBUG: Using file match: \.cabal$ for manager haskell-cabal (repository=local)
DEBUG: Using file match: (^|/)requirements\.ya?ml$ for manager helm-requirements (repository=local)
DEBUG: Using file match: (^|/)values\.ya?ml$ for manager helm-values (repository=local)
DEBUG: Using file match: (^|/)helmfile\.ya?ml(?:\.gotmpl)?$ for manager helmfile (repository=local)
DEBUG: Using file match: (^|/)Chart\.ya?ml$ for manager helmv3 (repository=local)
DEBUG: Using file match: (^|/)bin/hermit$ for manager hermit (repository=local)
DEBUG: Using file match: ^Formula/[^/]+[.]rb$ for manager homebrew (repository=local)
DEBUG: Using file match: \.html?$ for manager html (repository=local)
DEBUG: Using file match: (^|/)plugins\.(txt|ya?ml)$ for manager jenkins (repository=local)
DEBUG: Using file match: (^|/)jsonnetfile\.json$ for manager jsonnet-bundler (repository=local)
DEBUG: Using file match: ^.+\.main\.kts$ for manager kotlin-script (repository=local)
DEBUG: Using file match: (^|/)kustomization\.ya?ml$ for manager kustomize (repository=local)
DEBUG: Using file match: (^|/)project\.clj$ for manager leiningen (repository=local)
DEBUG: Using file match: (^|/|\.)pom\.xml$ for manager maven (repository=local)
DEBUG: Using file match: ^(((\.mvn)|(\.m2))/)?settings\.xml$ for manager maven (repository=local)
DEBUG: Using file match: (^|/)\.mvn/extensions\.xml$ for manager maven (repository=local)
DEBUG: Using file match: (^|\/).mvn/wrapper/maven-wrapper.properties$ for manager maven-wrapper (repository=local)
DEBUG: Using file match: (^|/)package\.js$ for manager meteor (repository=local)
DEBUG: Using file match: (^|/)Mintfile$ for manager mint (repository=local)
DEBUG: Using file match: (^|/)\.?mise\.toml$ for manager mise (repository=local)
DEBUG: Using file match: (^|/)\.?mise/config\.toml$ for manager mise (repository=local)
DEBUG: Using file match: (^|/)mix\.exs$ for manager mix (repository=local)
DEBUG: Using file match: (^|/)flake\.nix$ for manager nix (repository=local)
DEBUG: Using file match: (^|/)\.node-version$ for manager nodenv (repository=local)
DEBUG: Using file match: (^|/)package\.json$ for manager npm (repository=local)
DEBUG: Using file match: (^|/)pnpm-workspace\.yaml$ for manager npm (repository=local)
DEBUG: Using file match: \.(?:cs|fs|vb)proj$ for manager nuget (repository=local)
DEBUG: Using file match: \.(?:props|targets)$ for manager nuget (repository=local)
DEBUG: Using file match: (^|/)dotnet-tools\.json$ for manager nuget (repository=local)
DEBUG: Using file match: (^|/)global\.json$ for manager nuget (repository=local)
DEBUG: Using file match: (^|/)\.nvmrc$ for manager nvm (repository=local)
DEBUG: Using file match: (^|/)src/main/features/.+\.json$ for manager osgi (repository=local)
DEBUG: Using file match: (^|/)pyproject\.toml$ for manager pep621 (repository=local)
DEBUG: Using file match: (^|/)[\w-]*requirements([-.]\w+)?\.(txt|pip)$ for manager pip_requirements (repository=local)
DEBUG: Using file match: (^|/)setup\.py$ for manager pip_setup (repository=local)
DEBUG: Using file match: (^|/)Pipfile$ for manager pipenv (repository=local)
DEBUG: Using file match: (^|/)pyproject\.toml$ for manager pixi (repository=local)
DEBUG: Using file match: (^|/)pixi\.toml$ for manager pixi (repository=local)
DEBUG: Using file match: (^|/)pyproject\.toml$ for manager poetry (repository=local)
DEBUG: Using file match: (^|/)\.pre-commit-config\.ya?ml$ for manager pre-commit (repository=local)
DEBUG: Using file match: (^|/)pubspec\.ya?ml$ for manager pub (repository=local)
DEBUG: Using file match: (^|/)Puppetfile$ for manager puppet (repository=local)
DEBUG: Using file match: (^|/)\.python-version$ for manager pyenv (repository=local)
DEBUG: Using file match: (^|/)\.ruby-version$ for manager ruby-version (repository=local)
DEBUG: Using file match: (^|/)runtime.txt$ for manager runtime-version (repository=local)
DEBUG: Using file match: \.sbt$ for manager sbt (repository=local)
DEBUG: Using file match: project/[^/]*\.scala$ for manager sbt (repository=local)
DEBUG: Using file match: project/build\.properties$ for manager sbt (repository=local)
DEBUG: Using file match: (^|/)repositories$ for manager sbt (repository=local)
DEBUG: Using file match: (^|/)\.scalafmt.conf$ for manager scalafmt (repository=local)
DEBUG: Using file match: (^|/)setup\.cfg$ for manager setup-cfg (repository=local)
DEBUG: Using file match: (^|/)Package\.swift for manager swift (repository=local)
DEBUG: Using file match: \.tf$ for manager terraform (repository=local)
DEBUG: Using file match: (^|/)\.terraform-version$ for manager terraform-version (repository=local)
DEBUG: Using file match: (^|/)terragrunt\.hcl$ for manager terragrunt (repository=local)
DEBUG: Using file match: (^|/)\.terragrunt-version$ for manager terragrunt-version (repository=local)
DEBUG: Using file match: \.tflint\.hcl$ for manager tflint-plugin (repository=local)
DEBUG: Using file match: ^\.travis\.ya?ml$ for manager travis (repository=local)
DEBUG: Using file match: (^|/)\.vela\.ya?ml$ for manager velaci (repository=local)
DEBUG: Using file match: (^|/)vendir\.yml$ for manager vendir (repository=local)
DEBUG: Using file match: ^\.woodpecker(?:/[^/]+)?\.ya?ml$ for manager woodpecker (repository=local)
DEBUG: Using file match: openshift-deploy/.*/helmRequirements\.y[a]*ml$ for manager jsonata (repository=local)
DEBUG: Matched 1 file(s) for manager jsonata: openshift-deploy/base/helmRequirements.yml (repository=local)
DEBUG: manager extract durations (ms) (repository=local)
       "managers": {"jsonata": 86}
DEBUG: Found jsonata package files (repository=local)
DEBUG: Found 1 package file(s) (repository=local)
 INFO: Dependency extraction complete (repository=local)
       "stats": {
         "managers": {"jsonata": {"fileCount": 1, "depCount": 2}},
         "total": {"fileCount": 1, "depCount": 2}
       }
DEBUG: hostRules: no authentication for helm.vector.dev (repository=local)
DEBUG: Using queue: host=helm.vector.dev, concurrency=16 (repository=local)
DEBUG: Using queue: host=grafana.github.io, concurrency=16 (repository=local)
DEBUG: PackageFiles.add() - Package file saved for base branch (repository=local)
DEBUG: Package releases lookups complete (repository=local)
DEBUG: Repository libYears (repository=local)
       "managerLibYears": {"jsonata": 0.8094988606354643},
       "totalLibYears": 0.8094988606354643,
       "totalDepsCount": 2,
       "outdatedDepsCount": 2
DEBUG: branchifyUpgrades (repository=local)
DEBUG: detectSemanticCommits() (repository=local)
DEBUG: getCommitMessages (repository=local)
DEBUG: semanticCommits: detected "unknown" (repository=local)
DEBUG: semanticCommits: disabled (repository=local)
DEBUG: 2 flattened updates found: vector, loki (repository=local)
DEBUG: Returning 2 branch(es) (repository=local)
DEBUG: config.repoIsOnboarded=true (repository=local)
DEBUG: packageFiles with updates (repository=local)
       "config": {
         "jsonata": [
           {
             "deps": [
               {
                 "depName": "vector",
                 "currentValue": "0.38.0",
                 "datasource": "helm",
                 "versioning": "semver",
                 "registryUrls": ["https://helm.vector.dev/"],
                 "updates": [
                   {
                     "bucket": "non-major",
                     "newVersion": "0.41.0",
                     "newValue": "0.41.0",
                     "newDigest": "8ca47d62d684ca345bbcf83ef04f9a035003e6a3d8baa643d374758ed1709864",
                     "releaseTimestamp": "2025-02-24T20:44:40.769Z",
                     "newVersionAgeInDays": 15,
                     "newMajor": 0,
                     "newMinor": 41,
                     "newPatch": 0,
                     "updateType": "minor",
                     "libYears": 0.2248567626839168,
                     "branchName": "renovate/vector-0.x"
                   }
                 ],
                 "packageName": "vector",
                 "warnings": [],
                 "sourceUrl": "https://github.com/vectordotdev/helm-charts",
                 "registryUrl": "https://helm.vector.dev",
                 "homepage": "https://vector.dev/",
                 "currentVersion": "0.38.0",
                 "currentVersionTimestamp": "2024-12-04T18:59:57.901Z",
                 "currentVersionAgeInDays": 97,
                 "isSingleVersion": true,
                 "fixedVersion": "0.38.0"
               },
               {
                 "depName": "loki",
                 "currentValue": "6.10.0",
                 "datasource": "helm",
                 "versioning": "semver",
                 "registryUrls": ["https://grafana.github.io/helm-charts"],
                 "updates": [
                   {
                     "bucket": "non-major",
                     "newVersion": "6.28.0",
                     "newValue": "6.28.0",
                     "newDigest": "b63a2dc23cdf1d3ec503fc53837bacb83265da32de8f41cfddc8d9ac3f8c1c75",
                     "releaseTimestamp": "2025-03-10T20:07:49.614Z",
                     "newVersionAgeInDays": 1,
                     "newMajor": 6,
                     "newMinor": 28,
                     "newPatch": 0,
                     "updateType": "minor",
                     "libYears": 0.5846420979515474,
                     "branchName": "renovate/loki-6.x"
                   }
                 ],
                 "packageName": "loki",
                 "warnings": [],
                 "sourceUrl": "https://github.com/grafana/helm-charts",
                 "registryUrl": "https://grafana.github.io/helm-charts",
                 "homepage": "https://grafana.github.io/helm-charts",
                 "currentVersion": "6.10.0",
                 "currentVersionTimestamp": "2024-08-09T10:39:56.413Z",
                 "currentVersionAgeInDays": 215,
                 "isSingleVersion": true,
                 "fixedVersion": "6.10.0"
               }
             ],
             "matchStrings": [
               "$each(*, function($v, $n) { { 'depName': $v.chart, 'currentValue': $v.version, 'registryUrl': $v.repo } })"
             ],
             "fileFormat": "yaml",
             "datasourceTemplate": "helm",
             "versioningTemplate": "semver",
             "packageFile": "openshift-deploy/base/helmRequirements.yml"
           }
         ]
       }
DEBUG: detectSemanticCommits() (repository=local)
DEBUG: semanticCommits: returning "disabled" from cache (repository=local)
DEBUG: Repository timing splits (milliseconds) (repository=local)
       "splits": {"init": 1938, "extract": 1196, "lookup": 3930},
       "total": 7097
DEBUG: Package cache statistics (repository=local)
       "get": {"count": 2, "avgMs": 39, "medianMs": 72, "maxMs": 72, "totalMs": 77},
       "set": {"count": 2, "avgMs": 61, "medianMs": 104, "maxMs": 104, "totalMs": 122}
DEBUG: HTTP statistics (repository=local)
       "hosts": {
         "grafana.github.io": {
           "count": 1,
           "reqAvgMs": 152,
           "reqMedianMs": 152,
           "reqMaxMs": 152,
           "queueAvgMs": 0,
           "queueMedianMs": 0,
           "queueMaxMs": 0
         },
         "helm.vector.dev": {
           "count": 1,
           "reqAvgMs": 3437,
           "reqMedianMs": 3437,
           "reqMaxMs": 3437,
           "queueAvgMs": 19,
           "queueMedianMs": 19,
           "queueMaxMs": 19
         }
       },
       "requests": 2
DEBUG: HTTP cache statistics (repository=local)
DEBUG: Lookup statistics (repository=local)
       "helm": {"count": 2, "avgMs": 3582, "medianMs": 3660, "maxMs": 3660, "totalMs": 7164}
 INFO: Repository finished (repository=local)
       "cloned": undefined,
       "durationMs": 7097
DEBUG: Checking file package cache for expired items
DEBUG: Deleted 0 of 2 file cached entries in 89ms
```
