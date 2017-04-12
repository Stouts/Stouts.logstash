Stouts.logstash
==============

[![Build Status](http://img.shields.io/travis/Stouts/Stouts.logstash.svg?style=flat-square)](https://travis-ci.org/Stouts/Stouts.logstash)
[![Galaxy](http://img.shields.io/badge/galaxy-Stouts.logstash-blue.svg?style=flat-square)](https://galaxy.ansible.com/list#/roles/1995)

Ansible role which manage [Logstash](http://www.elasticsearch.org/overview/logstash/)

* Install and configure logstash

#### Dependencies

The roles are recomended to install:

* [Stouts.elasticsearch](https://github.com/Stouts/Stouts.elasticsearch) - for using as DB

#### Variables

Here is the list of all variables and their default values:

```yaml
logstash_enabled: yes                       # The role is enabled

logstash_apt_key: http://packages.elasticsearch.org/GPG-KEY-elasticsearch
logstash_apt_repo: "deb http://packages.elasticsearch.org/logstash/2.3/debian stable main"

logstash_home: /opt/logstash

logstash_plugins:
- logstash-input-beats

logstash_confdir: /etc/logstash

# Certificates (please replace with your own files)
logstash_ssl_cert_file: "logstash-nosafe.crt"
logstash_ssl_key_file: "logstash-nosafe.key"

# Logstash inputs
logstash_config_inputs: |
  file {
    path => [ "/var/log/syslog" ]
    type => "syslog"
  }
  beats {
    port => 5044
  }

# Logstash filters
logstash_config_filters: |
  if [type] == "syslog" {
    if [message] =~ /last message repeated [0-9]+ times/ {
      drop { }
    }
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }

  else if [type] == "nginx" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }

# Logstash outputs
logstash_config_outputs: |
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
    document_type => "%{[@metadata][type]}"
  }
```

#### Usage

Add `Stouts.logstash` to your roles and setup the variables in your playbook file.

Example:

```yaml

- hosts: all

  roles:
    - Stouts.elasticsearch
    - Stouts.logstash

  vars:
    logstash_config_inputs: |
      file { path => [ "/var/log/syslog" ], type => "syslog" }
      beats {
        port => 4000
      }
```

#### License

Licensed under the MIT License. See the LICENSE file for details.

#### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/Stouts/Stouts.logstash/issues)!
