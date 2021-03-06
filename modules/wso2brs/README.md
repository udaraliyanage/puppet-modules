# WSO2 Business Rules Server Puppet Module

This repository contains the Puppet Module for installing and configuring WSO2 Business Rules Server on various environments. It supports multiple versions of WSO2 Business Rules Server. Configuration data is managed using [Hiera](http://docs.puppetlabs.com/hiera/1/). Hiera provides a mechanism for separating configuration data from Puppet scripts and managing them in a separate set of YAML files in a hierarchical manner.

## Supported Operating Systems

- Debian 6 or higher
- Ubuntu 12.04 or higher

## Supported Puppet Versions

- Puppet 2.7, 3 or newer

## How to Contribute
Follow the steps mentioned in the [wiki](https://github.com/wso2/puppet-modules/wiki) to setup a development environment and update/create new puppet modules.

## Packs to be Copied

Copy the following files to their corresponding locations.

1. WSO2 Business Rules Server distribution (v2.2.0) to `<PUPPET_HOME>/modules/wso2brs/files`
2. JDK 1.7_80 distribution to `<PUPPET_HOME>/modules/wso2base/files`

## Running WSO2 Business Rules Server in the `default` profile
No changes to Hiera data are required to run the `default` profile.  Copy the above mentioned files to their corresponding locations and apply the Puppet Modules.

## Running WSO2 Business Rules Server with clustering in specific profiles
Do the below changes to relevant Business Rules Server profiles (`worker`, `manager`) Hiera YAML files to start the server in distributed setup. For more details refer [WSO2 Business Rules Server 2.2.0](https://docs.wso2.com/display/CLUSTER44x/Clustering+Business+Rules+Server) clustering guide.

1. If the Clustering Membership Scheme is `WKA`, add the Well Known Address list.

   Ex:
    ```yaml
    wso2::clustering :
        enabled: true
        local_member_host: 192.168.100.13
        local_member_port: 4000
        membership_scheme: wka
        sub_domain: mgt
        wka:
           members:
             -
               hostname: 192.168.100.43
               port: 4000
             -
               hostname: 192.168.100.44
               port: 4000
    ```

2. Add external databases to master datasources

   Ex:
    ```yaml
    wso2::master_datasources:
     wso2_config_db:
       name: WSO2_CONFIG_DB
       description: The datasource used for config registry
       driver_class_name: org.h2.Driver
       url: jdbc:h2:repository/database/WSO2_CONFIG_DB;DB_CLOSE_ON_EXIT=FALSE;LOCK_TIMEOUT=60000
       username: "%{hiera('wso2::datasources::common::username')}"
       password: "%{hiera('wso2::datasources::common::password')}"
       jndi_config: jdbc/WSO2_CONFIG_DB
       max_active: "%{hiera('wso2::datasources::common::max_active')}"
       max_wait: "%{hiera('wso2::datasources::common::max_wait')}"
       test_on_borrow: "%{hiera('wso2::datasources::common::test_on_borrow')}"
       validation_query: "%{hiera('wso2::datasources::h2::validation_query')}"
       validation_interval: "%{hiera('wso2::datasources::common::validation_interval')}"

    ```

3. Configure registry mounting

   Ex:
    ```yaml
    wso2_config_db:
      path: /_system/config
      target_path: /_system/config/brs
      read_only: false
      registry_root: /
      enable_cache: true

    wso2_gov_db:
      path: /_system/governance
      target_path: /_system/governance
      read_only: false
      registry_root: /
      enable_cache: true
    ```

4. Configure deployment synchronization

    Ex:
    ```yaml
    wso2::dep_sync:
        enabled: true
        auto_checkout: true
        auto_commit: true
        repository_type: svn
        svn:
           url: http://svnrepo.example.com/repos/
           user: username
           password: password
           append_tenant_id: true
    ```

## Running WSO2 Business Rules Server with Secure Vault
WSO2 Carbon products may contain sensitive information such as passwords in configuration files. [WSO2 Secure Vault](https://docs.wso2.com/display/Carbon444/Securing+Passwords+in+Configuration+Files) provides a solution for securing such information.

Uncomment and modify the below changes in Hiera file to apply Secure Vault.

1. Enable Secure Vault

    ```yaml
    wso2::enable_secure_vault: true
    ```

2. Add Secure Vault configurations as below

    ```yaml
    wso2::secure_vault_configs :
      <secure_vault_config_name>:
        secret_alias: <secret_alias>
        secret_alias_value: <secret_alias_value>
        password: <password>
    ```

    Ex:
    ```yaml
    wso2::secure_vault_configs :
      key_store_password :
        secret_alias: Carbon.Security.KeyStore.Password
        secret_alias_value: repository/conf/carbon.xml//Server/Security/KeyStore/Password,false
        password: wso2carbon
    ```

3. Add Cipher Tool configuration file templates to `template_list`

    ```yaml
    wso2::template_list:
      - repository/conf/security/cipher-text.properties
      - repository/conf/security/cipher-tool.properties
      - bin/ciphertool.sh
      - password-tmp
    ```

## Running WSO2 Business Rules Server on Kubernetes
WSO2 BRS Puppet module ships Hiera data required to deploy WSO2 Business Rules Server on Kubernetes. 
For more information refer to the documentation on [deploying WSO2 products on Kubernetes using WSO2 Puppet Modules](https://docs.wso2.com/display/PM200/Deploying+WSO2+Products+on+Kubernetes+Using+WSO2+Puppet+Modules).
