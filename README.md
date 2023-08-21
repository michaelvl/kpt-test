# kpt-test

This repo reproduces the bug initially described in the [kpt Slack](https://kubernetes.slack.com/archives/C0155NSPJSZ/p1690049245160149)

## Summary

A `kpt pkg get` followed by a `kpt pkg updata --strategy
force-delete-replace` leaves the old version of `Kptfile` and not as
expected the version from the update.

Tested using `kpt` version `1.0.0-beta.38` and `1.0.0-beta.43`.

## Description

This repo have two versions of the `pkgtst` package, an 'old' one with
two functions in the `Kptfile` pipeline and a 'new' one with three
functions. Functions in the pipeline are named.

A `Makefile` implements commands to reproduce the problem.

Show history and versions of `Kptfile`s:

```
cd cfg-test
make show-versions
```
This will show the 'old' and 'new' versions of `Kptfile`:

```
----------------------------
>>>> Showing version b07ab69 (aka. old version)
git show b07ab69:../pkgtst/Kptfile
apiVersion: kpt.dev/v1
kind: Kptfile
metadata:
  name: pkgtst
  annotations:
    config.kubernetes.io/local-config: "true"
info:
  description: kpt package for provisioning namespace
pipeline:
  mutators:
    - name: set package-context namespace
      image: gcr.io/kpt-fn/set-namespace:v0.4.1
      configPath: package-context.yaml
    - name: update role binding
      image: gcr.io/kpt-fn/apply-replacements:v0.1.1
      configPath: update-rolebinding.yaml
  validators:
    - name: validation
      image: gcr.io/kpt-fn/kubeval:v0.3
----------------------------
>>>> Showing version d60dafa (aka. new version)
git show d60dafa:../pkgtst/Kptfile
apiVersion: kpt.dev/v1
kind: Kptfile
metadata:
  name: pkgtst
  annotations:
    config.kubernetes.io/local-config: "true"
info:
  description: kpt package for provisioning namespace
pipeline:
  mutators:
    - name: set package-context namespace
      image: gcr.io/kpt-fn/set-namespace:v0.4.1
      configPath: package-context.yaml
    - name: set some values
      image: gcr.io/kpt-fn/apply-setters:v0.2.0
      configPath: setters.yaml
    - name: update role binding
      image: gcr.io/kpt-fn/apply-replacements:v0.1.1
      configPath: update-rolebinding.yaml
  validators:
    - name: validation
      image: gcr.io/kpt-fn/kubeval:v0.3
----------------------------
git diff b07ab69..d60dafa
diff --git a/pkgtst/Kptfile b/pkgtst/Kptfile
index a30151f..5b28a9b 100644
--- a/pkgtst/Kptfile
+++ b/pkgtst/Kptfile
@@ -11,6 +11,9 @@ pipeline:
     - name: set package-context namespace
       image: gcr.io/kpt-fn/set-namespace:v0.4.1
       configPath: package-context.yaml
+    - name: set some values
+      image: gcr.io/kpt-fn/apply-setters:v0.2.0
+      configPath: setters.yaml
     - name: update role binding
       image: gcr.io/kpt-fn/apply-replacements:v0.1.1
       configPath: update-rolebinding.yaml
```

Initially we get the 'old' version of the package (will run `kpt pkg get ...`):

```
make bootstrap-named
```

Which will show the content of the old `Kptfile` (output abbreviated for clarity):

```
kpt pkg get git@github.com:michaelvl/kpt-test.git/pkgtst@b07ab69 test-pkg
cat test-pkg/Kptfile
...
pipeline:
  mutators:
    - image: gcr.io/kpt-fn/set-namespace:v0.4.1
      configPath: package-context.yaml
      name: set package-context namespace
    - image: gcr.io/kpt-fn/apply-replacements:v0.1.1
      configPath: update-rolebinding.yaml
      name: update role binding
```

Next, we update the package to the 'new' version using `--strategy force-delete-replace`:

```
make update-named-w-force-delete-replace
```

Which will show the update content of the `Kptfile` (output abbreviated for clarity):

```
kpt pkg update test-pkg@d60dafa --strategy force-delete-replace
cat test-pkg/Kptfile
...
upstream:
  type: git
  git:
    repo: git@github.com:michaelvl/kpt-test
    directory: /pkgtst
    ref: d60dafa
  updateStrategy: force-delete-replace
upstreamLock:
  type: git
  git:
    repo: git@github.com:michaelvl/kpt-test
    directory: /pkgtst
    ref: d60dafa
    commit: d60dafa
...
pipeline:
  mutators:
    - image: gcr.io/kpt-fn/set-namespace:v0.4.1
      configPath: package-context.yaml
      name: set package-context namespace
    - image: gcr.io/kpt-fn/apply-replacements:v0.1.1
      configPath: update-rolebinding.yaml
      name: update role binding
  validators:
    - image: gcr.io/kpt-fn/kubeval:v0.3
      name: validation
```

**The expected result was the upstream version of `Kptfile` at
`d60dafa`, except for fields related to git (e.g. `upstream` and `upstreamLock`)**:

```
git show d60dafa:../pkgtst/Kptfile

apiVersion: kpt.dev/v1
kind: Kptfile
metadata:
  name: pkgtst
  annotations:
    config.kubernetes.io/local-config: "true"
info:
  description: kpt package for provisioning namespace
pipeline:
  mutators:
    - name: set package-context namespace
      image: gcr.io/kpt-fn/set-namespace:v0.4.1
      configPath: package-context.yaml
    - name: set some values
      image: gcr.io/kpt-fn/apply-setters:v0.2.0
      configPath: setters.yaml
    - name: update role binding
      image: gcr.io/kpt-fn/apply-replacements:v0.1.1
      configPath: update-rolebinding.yaml
  validators:
    - name: validation
      image: gcr.io/kpt-fn/kubeval:v0.3
```
