# Ansible role for Logstash

Configure and start [Logstash](https://www.elastic.co/products/logstash)  in a docker container using the [official docker image](https://www.docker.elastic.co/#logstash) `docker.elastic.co/logstash/logstash:6.2.4`. You can change the release version yourself.  
[More information](https://www.elastic.co/guide/en/logstash/current/docker.html) about using docker image.

## Role Variables

- `docker_logstash_name` (default: logstash): This name used in name of home folder path and in name of container.
- `docker_logstash_image` (default: docker.elastic.co/logstash/logstash:6.2.4)
- `docker_logstash_pipeline`: Body of pipeline file.
- `docker_logstash_uninstall_plugins` (default: []): List logstash plugins for uninstall from docker container.
- `docker_logstash_plugins` (default: []): List logstash plugins for install to docker container.
- `docker_logstash_ports` (default: [5044])
- `docker_logstash_env` (optional)
- `docker_logstash_network_mode` (default: host): Connect the container to a network.
- `docker_logstash_networks` (default: []): List of networks the container belongs to.
- `docker_logstash_log_driver` (default: json-file): Specify the logging driver.
- `docker_logstash_log_options` (optional): Dictionary of options specific to the chosen log_driver. See [Configure logging drivers](https://docs.docker.com/engine/admin/logging/overview/) for details.

## Dependencies

- `docker`

## Example Playbook

```yaml
- hosts: all
  become: yes
  become_method: sudo
  vars:
    elasticsearch_url: "http://elasticsearch:9200"
    influxdb_host: "influxdb"
  roles:
    - role: levonet.docker_logstash
      docker_logstash_image: docker.elastic.co/logstash/logstash:6.3.1
      docker_logstash_ports:
        - 5043:5043
        - 5044:5044
      docker_logstash_env:
        LOG_LEVEL: debug
      docker_logstash_uninstall_plugins:
        - x-pack
      docker_logstash_plugins:
        - logstash-output-influxdb
      docker_logstash_log_driver: syslog
      docker_logstash_log_options:
        syslog-facility: local0
        tag: "{{ docker_logstash_name }}"
      docker_logstash_pipeline: |
        input {
          beats {
            port => 5043
            codec => json
          }
          beats {
            port => 5044
          }
        }
        filter {
          # ...
        }
        output {
          elasticsearch {
            hosts => "{{ elasticsearch_url }}"
            user => "elastic"
            password => "XXXXXX"
            index => "MONITORING-%{+YYYY.MM.dd}"
          }
          influxdb {
            host => "{{ influxdb_host }}"
            use_event_fields_for_data_points => true
            data_points => {}
            exclude_fields => ["@timestamp", "@version", "sequence", "message"]
          }
        }

```

## License

[MIT](https://opensource.org/licenses/MIT)

## Author Information

This role was created by [Pavlo Bashynskyi](https://github.com/levonet)
