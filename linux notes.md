# Linux Notes

## Docker and Command Line Tips

```bash
yes | docker system prune  # yes | is used to give input to the upcoming request
```

- `ctl + r` -- to search the recent commands
- `which ls` or any command -- give the path of the binary
- Can also edit files using `echo \"\" > filename` ||| `cat >> filename` and type the matter and exit using `ctl + d`
- `sudo -i` -- to switch user to root
- `hostnamectl set-hostname cicdserver` -- to set hostname
- `cat /etc/os-release` to know os info
- `du -sh *` -- to know the size of the files in pwd

```bash
sudo timedatectl set-timezone Asia/Kolkata  # to set time to Indian time

aws s3 cp /path/to/local/file.txt s3://your-bucket-name/
```

- `file filename` -- to find the type of file
- When `yum install docker` or any is written it uses the rpm

## RPM Commands (Red Hat Package Manager)

```bash
rpm -ivh package.rpm  # to install a package
rpm -Uvh package.rpm  # to update a package
rpm -qa  # to list the packages
rpm -qa | grep docker  # to find a package
rpm -qi packagename  # to view the info of the package
```

## File System

- `/` -- root dir
- `~` -- `/home/user` home dir of the user logged in

### Directory Descriptions

- `/sbin` -- system binaries   pre installed system binaries only root users can use
- `/lib` -- Shared libraries and kernel modules
- `/boot` -- Stores files needed for booting the system
- `/bin` -- user binaries any user can use without admin access
- `/srv` -- Holds data for services like web servers (rarely used in containers)
- `/var` -- Stores logs, caches, and temporary files that change frequently
- `/etc` -- Stores system configuration files
- `/opt` -- Used for installing optional third-party software
- `/tmp` -- Temporary files (cleared on reboot)
- `/run` -- Holds runtime data for processes
- `/proc` -- Virtual filesystem for process and system information
- `/sys` -- Virtual filesystem for hardware and kernel information
- `/dev` -- Contains device files (e.g., `/dev/null`, `/dev/sda`)
- `/mnt` -- Temporary mount point for external filesystems
- `/data` -- Likely your mounted volume from Windows (C:/ubuntu-data)

### File Management Commands

```bash
ls - list -l - long list
ls -lart -- to list the dir
ls -lh my_archive.tar.gz -- to know the size of particular file
cd - change directory
mkdir - make directory
rm - remove file
rmdir - remove directory
rm -rf - to force remove a directory || rm * to remove everything
touch - create an empty file
cat - execute a file
cp - cp filename filename it is used to copy one file content to other
cp -f -- to copy a dir
mv - mv filename directory/filename is creates same file in another direcotory
cd ~- - back in previous directory
vi - create or edit an file
w to move forward one word i to insert and edit the file
escape :wq! to exit
```

- `sudo -i` -- to switch user to root
- `ctl c` to stop the process
- escape `yy` to copy and `p` to paste and `dd` to delete
- `jkhl` to move
- `whoami` - to know the user
- `id` - to know the id

## User Management

Generally in Linux root has every permission and that can also be achived by adding users to the sudoers group

Now everytime we install an application an group will be created and we have to add user to that group to get permission to run application

A user can be grouped to multiple groups so user can have multiple permmisons like dev docker and Jenkins etc

This user has access to data created by dev docker and Jenkins

Dev is also a group created by admin and added users to it so what ever the data or actions that user perform can be read and accessed by user in that group

### 1. Root User and Sudoers

The root user is the superuser in Linux. It has complete access to all files, commands, and system operations.

For security and best practices, we don't log in as root. Instead, we give admin privileges to other users by adding them to the sudoers group (usually achieved by adding to the sudo or wheel group, depending on the distro).

```bash
sudo usermod -aG sudo username
```

### 2. Application-Specific Groups

When you install certain applications (like Docker, Jenkins, etc.), they often create their own groups during installation.
These groups control who can interact with the application safely.
To grant a user permission to use a specific application, the user must be added to the application's group.

Example: To allow user john to use Docker:
```bash
sudo usermod -aG docker john
```

### 3. Users Belonging to Multiple Groups

A user can be a part of multiple groups, allowing flexible permission management.

For example:
- The user alice can be part of:
  - dev – for development permissions.
  - docker – to use Docker without sudo.
  - jenkins – to manage Jenkins jobs/pipelines.

This means that:
✅ She can access files created by these groups.
✅ She can perform actions allowed by these groups.

### 4. Group-Based File and Directory Access

Groups are not just for applications—they're a general way to share access across users.

An admin can create a group like dev for development teams.
Any file or directory assigned to group dev with proper group permissions can be accessed by its members.

To list groups a user belongs to: `groups alice`

## User Management Commands

```bash
adduser (username)  # used to add a user | useradd command created user but doesn't create a home dir  useradd -m username -- to create user with home dir
passwd (username)  # to create a password

/etc/passwd -- contains the list of users created

vi /etc/sudoers to go into into the sudo file and have to copy and edit root to user name

su username to switch user in Linux
sudo -i to switch to root user in ec2 server

# to switch user:
su - username

# the user info will be stored in /etc/passwd file you can view using cat command
# and the password will be stored in /etc/shadow file

groupadd name -- to create a group

# modifying user accounts:
/etc/group to view group files

usermod to change group

usermod -a -G 1000(code) name
usermod -aG groupname username -- to add a user to the group
usermod -aG sudo username -- to add user to the sudoer group

# setting up an expiry date for an account:
usermod -e (date) acname

# changing user shell:
usermod -s /bin/sh name

# to delete a user account:
userdel username -- to delete user
userdel -r name -- to delete user with home dir

# to force password changes:
chage -m(minimum)1 -M(maximum)90 or use chage man to show Manuel
# and to view
chage -l nameofaccount
```

- `/etc/passwd` – Stores user account details.
- `/etc/shadow` – Stores encrypted user passwords.
- `/etc/group` – Stores group information.
- `/etc/gshadow` – Stores secure group details.

To connect to a user with username and password and ip without pem file is:

```bash
ssh username@ip  # and then prompt the password
```

## File Permissions

If you want to search for a word in vi mode then use `esc /(word)`

| File or Dir | User | Group | Other |
|-------------|------|-------|-------|
| -           | ---  | ---   | ---   |

### File Permissions

- read - 4                        read only - 400
- write - 2                       write only - 200
- execute - 1
- no permission - 0

- Owner (User): The creator of the file.
- Group: Users belonging to the assigned group.
- Others: All other users on the system.

### Modify Permissions Using Symbols

Add (+), remove (-), or set (=) permissions.
Examples:

```bash
chmod u+x filename  # Add execute for user
chmod g-w filename  # Remove write for group
chmod o=r filename  # Set read-only for others
chmod u=rwx,g=rx,o= filename  # Set full access for user, read/execute for group, and no access for others

chmod 755 filename  # User (rwx), Group (r-x), Others (r-x)
chmod 644 filename  # User (rw-), Group (r--), Others (r--)
chmod 700 filename  # User (rwx), No access for others
```

To change permission use chmod (change modification) - `chmod 777 filename` or directory name

Example: if a Linux user leaves the org and need other user to manage the files we just change the owner permission using chmod:

```bash
chown -R username:groupname file/folder  # -R means it will apply to all files in the directory
```

## Networking Commands

```bash
ping google.com  # Checks connectivity to a remote server.
ifconfig  # Displays network interfaces (deprecated, use ip).
ip a  # Shows IP addresses of network interfaces.
netstat -tulnp  # Displays open network connections.
curl https://example.com  # Fetches a webpage's content.
wget https://example.com/file.zip  # Downloads a file from the internet.
```

## Package Management

```bash
yum install (package name) -y  # (to force yes)

yum list installed  # to list the install packages (ex: yum list installed | grep -i httpd)

# uninstalling a package:
yum remove (package name)

# to view yum repo:
cd /etc/yum.repos.d
```

- `|` (pipeline) first command output is sent as the second command input
- `grep` is used to filter the required thing and `-i` when you are uncertain of capital or small letter

## File Comparison, Compression, and Management

```bash
cmp file1 file2  # to compare both files
diff file1 file2  # to diff both files

gzip -k filename  # to zip the file -k to keep the original file
gunzip filename  # to decompress the file

tar -czf filename.tar.gz dir dir file file  # this compresses a dir in linux
tar -xzf filename  # to decompress the directory

printenv  # to get the env variables

wget url-of-file  # to download a file
wget -o filename url  # to download file with particular name
curl santosh.website  # to curl the webpage
```

## Process Management

```bash
ps -ef  # to know the process running in the system
ps aux  # View all running processes -- give memory and cpu utilization

kill -STOP PID  # Stop a running process
kill -CONT PID  # Resume a stopped process
renice -n 10 -p PID  # Lower priority of a process
renice -n -5 -p PID  # Increase priority of a process (requires root)

command &  # Run a command in the background
nice -n 10 command  # Run a command with a specific priority
renice -n -5 -p PID  # Change priority of an existing process

ps -u username  # View processes for a specific user

kill -9 PID  # to kill the process forcefully
top  # is used to know what process is utilizing the cpu
iostat  # to know cpu utilization and all
free -m(mb) or -g(gb) or -h  # to know the available ram

du -k dir/  # to know the memory usage in folder
free -m
du -sh *  # to show the size of each file in pwd
du -sh /path  # Show disk usage of a specific directory
df -kh  # is used to check storage space
lsblk  # to know the storage devices attached

curl  # is used to check http connection curl https://google.com → fetches webpage HTML content
netstat -tulnp  # Show listening ports
nslookup example.com  # Checking DNS Resolution
ping google.com  # Test internet connection
traceroute google.com  # Trace the path to Google
```

## File Management

### File and Directory Management

```bash
ls  # Lists files and directories in the current location.
cd /path/to/directory  # Changes the working directory.
pwd  # Prints the current working directory.
mkdir new_folder  # Creates a new directory.
rmdir empty_folder  # Removes an empty directory.
rm file.txt  # Deletes a file.
rm -r folder  # Deletes a folder and its contents.
cp file1.txt file2.txt  # Copies a file.
cp -r dir1 dir2  # Copies a directory recursively.
mv old_name new_name  # Moves or renames a file or directory.
```

### File Viewing and Editing

```bash
cat file.txt  # Displays file content.
tac file.txt  # Displays file content in reverse order.
less file.txt  # Opens a file for viewing with scrolling support.
more file.txt  # Similar to less, but only moves forward.
head -n 10 file.txt  # Displays the first 10 lines of a file.
tail -n 10 file.txt  # Displays the last 10 lines of a file.
nano file.txt  # Opens a simple text editor.
vi file.txt  # Opens a powerful text editor.
echo 'Hello' > file.txt  # Writes text to a file, overwriting existing content.
echo 'Hello' >> file.txt  # Appends text to a file without overwriting.
```

## VI Editor Shortcuts

### Modes in VI Editor

- Normal Mode (default) – Used for navigation and command execution.
- Insert Mode – Used for text editing (press i to enter, Esc to exit).
- Command Mode – Used for saving, quitting, and searching (press : in Normal mode).

### Basic Navigation

- `h` – Move left
- `l` – Move right
- `j` – Move down
- `k` – Move up
- `0` – Move to the beginning of the line
- `^` – Move to the first non-blank character of the line
- `$` – Move to the end of the line
- `w` – Move to the next word
- `b` – Move to the previous word
- `gg` – Move to the start of the file
- `G` – Move to the end of the file
- `:n` – Move to line number n

### Insert Mode Shortcuts

- `i` – Insert before cursor
- `I` – Insert at the beginning of the line
- `a` – Append after cursor
- `A` – Append at the end of the line
- `o` – Open a new line below
- `O` – Open a new line above
- `Esc` – Exit insert mode

### Editing Text

- `x` – Delete a character
- `X` – Delete a character before cursor
- `dw` – Delete a word
- `dd` – Delete a line
- `d$` – Delete from cursor to end of line
- `d0` – Delete from cursor to beginning of line
- `D` – Delete from cursor to end of line
- `u` – Undo last action
- `Ctrl + r` – Redo an undone change
- `yy` – Copy (yank) a line
- `yw` – Copy (yank) a word
- `p` – Paste after the cursor
- `P` – Paste before the cursor

### Search and Replace

- `/pattern` – Search forward for a pattern
- `?pattern` – Search backward for a pattern
- `n` – Repeat last search forward
- `N` – Repeat last search backward
- `:%s/old/new/g` – Replace all occurrences of "old" with "new"
- `:s/old/new/g` – Replace all occurrences in the current line

### Working with Multiple Files

- `:e filename` – Open a new file
- `:w` – Save file
- `:wq` – Save and exit
- `:q!` – Quit without saving
- `:split filename` – Split screen horizontally and open another file
- `:vsplit filename` – Split screen vertically
- `Ctrl + w + w` – Switch between split screens

## Find Command

```bash
find  # used to find a file in the directory
find ./(directory) -name (nameoffile)
find ./(directory) -name *.txt
find ./(directory) -name (nameoffile) -exec rm -i {} \\;  # to find and remove a file
find ./(directory) -empty  # to find an empty file
find ./(directory) -perm 777  # find files with same permissions
find ./ -type f -name "*.txt" -exec grep '(text)' {} \\;  # to find text in some file
```

## Cron Jobs

```bash
cd /var/spool/cron  # to view crontab files
crontab -l  # to show all current jobs
crontab -e  # to edit or add new jobs
```

Cron format:
```
*min *hr *date  *month  *day of week 
****** sh /path to script
```

Example:
```bash
# makefile.sh
#!/bin/bash 
touch newfile

# crontab -e 
20 3 * * * /bin/bash /root/makefile.sh
# this will make a new file at 3.20
```

## Systemctl Command

The systemctl command in Linux is a command-line tool that manages the systemd system and service manager. It allows users to control and manage system services and units by using commands to start, stop, restart, and check their status

```bash
systemctl status (service name)
systemctl start (service name)
systemctl stop (service name)
systemctl restart (service name)
systemctl enable <service>  # Enable a service to start at boot.
systemctl disable <service> # Disable a service from starting at boot.
```

### To Configure Date and Time

Use:
```bash
yum install ntpd
```

## Sed Command

s - source, g - globally

```bash
sed -i 's/text before / replaces the before text /g' name.txt  # used to replace the text

sed -i "s/santoshpalla27\\/cicd:.*/santoshpalla27\\/cicd:${BUILD_NUMBER}/g" deploymentfiles/deployment.yml

sed -i "s#santoshpalla27/3-tier:backend.*#santoshpalla27/3-tier:backend-${BUILD_NUMBER}#" # CAN BE USED INSTEAD OF / SO WE CAN USE / WITHOUT CONFLICT

# when there is / it will get confused so before natural / add  this \\/ so it thinks it is not the end
```

## Storage Management

```bash
lsblk  # to view the storage and new block storage and its partitions

# create a mount dir where you want to mount the new storage

mkfs -t ext4 /dev/nameofdisk

mount /dev/nameofdisk  dirtomount
```