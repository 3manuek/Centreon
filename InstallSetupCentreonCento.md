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

