# Intro

In this section we will show you step by step how to build an Execution Environment (EE) from code. You will also be publishing this EE to your own private automation hub which you will use in the rest of the tasks.

## Step 1

Ensure that you have `ansible-core` installed on your machine.

```console
dnf install ansible-core
```

## Step 2

Install our ee_utilities collection using `ansible-galaxy` command.

```console
ansible-galaxy collection install redhat_cop.ee_utilities
```
