# Intro

In this section we will show you step by step how to add pre commit linting to a repository. You will be able to use this on the command line to lint your code and use it in a github repository.

## Step 1

Ensure that you have `pre-commit` installed on your machine.

```console
dnf install pre-commit
```

Further documentation for those who are interested to learn more see:

- [Pre-commit installation](https://pre-commit.com/#installation)

## Step 2

Create a file that links to the pre-commit hooks that you want to use. In our case, some general cleanup hooks, yamllint, and ansible-lint.

Create a file in this folder path `.pre-commit-config.yml`

```yaml
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

## Step 3

To use this in github set the workflow action in a file.

Create a file in this folder path `.github/workflows/pre-commit.yml`

```yaml
---
name: Yaml and Ansible Lint

on: [push, pull_request]  # yamllint disable-line rule:truthy

jobs:
  pre-commit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
      - name: Install Collections
        run: |
          sudo apt install software-properties-common
          sudo apt-add-repository --yes --update ppa:ansible/ansible
          sudo apt install ansible
      - uses: pre-commit/action@v2.0.0
...
```

Further documentation for more hooks that can be added can be found here:

- [Supported hooks](https://pre-commit.com/hooks.html)
