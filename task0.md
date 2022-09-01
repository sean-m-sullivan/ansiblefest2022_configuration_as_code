# Intro

In this section we will show you step by step how to add pre commit linting to a repostiory. You will be able to use this on the command line to lint your code and use it in a github repostiroy.

## Step 1

Ensure that you have `pre-commit` and `` installed on your machine.

```console
dnf install pre-commit
```

Further documentation for those who are interested to learn more see:

- [Pre-commit installation](https://pre-commit.com/#installation)

## Step 2

Create a file in this folder path `.pre-commit-config.yml`

```yaml
# User may update controller/hub auth creds to this file and encrypt it using `ansible-vault`
---
---
repos:
  - repo: 'https://github.com/pre-commit/pre-commit-hooks'
    rev: v4.3.0
    hooks:
      - id: end-of-file-fixer
      - id: trailing-whitespace
  - repo: 'https://github.com/adrienverge/yamllint.git'
    rev: v1.27.1
    hooks:
      - id: yamllint
  - repo: 'https://github.com/ansible-community/ansible-lint.git'
    rev: v6.5.1
    hooks:
      - id: ansible-lint
        pass_filenames: false
        always_run: true
        entry: "ansible-lint"
...
```

Further documentation for more hooks that can be added can be found here:

- [Supported hooks](https://pre-commit.com/hooks.html)
