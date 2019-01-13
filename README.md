# bsdops

DebOps-derived reusable, integrated Ansible configs for FreeBSD-based machines.

**Note: This is currently in a very very very early stage of development. Currently only `debops bootstrap` works, and "extensive testing" is a dream.**

## How to use

### 1. Install

1. Install `debops` normally:
   * https://docs.debops.org/en/master/user-guide/install.html
   * I prefer to install it into a venv: `pipenv install ansible debops`

2. Create a project directory for your configuration:
  ```
  mkdir my-project
  cd my-project
  # if you installed with pipenv: pipenv shell
  debops-init .
  ```

3. Tell `debops` to use [my fork](https://github.com/AnotherKamila/debops) instead of the official DebOps repo: put it into a `debops/` subdirectory:
  ```
  # (inside the project directory)
  git clone https://github.com/AnotherKamila/debops debops
  # or git submodule add ...
  # or ln -s ...
  ```

  Or with `git subrepo`:
  ```
  git subrepo clone https://github.com/anotherkamila/debops.git debops/
  ```

### Bootstrap host(s)

1. Add your host(s) to the inventory: `$EDITOR ansible/inventory/hosts`
   * All FreeBSD hosts need to be in a group named `freebsd`  (this is used to run the right OS-specific things).  
      Example inventory file:
      ```
      [freebsd]
      bsdops-test

      [debops_all_hosts:children]
      freebsd
      ```
   * Create the file `group_vars/freebsd/paths.yml`. You can copy the one in this repo. It contains the following variables:
       * `ansible_python_interpreter: /usr/local/bin/python2`. This is the Python path used by Ansible on the host. Ansible hard-codes it to `/usr/bin/python`, so we need to override it.
       * `pkg_base_path: /usr/local/`. The base path for things not in the base system.

2. Bootstrap your host(s):
  Make sure that you can log in without password (`--ask-pass` doesn't work with FreeBSD).  
  For example: `ssh-copy-id myhost`.
  Then run:
  ```
  debops bootstrap <parameters-needed-to-connect> [--limit myhost]
  # examples:
  # debops bootstrap -u root  # if you can log in as root
  # debops bootstrap -u admin --become --ask-become-pass  # if you can log in as a user that has sudo
  ```

### Configuration

Everything is the same as DebOps, so in theory this section is unnecessary. However, as a quick reference, this is a list of things one might want to change often:

* `bootstrap__admin_system`: set to False to have the automatically created admin user get a home directory under `/home` instead of under `/usr/local`.

# Development

**This repo contains the documentation (so, this README) and an example debops project that you can use as a starting point. The repo with my patched version of debops lives at [anotherkamila/debops](https://github.com/AnotherKamila/debops).**

Development workflow with `git subrepo`:

```
git subrepo clone git@github.com:anotherkamila/debops.git debops/
[hack hack]
[git commit ...]
git subrepo push debops
git push

```

Updating `debops` from upstream with minimum suffering:

```
git subrepo push debops
cd WHEREVER_I_CHECKED_OUT_TOPLEVEL_REPO
git pull origin master
git fetch --all
git merge upstream/master
git push
cd BACK_TO_HERE
git subrepo pull debops
```
