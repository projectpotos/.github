# Contributing to Potos

Thank you for considering contributing to Potos. Potos is a series of [Ansible Roles](https://github.com/orgs/projectpotos/repositories?q=ansible-role), [some magic](https://github.com/projectpotos/ansible-plays-potos) and a client specific repo ([Potos Example](https://github.com/projectpotos/ansible-specs-potos)) to create an enterprise ready Linux Client. Before submitting a contribution, please take some time to read the following content to ensure that the contribution is in line with the specification and can help the community.

## Bugs
### Where to find Known Issues
We are using Github Issues for our public bugs. We keep a close eye o nthis and try to make it clear when we have an internal fix in progress. Before filing a new task, try to make sure your problem doesn't already exist.

### Reporting New Issues
The best way to get your bug fixed is to provide a clear guide on how to reproduce the issue.

## How to contribute

Best have a look at the open [issues](issues) and open a [Github pull request](compare).

 * Fork the repo you'd like to make a contribution to
 * Clone your fork to your local workstation
 * Create a new branch for the issue
 * Make the necessary changes on that branch
   * Ensure the new code is tested with Github Action / Molecule
 * Commit that change
   * We'd love to see if you [sign your commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
   * We try to follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
 * Push that branch to github
 * Make a Pull Request against the repo

### Semantic Versioning
Potos follows semantic versioning. We release patch versions for critical bugfixes, minor versions for new features or non-essential changes, and major versions for any breaking changes. When we make breaking changes, we also introduce deprecations warning in a minor version so that our users learn about the upcoming changes and migrate their code in advance. Learn more about our commitment to stability and incremental migration in our versioning policy.

Every significant change is documented in the changelog file.

### Branch Organization
Submit all changes directly to the main branch. We don't use separate branches for developing or upcoming releases. We do our best to keep `main` in good shapre, with all tests. passing.

Code that lands in `main` must be compatible with the latest stable release. It may contain additional features, but no breaking changes. We should be able to release a new minor version from the tip of main at any time.

## Development Environment
Probably the best way to setup a development environment is to create a virtual machine running with Potos. 

Until we have the iso creation process available, one can use the following process to convert a plain Ubuntu 22.04 system into a Potos Client.

* Install the requirements
```bash
sudo apt install git python3-pip
sudo pip3 install ansible-core
```

* Create the client specification file with the spec repository you want to use
```bash
sudo mkdir -p /etc/potos
sudo cat > /etc/potos/specs_repo.yml <<EOF
---
client_name: "Potos Vanilla"
git_url: "https://github.com/projectpotos/"
git_repo: "ansible-specs-potos"
git_branch: "main"
EOF
```

* create a temporary directory for the initial ansible run
```bash
sudo git clone https://github.com/projectpotos/ansible-plays-potos.git
sudo cd ansible-plays-potos
sudo ansible-playbook prepare.yml
sudo ansible-playbook playbook.yml
```

* Now you have a Potos Linux Client with all default
---
* For testing your own role you probably want to create a test playbook derrived from the normal playbook but without dynamic role downloading and executin i.e. something like:
```yaml
---

- name: Potos test playbook
  hosts: localhost
  connection: local
  become: True
  gather_facts: True
  ignore_errors: False

  vars:
    # define default run type
    potos_runtype: 'daily'

  tasks:
    - name: run all the required roles
      ansible.builtin.include_role:
        name: '{{ potos_playbook_role }}'
        apply:
          tags:
            - always
      loop:
        - 'role_to_test'
      loop_control:
        loop_var: 'potos_playbook_role'
```

* For running the molecule tests locally you need a few other dependencies installed:
```bash
# install requirements to add new repository
sudo apt update
sudo apt install ca-certificates curl gnupg sb-release -y

# add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
# setup the repository 
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# update repos and install docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
# install molecule
sudo pip3 install molecule molecule-docker
```
* Run the test with
```bash
sudo molecule test
```
