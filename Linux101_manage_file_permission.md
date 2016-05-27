## Learn Linux 101 - Manage file permissions and ownership

### Overview
In this note, we will learn to control file access through correct use of file
and directory permissions:

- Manage access permissions on both regular and special files as well as
  directories.
- Maintain security using access modes such as suid, sgid and the sticky
  bit.
- Change the file creation mask
- Grant file access to group members

### User and Group

To start, let's review some basic user and group information.
You can use the `whoami` command to check your current effective id.
```
[ian@attic4-cent ~]$ whoami
ian
[ian@attic4-cent ~]$ ksh -l
$ whoami
ian
$ su jenni
Password: 
[jenni@attic4-cent ian]$ whoami
jenni
[jenni@attic4-cent ian]$ echo "$PS1"
[\u@\h \W]\$ 
[jenni@attic4-cent ian]$ su - mary
Password: 
[mary@attic4-cent ~]$ whoami
mary
```

To check group you are in by using `groups` command.
```
[ian@attic4-cent ~]$ groups
ian development editor
[ian@attic4-cent ~]$ groups jenni
jenni : jenni development
```

### File ownership and permissions
- Regular file
Use `ls -l` command to display the owner and group
```
[ian@attic4-cent ~]$ ls -l /bin/bash .bashrc helloworld.C
-rw-r--r--. 1 ian  ian            124 Oct 16  2014 .bashrc
-rwxr-xr-x. 1 root root        906152 Jul 23 14:55 /bin/bash
-rw-rw-r--. 1 ian  development    116 Aug  8 21:40 helloworld.C
```

The Linux permission model has three types of permissions for each file system
object. The permissions are read(r - 4), write (w - 2) and execute (x -
1). Write permission includes the ability to alter or delete an object. In
addtion, these permissions are specified for the file's owner, members of
file's group, and everyone else.

- Directories
Directories use the same permissions flag as regular file, but they are
interpreted differently.
    - Read permissions for a directory allows a user to list the contents of
      directory.
    - Write permissions means a user can create or delete files in the
      directory
    - Execute permissions allows user to enter the directory and access any
      subdirectories.
      
### Other filesystem objects
| Code | Object                   | 
| ---- |:------------------------:|
| -    | Regular file             |
| d    | Directory                |
| l    | Symbol link              |
| c    | Character special device |
| b    | Block special device     |
| p    | FIFO                     |
| s    | Socket                   |


### Changing permissions

- Adding permissions: You can use `chmod` command to change permissions of
  files or directoryies. Use +x to add execute permissions, +r to add read
  permissions and +w to add write permissions. In fact, you can use any
  combination of r, w and x together. To be more selective, you may prefix the
  mode expression with u to set the permission for users, g to set it for
  groups and o for others. 
  
```
[ian@attic4-cent ~]$ echo 'echo "Hello world!"'>hello2.sh
[ian@attic4-cent ~]$ chmod ug+xw hello2.sh
[ian@attic4-cent ~]$ ls -l hello2.sh
-rwxrwxr--. 1 ian ian 20 Aug  9 06:22 hello2.sh
```

- Removing permissions: Sometimes you need to remove permission rather than
  add them. Simply change the + to a - and you remove any of the specified
  permissions that are set.
  
- Setting permissions: If you want to set different permissions for user,
  group, or other, you can seperate different expressions by commas - for
  example, ug=rwx,o=rx.
  
### Access Modes
- suid and sgid: The Linux permissions model has two special access modes
  called suid (set user id) and sgid (set group id). When an executable
  program has the suid access modes set, it will run as if it had been started
  by the file's owner, rather than by the user who really started
  it. Similarly, with sgid access modes set, the program will run if the
  initiating user belonged to the file's group rather than to his own group.
  
```
[ian@attic4-cent ~]$ ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 30768 Feb 22  2012 /usr/bin/passwd
```
As the above example, the suid and sgid bits occupy the same space as the x
bits for user and group. If the file is executable, the suid and sgid bits, if
set, it will be displayed as lowercase s. Otherwise, they are displayed as
uppercase S.

- Setting suid and sgid: The suid and sgid are set and reset using the letter
  s. For example, u+s sets the suid access mode, g-s removes the sgid mode.
  
- Directories and sgid: When a directory has the sgid mode enabled, any files
  or directories created in it will inherit the group ID of directory. This
  useful for people working on the same project. But it can't prevent files
  deletion each other. Fortunately, there is a solution - that is `sticky bit`
  
- The sticky bit: It is represented symbolically by t and numberically as a 1
  in the high-order octal digit. It is displayed in the place of executable
  flag for other users. If set for directory, it permits only the owning user
  or the supper user (root) to delete or unlink a file.
  
```
[greg@attic4-cent ~]$ chmod +t lpi101
[greg@attic4-cent ~]$ ls -ld lpi101 /tmp
drwxrwsr-t.  2 greg development 4096 Aug  9 06:51 lpi101
drwxrwxrwt. 42 root root        4096 Aug  9 06:53 /tmp
```

- Access mode summary

| Access mode | Symbolic | Octal |
| ----------- |:--------:|:-----:|
| suid        | s with u | 4000  |
| sgid        | s with g | 2000  |
| sticky      | t        | 1000  |

Printing symbolic and octal permissions:
```
[greg@attic4-cent ~]$ find . -name lpi101  -printf "%M %m %f\n"
drwxrwsr-t 3775 lpi101
```

- Immutable files: It is also know is the immutable attribute. If this is set,
  even root cannot delete the file until the attribute is unset. We can use
  `lsattr` command to see whether the immutable flag (or any other attribute)
  is set for a file or directory. To make a file immutable, use the `chattr`
  command with -i flag.

```
[root@attic4-cent ~]# touch keep.me
[root@attic4-cent ~]# chattr +i keep.me
[root@attic4-cent ~]# lsattr keep.me
----i---------- keep.me
[root@attic4-cent ~]# rm -f keep.me
rm: cannot remove `keep.me': Operation not permitted
[root@attic4-cent ~]# chattr -i keep.me
[root@attic4-cent ~]# rm -f keep.me
```

### The file creation mask
When the new file is created, the creation process specifies the permissions
(default) that new file should have. Often, the mode requested is 0666 for
files and 0777 for directory. However, this permissive creation is affected
by a `umask` value. The system uses the umask value to reduce the originally
requested permissions.
```
[ian@attic4-cent ~]$ umask
022
```
Remember that the umask value specifies which permissions should not be
granted. 0022 which removes group and other write permission from new files,
while 0002 which removes write permission for other users. Use -S option to
display symbollically.

```
[ian@attic4-cent ~]$ umask -S
u=rwx,g=rwx,o=rx
[ian@attic4-cent ~]$ umask u=rwx,g=,o=
[ian@attic4-cent ~]$ umask
0077
[ian@attic4-cent ~]$ touch newfile
[ian@attic4-cent ~]$ ls -l newfile
-rw-------. 1 ian ian 0 Aug  9 07:09 newfile
```

Source: http://www.ibm.com/developerworks/library/l-lpic1-104-5/index.html
