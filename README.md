# ansible-role-oraclejdk
An Ansible role to install Oracle JDK.

[![Platform](http://img.shields.io/badge/platform-ubuntu-dd4814.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-debian-a80030.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-redhat-cc0000.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-centos-932279.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-fedora-333fff.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-suse-008000.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-opensuse-49BD31.svg?style=flat)](#)

[![OracleJDK](http://img.shields.io/badge/oracle%20jdk-%20%208%20%20-cc0000.svg?style=flat)](#)
[![OracleJDK](http://img.shields.io/badge/oracle%20jdk-%20%209%20%20-cc0000.svg?style=flat)](#)

## Summary

This Ansible role installs Oracle JDK and Java Cryptography Extension (optional) on the target host. OpenJDK is not supported. It has been tested with Oracle JDK 8u144/8u151 and JDK 9u181/9.0.1 on the following Linux distributions.

* Ubuntu 16.04
* Debian 9
* RHEL 7.4
* CentOS 7.3
* Fedora 26
* SLE 12 SP2
* openSUSE 42.2

## Dependencies

Ansible >= 2.x (tested with 2.3.2.0 and 2.4.0.0)

In addition this role requires the following packages to be installed on the target host.

* `tar` - Always required to extract the JDK tar.gz archive
* `unzip` - Only required if JCE for JDK8 should be installed

## Installation

To use this role it must first get downloaded from [Ansible Galaxy](https://galaxy.ansible.com). To install the role execute the command below on your Ansible controle machine. For more information please refer to the [official documentation](https://galaxy.ansible.com/intro#download).

`ansible-galaxy install frieder.oraclejdk`

The command above will always pull the latest version off of Galaxy. Another possibility is to pull the role from Github and define a specific release version to download. This allows for better control over which version of this role to be used. Create a `requirements.yml` and put the following content in it.

```yaml
---
- name: frieder.oraclejdk
  src: https://github.com/frieder/ansible-role-oraclejdk
  version: 2.0.0
```

Next you can execute `ansible-galaxy install -r ./requirements.yml --ignore-errors` and it will download all dependencies defined in this list. `--ignore-errors` will make sure that the whole list is processed even when some dependencies are already downloaded. A nice overview of possible entries for `requirements.yml` can be found [here](https://zaiste.net/posts/automatically_install_ansible_galaxy_roles_with_requirements_yml/).

## Role Variables

All role variables are defined in `defaults/main.yml`. One can overwrite these values in the playbook (see examples at the bottom).

| Variable | Default | Description |
|--:|:-:|:--|
| oraclejdk_license_accept | false | When installing a JDK this must be set to true otherwise the play will fail. Not required when removing a JDK. |
| oraclejdk_state | present | Defines whether to add or remove a JDK from the target host. Possible values are `[present,absent]`. |
| oraclejdk_cleanup | true | Remove all temp files/folders (both local and remote) after installation. When set to true this will result in "changed" plays. |
| oraclejdk_dl_dir | /tmp/oraclejdk | The folder (both local and remote) to which the archives get downloaded/copied/extracted. |
| oraclejdk_home | | The Java home directory to which the JDK should get extracted to and to which JAVA_HOME will point to if set. |
| oraclejdk_profile_file | /etc/profile.d/java.sh | The file in which the role will set the JAVA_HOME and PATH export. |
| oraclejdk_cookie | Cookie:oraclelicense=accept-securebackup-cookie | The cookie required for automated downloads from oracle.com. License check is done with a different variable. |
| oraclejdk_url | | The URL of the JDK archive either at oracle.com or a local corporate repository (recommended). |
| oraclejdk_checksum | | The SHA256 checksum of the JDK archive. This is used to verify that the downloaded file is valid. To disable this check just provide an empty checksum value (`oraclejdk_checksum: ''`). |
| oraclejdk_sethome | true | When set to true it will update the global variable JAVA_HOME to point to the installation directory of the JDK and add the binaries to the PATH variable. Also have a look at the `oraclejdk_profile_file` variable. |
| oraclejdk_alternative_upd | true | When set to true it will set the alternative for java (`update-alternatives --config java`) to the current JDK. |
| oraclejdk_alternative_prio | 1 | The priority used for the `update-alternatives` command. The JDK with the highest priority wins. |
| oraclejdk_alternative_items | <ul><li>jar</li><li>java</li><li>javac</li><li>jcmd</li><li>jconsole</li><li>jmap</li><li>jps</li><li>jstack</li><li>jstat</li><li>jstatd</li></ul> | This property allows to define what kind of alternatives should be configured for the JDK. |
| oraclejdk_jce_install | false | When set to true it will install and add the latest Java Cryptography Extension (JCE) to the JDK. Please note that with JDK9 the unlimited key strength is enabled by default and no additional action are required. For more information please refer to the [security-dev mailinglist](http://mail.openjdk.java.net/pipermail/security-dev/2016-October/014943.html). |
| oraclejdk_jce_name | UnlimitedJCEPolicyJDK8 | **JDK8 only**. The name of the folder inside the JCE archive. |
| oraclejdk_jce_url | | **JDK8 only**. The URL of the JCE archive either at oracle.com or a local corporate repository (recommended). |
| oraclejdk_jce_checksum | | **JDK8 only**. The SHA256 checksum of the JCE archive. To disable this check just provide an empty checksum value (`oraclejdk_jce_checksum: ''`). |

## Example Playbooks

Following are some examples how to use this role in an Ansible playbook.

**JDK8 & JDK9 together, set home for JDK8, update alternatives for both JDKs with different priorities**
```yaml
- hosts: jdk
  pre_tasks:
  - name: Install required packages (127.0.0.1)
    delegate_to: 127.0.0.1
    package:
      name: '{{ item }}'
      state: present
    with_items:
    - tar
    - unzip
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/jdk9.0.1
    oraclejdk_sethome: false
    oraclejdk_alternative_prio: 100
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/9.0.1+11/jdk-9.0.1_linux-x64_bin.tar.gz'
    oraclejdk_checksum: 'sha256:2cdaf0ff92d0829b510edd883a4ac8322c02f2fc1beae95d048b6716076bc014'
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/jdk8u151
    oraclejdk_alternative_prio: 200
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz'
    oraclejdk_checksum: 'sha256:c78200ce409367b296ec39be4427f020e2c585470c4eed01021feada576f027f'
    oraclejdk_jce_install: true
    oraclejdk_jce_url: 'http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip'
    oraclejdk_jce_checksum: 'sha256:f3020a3922efd6626c2fff45695d527f34a8020e938a49292561f18ad1320b59'
```

**JDK8 w/ minimal configuration**
```yaml
- hosts: jdk8
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/jdk8_151
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz'
    oraclejdk_checksum: 'sha256:c78200ce409367b296ec39be4427f020e2c585470c4eed01021feada576f027f'
```

**JDK8 w/ JCE**
```yaml
- hosts: jdk8
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/java-8-151
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz'
    oraclejdk_checksum: 'sha256:c78200ce409367b296ec39be4427f020e2c585470c4eed01021feada576f027f'
    oraclejdk_jce_install: true
    oraclejdk_jce_url: 'http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip'
    oraclejdk_jce_checksum: 'sha256:f3020a3922efd6626c2fff45695d527f34a8020e938a49292561f18ad1320b59'
```

**JDK8 full config**
```yaml
- hosts: jdk8
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_cleanup: true
    oraclejdk_dl_dir: /tmp/java_download
    oraclejdk_home: /opt/java/java-8-151
    oraclejdk_sethome: true
    oraclejdk_alternative_upd: true
    oraclejdk_alternative_prio: 123
    oraclejdk_alternative_items:
    - jar
    - java
    - javac
    - jcmd
    - jconsole
    - jmap
    - jps
    - jstack
    - jstat
    - jstatd
    oraclejdk_profile_file: /etc/profile.d/java.sh
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz'
    oraclejdk_checksum: 'sha256:c78200ce409367b296ec39be4427f020e2c585470c4eed01021feada576f027f'
    oraclejdk_jce_install: true
    oraclejdk_jce_url: 'http://download.oracle.com/otn-pub/java/jce/8/jce_policy-8.zip'
    oraclejdk_jce_checksum: 'sha256:f3020a3922efd6626c2fff45695d527f34a8020e938a49292561f18ad1320b59'
```

**JDK9**
```yaml
- hosts: jdk9
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/jdk9.0.1
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/9.0.1+11/jdk-9.0.1_linux-x64_bin.tar.gz'
    oraclejdk_checksum: 'sha256:2cdaf0ff92d0829b510edd883a4ac8322c02f2fc1beae95d048b6716076bc014'
```

**JDK9 w/o checksum check**
```yaml
- hosts: - hosts: jdk9
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/jdk9.0.1
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/9.0.1+11/jdk-9.0.1_linux-x64_bin.tar.gz'
    oraclejdk_checksum: ''
```

**JDK9 add new JDK, remove old JDK**
```yaml
- hosts: jdk9
  roles:
  - role: frieder.oraclejdk
    oraclejdk_license_accept: true
    oraclejdk_home: /opt/java/jdk9.0.1
    oraclejdk_url: 'http://download.oracle.com/otn-pub/java/jdk/9.0.1+11/jdk-9.0.1_linux-x64_bin.tar.gz'
    oraclejdk_checksum: ''
  - role: frieder.oraclejdk
    oraclejdk_state: absent
    oraclejdk_home: /opt/java/jdk9.0.1
```


