# System Administration Project
This project consists of an directory information system based on LDAP for authentication and for exporting user directories using NFS.

# Goals 
1. Have a RAID/LVM system on the [OMV](#server---openmediavault) machine.
2. Export directories on the [OMV](#server---openmediavault) via NFS or SAMBA.
3. Have openLDAP on the [server](#server---openldap) to authenticate users on [Desktop](#fedora---workstation) and [Win10](#windows-10).
4. Use LDAP to know which directories are needed to mount for the user authenticated.
5. Mount on [Desktop](#fedora---workstation) the directories of the authenticated user via NFS or SAMBA. 

![System Architecture](./Images/network.-trab.png)

# Table of Contents
- [System Administration Project](#system-administration-project)
- [Goals](#goals)
- [Table of Contents](#table-of-contents)
- [Virtual machines specs](#virtual-machines-specs)
  - [Fedora - Workstation](#fedora---workstation)
  - [Server - OpenMediaVault](#server---openmediavault)
  - [Server - OpenLDAP](#server---openldap)
  - [Windows 10](#windows-10)
- [Fedora - Workstation](#fedora---workstation-1)
  - [Remote Desktop (GUI)](#remote-desktop-gui)
- [Server - OpenMediaVault](#server---openmediavault-1)
  - [Installing OpenMediaVault on debian 11](#installing-openmediavault-on-debian-11)
- [Server - OpenLDAP](#server---openldap-1)
  - [Install OpenLDAP on CentOS 7](#install-openldap-on-centos-7)

# Virtual machines specs
## Fedora - Workstation

    Type: e2-micro
    Location: europe-west1-d
    OS: Fedora-36
    External IP: 34.77.37.203

## Server - OpenMediaVault

    Type: e2-micro
    Location: europe-west1-b
    OS: Debian-11
    External IP: 35.195.233.30

## Server - OpenLDAP

    Type: e2-micro
    Location: europe-west1-b
    OS: CentOS-7
    External IP: 35.240.73.58

## Windows 10

# Fedora - Workstation

## Remote Desktop (GUI)

**Note: To use a GUI it is required to activate the Display Device on the Google Cloud options**

# Server - OpenMediaVault

## Installing OpenMediaVault on debian 11

**Note: Remember to permit HTTP/HTTPS traffic (WebUI must run on port 80 or 443 or it will be blocked by the firewall)**

[Instruction Used](https://docs.openmediavault.org/en/6.x/installation/on_debian.html)

# Server - OpenLDAP

## Install OpenLDAP on CentOS 7

1. Update the server
    ```
    yum update -y
    ```
2. Install OpenLDAP Server 
    ```
    yum install openldap openldap-servers -y
    ```
    
3. Install OpenLDAP Client 
    ```
    yum install openldap-clients -y
    ```
4. Start and Enable OpenLDAP services 
    ```
    systemctl start slapd # Starting the service
    systemctl enable slapd # Makes service start at boot 
    systemctl status slapd # To check the state of the service
    ```
5. Setup OpenLDAP root user password (Outputs a hashed password to use in the next step)
    ```
    slappasswd
    ```
    - **Slappasswd**  is  used  to  generate  an userPassword value suitable for use with ldapmodify(1), ...

6. Configure OpenLDAP Server
    ```
    [root@server-openldap ~]# vi ldaprootpasswd.ldif
    dn: olcDatabase={0}config,cn=config
    changetype: modify
    add: olcRootPW
    olcRootPW: {SSHA}PASSWORD_CREATED
    ```

    - **oldcDatabase={0}** : database instance which can be found in /etc/openldap/slapd.d/cn=config.

    - **changetype** : type of operations needs to perform - add/modify/delete

    - **add** : perform add operation

    - **olcRootPW** :  Specify the Administrative user hashed password.

    Add the entry above by using the ldapadd command. \
    `ldapadd -Y EXTERNAL -H ldapi:/// -f ldaprootpasswd.ldif`

    - **-Y** : Specify the SASL mechanism to be used for authentication. If it's not specified, the program will choose the best mechanism the server knows. More can be checked on ldapadd Man Page.

    - **-H** : Specify URI(s) referring to the ldap server(s) only the protocol/host/port fields are allowed.

    - **-f** : Read the entry modification information from file instead of from standard input.

7.  Configure OpenLDAP Sample Database 

    Copy the DB_CONFIG Example.
    ```
    cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
    ```

    Change the permission.
    ```
    chown -R ldap:ldap /var/lib/ldap/DB_CONFIG
    ```
      - -R: recursive 
      - ldap:ldap = [Owner][:[group]]

    Restart the slapd service
    ```
    systemctl restart slapd
    ```
    Adding customized schemas distributed by Red Hat.
    ```
    ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
    ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
    ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
    ```
8. Add Domain Configuration

    ```
    vi chdomain.ldif 
        # replace to your own domain name for "dc=***,dc=***" section
        # specify the password generated above for "olcRootPW" section

        dn: olcDatabase={1}monitor,cn=config
        changetype: modify
        replace: olcAccess
        olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0 cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=ads,dc=dcc" read by * none

        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        replace: olcSuffix
        olcSuffix: dc=ads,dc=dcc

        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        replace: olcRootDN
        olcRootDN: cn=Manager,dc=ads,dc=dcc

        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        add: olcRootPW
        olcRootPW: {SSHA}LVtjdrLgXyb3PrZNOxWe1Q8zQk+zIvtz

        dn: olcDatabase={2}hdb,cn=config
        changetype: modify
        add: olcAccess
        olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=Manager,dc=ads,dc=dcc" write by anonymous auth by self write by * none
        olcAccess: {1}to dn.base="" by * read
        olcAccess: {2}to * by dn="cn=Manager,dc=ads,dc=dcc" write by * read

    ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 

    vi basedomain.ldif
        # replace to your own domain name for "dc=***,dc=***" section

        dn: dc=ads,dc=dcc  
        objectClass: top
        objectClass: dcObject
        objectclass: organization
        o: Server World
        dc: ads

        dn: cn=Manager,dc=ads,dc=dcc  
        objectClass: organizationalRole
        cn: Manager
        description: Directory Manager

        dn: ou=People,dc=ads,dc=dcc
        objectClass: organizationalUnit
        ou: People

        dn: ou=Group,dc=ads,dc=dcc  
        objectClass: organizationalUnit
        ou: Group

    ldapadd -x -D cn=Manager,dc=ads,dc=dcc -W -f basedomain.ldif 
    ```
9. Add firewall exception
    ```
    firewall-cmd --add-service=ldap --permanent 
    firewall-cmd --reload 
    ```