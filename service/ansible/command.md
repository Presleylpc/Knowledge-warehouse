# command

## [Synopsis](https://docs.ansible.com/ansible/latest/modules/command_module.html#id1)

- The `command` module takes the command name followed by a list of space-delimited arguments.
- The given command will be executed on all selected nodes. It will not be processed through the shell, so variables like `$HOME` and operations like `"<"`, `">"`, `"|"`, `";"` and `"&"` will not work (use the [shell](https://docs.ansible.com/ansible/latest/modules/shell_module.html#shell-module) module if you need these features).
- For Windows targets, use the [win_command](https://docs.ansible.com/ansible/latest/modules/win_command_module.html#win-command-module) module instead.

## [Parameters](https://docs.ansible.com/ansible/latest/modules/command_module.html#id2)

| Parameter                     | Choices/Defaults        | Comments                                                     |
| ----------------------------- | ----------------------- | ------------------------------------------------------------ |
| **argv**  (added in 2.6)      |                         | Allows the user to provide the command as a list vs. a string. Only the string or the list form can be provided, not both. One or the other must be provided. |
| **chdir**                     |                         | Change into this directory before running the command.       |
| **creates**                   |                         | A filename or (since 2.0) glob pattern. If it already exists, this step **won't** be run. |
| **free_form**  required       |                         | The command module takes a free form command to run. There is no parameter actually named 'free form'. See the examples! |
| **removes**                   |                         | A filename or (since 2.0) glob pattern. If it already exists, this step **will** be run. |
| **stdin**  (added in 2.4)     |                         | Set the stdin of the command directly to the specified value. |
| **warn**  bool (added in 1.8) | **Choices:**no**yes** ← | If command_warnings are on in ansible.cfg, do not warn about this particular line if set to `no`. |

## [Notes](https://docs.ansible.com/ansible/latest/modules/command_module.html#id3)

Note

- If you want to run a command through the shell (say you are using `<`, `>`, `|`, etc), you actually want the [shell](https://docs.ansible.com/ansible/latest/modules/shell_module.html#shell-module) module instead. Parsing shell metacharacters can lead to unexpected commands being executed if quoting is not done correctly so it is more secure to use the `command` module when possible.
- `creates`, `removes`, and `chdir` can be specified after the command. For instance, if you only want to run a command if a certain file does not exist, use this.
- Check mode is supported when passing `creates` or `removes`. If running in check mode and either of these are specified, the module will check for the existence of the file and report the correct changed status. If these are not supplied, the task will be skipped.
- The `executable` parameter is removed since version 2.4. If you have a need for this parameter, use the [shell](https://docs.ansible.com/ansible/latest/modules/shell_module.html#shell-module)module instead.
- For Windows targets, use the [win_command](https://docs.ansible.com/ansible/latest/modules/win_command_module.html#win-command-module) module instead.

## [Examples](https://docs.ansible.com/ansible/latest/modules/command_module.html#id4)

```
- name: return motd to registered var
  command: cat /etc/motd
  register: mymotd

- name: Run the command if the specified file does not exist.
  command: /usr/bin/make_database.sh arg1 arg2
  args:
    creates: /path/to/database

# You can also use the 'args' form to provide the options.
- name: This command will change the working directory to somedir/ and will only run when /path/to/database doesn't exist.
  command: /usr/bin/make_database.sh arg1 arg2
  args:
    chdir: somedir/
    creates: /path/to/database

- name: use argv to send the command as a list.  Be sure to leave command empty
  command:
  args:
    argv:
      - echo
      - testing

- name: safely use templated variable to run command. Always use the quote filter to avoid injection issues.
  command: cat {{ myfile|quote }}
  register: myoutput
```

## [Return Values](https://docs.ansible.com/ansible/latest/modules/command_module.html#id5)

Common return values are documented [here](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html#common-return-values), the following are the fields unique to this module:

| Key               | Returned | Description                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| **cmd**  list     | always   | the cmd that was run on the remote machine **Sample:**['echo', 'hello'] |
| **delta**  string | always   | cmd end time - cmd start time **Sample:**0.001529            |
| **end**  string   | always   | cmd end time **Sample:**2017-09-29 22:03:48.084657           |
| **start**  string | always   | cmd start time **Sample:**2017-09-29 22:03:48.083128         |