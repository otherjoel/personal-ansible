# Notes

Things I had to figure out that weren’t well-documented or were otherwise not obvious.

## Ansible stuff

What’s the best way to use roles made by other people?
: Use `ansible-galaxy`. The roles get downloaded to `/usr/local/etc/ansible` so they don’t have to become part of your playbook’s repo.

Does the `get_url` module treat the destination as a folder when specified as a filename with no extension?
: No (which is good)! This is important because it means you can use this module with an extension-less file and it won’t re-download the file every time.

The [docs for the `get_url` module](http://docs.ansible.com/ansible/get_url_module.html) mention a `mode` option in the examples which is not documented in the Options section. Does this option accept `a+x` style permissions as well as numeric ones?
: Yes. Also, the docs don’t explicitly say so, but it looks like `get_url` probably supports most of the same options as [the `file` module](http://docs.ansible.com/ansible/file_module.html).

How do I convert a list variable into a comma-separated string for use as a command-line argument?
: Like this: `{{ listvar|join(',') }}`  ([Source](https://www.sbarjatiya.com/notes_wiki/index.php/Convert_list_variable_to_comma_separated_list_in_ansible))

Can a role refer to a handler from another role?
: See <http://stackoverflow.com/questions/22649333/ansible-notify-handlers-in-another-role>. The simplest way is if you are in a play that includes both roles, then any of the included roles can refer to any other role’s handlers. I ended up not going down that road, however.

## Certbot stuff

How idempotent is `certbot-auto`?
: Certbot [claims](https://certbot.eff.org/docs/intro.html) that “since `certbot-auto` is a wrapper to `certbot`, it accepts exactly the same command line flags and arguments.” This is true with one important (and undocumented) exception: the first time it is run it will only take a limited number of flags. I get around this by testing for the existence of `/etc/letsencrypt/options-ssl-apache.conf` and running `certbot-auto --noninteractive` once if that file does not exist, then running it with the complete set of options.

Will certbot create a new certificate each time it is called?
: Nope. By keeping an eye on the key fingerprint, I’ve found it can be called repeatedly without grabbing a new certificate each time.
