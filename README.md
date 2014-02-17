Puppet Cloudera Manager and CDH4 Module
=======================================

master branch: [![Build Status](https://secure.travis-ci.org/razorsedge/puppet-cloudera.png?branch=master)](http://travis-ci.org/razorsedge/puppet-cloudera)
develop branch: [![Build Status](https://secure.travis-ci.org/razorsedge/puppet-cloudera.png?branch=develop)](http://travis-ci.org/razorsedge/puppet-cloudera)

Introduction
------------

This module manages the installation of [Cloudera Manager](http://www.cloudera.com/content/cloudera/en/products/cloudera-manager.html).  It follows the standards written in the [Cloudera Manager Installation Guide](https://ccp.cloudera.com/display/ENT41DOC/Cloudera+Manager+Installation+Guide) [Installation Path B - Installation Using Your Own Method](https://ccp.cloudera.com/display/ENT41DOC/Installation+Path+B+-+Installation+Using+Your+Own+Method).  By default, this module assumes that [parcels](http://blog.cloudera.com/blog/2013/05/faq-understanding-the-parcel-binary-distribution-format/) will be used to deploy CDH, Impala, and Search.  If parcels are not desired, this module can also manage the installation of [Cloudera's Distribution, including Apache Hadoop (CDH)](http://www.cloudera.com/content/cloudera/en/products/cdh.html), [Cloudera Impala](http://www.cloudera.com/content/cloudera-content/cloudera-docs/Impala/latest/Installing-and-Using-Impala/Installing-and-Using-Impala.html), [Cloudera Search](http://www.cloudera.com/content/cloudera-content/cloudera-docs/Search/latest/Cloudera-Search-Installation-Guide/Cloudera-Search-Installation-Guide.html), and [LZO compression](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Installation-Guide/cmig_install_LZO_Compression.html).

Actions:

* Installs the Cloudera software repository for CM.
* Installs Oracle JDK 6.
* Installs CM 4 agent.
* Configures the CM agent to talk to a CM server.
* Starts the CM agent.
* Separately installs the CM server and database connectivity (by default to the embedded database server).
* Separately starts the CM server.

Optional Actions (non-parcel):

* Installs the Cloudera software repositories for CDH, Impala, and Search.
* Installs most components of CDH 4.
* Installs Impala 1.
* Installs Search 1.
* Installs GPL Extras (LZO) 4.

Software Support:

* Cloudera Manager    - tested with 4.1.2 and 4.8.0
* CDH                 - tested with 4.1.2 and 4.5.0
* Cloudera Impala     - tested with 1.0 and 1.2.3
* Cloudera Search     - tested with 1.1.0
* Cloudera GPL Extras - tested with 4.3.0

OS Support:

Cloudera official [supported operating systems](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Installation-Guide/cmig_cm_requirements.html).

* RedHat family - tested on CentOS 6.4
* SuSE family   - tested on SLES 11SP1
* Debian family - tested on Debian 6.0.7, Ubuntu 10.04.4 LTS, and Ubuntu 12.04.2 LTS

Class documentation is available via puppetdoc.

Class Descriptions
------------------

### Class['cloudera']

Meta-class that includes:

* Class['cloudera::cm::repo']
* Class['cloudera::java']
* Class['cloudera::cm']

Requires the parameter `cm_server_host`.

### Class['cloudera::cm::repo']

This class handles installing the Cloudera Manager software repository.

### Class['cloudera::java']

This class handles installing the Oracle JDK from the Cloudera Manager repository.

### Class['cloudera::java::jce']

This class handles installing the Oracle Java Cryptography Extension (JCE) unlimited strength jurisdiction policy files.  Manual setup is requied in order to download the required software from Oracle.  See the files/README_JCE.md file for details.

### Class['cloudera::cm']

This class handles installing and configuring the Cloudera Manager Agent.  This agent should be running on every node in the cluster so that Cloudera Manager can deploy software configurations to the node.  Requires the parameter `server_host` which is passed in from Class['cloudera'].

### Class['cloudera::cm::server']

This class handles installing and configuring the Cloudera Manager Server.  This class should only be included on one node of your environment.  By default it will install the embeded PostgreSQL database on the same node.  With the correct parameters, it can also connect to local or remote MySQL, PostgreSQL, and Oracle RDBMS databases.


### Class['cloudera::cdh::repo']

This class handles installing the Cloudera Hadoop software repositories.

### Class['cloudera::cdh']

This class handles installing the Cloudera Distribution, including Apache Hadoop.  No configuration is performed on the CDH software and all daemons are forced off so that Cloudera Manager can manage them.  This class installs Bigtop utils, Hadoop (HDFS, MapReduce, YARN), Hue-plugins, HBase, Hive, Oozie, Pig, ZooKeeper, and Flume-NG.

### Class['cloudera::cdh::hue']

This class handles installing Hue.  This class is not currently included in Class['cloudera::cdh'] as this would conflict with the Cloudera installation instructions.

### Class['cloudera::impala::repo']

This class handles installing the Cloudera Impala software repositories.

### Class['cloudera::impala']

This class handles installing Cloudera Impala.  No configuration is performed on the Impala software and all daemons are forced off so that Cloudera Manager can manage them.

### Class['cloudera::search::repo']

This class handles installing the Cloudera Search software repositories.

### Class['cloudera::search']

This class handles installing Cloudera Search.  No configuration is performed on the Search software and all daemons are forced off so that Cloudera Manager can manage them.

### Class['cloudera::gplextras::repo']

This class handles installing the Cloudera GPL Extras software repositories.

### Class['cloudera::gplextras']

This class handles installing Cloudera's GPL Extras (LZO compression libraries).  No configuration is performed on any software.


Examples
--------

Most nodes in the cluster will use this declaration:
```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
}
```

The node that will be the CM server may use this declaration:
```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
} ->
class { 'cloudera::cm::server': }
```

### Parcels

[Parcel](http://blog.cloudera.com/blog/2013/05/faq-understanding-the-parcel-binary-distribution-format/) is an alternative binary distribution format supported by Cloudera Manager 4.5+ that simplifies distribution of CDH and other Cloudera products.  By default, this module assumes software deployment via parcel.  To allow Cloudera Manager to install RPMs (or DEBs) instead of parcels, just set `use_parcels => false`.

Nodes that will be cluster members will use this declaration:
```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
  use_parcels    => false,
}
```

Nodes that will be Gateways may use this declaration:
```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
  use_parcels    => false,
}
class { 'cloudera::cdh::hue': }
class { 'cloudera::cdh::mahout': }
class { 'cloudera::cdh::sqoop': }
# Install Oozie WebUI support (optional):
#class { 'cloudera::cdh::oozie::ext': }
# Install MySQL support (optional):
#class { 'cloudera::cdh::hue::mysql': }
#class { 'cloudera::cdh::oozie::mysql': }
```

The node that will be the CM server may use this declaration:
(This will skip installation of the CDH software as it is not required.)
```puppet
class { 'cloudera::cm::repo':
  cm_version => '4.1',
} ->
class { 'cloudera::java': } ->
class { 'cloudera::java::jce': } ->
class { 'cloudera::cm': } ->
class { 'cloudera::cm::server': }
```

### TLS
Level 1: [Configuring TLS Encryption only for Cloudera Manager](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Administration-Guide/cmag_config_tls_encr.html)
Level 2: [Configuring TLS Authentication of Server to Agents and Users](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Administration-Guide/cmag_config_tls_auth.html)
Level 3: [Configuring TLS Authentication of Agents to Server](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Administration-Guide/cmag_config_tls_agent_auth.html)

This module's deployment of TLS provides both level 1 and level 2 configuration (encryption and authentication of the server to the agents).  Level 3 is presently much more difficult to implement.  You will need to provide a TLS certificate and the signing certificate authority for the CM server.  See the File resources in the below example for where the files need to be deployed.

There are some settings inside CM that can only be configured manually.  See the [Level 1](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Administration-Guide/cmag_config_tls_encr.html) instructions for the details of what to change in the WebUI and use the below values:

    Setting                       Value
    Use TLS Encryption for Agents (check)
    Path to TLS Keystore File     /etc/cloudera-scm-server/keystore
    Keystore Password             The value of server_keypw in Class['cloudera::cm::server'].
    Use TLS Encryption for        (check)
      Admin Console

```puppet
# The node that will be the CM agent may use this declaration:
class { 'cloudera':
  server_host => 'smhost.example.com',
  use_tls     => true,
  install_jce => true,
}
file { '/etc/pki/tls/certs/cloudera_manager.crt': }
```

```puppet
# The node that will be the CM server may use this declaration:
class { 'cloudera':
  server_host => 'smhost.example.com',
  use_tls     => true,
  install_jce => true,
} ->
class { 'cloudera::cm::server':
  use_tls      => true,
  server_keypw => 'myPassWord',
}
file { '/etc/pki/tls/certs/cloudera_manager.crt': }
file { '/etc/pki/tls/certs/cloudera_manager-ca.crt': }
file { "/etc/pki/tls/certs/${::fqdn}-cloudera_manager.crt": }
file { "/etc/pki/tls/private/${::fqdn}-cloudera_manager.key": }
```

### LZO Compression

[LZO](http://www.oberhumer.com/opensource/lzo/) Compression libraries are available in the GPL Extras repository.  To deploy the software on a non-parcel system just add `use_gplextras => true` to the class declaration.  Additional configuration in Cloudera Manager will be required to [activate](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Installation-Guide/cmig_install_LZO_Compression.html) the functionality (ignore the mention of parcels in the link to the documentation).

```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
  use_parcels    => false,
  use_gplextras  => true,
}
```

Notes
-----

* Supports Top Scope variables (i.e. via Dashboard) and Parameterized Classes.
* Installing CDH3 will not be supported.
* Based on the [Cloudera Manager 4.1 Installation Guide](https://ccp.cloudera.com/download/attachments/22151983/CM-4.1-enterprise-install-guide.pdf?version=3&modificationDate=1358553325305)
* TLS certificates must be in PEM format and are not deployed by this module.
* When using parcels, the CDH software is not deployed by Puppet.  Puppet will only install the Cloudera Manager server/agent.  You must then configure Cloudera Manager to deploy the parcels.
* When installing packages and not parcels on SLES, SP2 is required as the hadoop-2.0.0+1518-1.cdh4.5.0.p0.24.sles11.x86_64 package requires netcat-openbsd which is not avalable on SLES 11SP1.

Issues
------

* Need external module support for the Oracle Instant Client JDBC.
* When using an external PostgreSQL server that is on the same host as the CM server, PostgreSQL must be configured to accept connections with md5 password authentication.

TODO
----

See TODO.md for more items.

Deprecation Warning
-------------------

The default for `use_parcels` will switch to `true` before the 1.0.0 release.

This:

```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
}
```

would become this:

```puppet
class { 'cloudera':
  cm_server_host => 'smhost.example.com',
  use_parcels    => false,
}
```

The [puppetlabs/mysql](https://forge.puppetlabs.com/puppetlabs/mysql) dependency will update to version 2.  Make sure to review its changelog in the case of an upgrade.

The class `cloudera::repo` will be renamed to `cloudera::cdh::repo` and the Impala repository will be split out into `cloudera::impala::repo`.

This:

```puppet
class { 'cloudera::repo':
  cdh_version => '4.1',
  cm_version  => '4.1',
}
```

would become this:

```puppet
class { 'cloudera::cdh::repo':
  cdh_version => '4.1',
}
class { 'cloudera::impala::repo':
  ci_version => '4.1',
}
```

Contributing
------------

Please see DEVELOP.md for contribution information.

License
-------

Please see LICENSE file.

Copyright
---------

Copyright (C) 2013 Mike Arnold <mike@razorsedge.org>

[razorsedge/puppet-cloudera on GitHub](https://github.com/razorsedge/puppet-cloudera)

[razorsedge/cloudera on Puppet Forge](http://forge.puppetlabs.com/razorsedge/cloudera)

