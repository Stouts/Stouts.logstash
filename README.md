Stouts.logstash
==============

[![Build Status](http://img.shields.io/travis/Stouts/Stouts.logstash.svg?style=flat-square)](https://travis-ci.org/Stouts/Stouts.logstash)
[![Galaxy](http://img.shields.io/badge/galaxy-Stouts.logstash-blue.svg?style=flat-square)](https://galaxy.ansible.com/list#/roles/1995)

Ansible role which manage [Logstash](http://www.elasticsearch.org/overview/logstash/)

* Install and configure logstash/logstash-forwarder

#### Dependencies

The roles are recomended to install:

* [Stouts.elasticsearch](https://github.com/Stouts/Stouts.elasticsearch) - for using as DB

#### Variables

Here is the list of all variables and their default values:

```yaml
logstash_enabled: yes                       # The role is enabled
logstash_apt_repositories:                  # These repositories will be added to system
- ppa:webupd8team/java
- deb http://packages.elasticsearch.org/logstash/1.4/debian stable main
- deb http://packages.elasticsearch.org/logstashforwarder/debian stable main
logstash_apt_key: http://packages.elasticsearch.org/GPG-KEY-elasticsearch
logstash_apt_pkgs:
- acl
- oracle-java7-installer
- logstash
- logstash-contrib

logstash_server_enabled: yes                # Setup logstash server
logstash_forwarder_enabled: no              # Setup logstash forwarder
logstash_web_enabled: no                    # Setup logstash web

logstash_user: logstash                     # logstash user
logstash_group: "{{logstash_user}}"         # logstash group

logstash_home: /opt/logstash
logstash_forwarder_home: /opt/logstash-forwarder
logstash_confdir: /etc/logstash
logstash_logdir: /var/log/logstash

# Certificates (please replace with your own files)
logstash_ssl_cert_file: "logstash-nosafe.crt"
logstash_ssl_key_file: "logstash-nosafe.key"

# Grant read permissions to logstash user
logstash_grant_permissions:
- /var/log/syslog

# Logstash server options
# -----------------------

# Logstash inputs
logstash_config_inputs: |
  file { path => [ "/var/log/syslog" ], type => "syslog" }
  lumberjack {
    port => 5000
    type => "lumberjack"
    ssl_certificate => "{{logstash_confdir}}/logstash.crt"
    ssl_key => "{{logstash_confdir}}/logstash.key"
  }

# Logstash filters
logstash_config_filters: |
  if [type] == "syslog" {
    grok { pattern => "%SYSLOGBASE" }
    date { match => ["timestamp", "MMM dd HH:mm:ss"] }
  }
  else if [type] == "nginx-access" {
    grok { pattern => "%{NGINXACCESS}" }
    geoip { source => "clientip" }
  }

# Logstash outputs
logstash_config_outputs: |
  elasticsearch {
    host => "localhost"
  }

# Logstash patterns
logstash_config_patterns:
  nginx: |
    NGUSERNAME [a-zA-Z\.\@\-\+_%]+
    NGUSER %{NGUSERNAME}
    NGINXACCESS %{IPORHOST:clientip} %{NGUSER:ident} %{NGUSER:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{URIPATHPARAM:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response} (?:%{NUMBER:bytes}|-) (?:"(?:%{URI:referrer}|-)"|%{QS:referrer}) %{QS:agent}

# Logstash forwarder options
# --------------------------
logstash_forwarder_servers: [ "127.0.0.1:5000" ]
logstash_forwarder_network:
  servers: "{{logstash_forwarder_servers}}"
  timeout: 15
  "ssl ca": "{{logstash_confdir}}/logstash.crt"
logstash_forwarder_files:
  - paths: [ /var/log/syslog ]
    fields:
      type: syslog
```

#### Usage

Add `Stouts.logstash` to your roles and setup the variables in your playbook file.

Example (server setup):

```yaml

- hosts: all

  roles:
    - Stouts.elasticsearch
    - Stouts.logstash

  vars:
    logstash_config_inputs: |
      file { path => [ "/var/log/syslog" ], type => "syslog" }
      lumberjack {
        port => 4000
        type => "lumberjack"
        ssl_certificate => "{{logstash_confdir}}/logstash.crt"
        ssl_key => "{{logstash_confdir}}/logstash.key"
      }
```

Example (forwarder setup):

```yaml

- hosts: all

  roles:
    - Stouts.elasticsearch
    - Stouts.logstash

  vars:
    logstash_enabled: no
    logstash_forwarder_enabled: yes
    logstash_forwarder_servers:
    - "my.server.com:5000"
    logstash_forwarder_files:
    - paths: [ "/var/log/syslog", "/var/log/auth.log" ]
      fields: { type: syslog }
    - paths: [ "/usr/lib/ticketscloud/log/ticketscloud-access.log" ]
      fields: { type: nginx-access }
```

#### License

Licensed under the MIT License. See the LICENSE file for details.

#### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/Stouts/Stouts.logstash/issues)!
