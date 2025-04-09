# Oracle RAC Performance Tests

## Prerequisites
* A RHEL9 or RHEL8 virtual machine instance accessible via ssh
* Access to the command line interface as the `root` user
* An avaialble Oracle database RAC cluster or database instance

# Red Hat Enterprise Linux VM Setup Guide

Set up a Red Hat Enterprise Linux (RHEL) virtual machine. The VM can be an AWS EC2 instance or one provided on OpenShift Virtualization VM.

## Initial Configuration
### Become the Root User
```bash
sudo su - root
```
### Set Passwords for Users
Set passwords for users including for root:
```bash
passwd
passwd ec2-user # substitute with your user e.g. cloud-user
```

## Register with Red Hat Connect
Register your system with Red Hat Connect:
```bash
rhc connect -a <your activation key> -o <your org>
```

## Update and Install GNOME Desktop (Optional - but recommended)
### Update DNF
Update the DNF package manager:
```bash
dnf update -y
```
### Install Server with GUI
Install the "Server with GUI" package group:
```bash
dnf groupinstall "Server with GUI" -y
```
Set the default target to `graphical.target`:
```bash
systemctl set-default graphical.target
```

## Install EPEL Release Package
For RHEL 9, install the EPEL release package:
```bash
yum install
https://d2lzkl7pfhq30w.cloudfront.net/pub/epel/epel-release-latest-9.noarchhttps://d2lzkl7pfhq30w.cloudfront.net/pub/epel/epel-release-latst-9.noarch.rpm
```
Alternatively, for RHEL 8:
```bash
yum install
https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/e/epel-rhttps://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packaes/e/epel-release-8-21.el8.noarch.rpm
```

## Install Required Packages
Install required packages using DNF:
```bash
dnf install epel-release
dnf install xrdp tigervnc-server xterm -y
```
Enable the XRDP service:
```bash
systemctl enable xrdp
systemctl enable xrdp-sesman
```

## Start and Check Services
Start the XRDP service:
```bash
systemctl start xrdp
```
Check the status of the XRDP service:
```bash
systemctl status xrdp
```

## Configure Firewall Rules
Add a firewall rule for port 3389 (TCP):
```bash
firewall-cmd --add-port=3389/tcp --permanent
```
Reload the firewall configuration:
```bash
firewall-cmd --reload
```

## Configure X Clients
Create an `.Xclients` file with the desired desktop environment:
```bash
echo "gnome-session" > ~/.Xclients
chmod a+x ~/.Xclients
```
Alternatively, install and configure Cinnamon desktop if you like another desktop flavor:
```bash
dnf install cinnamon -y
echo "cinnamon-session" > ~/.Xclients
chmod a+x ~/.Xclients
```

## Access the newly configured VM from a client/developer machine
Use a RDP App from your machine to access the VM you just configured. E.g. Windows App on Mac or Remmina for Linux.

# Oracle Database Client Tools Setup

## Download Oracle Instant Client Packages
Download the Oracle Instant Client basic package:
```bash
curl -L https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient/x86_64/getPackage/oracle-instantclient19.26-basic-19.26.0.0.0-1.el8.x86_64.rpm -o
/tmp/oracle-instantclient19.26-basic-19.26.0.0.0-1.el8.x86_64.rpm
```
Download the Oracle Instant Client SQLPlus package:
```bash
curl -L https://yum.oracle.com/repo/OracleLinux/OL8/oracle/instantclient/x86_64/getPackage/oracle-instantclient19.26-sqlplus-19.26.0.0.0-1.el8.x86_64.rpm -o
/tmp/oracle-instantclient19.26-sqlplus-19.26.0.0.0-1.el8.x86_64.rpm
```

## Install Oracle Instant Client Packages
Install the Oracle Instant Client basic package:
```bash
dnf install  /tmp/oracle-instantclient19.26-basic-19.26.0.0.0-1.el8.x86_64.rpm
```
Install the Oracle Instant Client SQLPlus package:
```bash
dnf install  /tmp/oracle-instantclient19.26-sqlplus-19.26.0.0.0-1.el8.x86_64.rpm
```

## Connect to Oracle Database using SQLPlus
Connect to the Oracle database using SQLPlus:
```sql
sqlplus sys/<SYS_PASSWORD>@<DB Host>:<DB Port e.g. 1521>/<DB Name> as sysdba
```

# HammerDB Setup
## Install Required Packages for RHEL8
Install the required packages using DNF:
```bash
sudo dnf install -y tcl tcl-devel libaio unzip curl
```

## Download and Extract HammerDB
Download the HammerDB package:
```bash
wget https://github.com/TPC-Council/HammerDB/releases/download/v4.12/HammerDB-4.12-RHEL8.tar.gz
```
Extract the package:
```bash
tar -xvf ./HammerDB-4.12-RHEL8.tar.gz
```
Change into the extracted directory:
```bash
cd HammerDB-4.12/
```

## Run HammerDB in GNOME like Desktop Envirornment
Run HammerDB UI (applicable if you installed desktop environment such as GNOME):
```bash
./hammerdb
```

## Configure Oracle Database Connection
Create a `tnsnames.ora` file in `/usr/lib/oracle/19.26/client64/lib/network/admin` with the following content:
```markdown
<YOUR SERVICENAME> =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST=<DB Host>)(PORT=1521))
    (CONNECT_DATA =
      (SERVICE_NAME=<YOUR SERVICENAME>)
    )
  )
```

## Set Oracle Database Environment Variables
Set the Oracle database environment variables:
```bash
export ORACLE_SID=<your SID>
export ORACLE_BASE=/usr/lib/oracle
export ORACLE_HOME=$ORACLE_BASE/19.26/client64
export ORACLE_LIBRARY=$ORACLE_HOME/lib/libclntsh.so
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
export PATH=$ORACLE_HOME/bin:$PATH
export TNS_ADMIN=/usr/lib/oracle/19.26/client64/lib/network/admin
```

## Configure firewall to allow communication to the database
```bash
firewall-cmd --add-port=1521/tcp --permanent
firewall-cmd --reload
```

## Verify Oracle Library Configuration
Run the following command to verify if the Oracle library is configured correctly with Success log message:
```bash
./hammerdbcli

librarycheck
```

# Run HammerDB test with hammerdbcli
Invoke the hammerdb cli in a terminal with Oracle database environment variables set as described earlier.
```bash
hammerdbcli
```

Execute following lines with substituted values based on your environment
```tcl
./hammerdbcli

dbset db ora
dbset bm TPROC-C
diset connection system_user system
diset connection system_password <db passwor>
diset connection instance <DB Instance>
diset connection rac 1

diset tpcc count_ware 2
diset tpcc num_vu 2
diset tpcc tpcc_user C##hammerdb
diset tpcc tpcc_pass  <desired password for the user above>
diset tpcc tpcc_def_tab users
diset tpcc tpcc_ol_tab users
diset tpcc tpcc_def_temp temp
diset tpcc partition false
diset tpcc hash_clusters false
diset tpcc tpcc_tt_compat false
diset tpcc total_iterations 10000000
diset tpcc raiseerror false
diset tpcc keyandthink false
diset tpcc checkpoint false
diset tpcc ora_driver timed
diset tpcc rampup 2
diset tpcc duration 5
diset tpcc allwarehouse false
diset tpcc ora_timeprofile false
diset tpcc async_scale false
diset tpcc async_client 10
diset tpcc async_verbose false
diset tpcc async_delay 1000
diset tpcc connect_pool false

buildschema
vurun
```