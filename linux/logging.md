# Logging in Linux (syslog)

- Main config file **/etc/syslog.conf** (**/etc/rsyslog.conf**)
- performing by daemon **syslogd** (**rsyslogd** - in new distributions)

- **/dev/log** - is used for log info from local machine 
- **UDP 514** is used for receiving log info from remove machine 

## Configuration file syslog.conf

- syslog.conf is set of rules
- each rule is row consists of **selector** and **action**
- *selector* is an entry like **source.priority* 

### Source (Category) types

- 0 - **kern** - kernel messages
- 1 - **user** - user program messages
- 2 - **mail** - mail system messages
- 3 - **daemon** - messages from system daemons which hasn't categories
- 4 - **auth** - user authorization, like *login*, *su*
- 5 - **syslog** - system may log itself
- 6 - **lpr** - messages from printing service
- 7 - **news** - messages from news server (not used)
- 8 - **uucp** - unix-to-unix copy protocol (not used)
- 9 - **cron** - messages from system planner
- 10 - **authpriv** - same as **auth**, may consist open passwords
- 11 - **ftp** - ftp logging
- 12 - **ntp** - messages from time server
- 13 - **log audit**
- 14 - **log alert** 
- 15 - **clock daemon** - time daemon messages
- 16 - 23 - local0 - local7 - reserved for sysadmin. local7 is usually used for system boot messages
- **mark** - messages which are generated by syslogd

### Message priorities

- 0 - **emerg** - (old name PANIC) - System doesn't work
- 1 - **alert** - Alert! Action need immidiately
- 2 - **crit** - Critical state
- 3 - **err** - (old name ERROR) Error message
- 4 - **warning** - (old anme WARN) - Warning
- 5 - **notice** - Important but normal message
- 6 - **info** - Information message
- 7 - **debug** - Debugging message

### Message destinations

- **File** - path start with "/"
- **Naming channels** - use "|" for *fifo* (first-in-first-out) or **named pipe**
- **Terminal or console** - /dev/console
- **Remote machine** 
- **User list** - f.e. root

```
kern.*              /dev/console
*.info;cron.none    /var/log/syslog
authpriv.*          /var/log/secure
mail.*              /var/log/maillog
cron.*              /var/log/cron

# Send emergency messages for all system ysers
*.emerg             *

# Save system boot messages to boot.log
local7.*            /var/log/boot.log
``` 

```
# Send all messages from kernel to /var/log/kernel
# Send all messagess with priority *crit* to sysloger machine and to console
# Send all messages with *info, notice and warning* to /var/log/kernel-info

kern.*                  /var/log/kernel
kern.crit               @sysloger
kern.crit               /dev/console
kern.info;kern.!err     /var/log/kernel-info

# Send all messages from mail system, except info level to /var/log/mail
mail.*;mail.!=info      /var/log/mail
```

### syslog daemon start

- **syslog daemon** starts by script **/etc/rc.d/init/d/syslog**
- Startup config file is located at **/sysconfig/syslog (/etc/default/syslog)**

Startup parameters:
- **-a /folder/socket** - specifying additional listening socket
- **-d** - debugging mode. Daemon show all messages in terminal
- **-f name-of-config-file** - define alternative config file. Instead of default */etc/syslog.conf*
- **-l hosts-list** - host exceptions for which FQDN doesn't used
- **-m minutes** - change time interval for writing *mark* time stamps
- **-p socket** - define alternative UNIX socket (insted of default */dev/log*)
- **-r** - permission of receiving messages from remote hosts
- **-x** - prohibition of defining hostname by it's address for preventing freezing on working with one host with DNS
- **-v** - show version and finish work

After **syslogd** start status file */var/lock/subsys/syslog* is created and file with process ID */var/run/syslogd.pid*

Packet **sysklogd** consists of two daemons
- *syslogd*
- *klogd* - used for logging system core events

### Automatic log rotation and archivation

Main config file **/etc/logrotate.conf**

logrotate.conf example
```
# see "man logrotate" for details
# rotate log files weekly
weekly

# use the syslog group by default, since this is the owning group
# of /var/log/syslog.
su root syslog

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp, or btmp -- we'll rotate them here
/var/log/wtmp {
    missingok
    monthly
    create 0664 root utmp
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0660 root utmp
    rotate 1
}

# system-specific logs may be configured here
```
