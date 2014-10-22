Stouts.logstash
==============

[![Build Status](https://travis-ci.org/Stouts/Stouts.logstash.png)](https://travis-ci.org/Stouts/Stouts.logstash)

Ansible role which manage [Logstash](http://www.elasticsearch.org/overview/logstash/)

* Install and configure logstash/logstash-forwarder

#### Dependencies

The roles are recomended to install:

* [Stouts.elasticsearch](https://github.com/Stouts/Stouts.elasticsearch) - for using as DB

#### Variables

Here is the list of all variables and their default values:

```yaml
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
    logstash_config_inputs:
      lumberjack:
        port: 5000
        type: logs
        ssl_certificate: "{{logstash_confdir}}/logstash.crt"
        ssl_key: "{{logstash_confdir}}/logstash.key"

    logstash_config_outputs:
      elasticsearch:
        host: localhost
```

#### License

Licensed under the MIT License. See the LICENSE file for details.

#### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/Stouts/Stouts.logstash/issues)!
