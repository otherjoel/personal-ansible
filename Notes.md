# Notes

Things I had to figure out that weren’t well-documented:

What’s the best way to use roles made by other people?
: Use `ansible-galaxy`. The roles get downloaded to `/usr/local/etc/ansible` so they don’t have to become part of your playbook’s repo.

Does the `get_url` module treat the destination as a folder when specified as a filename with no extension?
: No (which is good)! This is important because it means you can use this module with an extension-less file and it won’t re-download the file every time.

The [docs for the `get_url` module](http://docs.ansible.com/ansible/get_url_module.html) mention a `mode` option in the examples which is not documented in the Options section. Does this option accept `a+x` style permissions as well as numeric ones?
: Yes. Also, the docs don’t explicitly say so, but it looks like `get_url` probably supports most of the same options as [the `file` module](http://docs.ansible.com/ansible/file_module.html).

How do I convert a list variable into a comma-separated string for use as a command-line argument?
: Like this: `{{ listvar|join(',') }}`  ([Source](https://www.sbarjatiya.com/notes_wiki/index.php/Convert_list_variable_to_comma_separated_list_in_ansible))
