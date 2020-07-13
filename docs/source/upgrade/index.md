# Upgrade to Gluu Server 4.2 Beta

## Overview
The Gluu Server **cannot** be upgraded with a simple `apt-get upgrade`. You will need to either use our in-place upgrade script or explicitly install the new version and export/import your data. Find the existing version below for upgrade instructions to Gluu Server 4.2. 

### Pre-requisites

- Before upgrading, make sure to [back up](../operation/backup.md) the Gluu container or LDAP LDIF. 
- Upgrades should always be thoroughly scoped and tested on a development environment *first*.

### Upgrading from 3.1.x to 4.2

At this time, only Gluu Server version 3.1.x can be upgraded to version 4.2 Beta. The upgrade script works on CentOS 7, Ubuntu 16, and RedHat 7. Upgrade script performs the following steps:

- Upgrades Java to Amazon Corretto. Extracts certificates from the existing Java keystore to `hostname_service.crt` in the upgrade directory. After upgrading Java, imports to keystore
- Upgrades all Gluu WAR files, NodeJS, and Passport components
- Transfers all data from LDAP to `gluu.ldif` in the upgrade directory
- Upgrades to [WrenDS](https://github.com/WrenSecurity/wrends) (a community maintained fork of OpenDJ). If you are currently running OpenLDAP, it will be backed up and migrated to WrenDS
- Processes `gluu.ldif` to convert the existing data set to the new model. Removes all inums. Depending on the data
size, this step will take some time. Writes resulting data to `gluu_noinum.ldif`. Your current passport configuration
will be moved to `gluuPassportConfiguration.json` for future reference
- Imports `gluu_noinum.ldif` to newly installed WrenDS. Rejected and Skipped entries will be written to 
`opendj_rejects.txt` and `opendj_skips.txt` to the upgrade directory
- Upgrade script uses setup.py to updated the configuration. All activities will be logged to `setup/update.log` and
`update_error.log`
- All files will be backed up with `file_name.gluu-version-#~` where # is a consecutive number, unless backup is specified in
another way.
- Sets the OpenID Connect `claimsParameterSupported` property to `false` by default to ensure clients are unable to gather unwanted claims. If a client in use depends on this property, it can be set back to `true` in the JSON configuration.

!!! Note
    If you are using custom schema:  
    (a) OpenDJ Users: Back up the schema file  
    (b) OpenLDAP users: Convert the schema according to [this guide](https://backstage.forgerock.com/docs/opendj/3.5/admin-guide/#chap-schema)  
    
    When the upgrade script prompts:  
    
    ```
    If you have custom ldap schema, add them now and press c  
    If you don't have any custom schema you can continue with pressing c
    ```
    
    Put the schema file in `/opt/opendj/config/schema/`


There are two options to perform the upgrade (both methods work inside the container):

#### Online Upgrade
!!! Note
    Upgrade script runs on Python 3. You need to install Python 3 before running the script.
    * On CentoOS/RHEL: `yum install -y python3`
    * On Ubuntu/Debian: `apt-get update && apt-get install -y python3`

The upgrade script downloads all needed software and applications from the internet. You can perform an online upgrade by following these steps:

* Download the upgrade script

```
wget https://raw.githubusercontent.com/GluuFederation/community-edition-package/master/update/4.2.0/upg4xto420.py
```

* Execute the script:

```
python3 upg4xto420.py
```

Your upgrade directory will be the current directory. The script will create these directories: `app`, `war`, `temp`, `setup`
<!--
#### Static Upgrade
The static, self-extracting upgrade package contains all components for the upgrade. You still need an internet connection to install the libraries that are needed by the upgrade script. To perform a static upgrade, follow these steps:

* Download the self-extracting package

```
wget http:// ...... /4.2-upg.sh
```

* Execute the script

```
sh 4.2-upg.sh
```

The upgrade directory will be `/opt/upd/4.2-upg`
-->
