# Linux shells
The first thing we need to know about is Linux Shells. The shell sits between the application and the operating system and acts as a bridge between the user and the system, while receiving, interpreting, and executing commands from the user. This diagram below shows the relation and the order among the hardware, OS, Shell, Application and User
![image](https://hackmd.io/_uploads/r1l6-Ns5R.png)
There are two types of shells: graphical user interface (GUI) and command-line interfaces (CLI) shells, of which Windows Explorer, Macintosh Finder, and X Window (primarily Unix) are the most common. For CLIs, bash shells and command prompts (cmd.exe, cmd) are commonly used.
Regarding Unix shells, we have 
* Bourne shell (sh): executable file that replaced the first shell, Thompson Shell, developed by Stephen Bourne at AT&T Bell Labs. It can be devided to 2 types: Almquist shell (ash) and Bash shell (extends various features from the main shell (sh))
* C shell (csh): base of Tenex C shell (tcsh)
* KornShell (ksh), Scheme shell (Scsh), Z shell (zsh)

![image](https://hackmd.io/_uploads/B1YmSSoqR.png)
Above is a table comparing these shells.
Before going into other parts, I'd like to introduce you:
* `~` is a shorthand for `/home` folder. For example, if the user is `thd` then `~` is equillvalent to `/home/thd`
* `etc` is a folder which contain all your system configuration files in it.
* **Interactive shell**: A shell session where the user interacts directly with the shell, typically through a terminal. It expects user input and provides real-time feedback. Terminal, CMD or SSH into a remote server can be listed as some notable examples.
* **Non-interactive shell**: A shell session that is executed automatically, without expecting input from the user. It runs predefined scripts or commands. For example, it could be: Running a shell script, executing a command via SSH like `ssh user@server "ls -l"` or a shell invoked by a cron job.
# Login shells
Regarding this category, we'll discuss about `~/.bash_profile` and `/etc/profile`.
At your working directory, use `ls -a` or `la` command and receive this: 
![image](https://hackmd.io/_uploads/BkAZxLsqC.png)
Hmmm seem like it doesn't appear in my WSL2, but it has `.profile` file, which I would refer later because it has some the same features as `~/.bash_profile`. But never mind, the `~/.bash_profile` file contains command to set up environment variables when a user logs in. Consequently, all the future shells will inherit these variables. 
When you change your working directory by `cd /etc` command and use `la` again, you will see there is a `profile` file ($4^{th}$ column in the below screenshot). `/etc/profile` serves a similar purpose but on a system-wide level, establishing initial environmental variables for all shell users. `/etc/profile` also executes scripts in `/etc/profile.d/*.sh`. For custom system-wide environmental settings, it's advisable to place your scripts in this directory.
![image](https://hackmd.io/_uploads/HJUBGai9R.png)
In an interactive login shell, Bash first looks for the `/etc/profile` file. If found, Bash reads and executes it in the current shell. 
Simirlarly, Bash then checks if `.bash_profile` exists in the home directory. If it does, then Bash executes `.bash_profile` in the current shell. If Bash doesn’t find `.bash_profile`, then it looks for `.bash_login` and `.profile`, in that order, and executes the first readable file only. `.profile` can hold the same configurations as `.bash_profile` or `.bash_login`. It controls prompt appearance, keyboard sound, shells to open, and individual profile settings that override the variables set in the `/etc/profile` file.
To conclude, both two files are executed for login shells. `~/.bash_profile` relates to current users whereas `/etc/profile` is used for all users.
# Non-login Interactive Shells
Non-login interactive shell is a type of shell session where the user interacts directly with the shell (it is interactive), but it does not involve logging into the system (it is non-login). `~/.bashrc` and `/etc/bash.bashrc` are two notable files executed for this shell type. 
`~/.bashrc` contains commands that are specific to the Bash shells. Every interactive non-login shell reads `.bashrc` first. Normally `.bashrc` is the best place to add aliases ~~(shortcut name for command and filename)~~ and Bash related functions. The Bash shell looks for the `.bashrc` file in the home directory and executes it in the current shell using [source](https://www.baeldung.com/linux/source-command). On the other hand, `/etc/bash.bashrc` acts as system-wide equivalents of `.bashrc`, but it is executed for all users.
**Comparing these files of the first two mentioned shells**, we can see:
* Environment variables are put into `.bash_profile`. Since the interactive login shell is the first shell, all the default settings required for the environment setup are put in `.bash_profile`. Consequently,  they are set only once but inherited in all child shells.
* Aliases and functions are put into .bashrc to ensure that these are loaded every time a shell is launched from within the existing environment.
* To avoid login and non-login interactive shell setup difference, `.bash_profile` calls `.bashrc`. As a result, we’ll see the below code piece is inserted in `.bash_profile`, so that on every interactive login shell, `.bashrc` is also executed in the same shell.
# Systemd and unit files
## 1. Systemd and Unit files
Linux distributions are adopting the `systemd` init system. This powerful suite of software can manage many aspects of your server, from services to mounted devices and system states. In `systemd`, a `unit` refers to any resource that the system knows how to operate on and manage. This is the primary object that the `systemd` tools know how to deal with. These resources are defined using configuration files called `unit` files. They are located in directories like `/etc/systemd/system/`, `/lib/systemd/system/`, or `/usr/lib/systemd/system/`. These files define services, timers, devices, mounts, and other entities systemd manages.
* **Service unit** (`.service`): Defines services, like daemons that start and run in the background ( `nginx.service`, `ssh.service`,...).
* **Target unit** (`.target`): Group services and other units into logical groups (`multi-user.target`, `graphical.target`,...).
* **Mount Units** (`.mount`): Handle filesystem mount points (`home.mount`,...).
* **Timer Units** (`.timer`): Defines timers to trigger service units at specific times (`backup.timer`,...).
* **Socket Units** (`.socket`): Manages inter-process communication sockets (`sshd.socket`,...).
## 2. `/etc/systemd/system/` and  `/lib/systemd/system/`
![image](https://hackmd.io/_uploads/SJCGgUh5C.png)
* `/etc/systemd/system` is the prime directory for your custom unit files. If you want to add a new service or modify an existing one without affecting the package-provided defaults, this is where you’d place your files.
-- It is the location where you place your custom service files or override existing ones.
-- If you need to modify or override a service, you would create a file here with the same name as the original file in `/lib/systemd/system/`. Files here take precedence over those in `/lib/systemd/system/`
-- Any manual changes or overrides to unit files should be done here, as this directory is not affected by system updates.
-- Files in `/etc/systemd/system/` are user-managed and persist across updates, ensuring that your custom configurations are maintained.
* `/lib/systemd/system/` or `/usr/lib/systemd/system/` is for unit files installed by packaged software. You should avoid making changes here because package updates can overwrite your modifications.
-- This directory contains unit files provided by the system’s package manager (e.g., those installed by default or from packages).
-- It holds the default configurations for services and targets as provided by your Linux distribution. If you need to reference or view the default configuration of a service, you check this directory.
-- Modifications should not be made directly here because they will be **overwritten** during updates.
-- When systemd starts a service, it first checks `/etc/systemd/system/` for the unit file. If not found, it checks `/lib/systemd/system/`.

For instance, you have service named `myservice.service`. We could figure that the original unit file is located in `/lib/systemd/system/myservice.service`. If you want to customize how it starts, so you copy it to `/etc/systemd/system/myservice.service` and modify it there. Your customized version will override the default one.
# Crontab
Crontab (CRON TABLE) is a utility in Unix-like operating systems used to schedule automated tasks at predefined times. There are different types of crontab, each serving a specific role in managing scheduled tasks. There are three level of crontab:
* User crontab (`crontab -e`): the most common crontab, used to manage automated tasks for individual users' personal jobs. Each user on the system can have their own crontab file and tasks configured in a user crontab run with the privileges of that user.
* System-wide crontab (`/etc/crontab`): the global crontab file managed by the system administrator, including an additional field to specify which user will execute the command. It is typically used for system maintenance tasks.
* Cron directories (`/etc/cron.d/`, `/etc/cron.daily`/, `/etc/cron.hourly/`, `/etc/cron.weekly/`, `/etc/cron.monthly/`): These directories contain script files that are scheduled to run at fixed intervals (hourly, daily, weekly, monthly). Administrators can place scripts in these directories to have the system automatically execute them at the appropriate times.
```bash
.---------------- minute (0 - 59)
|  .------------- hour (0 - 23)
|  |  .---------- day of month (1 - 31)
|  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
|  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
|  |  |  |  |
*  *  *  *  * user-name command to be executed
```
Above is the structure of crontab command. Each `*` character is assigned with a value having its validation, for example, `minute` is ranged in `0-59`. Besides, there is some special characters we should know:
* `*`: Represents all possible values (e.g., `*` in the "hour" field means every hour)
* `,`: Separates values (e.g., `1,15` means the 1st and 15th days)
* `-`: Specifies a range of values (e.g., `1-5` means Monday through Friday)
* `/`: Specifies intervals (e.g., `*/2` in the "minute" field means every 2 minutes)

To be more specific, let's continue with some example:
* Run command every day at 17:30
```bash
30 17 * * * /path/to/your/command
```
* Run command every 10 minutes
```bash
*/10 * * * * /path/to/your/command
```
* Run a command on the 1st and 15th of every month at 12:00 PM
```bash
0 12 1,15 * * /path/to/your/command
```
* Run a command every weekday during business hours (9:00 AM to 5:00 PM)
```bash
0 9-17 * * 1-5 /path/to/your/command
```

# rc.local
The `rc.local` file is a traditional script found in Unix-like operating systems, particularly in systems using the SysV init system. It was used to execute commands at the end of the system’s boot process. This file is commonly located at `/etc/rc.local`. On some distributions, especially newer ones, this file may be missing or disabled by default due to the adoption of `systemd`. If your system doesn't have this file, try this [methods](https://www.cyberciti.biz/faq/how-to-enable-rc-local-shell-script-on-systemd-while-booting-linux-system/).
The `rc.local` script is executed with root privileges at the end of the boot process. It’s typically used for running custom commands or scripts that you want to run automatically when the system starts.
`rc.local` files always start with this below line:
```bash
#!/bin/sh -e
```
It is followed by the commands or scripts you want to execute, and finally ends with `exit 0` to ensure the script completes successfully. Here's a sample bash code:
```bash!
#!/bin/sh -e
# Turn on an LED indicator to signal the boot is complete
echo 1 > /sys/class/leds/led0/brightness

# Start a custom service
/usr/local/bin/start_custom_service.sh &

# Mount a network folder
mount -t nfs 192.168.1.100:/shared /mnt/nfs

exit 0
```
With root privilege, you can execute any scripts as you want to do to manage the startup.
# Previous autostart service
Before `systemd` became the standard in most modern Linux OS, service management and system startup were primarily handled through scripts located in the `/etc/init.d/` directory. This system used the `SysV init` (short for System V init), which relied on scripts to control services such as starting, stopping, and checking their status. The `/etc/init.d/` directory contains startup scripts (init scripts) corresponding to each service. Each script can perform standard actions like start, stop, restart, and status.
![image](https://hackmd.io/_uploads/r13QT7xjC.png)
The SysV init system is based on runlevels to determine the system’s current state:
* `0`: Shutdown the system
* `1`: Single-user mode
* `2`: Multi-user mode without networking
* `3`: Multi-user mode with networking
* `5`: Graphical mode
* `6`: Reboot the system

The scripts in `/etc/init.d/` are linked to these runlevels via directories like `/etc/rc[0-6].d/`, where symbolic links point to the corresponding scripts in `/etc/init.d/`.
Each script in `/etc/init.d/` supports standard parameters:
* `start`: Start the service
* `stop`: Stop the service
* `restart`: Restart the service
* `status`: Check the status of the service

```bash
sudo /etc/init.d/dbus start
sudo /etc/init.d/dbus stop
```
It also support boot process. During system startup, based on the current runlevel, scripts in the corresponding directory (`/etc/rc[0-6].d/`) are executed in order. Links starting with `S` are called to start services, while links starting with `K` are used to stop services. For instance, in `/etc/rc3.d/`, there might be links like `S20dbus`, where `20` determines the startup order, and `dbus` points to the script `/etc/init.d/dbus`.
The scripts in `/etc/init.d/` can be customized to add commands or change how a service starts. However, manual edits can be risky if not done carefully due to these following limitations:
* **Fixed Startup Order**: Services start in a rigid order determined by the number in the link name, without the ability to start in parallel
* **Difficult to Manage**: As the number of services grows, managing and coordinating the startup order becomes more complex
* **Limited Features**: It lacks advanced features like state tracking or automatically restarting services when they fail

After that, `systemd` was introduced to tackle with the limitations of SysV init by providing better service management, parallel startup, tracking, and automatic service restarts. In  `systemd`, `.service` files replace the scripts in `/etc/init.d/`, offering a more modern and flexible approach. That's why `/etc/init.d` was replaced.
# UDEV rules
`udev` is a device manager for the Linux kernel that manages device nodes in the `/dev` directory and handles dynamic device management, such as when devices are added or removed from the system. In short, it helps your computer find your robot easily. `udev` rules are text files that instruct udev how to manage devices and used to identify and assign specific actions to devices based on various attributes like device name, type, vendor, serial number, etc.
The rules files are taken from `/etc/udev/rules.d` for custom rules created by the user, `/lib/udev/rules.d` for system-installed rules and `/run/udev/rules.d` for dynamically generated rules. Especially, rules in `/etc/udev/rules.d` are given the highest priority that they can override rules in the other directories. Meanwhile, rules in `/lib/udev/rules.d/` can be overwritten when packages are updated. This is a key reason why this is the best place to put your custom rules without being affected by system updates. Here's a typical udev rule format:
```bash
<match criteria>, <match criteria>, ... <key>=<value>, <key>=<value>, ...
```
| Match<br>Criteria | Description                                                                  |
|:----------------- | ---------------------------------------------------------------------------- |
| `KERNEL`          | Matches the kernel device name (e.g., `sd*` for storage devices)             |
| `SUBSYSTEM`       | Matches the subsystem of the device (e.g., `usb`, `block`, `net`)            |
| `ATTR{}`          | Matches device attributes, such as `idVendor`, `idProduct`, or `serial`      |
| `ACTION`          | Matches the action taken on the device, such as `add`, `remove`, or `change` |
|`ENV{}`| Matches environment variables|
|`DEVPATH`| Matches the device path in the /dev directory|

Here's some common keys:

| Keys               | Description                                           |
| ------------------ | ----------------------------------------------------- |
| `NAME`               | Specifies the name of the device node                 |
| `SYMLINK`            | Creates a symbolic link to the device node            |
| `OWNER`, `GROUP`, `MODE` | Sets the ownership and permissions of the device node |
| `RUN`|Executes a specified command or script|
| `IMPORT`| Imports properties from other rules or commands|

Let's breakdown this instance:
```bash
SUBSYSTEM=="usb", ATTR{idVendor}=="1234", ATTR{idProduct}=="5678", MODE="0666", SYMLINK+="my_usb_device"
```
In this instance, the rules apply to USB devices with `idVendor = 1234`, `idProduct = 5678`. Otherwise, we set the permissions of the device node to `0666`, making it readable and writable by all users, and create a symlink `/dev/my_usb_device` pointing to the device.
After knowing the syntax, let's check what these rules allow you to do:
* Assign a specific name to a device: 
```bash!
# Assign a name to a specific USB device
SUBSYSTEM=="block", ATTRS{serial}=="1234567890", SYMLINK+="custom_usb_drive"
```
* Change the permissions of a device
```bash
# Set permissions for a device
KERNEL=="sd*", ATTRS{serial}=="987654321", MODE="0666", GROUP="users"
```
* Run specific scripts or commands when a device is plugged or removed
```bash 
# Run a script when a USB device is plugged in
SUBSYSTEM=="usb", ACTION=="add", ATTR{idVendor}=="abcd", RUN+="/path/to/script.sh"
```
* Create symbolic links to device nodes
```bash
# Create a symbolic link for a USB device
SUBSYSTEM=="usb", ATTR{idVendor}=="abcd", ATTR{idProduct}=="ef01", SYMLINK+="my_device"
```
# Startup and Desktop Environment
## 1. `~/config/autostart` and `/etc/xdg/autostart/`
The `~/.config/autostart/` directory is used in many Linux desktop environments to automatically start applications when a user logs into their desktop session. This directory is part of the [freedesktop.org](https://www.freedesktop.org/wiki/) XDG Autostart Specification, which defines how applications should be automatically started in environments like GNOME, KDE Plasma, XFCE, and others. It contains `.desktop` files that define the applications to be started automatically when the user logs in. Each `.desktop` file specifies the command to run, the name of the application, and other relevant properties. They follow the [freedesktop.org](https://www.freedesktop.org/wiki/) standard and have a format like this:
```ini
[Desktop Entry] #standard header for a desktop entry
Type=Application 
Name=MyApp
Exec=/path/to/your/app #The command that is executed to start the application
X-GNOME-Autostart-enabled=true 
#Ensures that the application is enabled for autostart in GNOME environment
```
It also assist you on adding applications (create a `.desktop` file manually and place it in `~/.config/autostart/`), disabling autostart applications and editing autostart applications.
![image](https://hackmd.io/_uploads/H1kc8QejR.png)
Meanwhile, `/etc/xdg/autostart/`  is a system-wide directory that conta`ins .desktop files similar to those in `~/.config/autostart/, but the files here affect all users on the system. Both these two above directories customize applications that automatically start when the user logs in (the `/etc/xdg/autostart` is for all users). This approach is widely supported and easy to manage. 
## 2. `/etc/systemd/system/`
The definition I've written above, you can read again [here](https://hackmd.io/@d4tb30/rkawpyi9R#2-etcsystemdsystem-and-libsystemdsystem).
You can create a unit file for services or applications that you want to start automatically when the system boots. These unit files can be configured to start services or applications at boot time or when a user logs in. Here's an example:
```ini
[Unit]
Description=My Application

[Service]
ExecStart=/path/to/myapp
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
Now let's enable the service above:
```bash 
sudo systemctl enable myapp.service
sudo systemctl start myapp.service
```
In short, we can using this folder to manage services and applications that start automatically using `systemd`. 
# Scripting and Configuration
Both the directories `/etc/profile.d/` and `/etc/systemd/system/[unit_name].service.d/` play important roles in configuring the system environment and managing startup services. Each directory has a distinct role in influencing the system's startup and operation.
## 1. `/etc/profile.d/`
`/etc/profile.d/` directory is commonly found in Unix-like operating systems, and it plays a significant role in configuring the environment settings for all users when they log into a system. 
This directory contains shell scripts (`.sh` file) that are executed when a user logs into the system. These scripts are sourced by the global `/etc/profile` script and apply system-wide environment settings for all users. When a user logs in (either through a terminal, SSH, or a graphical login), the system sources the `/etc/profile` file, which in turn sources all the scripts located in directory `/etc/profile.d/`, which contains file executed for all users, allowing you to apply default system-wide environment configurations.
Here is the `/etc/profile.d/java.sh` file. When this file is executed, the `JAVA_HOME` variable and the `PATH` are configured for all users.
```bash
#!/bin/sh
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
export PATH=$PATH:$JAVA_HOME/bi
```
To sum up, directory `/etc/profile.d/` affects user environment configurations when they log into the system, where you can add shared environment settings for the entire system.
## 2. `/etc/systemd/system/[unit_name].service.d/`
The directory `/etc/systemd/system/[unit_name].service.d/` is used to customize and override settings for specific systemd service units without directly modifying the original unit files. It allows you to add or modify service configurations without directly editing the original unit file. This approach is safer and more maintainable, as your changes are kept separate from the original unit files provided by system packages.
In this directory, you can create configuration files with a `.conf` extension. These files contain additional or modified configuration directives for the original service unit. When systemd loads a service, it checks if any additional configuration files are present in `/etc/systemd/system/[unit_name].service.d/`. If it finds any, it reads and applies those settings in addition to or overriding the settings in the main service unit file.
Here is the `/etc/systemd/system/[unit_name].service.d/override.conf` file:
```ini
[Service]
Environment="ENV_VAR_NAME=custom_value"
ExecStart=
ExecStart=/path/to/new/executable --new-option
```
In the above example, the environment variable `ENV_VAR_NAME` is set to a new value. The original `ExecStart` command of the service is cleared (by setting `ExecStart= empty`) and replaced with a new command.
In conclusion, `/etc/systemd/system/[unit_name].service.d/` affects the configuration of system startup services. It’s where you can safely and flexibly adjust or extend configurations for `systemd` services.
