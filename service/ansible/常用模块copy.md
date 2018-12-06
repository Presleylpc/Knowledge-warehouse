# copy - Copies files to remote locations

## [Synopsis](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id1)

- The `copy` module copies a file from the local or remote machine to a location on the remote machine. Use the [fetch](https://docs.ansible.com/ansible/latest/modules/fetch_module.html#fetch-module) module to copy files from remote locations to the local box. If you need variable interpolation in copied files, use the [template](https://docs.ansible.com/ansible/latest/modules/template_module.html#template-module) module.
- For Windows targets, use the [win_copy](https://docs.ansible.com/ansible/latest/modules/win_copy_module.html#win-copy-module) module instead.

## [Parameters](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id2)

| Parameter                              | Choices/Defaults        | Comments                                                     |
| -------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| **attributes**  (added in 2.3)         |                         | Attributes the file or directory should have. To get supported flags look at the man page for *chattr* on the target system. This string should contain the attributes in the same order as the one displayed by *lsattr*.`=` operator is assumed as default, otherwise `+` or `-` operators need to be included in the string. aliases: attr |
| **backup**  bool                       | **Choices:****no** ←yes | Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly. |
| **checksum**  (added in 2.5)           |                         | SHA1 checksum of the file being transferred. Used to validate that the copy of the file was successful.If this is not provided, ansible will use the local calculated checksum of the src file. |
| **content**                            |                         | When used instead of *src*, sets the contents of a file directly to the specified value. For anything advanced or with formatting also look at the template module. |
| **decrypt**  bool (added in 2.4)       | **Choices:**no**yes** ← | This option controls the autodecryption of source files using vault. |
| **dest**  required                     |                         | Remote absolute path where the file should be copied to. If *src* is a directory, this must be a directory too. If *dest* is a nonexistent path and if either *dest*ends with "/" or *src* is a directory, *dest* is created. If *src* and *dest* are files, the parent directory of *dest* isn't created: the task fails if it doesn't already exist. |
| **directory_mode** (added in 1.5)      |                         | When doing a recursive copy set the mode for the directories. If this is not set we will use the system defaults. The mode is only set on directories which are newly created, and will not affect those that already existed. |
| **follow**  bool (added in 1.8)        | **Choices:****no** ←yes | This flag indicates that filesystem links in the destination, if they exist, should be followed. |
| **force**  bool                        | **Choices:**no**yes** ← | the default is `yes`, which will replace the remote file when contents are different than the source. If `no`, the file will only be transferred if the destination does not exist. aliases: thirsty |
| **group**                              |                         | Name of the group that should own the file/directory, as would be fed to *chown*. |
| **local_follow**  bool (added in 2.4)  | **Choices:**no**yes** ← | This flag indicates that filesystem links in the source tree, if they exist, should be followed. |
| **mode**                               |                         | Mode the file or directory should be. For those used to */usr/bin/chmod*remember that modes are actually octal numbers. You must either add a leading zero so that Ansible's YAML parser knows it is an octal number (like `0644` or `01777`) or quote it (like `'644'` or `'1777'`) so Ansible receives a string and can do its own conversion from string into number. Giving Ansible a number without following one of these rules will end up with a decimal number which will have unexpected results. As of version 1.8, the mode may be specified as a symbolic mode (for example, `u+rwx` or `u=rw,g=r,o=r`). As of version 2.3, the mode may also be the special string `preserve`. `preserve`means that the file will be given the same permissions as the source file. |
| **owner**                              |                         | Name of the user that should own the file/directory, as would be fed to *chown*. |
| **remote_src**  bool (added in 2.0)    | **Choices:****no** ←yes | If `no`, it will search for *src* at originating/master machine.If `yes` it will go to the remote/target machine for the *src*. Default is `no`.Currently *remote_src* does not support recursive copying.*remote_src* only works with `mode=preserve` as of version 2.6. |
| **selevel**                            | **Default:** s0         | Level part of the SELinux file context. This is the MLS/MCS attribute, sometimes known as the `range`. `_default` feature works as for *seuser*. |
| **serole**                             |                         | Role part of SELinux file context, `_default` feature works as for *seuser*. |
| **setype**                             |                         | Type part of SELinux file context, `_default` feature works as for *seuser*. |
| **seuser**                             |                         | User part of SELinux file context. Will default to system policy, if applicable. If set to `_default`, it will use the `user` portion of the policy if available. |
| **src**                                |                         | Local path to a file to copy to the remote server; can be absolute or relative. If path is a directory, it is copied recursively. In this case, if path ends with "/", only inside contents of that directory are copied to destination. Otherwise, if it does not end with "/", the directory itself with all contents is copied. This behavior is similar to Rsync. |
| **unsafe_writes**  bool (added in 2.2) | **Choices:****no** ←yes | By default this module uses atomic operations to prevent data corruption or inconsistent reads from the target files, but sometimes systems are configured or just broken in ways that prevent this. One example is docker mounted files, which cannot be updated atomically from inside the container and can only be written in an unsafe manner.This option allows Ansible to fall back to unsafe methods of updating files when atomic operations fail (however, it doesn't force Ansible to perform unsafe writes). IMPORTANT! Unsafe writes are subject to race conditions and can lead to data corruption. |
| **validate**                           |                         | The validation command to run before copying into place. The path to the file to validate is passed in via '%s' which must be present as in the example below. The command is passed securely so shell features like expansion and pipes won't work. |

## [Notes](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id3)

Note

- The [copy](https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module) module recursively copy facility does not scale to lots (>hundreds) of files. For alternative, see [synchronize](https://docs.ansible.com/ansible/latest/modules/synchronize_module.html#synchronize-module) module, which is a wrapper around `rsync`.
- For Windows targets, use the [win_copy](https://docs.ansible.com/ansible/latest/modules/win_copy_module.html#win-copy-module) module instead.

## [Examples](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id4)

```
- name: example copying file with owner and permissions
  copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: 0644

- name: The same example as above, but using a symbolic mode equivalent to 0644
  copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: u=rw,g=r,o=r

- name: Another symbolic mode example, adding some permissions and removing others
  copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: u+rw,g-wx,o-rwx

- name: Copy a new "ntp.conf file into place, backing up the original if it differs from the copied version
  copy:
    src: /mine/ntp.conf
    dest: /etc/ntp.conf
    owner: root
    group: root
    mode: 0644
    backup: yes

- name: Copy a new "sudoers" file into place, after passing validation with visudo
  copy:
    src: /mine/sudoers
    dest: /etc/sudoers
    validate: /usr/sbin/visudo -cf %s

- name: Copy a "sudoers" file on the remote machine for editing
  copy:
    src: /etc/sudoers
    dest: /etc/sudoers.edit
    remote_src: yes
    validate: /usr/sbin/visudo -cf %s

- name: Copy using the 'content' for inline data
  copy:
    content: '# This file was moved to /etc/other.conf'
    dest: /etc/mine.conf'
```

## [Return Values](https://docs.ansible.com/ansible/latest/modules/copy_module.html#id5)

Common return values are documented [here](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html#common-return-values), the following are the fields unique to this module:

| Key                     | Returned                  | Description                                                  |
| ----------------------- | ------------------------- | ------------------------------------------------------------ |
| **backup_file**  string | changed and if backup=yes | name of backup file created **Sample:**/path/to/file.txt.2015-02-12@22:09~ |
| **checksum**  string    | success                   | sha1 checksum of the file after running copy **Sample:**6e642bb8dd5c2e027bf21dd923337cbb4214f827 |
| **dest**  string        | success                   | destination file/path **Sample:**/path/to/file.txt           |
| **gid**  int            | success                   | group id of the file, after execution **Sample:**100         |
| **group**  string       | success                   | group of the file, after execution **Sample:**httpd          |
| **md5sum**  string      | when supported            | md5 checksum of the file after running copy **Sample:**2a5aeecc61dc98c4d780b14b330e3282 |
| **mode**  string        | success                   | permissions of the target, after execution **Sample:**420    |
| **owner**  string       | success                   | owner of the file, after execution **Sample:**httpd          |
| **size**  int           | success                   | size of the target, after execution **Sample:**1220          |
| **src**  string         | changed                   | source file used for the copy on the target machine **Sample:**/home/httpd/.ansible/tmp/ansible-tmp-1423796390.97-147729857856000/source |
| **state**  string       | success                   | state of the target, after execution **Sample:**file         |
| **uid**  int            | success                   | owner id of the file, after execution **Sample:**100         |