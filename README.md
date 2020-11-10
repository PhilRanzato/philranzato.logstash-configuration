philranzato.logstash-configuration
=========

Configure Logstash

Requirements
------------

- Sudo permissions
- The Cluster CA if using SSL
- Certificates for Logstash of using certificates
- A JSON file containing elastic and logstash_system passwords in the format:
    
    ```json
    [
        {
            "password": "xxxx",
            "user": "elastic"
        },
        {
            "password": "yyyy",
            "user": "logstash_system"
        }
    ]
    ```
- Logstash installed on the target machine
- An elasticsearch master available to be reached through API call


Role Variables
--------------

```yaml
---
# The password for the logstash keystore 
keystore_pass: 'L0gst4sh!'
os_environment:
- key: LOGSTASH_KEYSTORE_PASS 
  value: "{{ keystore_pass }}"  


# logstash_system user
logstash_system_user: logstash_system
logstash_system_password: "" # Leave it blank if you have the builtin_passwords file

# elastic user
elastic_password: "" # Leave it blank if you have the builtin_passwords file

# Logstash Internal user with the role logstash_writer
logstash_internal:
  password: "Logstash-Internal"

# Role logstash_writer
logstash_writer:
  # index pattern for the logstash_writer
  index_patterns:
  - "logstash-*"

# Wheter to include an example pipeline for output
include_example_pipeline: false
# Wheter to include an example pipeline for filebeat input
include_filebeat_pipeline: false
# Filebeat input configuration
filebeat:
  ca_path: /etc/logstash/elastic-CA.crt
  logstash_server: 
    cert: /etc/logstash/logstash-filebeat.crt
    # THIS KEY MUST BE IN PKCS8 FORMAT, see documentation
    key:  /etc/logstash/logstash-filebeat.key
    key_pass: "P4ssword!"

# CA parameters
ca:
  dir: /etc/logstash
  cert: "elastic-CA.crt"
  key: "elastic-CA.key"

# Certificate parameters for logstash
logstash:
  conf_dir: "/etc/logstash"
  data_dir: "/var/lib/logstash"
  logs_dir: "/var/log/logstash"
  truststore:
    path: "/etc/logstash/lgs.p12"
    password: "test"
  keystore:
    path: "/etc/logstash/lgs.p12"
    password: "test"
  user: "{{ logstash_system_user }}"
  password: "{{ logstash_system_password }}"

# Elasticsearch masters variables
elasticsearch_http_port: "9200"
elasticsearch_primary_master_group_name: "elasticsearch_primary_master"
elasticsearch_masters_group_name: "elasticsearch_master"

# /etc/logstash/logstash.yml
node:
  name: "{{ inventory_hostname | regex_replace('\\..+', '') }}"

path:
  data: "{{ logstash.data_dir }}"
  logs: "{{ logstash.logs_dir }}"

pipeline:
  ordered: auto

xpack:
  monitoring:
      enabled: true
      elasticsearch:
        username: "{{ logstash.user }}"
        password: "{{ logstash.password }}"
        sniffing: false
        ssl:
          enabled: false
          certificate_authority: "{{ ca.dir }}/{{ ca.cert }}"
          truststore:
            path: "{{ logstash.truststore.path }}"
            password: "{{ logstash.truststore.password }}"
          keystore:
            path: "{{ logstash.keystore.path }}"
            password: "{{ logstash.keystore.password }}"
          verification_mode: certificate
      collection:
        interval: 10s
        pipeline:
          details:
            enabled: true
```

Dependencies
------------

None.

Example Playbook
----------------

```yaml
---
- name: "Configure Logstash"
  hosts: logstash
  roles:
  - role: philranzato.logstash-configuration
  vars_files:
  - vars/logstash-configuration.yml
```

License
-------

BSD

Author Information
------------------

Check me on [LinkedIn](www.linkedin.com/in/phil-ranzato-47b8bb194)
