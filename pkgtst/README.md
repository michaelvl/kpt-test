# pkgtst

## Description
kpt package for provisioning namespace

## Usage

### Fetch the package
`kpt pkg get REPO_URI[.git]/PKG_PATH[@VERSION] pkgtst`
Details: https://kpt.dev/reference/cli/pkg/get/

### View package content
`kpt pkg tree pkgtst`
Details: https://kpt.dev/reference/cli/pkg/tree/

### Apply the package
```
kpt live init pkgtst
kpt live apply pkgtst --reconcile-timeout=2m --output=table
```
Details: https://kpt.dev/reference/cli/live/
