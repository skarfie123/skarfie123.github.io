It's common to use code formatters and linters to ensure code quality. To make sure your code conforms before committing, you can use pre-commit hooks.

To set this up write a script at `.git/hooks/pre-commit` that runs your formatter, linters and any other checks you want. This script will be executed every time you run `git commit` and will abort the commit if any of the checks return a non zero exit code.

## Examples

### Prevent commits on main branch

```bash
#!/bin/bash

if [ "$(git rev-parse --abbrev-ref HEAD)" = "main" ]; then
  echo "You can't commit directly to the main branch"
  exit 1
fi
```

### Format terraform files

```bash
#!/bin/bash

cd "$(git rev-parse --show-toplevel)"

echo "Running pre-commit hook: terraform fmt"
terraform fmt -check -recursive
```

## `pre-commit` Framework

You can also use the `pre-commit` framework to manage your pre-commit hooks. This allows you to define hooks in a `.pre-commit-config.yaml` file and install them with a single command.

You can configure it with hooks shared by the community or write your own.

See more at [pre-commit.com](https://pre-commit.com/).

## Other Git Hooks

There are several other Git hooks that can be used to automate tasks at different stages of the Git workflow. You can find samples in the git hooks folder:

```bash
❯ ls .git/hooks/
applypatch-msg.sample           fsmonitor-watchman.sample       pre-applypatch.sample           pre-merge-commit.sample         pre-rebase.sample               prepare-commit-msg.sample       update.sample
commit-msg.sample               post-update.sample              pre-commit.sample               pre-push.sample                 pre-receive.sample              push-to-checkout.sample
```
