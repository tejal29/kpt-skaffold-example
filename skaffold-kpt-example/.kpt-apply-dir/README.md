# .kpt-apply-dir

## Description
sample description

## Usage

### Fetch the package
`kpt pkg get REPO_URI[.git]/PKG_PATH[@VERSION] .kpt-apply-dir`
Details: https://kpt.dev/reference/cli/pkg/get/

### View package content
`kpt pkg tree .kpt-apply-dir`
Details: https://kpt.dev/reference/cli/pkg/tree/

### Apply the package
```
kpt live init .kpt-apply-dir
kpt live apply .kpt-apply-dir --reconcile-timeout=2m --output=table
```
Details: https://kpt.dev/reference/cli/live/
