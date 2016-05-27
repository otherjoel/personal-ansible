# personal-ansible

Ansible playbook(s) for personal servers. They automate my (not terribly unique) routine for setting up a new Ubuntu server:

* ‘common’ role: apply basic security hardening
* ‘webserver’ role: install and configure web server software *(coming soon)*

Many of the steps used in these playbooks are explained in more detail by these two posts:

 * [My First 5 Minutes On A Server](https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers) by Bryan Kennedy
 * [How to Set Up Your Linode for Maximum Awesomeness](http://feross.org/how-to-setup-your-linode/) by Feross Aboukhadijeh

I’m also making use of `ansible-vault` for keeping server credentials secure while still including them in the repo.

## Usage

Ensure workstation and VMs are configured according to ‘Setup’ (below).

When running against VMs for the first time:

    ansible-playbook site-initonce.yml -e "ansible_port=22" --ask-vault-pass

To run against a *single VM* for the first time (note ending comma)

    ansible-playbook site-initonce.yml -i "machine1," -e "ansible_port=22" --ask-vault-pass

Because this will disable root and password use in ssh as well change the ssh port, you should only be able to (and only need to) use this once.

For all subsequent runs to the same server(s):

    # ansible-playbook site.yml --ask-vault-pass

To test without actually performing any changes, use the `-C` flag.

### Shortcut script

Because all of that is a bit annoying to type, I’ve created a basic bash script `ap` to make it simpler.

    Usage: ap [-c] [-i] [-h host1,host2]" [-p playbook.yml]

      -c        Runs ansible-playbook in Check mode (-C)
      -i        Initialize (use site-initonce.yml w/port 22 and root login)
      -p file   Specify a playbook (can be overridden if followed by -i)
      -h hosts  Limit to specified hosts (comma-separated)

Example usage:

    ./ap -i                # Runs site-initonce.yml with port 22
    ./ap -i -h host1       #  -- same as above, limit to one host
    ./ap                   # Runs site.yml
    ./ap -h host1,host2    #  -- same as above, limit to two hosts

    ./ap -p web.yml        # Specifies a playbook
    ./ap -c                # Runs ansible in Check mode (-C)
    ./ap -ci -h host1      # Check mode, init playbook...you get the idea

## Setup

### Virtual Machines

These playbooks assume:

* Ubuntu 14.04 LTS
* Passwordless login via public key is already configured for root. (Most cloud providers will allow you to select public keys to be added to root when you provision the VM.)

### Local Environment (OS X)

1. Install ansible: `brew install ansible`

2. Install roles from galaxy (these get put into `/usr/local/etc/ansible`):
    * `ansible-galaxy install geerlingguy.apache`

3. Ensure `~/.ssh/id_rsa.pub` exists (generate if not)

3. Create the inventory of your servers:
    * `mkdir -p /usr/local/etc/ansible`
    * Create `/usr/local/etc/ansible/hosts` and add your servers ([instructions](http://docs.ansible.com/ansible/intro_inventory.html))
    * You may wish to `ln -s /usr/local/etc/ansible/hosts ./inv` for convenient editing of the inventory

4. Edit vars files under `group_vars/all` to match your setup

5. Set variables for passwords, port numbers, etc. with `ansible-vault edit group_vars/all/secret.yml`
    * Password vars with a `_PLAIN` suffix are the plain text of the password; other password vars are **hashes of the password**. You will need to generate these hashes using [one of these methods](http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module).
    * If anyone besides me uses this, you will need to replace `group_vars/all/secret.yml` with your own Ansible vault. Just make sure it includes all the vars with the `vault_` prefix referenced in `group_vars/all/vars.yml`.
    * You should have an Ansible vault password stored in your password manager (1Password, KeePass, etc).

## Files

Folders or files      |Description
----------------------|------------
`ap`                  |Shortcut script for using `ansible-playbook` without having to type out all the options (see above)
`site-initonce.yml`   |Playbook for new VMs (root login)
`site.yml`            |Playbook for all servers
`webservers.yml`      |Playbook for web servers
`group_vars/all/*.yml`|These vars are automatically loaded for all playbooks
`roles/`              |Any `main.yml` files are added to specific parts of the play (tasks, handlers, etc)  automatically for that role. Any files under `role/x/files/` can be referenced without specifying a path. ([more info](http://docs.ansible.com/ansible/playbooks_roles.html#roles))

## Planned Improvements

* Tasks for installing and renewing Let’s Encrypt SSL certs
   * May wait until EFF Certbot client is out of beta
   * Dependency installation
* Tasks for restoring specific sites from backup
