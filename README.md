# bsdops

DebOps-derived reusable, integrated Ansible configs for FreeBSD-based machines.

**Note: This is currently in a very early stage of development. Currently only `debops bootstrap` is fully "FreeBSD-ified", and "extensive testing" is a dream.**

## Why should I use this, especially when I know it will break?

Because:

1. you should use configuration management
2. Ansible is a decent configuration management option, so you likely want to use that
3. If you're going to use Ansible, you will have to write your own roles, because there is no "ready-made" option for FreeBSD
4. If you're going to write your own roles, you might as well start with a decent base that can benefit other people too, such as this one :-)

## How to use

### 1. Install

1.  Install `debops` normally:
    * https://docs.debops.org/en/master/user-guide/install.html
    * I prefer to install it into a venv: `pipenv install ansible debops`

2.  Create a project directory for your configuration:
    ```
    mkdir my-project
    cd my-project
    # if you installed with pipenv: pipenv shell
    debops-init .
    ```

3.  Tell `debops` to use [my fork](https://github.com/AnotherKamila/debops) instead of the official DebOps repo: put it into a `debops/` subdirectory:
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
    
    (this repository uses `git subrepo`)

### 2. Bootstrap host(s)

1.  Add your host(s) to the inventory: `$EDITOR ansible/inventory/hosts`
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
      
      (this step will not be necessary in a future version, see #1)

2.  Bootstrap your host(s):
    Make sure that you can log in without password (`--ask-pass` doesn't work with FreeBSD).  
    For example: `ssh-copy-id myhost`.
    Then run:
    ```
    debops bootstrap <parameters-needed-to-connect> [--limit myhost]
    # examples:
    # debops bootstrap -u root  # if you can log in as root
    # debops bootstrap -u admin --become --ask-become-pass  # if you can log in as a user that has sudo
    ```

### 3. Use DebOps roles to effortlessly set up your host(s)

This feature will be available in a future version :D

Seriously though: This project is a WIP and so far I have only "FreeBSD-ified" the roles used in the `bootstrap` playbook. Other roles might or might not work (often e.g. paths or package names might be wrong). However, patching these higher-level roles should be relatively straight-forward in many cases. So: Try it, see what breaks, and send a pull request!

The next roles I intend to convert are generic basic things like firewalling. My use-case is a router box, so the things I'll work on next will be going in that direction. If you want something else, I'll happily accept pull requests :-)

See [Common Differences](https://github.com/AnotherKamila/bsdops/wiki/Common-Differences) for a quick list of things which are often different on Linux vs FreeBSD and how I handle them when FreeBSD-ifying roles.

### Configuration

Everything is the same as DebOps, so in theory this section is unnecessary. However, as a quick reference, this is a list of things one might want to change often:

* `bootstrap__admin_system`: set to False to have the automatically created admin user get a home directory under `/home` instead of under `/usr/local`.

### Compatibility with Debian-based hosts

Ideally, my DebOps fork should work the same as original DebOps on non-FreeBSD hosts -- the idea is to add new functionality without breaking existing stuff. That said, I do not currently have the capacity to test that much. So, it may break. But if it does break, it's a bug, so please report it (ideally with a patch :-)).

# Development

**This repo contains the documentation (so, this README) and an example debops project that you can use as a starting point. The repo with my patched version of debops lives at [anotherkamila/debops](https://github.com/AnotherKamila/debops).**

Development workflow with `git subrepo`:

```
git subrepo clone git@github.com:anotherkamila/debops.git debops/  # use your fork
[hack hack]
[git commit ...]
git subrepo push debops
git push

```

Updating `debops` from upstream with minimum suffering:

```
git subrepo push debops
cd WHEREVER_I_CHECKED_OUT_DEBOPS_REPO_AS_TOPLEVEL
git pull origin master
git fetch --all
git merge upstream/master
git push
cd BACK_TO_HERE
git subrepo pull debops
```

# License

The contents of this repository EXCLUDING the `debops/` directory is licensed under BSD 2-clause license, as indicated by the LICENSE file.

As DebOps is licensed under GPL, the `debops/` directory and my changes in it are licensed under GPL. My apologies.
