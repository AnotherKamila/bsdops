# bsdops

DebOps-derived reusable, integrated Ansible configs for FreeBSD-based machines.

**Note: This is currently in a very very very early stage of development. Currently nothing is working. I am aiming to have the `bootstrap` playbook working soon.**

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

3. Tell `debops` to use my fork instead of the official DebOps repo: put it into a `debops/` subdirectory:
  ```
  # (inside the project directory)
  git clone https://github.com/AnotherKamila/debops debops
  # or git submodule add ...
  # or ln -s ...
  ```

   Or with `git subtree`:
  ```
  git remote add -f bsdops https://github.com/AnotherKamila/debops.git
  git subtree add --prefix debops/ bsdops master --squash
  ```

### Bootstrap host(s)

1. Add your host(s) to the inventory: `$EDITOR ansible/inventory/hosts`

2. Bootstrap your host(s):
  ```
  debops bootstrap <parameters-needed-to-connect> [--limit yourhostname]
  # examples:
  # debops bootstrap -u root --ask-pass  # if you can log in as root with a password
  # debops bootstrap -u admin --ask-pass --become --ask-become-pass  # if you can log in as a user that has sudo
  ```
