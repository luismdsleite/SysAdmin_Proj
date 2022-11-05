- [Systemd](#systemd)
- [LDAP](#ldap)
  - [Attributes](#attributes)
  - [Object Classes](#object-classes)
  - [Schemas](#schemas)

# Systemd

**A System and service manager, an init system used to bootstrap user space and manage user processes.**

Systemd is a system that is designed specifically for the Linux kernel. It replaces the sysvinit process to become the first process with PID = 1, which gets executed in user space during the Linux start-up process.  

Managing services with systemd:\
Below is the list of some useful systemd utilities along with a brief description of what they do:

**systemctl**: It Controls the systemd system and services.\
**journalctl**: Used To manage journal, systemd’s own logging system.\
**hostnamectl**: Can Control hostname.\
**localectl**: Helps Configure system local and keyboard layout.\
**timedatectl**: Used to Set time and date.\
**systemd-cgls**: It Shows cgroup contents.\
**systemadm**: It is a Front-end for systemctl command.\

# LDAP
[Simple Explanation of what is OpenLDAP, LDAP, slapd](https://www.youtube.com/watch?v=l8BwMlPRMF8)

LDAP = X.500 + TCP/IP

## Attributes 
- CN = Common Name
- OU = Organizational Unit
- DC = Domain Component
- DN = Distinguished Names 
- C = country

<figure>
<img src="Images/LDAP_Directory_Strucuture.gif" alt="Trulli" style="width:100%">
<figcaption align = "center"><b>Example of an LDAP directory structure with distinguished names and relative distinguished names.</b></figcaption>
</figure>

## Object Classes
Set of attributes
- Person = CN + SN
- Country = C
Note: Optional attributes are also possible in object classes

## Schemas  
Plugins (Attributes + Object Classes)
- PosixAccount composed by: VID, Shell, home, GID
Note: PosixAccount = Object class Used in /etc/passwd composed by attributes

The slapd configuration is stored as a special LDAP directory with a predefined schema and DIT. There are specific objectClasses used to carry global configuration options, schema definitions, backend and database definitions, and assorted other items

<figure>
<img src="Images/config_dit.png" alt="Trulli" style="width:100%">
<figcaption align = "center"><b>Sample configuration tree.</b></figcaption>
</figure>

<figure>
<img src="Images/slapd_configs.png" alt="Trulli" style="width:100%">
<figcaption align = "center"><b>Slapd config files from the OpenLDAP Server</b></figcaption>
</figure>

`olcDatabase={1}monitor.ldif`: olc = OpenLDAPConfig, the matching database, and who has access to the DB.

`olcDatabase={1}hdb.ldif`: hdb = The database where all the data is stored.  
