# terrax-action

GitHub Actions for [TerraX](https://github.com/israoo/terrax) — the interactive TUI executor for Terragrunt stacks.

## Actions

| Action | Description |
|--------|-------------|
| [`setup-terrax`](#setup-terrax) | Install the TerraX binary |
| [`find-stacks`](#find-stacks) | List stacks, optionally filtered by git diff |
| [`summary`](#summary) | Print a plan summary to stdout |

---

## `setup-terrax`

Install TerraX from GitHub Releases and add it to `PATH`.

### Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `version` | no | `latest` | Version to install (e.g. `v0.5.1` or `latest`). |

### Outputs

| Name | Description |
|------|-------------|
| `terrax-version` | Resolved version installed (e.g. `v0.5.0`). |

### Example

```yaml
- uses: israoo/terrax-action/setup-terrax@v1
  with:
    version: latest
```

---

## `find-stacks`

List Terragrunt stacks under a directory. With `base`, returns only stacks affected by changes between `base` and `HEAD` — including transitive dependents and stacks that consume changed YAML files.

> **Note:** When using `base`, set `fetch-depth: 0` on `actions/checkout`.

### Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `dir` | no | `.` | Root directory to scan. |
| `base` | no | `""` | Git ref for change detection (e.g. `origin/main`, a commit SHA). |

### Outputs

| Name | Description |
|------|-------------|
| `stacks` | JSON array of stack paths. |
| `count` | Number of stacks found. |

### Example

```yaml
- uses: israoo/terrax-action/find-stacks@v1
  id: find
  with:
    dir: infra/
    base: origin/main

- name: Show affected stacks
  run: echo '${{ steps.find.outputs.stacks }}'
```

---

## `summary`

Print a grouped terminal summary of pending vs. no-change stacks from `.terrax/plans/` plan files written by a previous `terrax run plan` step.

### Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `dir` | no | `.` | Working directory (same as `terrax summary --dir`). |

### Example

```yaml
- uses: israoo/terrax-action/summary@v1
  with:
    dir: infra/
```

---

## Full workflow example

```yaml
jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: israoo/terrax-action/setup-terrax@v1

      - uses: israoo/terrax-action/find-stacks@v1
        id: find
        with:
          dir: infra/
          base: origin/main

      - name: Run plan on affected stacks
        run: |
          echo '${{ steps.find.outputs.stacks }}' | jq -r '.[]' | while read -r stack; do
            terrax run plan --dir "$stack"
          done

      - uses: israoo/terrax-action/summary@v1
```

## License

Apache 2.0
