# How to install and Setup Centreon on Centos 7 on Distributed architecture with remote DBMS (vbox)

### Install and update Centos on VirtualBox

   - Download Centos ISO from server lists http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso
   - Check de integrity of the image https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7#head-9e717f2e95c7cb447ef66c0a0ae1315027b67684
   - Setup a classic install on VirtualBox, VMWare or another hypervisor.

### Update CentOS

Simply run yum update

```
# yum -y update
```
### Disable selinux

Edit selinux config and then restart de server

```
# cat /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 
```


### Disable firewall

```
# systemctl stop firewalld
# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)

Sep 19 17:11:30 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Sep 19 17:11:32 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
Sep 19 17:11:34 localhost.localdomain firewalld[644]: WARNING: ICMP type 'beyond-scope' is not supported by the kernel for ipv6.
Sep 19 17:11:34 localhost.localdomain firewalld[644]: WARNING: beyond-scope: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
Sep 19 17:11:34 localhost.localdomain firewalld[644]: WARNING: ICMP type 'failed-policy' is not supported by the kernel for ipv6.
Sep 19 17:11:34 localhost.localdomain firewalld[644]: WARNING: failed-policy: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
Sep 19 17:11:34 localhost.localdomain firewalld[644]: WARNING: ICMP type 'reject-route' is not supported by the kernel for ipv6.
Sep 19 17:11:34 localhost.localdomain firewalld[644]: WARNING: reject-route: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
Sep 19 17:56:52 poller1.centreon.gc systemd[1]: Stopping firewalld - dynamic firewall daemon...
Sep 19 17:56:52 poller1.centreon.gc systemd[1]: Stopped firewalld - dynamic firewall daemon.
```
## STOP VM & CLONE

### Clone the server four times

* First VM: A central Centreon server to display information
* Second: A RDBMS server to store collected data
* Thirth: Two remote servers to collect data (Pollers)
* Fourth: Check de arquitecture in https://documentation.centreon.com/docs/centreon/en/latest/installation/architecture/03c.html


## First VM

### Install MariaDB 5.5 

> (mysql57 not work for centreon)

```
# yum install mariadb-server.x86_64
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.brisanet.com.br
 * extras: centos.brisanet.com.br
 * updates: centos.brisanet.com.br
Resolving Dependencies
--> Running transaction check
---> Package mariadb-server.x86_64 1:5.5.56-2.el7 will be installed
................. OUTPUT CUT FOR SANITY.................
Installed:
  mariadb-server.x86_64 1:5.5.56-2.el7                                                                                                                                                                                                        

Dependency Installed:
  mariadb.x86_64 1:5.5.56-2.el7          perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7  perl-DBD-MySQL.x86_64 0:4.023-5.el7  perl-DBI.x86_64 0:1.627-4.el7  perl-Data-Dumper.x86_64 0:2.145-3.el7 
  perl-IO-Compress.noarch 0:2.061-2.el7  perl-Net-Daemon.noarch 0:0.48-5.el7           perl-PlRPC.noarch 0:0.2020-14.el7           

Complete!
# 
# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
# systemctl start mariadb
# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2017-09-22 11:41:08 -03; 3min 31s ago
  Process: 1162 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 1083 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 1161 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─1161 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─1323 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock

Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: MySQL manual for more instructions.
Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: Please report any problems at http://mariadb.org/jira
Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: The latest information about MariaDB is available at http://mariadb.org/.
Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: You can find additional information about the MySQL part at:
Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: http://dev.mysql.com
Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: Consider joining MariaDB's strong and vibrant community:
Sep 22 11:41:06 localhost.localdomain mariadb-prepare-db-dir[1083]: https://mariadb.org/get-involved/
Sep 22 11:41:06 localhost.localdomain mysqld_safe[1161]: 170922 11:41:06 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
Sep 22 11:41:06 localhost.localdomain mysqld_safe[1161]: 170922 11:41:06 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
Sep 22 11:41:08 localhost.localdomain systemd[1]: Started MariaDB database server.
# 
# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
# 
```

### Add users and grants privileges

```
MariaDB [(none)]> select user,host,password from mysql.user order by 1,2;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | 127.0.0.1 | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
| root | ::1       | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
| root | localhost | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
+------+-----------+-------------------------------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> drop user root@'::1';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> select user,host,password from mysql.user order by 1,2;
+------+-----------+-------------------------------------------+
| user | host      | password                                  |
+------+-----------+-------------------------------------------+
| root | 127.0.0.1 | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
| root | localhost | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
+------+-----------+-------------------------------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> 
MariaDB [(none)]> 
MariaDB [(none)]> show grants for root@localhost;
+----------------------------------------------------------------------------------------------------------------------------------------+
| Grants for root@localhost                                                                                                              |
+----------------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION                                                                           |
+----------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'root'@'[CENTRAL_CENTREON_IP]' IDENTIFIED BY PASSWORD '*5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> select user,host,password from mysql.user order by 1,2;
+------+------------+-------------------------------------------+
| user | host       | password                                  |
+------+------------+-------------------------------------------+
| root | 10.1.123.% | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
| root | 127.0.0.1  | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
| root | localhost  | *5011F6C1FXXXXXXXXXXXXXXXXXXXXXXB60FEA0C1 |
+------+------------+-------------------------------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> 
```

### my.cnf test

```
# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

innodb_file_per_table = 1


[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```

### Change limit files

```
# cat /etc/systemd/system/mariadb.service.d/limits.conf 
[Service]
LimitNOFILE=32000
#
# systemctl daemon-reload
# systemctl stop mariadb
# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─limits.conf
   Active: inactive (dead) since Fri 2017-09-22 13:49:21 -03; 15s ago
 Main PID: 11160 (code=exited, status=0/SUCCESS)

Sep 22 13:42:25 localhost.localdomain systemd[1]: Starting MariaDB database server...
Sep 22 13:42:26 localhost.localdomain mariadb-prepare-db-dir[11130]: Database MariaDB is probably initialized in /var/lib/mysql already, nothing is done.
Sep 22 13:42:26 localhost.localdomain mysqld_safe[11160]: 170922 13:42:26 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
Sep 22 13:42:26 localhost.localdomain mysqld_safe[11160]: 170922 13:42:26 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
Sep 22 13:42:28 localhost.localdomain systemd[1]: Started MariaDB database server.
Sep 22 13:49:18 localhost.localdomain systemd[1]: Stopping MariaDB database server...
Sep 22 13:49:21 localhost.localdomain systemd[1]: Stopped MariaDB database server.
# systemctl start mariadb
# systemctl status mariadb -l
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─limits.conf
   Active: active (running) since Fri 2017-09-22 13:49:43 -03; 4s ago
  Process: 11469 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 11438 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 11468 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─11468 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─11643 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock

Sep 22 13:49:41 localhost.localdomain systemd[1]: Starting MariaDB database server...
Sep 22 13:49:41 localhost.localdomain mariadb-prepare-db-dir[11438]: Database MariaDB is probably initialized in /var/lib/mysql already, nothing is done.
Sep 22 13:49:41 localhost.localdomain mysqld_safe[11468]: 170922 13:49:41 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
Sep 22 13:49:41 localhost.localdomain mysqld_safe[11468]: 170922 13:49:41 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
Sep 22 13:49:43 localhost.localdomain systemd[1]: Started MariaDB database server.
# 
```

### Install SNMP and setup

```
# yum install net-snmp.x86_64 net-snmp-agent-libs.x86_64 net-snmp-devel.x86_64 net-snmp-libs.x86_64 net-snmp-perl.x86_64
# cat /etc/snmp/snmp

#view    systemview    included   .1.3.6.1.2.1.1
#view    systemview    included   .1.3.6.1.2.1.25.1.1
view    systemview    included   .1

# systemctl status snmpd
● snmpd.service - Simple Network Management Protocol (SNMP) Daemon.
   Loaded: loaded (/usr/lib/systemd/system/snmpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
# systemctl enable snmpd
Created symlink from /etc/systemd/system/multi-user.target.wants/snmpd.service to /usr/lib/systemd/system/snmpd.service.
# systemctl start snmpd
# systemctl status snmpd
● snmpd.service - Simple Network Management Protocol (SNMP) Daemon.
   Loaded: loaded (/usr/lib/systemd/system/snmpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-09-25 14:25:17 -03; 4s ago
 Main PID: 1977 (snmpd)
   CGroup: /system.slice/snmpd.service
           └─1977 /usr/sbin/snmpd -LS0-6d -f

Sep 25 14:25:17 localhost.localdomain systemd[1]: Starting Simple Network Management Protocol (SNMP) Daemon....
Sep 25 14:25:17 localhost.localdomain snmpd[1977]: NET-SNMP version 5.7.2
Sep 25 14:25:17 localhost.localdomain systemd[1]: Started Simple Network Management Protocol (SNMP) Daemon..
# 
# ss -lnp4
Netid  State      Recv-Q Send-Q                                                                       Local Address:Port                                                                                      Peer Address:Port              
udp    UNCONN     0      0                                                                                        *:161                                                                                                  *:*                   users:(("snmpd",pid=1977,fd=6))
udp    UNCONN     0      0                                                                                        *:68                                                                                                   *:*                   users:(("dhclient",pid=664,fd=6))
udp    UNCONN     0      0                                                                                        *:52845                                                                                                *:*                   users:(("dhclient",pid=664,fd=20))
tcp    LISTEN     0      128                                                                                      *:22                                                                                                   *:*                   users:(("sshd",pid=853,fd=3))
tcp    LISTEN     0      100                                                                              127.0.0.1:25                                                                                                   *:*                   users:(("master",pid=939,fd=13))
tcp    LISTEN     0      128                                                                              127.0.0.1:199                                                                                                  *:*                   users:(("snmpd",pid=1977,fd=7))
tcp    LISTEN     0      50                                                                                       *:3306                                                                                                 *:*                   users:(("mysqld",pid=11643,fd=14))
# 
```

## Second VM

### CENTREON POLLER 1

```
# export http_proxy=http://user:pass@10.1.120.137:8080/
# curl -O http://yum.centreon.com/standard/3.4/el7/stable/noarch/RPMS/centreon-release-3.4-4.el7.centos.noarch.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4308  100  4308    0     0  36067      0 --:--:-- --:--:-- --:--:-- 36508
# ll
total 8
-rw-r--r--. 1 root root 4308 Sep 19 17:15 centreon-release-3.4-4.el7.centos.noarch.rpm
# yum install centreon-release-3.4-4.el7.centos.noarch.rpm 
Loaded plugins: fastestmirror
Examining centreon-release-3.4-4.el7.centos.noarch.rpm: centreon-release-3.4-4.el7.centos.noarch
Marking centreon-release-3.4-4.el7.centos.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package centreon-release.noarch 0:3.4-4.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================
 Package                                               Arch                                        Version                                               Repository                                                                      Size
==============================================================================================================================================================================================================================================
Installing:
 centreon-release                                      noarch                                      3.4-4.el7.centos                                      /centreon-release-3.4-4.el7.centos.noarch                                      3.1 k

Transaction Summary
==============================================================================================================================================================================================================================================
Install  1 Package

Total size: 3.1 k
Installed size: 3.1 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : centreon-release-3.4-4.el7.centos.noarch                                                                                                                                                                                   1/1 
  Verifying  : centreon-release-3.4-4.el7.centos.noarch                                                                                                                                                                                   1/1 

Installed:
  centreon-release.noarch 0:3.4-4.el7.centos                                                                                                                                                                                                  

Complete!

# yum install centreon-poller-centreon-engine.noarch
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.brisanet.com.br
 * extras: centos.brisanet.com.br
 * updates: centos.brisanet.com.br
Resolving Dependencies
--> Running transaction check
---> Package centreon-poller-centreon-engine.noarch 0:2.8.13-5.el7.centos will be installed
--> Processing Dependency: centreon-plugins = 2.8.13-5.el7.centos for package: centreon-poller-centreon-engine-2.8.13-5.el7.centos.noarch
--> Processing Dependency: centreon-common = 2.8.13-5.el7.centos for package: centreon-poller-centreon-engine-2.8.13-5.el7.centos.noarch
--> Processing Dependency: centreon-engine >= 1.6.0 for package: centreon-poller-centreon-engine-2.8.13-5.el7.centos.noarch
.....
systemtap-sdt-devel.x86_64 0:3.1-3.el7                                                                                tcp_wrappers-devel.x86_64 0:7.6-77.el7                                                                               
  time.x86_64 0:1.7-45.el7                                                                                              trousers.x86_64 0:0.3.14-2.el7                                                                                       
  urw-fonts.noarch 0:2.4-16.el7                                                                                         xdg-utils.noarch 0:1.1.0-0.17.20120809git.el7                                                                        
  xorg-x11-font-utils.x86_64 1:7.5-20.el7                                                                               xz-devel.x86_64 0:5.2.2-1.el7                                                                                        
  zlib-devel.x86_64 0:1.2.7-17.el7                                                                                     

Failed:
  centreon-common.noarch 0:2.8.13-5.el7.centos                                                                                                                                                                                                

Complete!
# hostname
localhost.localdomain

# 
# hostname poller1.centreon.gc
```

> If Centreon common install failed; then  retry

```
# yum install centreon-common.noarch
Loaded plugins: fastestmirror
base                                                                                                                                                                                                                   | 3.6 kB  00:00:00     
centreon-stable                                                                                                                                                                                                        | 2.9 kB  00:00:00     
centreon-stable-noarch                                                                                                                                                                                                 | 2.9 kB  00:00:00     
extras                                                                                                                                                                                                                 | 3.4 kB  00:00:00     
updates                                                                                                                                                                                                                | 3.4 kB  00:00:00     
(1/2): extras/7/x86_64/primary_db                                                                                                                                                                                      | 101 kB  00:00:00     
(2/2): centreon-stable-noarch/primary_db                                                                                                                                                                               | 240 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: centos.brisanet.com.br
 * extras: centos.brisanet.com.br
 * updates: centos.brisanet.com.br
Resolving Dependencies
--> Running transaction check
---> Package centreon-common.noarch 0:2.8.13-5.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================
 Package                                                  Arch                                            Version                                                       Repository                                                       Size
==============================================================================================================================================================================================================================================
Installing:
 centreon-common                                          noarch                                          2.8.13-5.el7.centos                                           centreon-stable-noarch                                          2.3 k

Transaction Summary
==============================================================================================================================================================================================================================================
Install  1 Package

Total download size: 2.3 k
Installed size: 0  
Is this ok [y/d/N]: y
Downloading packages:
centreon-common-2.8.13-5.el7.centos.noarch.rpm                                                                                                                                                                         | 2.3 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : centreon-common-2.8.13-5.el7.centos.noarch                                                                                                                                                                                 1/1 
  Verifying  : centreon-common-2.8.13-5.el7.centos.noarch                                                                                                                                                                                 1/1 

Installed:
  centreon-common.noarch 0:2.8.13-5.el7.centos                                                                                                                                                                                                

Complete!
```

## Thirth VM

### CENTRAL SERVER (WEB)

####  Install repo

```
# hostname centralweb.centreon.gc
# hostname
centralweb.centreon.gc
# curl -O http://yum.centreon.com/standard/3.4/el7/stable/noarch/RPMS/centreon-release-3.4-4.el7.centos.noarch.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4308  100  4308    0     0   5620      0 --:--:-- --:--:-- --:--:--  5616
```

### If you're behind a proxy

```
# curl --proxy http://10.1.120.137:8080 --proxy-user user:pass -O http://yum.centreon.com/standard/3.4/el7/stable/noarch/RPMS/centreon-release-3.4-4.el7.centos.noarch.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4308  100  4308    0     0   6599      0 --:--:-- --:--:-- --:--:--  6597
# ls -lhart
total 8.0K
drwxr-xr-x. 12 root root  131 Sep 19 14:53 ..
-rw-r--r--   1 root root 4.3K Sep 22 12:05 centreon-release-3.4-4.el7.centos.noarch.rpm
drwxr-xr-x.  2 root root   58 Sep 22 12:05 .
```

### Go on

```
# ll
total 8
-rw-r--r-- 1 root root 4308 Sep 19 15:57 centreon-release-3.4-4.el7.centos.noarch.rpm
# yum install --nogpgcheck centreon-release-3.4-4.el7.centos.noarch.rpm 
Loaded plugins: fastestmirror
Examining centreon-release-3.4-4.el7.centos.noarch.rpm: centreon-release-3.4-4.el7.centos.noarch
Marking centreon-release-3.4-4.el7.centos.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package centreon-release.noarch 0:3.4-4.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================================================================
 Package                                               Arch                                        Version                                               Repository                                                                      Size
==============================================================================================================================================================================================================================================
Installing:
 centreon-release                                      noarch                                      3.4-4.el7.centos                                      /centreon-release-3.4-4.el7.centos.noarch                                      3.1 k

Transaction Summary
==============================================================================================================================================================================================================================================
Install  1 Package

Total size: 3.1 k
Installed size: 3.1 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : centreon-release-3.4-4.el7.centos.noarch                                                                                                                                                                                   1/1 
  Verifying  : centreon-release-3.4-4.el7.centos.noarch                                                                                                                                                                                   1/1 

Installed:
  centreon-release.noarch 0:3.4-4.el7.centos                                                                                                                                                                                                  

Complete!
``` 

### Install central server

```
# yum install centreon-base-config-centreon-engine.noarch centreon
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.brisanet.com.br
 * extras: centos.brisanet.com.br
 * updates: centos.brisanet.com.br
Resolving Dependencies
--> Running transaction check
....... 

  t1lib.x86_64 0:5.1.2-14.el7                                                                                           tcp_wrappers-devel.x86_64 0:7.6-77.el7                                                                               
  time.x86_64 0:1.7-45.el7                                                                                              trousers.x86_64 0:0.3.14-2.el7                                                                                       
  urw-fonts.noarch 0:2.4-16.el7                                                                                         xdg-utils.noarch 0:1.1.0-0.17.20120809git.el7                                                                        
  xorg-x11-font-utils.x86_64 1:7.5-20.el7                                                                               xz-devel.x86_64 0:5.2.2-1.el7                                                                                        
  zlib-devel.x86_64 0:1.2.7-17.el7                                                                                     

Replaced:
  mariadb-libs.x86_64 1:5.5.56-2.el7                                                                                                                                                                                                          

Complete!
```

### Set timezone php

```
# grep timezone /etc/php.ini 
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = America/Argentina/Buenos_Aires
# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
# systemctl start httpd
# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2017-09-19 17:59:36 -03; 6s ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 20736 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)
  Process: 20695 ExecReload=/usr/sbin/httpd $OPTIONS -k graceful (code=exited, status=0/SUCCESS)
 Main PID: 20746 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd.service
           ├─20746 /usr/sbin/httpd -DFOREGROUND
           ├─20747 /usr/sbin/httpd -DFOREGROUND
           ├─20748 /usr/sbin/httpd -DFOREGROUND
           ├─20749 /usr/sbin/httpd -DFOREGROUND
           ├─20750 /usr/sbin/httpd -DFOREGROUND
           └─20751 /usr/sbin/httpd -DFOREGROUND

Sep 19 17:59:35 centralweb.centreon.gc systemd[1]: Starting The Apache HTTP Server...
Sep 19 17:59:36 centralweb.centreon.gc systemd[1]: Started The Apache HTTP Server.
```

## SnapShot BEFORE SETUP WEB CENTREON
### Setup Centreon

> Nota: Seguir los pasos de la documentacion oficial, salvando los siguiente issues encontrados en la doc
> https://documentation.centreon.com/docs/centreon/en/latest/installation/from_centreon.html#configuration 

En ese bloque de la doc seteamos las credenciales de acceso y strings de conexion a la base de datos.
Tener en cuenta que las password a usar deben cumplir con los requerimientos de seguridad de MySQL, sino el setup falla
Al encontrar el fallo y querer reintentar los pasos de instalacion a traves de la interfaz web, el proceso fallara 
porque ya estan generadas las bases de datos.
Por lo tanto, en caso de fallo se deberan borrar las bases de datos para empezar de nuevo el flujo.

# Credenciales usadas en el lab

##### Web user: admin/admin1234
##### OS user: centreon/centreon1234
##### DB user: root@'ipcentralcentreon'/IsolationLevel54$
##### DB user: centreon/C3ntre0n$

> https://documentation.centreon.com/docs/centreon/en/latest/installation/from_centreon.html#start-monitoring

Los pasos que se describen en ese bloque no coinciden con le menu de la version que instale, la ultima.
El unico menu de exportacion que encontre, no fue del engine, sino del poller que corre en el central server.
Para realizar el export: 

1- Ir a Configuration > Poller > Export Configuration
2- Click en Export
3- Uncheck Generate Configuration Files and Run monitoring engine debug (-v)
4- Check Move Export Files and Restart Monitoring Engine
5- Click on Export again
6- Log into the ‘root’ user on your server (central server in this case)
7- Start Centreon Broker
```
[root@localhost log]# systemctl status cbd 
● cbd.service - Centreon Broker watchdog
   Loaded: loaded (/etc/systemd/system/cbd.service; enabled; vendor preset: disabled)
   Active: inactive (dead)

Sep 22 14:12:59 centralweb.centreon.gc systemd[1]: Unit cbd.service cannot be reloaded because it is inactive.
[root@localhost log]# systemctl enable cbd 
[root@localhost log]# systemctl start cbd 
[root@localhost log]# systemctl status cbd -l
● cbd.service - Centreon Broker watchdog
   Loaded: loaded (/etc/systemd/system/cbd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2017-09-22 14:17:12 -03; 3s ago
 Main PID: 20878 (cbwd)
   CGroup: /system.slice/cbd.service
           ├─20878 /usr/sbin/cbwd /etc/centreon-broker/watchdog.xml
           ├─20880 /usr/sbin/cbd /etc/centreon-broker/central-broker.xml
           └─20881 /usr/sbin/cbd /etc/centreon-broker/central-rrd.xml

Sep 22 14:17:12 centralweb.centreon.gc systemd[1]: Started Centreon Broker watchdog.
Sep 22 14:17:12 centralweb.centreon.gc systemd[1]: Starting Centreon Broker watchdog...
[root@localhost log]# 
```

8- Start Centengine

```
[root@localhost log]# systemctl status centengine
● centengine.service - Centreon Engine
   Loaded: loaded (/etc/systemd/system/centengine.service; enabled; vendor preset: disabled)
   Active: inactive (dead)

Sep 22 14:12:59 centralweb.centreon.gc systemd[1]: Unit centengine.service cannot be reloaded because it is inactive.
[root@localhost log]# systemctl enable centengine
[root@localhost log]# 
[root@localhost log]# systemctl start centengine
[root@localhost log]# systemctl status centengine -l
● centengine.service - Centreon Engine
   Loaded: loaded (/etc/systemd/system/centengine.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2017-09-22 14:18:29 -03; 3s ago
 Main PID: 20963 (centengine)
   CGroup: /system.slice/centengine.service
           └─20963 /usr/sbin/centengine /etc/centreon-engine/centengine.cfg

Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/timeperiods.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/escalations.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/dependencies.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/connectors.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/meta_commands.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/meta_timeperiod.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/meta_host.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Processing object config file '/etc/centreon-engine/meta_services.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Reading resource file '/etc/centreon-engine/resource.cfg'
Sep 22 14:18:29 centralweb.centreon.gc centengine[20963]: [1506100709] [20963] Parsing of retention file failed: Can't open file '/var/log/centreon-engine/retention.dat'
[root@localhost log]# 
```

9- Start CentCore

```
[root@localhost log]# systemctl status centcore
● centcore.service - SYSV: centcore is a Centreon program that manage pollers
   Loaded: loaded (/etc/rc.d/init.d/centcore; bad; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:systemd-sysv-generator(8)
[root@localhost log]# systemctl enable centcore
centcore.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig centcore on
[root@localhost log]# 
[root@localhost log]# systemctl start centcore
[root@localhost log]# systemctl status centcore -l
● centcore.service - SYSV: centcore is a Centreon program that manage pollers
   Loaded: loaded (/etc/rc.d/init.d/centcore; bad; vendor preset: disabled)
   Active: active (running) since Fri 2017-09-22 14:19:34 -03; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 21009 ExecStart=/etc/rc.d/init.d/centcore start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/centcore.service
           └─21022 /usr/bin/perl /usr/share/centreon/bin/centcore --logfile=/var/log/centreon/centcore.log --severity=error --config=/etc/centreon/conf.pm

Sep 22 14:19:33 centralweb.centreon.gc systemd[1]: Starting SYSV: centcore is a Centreon program that manage pollers...
Sep 22 14:19:34 centralweb.centreon.gc runuser[21020]: pam_unix(runuser:session): session opened for user centreon by (uid=0)
Sep 22 14:19:34 centralweb.centreon.gc centcore[21009]: [36B blob data]
Sep 22 14:19:34 centralweb.centreon.gc systemd[1]: Started SYSV: centcore is a Centreon program that manage pollers.
[root@localhost log]# 
```

# Install Centreon License Manager

# Then install Centreon Plugin Pack Manager itself.

# Go to "Administration" > "Parameters" > "Centreon UI"  and complete Proxy Options.
# Then you can go to Configuration > plugin Pack and view plugin lists




-- https://documentation.centreon.com/docs/centreon/en/latest/installation/from_centreon.html#easy-monitoring-configuration

Luego de hacer los pasos de instalacion de los plugins, los cuales me funciono bien (no como en la iso propia de centreon), 
hay que hacer el setup que no se encuentra en la ruta que indica el documento.
Hay que ir a Configuration > Plugin Pack > Setup (este ultimo solo si es necesario, ie no cargue los plugins)
WALA! los plugins



