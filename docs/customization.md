[<-home page](../README.md)

### Customization

From this page you'll learn how to customize Logsearch-for-cloudfoundry using configuration settings. 

This way of customization is defenetely more preferable then changing source code, because in case of custom changes in source code you have to synchronize your repository with the upstream each time when you want to get recent updates from the upstream. This can be really painful especially in case of major changes in the upstream. From the other hand, not every piece of functionality can be tuned using configuration settings. 

This page lists most popular customizations that can be done using configuration.

#### Index name

To use custom index name it's enough to set `logstash_parser.elasticsearch.index` property in deployment manifest:

```yaml
properties:
  logstash_parser:
    elasticsearch:
      index: "logs-my_custom_name"
```

Please note that __logs-__ prefix should be kept in a new name so that [Elasticsearch mappings](features.md#elasticsearch-mappings) be still applied.

#### Parsing rules

To add custom parsing use `logstash_parser.filters` property of `parser` job. You can either pass a list of file paths there:

```yaml
- name: parser
  properties:
    logstash_parser:
      filters:
      - logsearch-for-cf: /var/vcap/packages/logsearch-config-logstash-filters/logstash-filters-default.conf
      - my-custom-rules: /path/to/my/custom/rules
```
or put a block of code with your parsing rules:
```yaml
- name: parser
  properties:
    logstash_parser:
      filters: { .. }
```
Note that the latter possibility overrides Logsearch-for-cloudfoundry parsing rules, while the former can be used in both ways - to append your custom parsing rules to the Logsearch-for-cloudfoundry parsing and to override them.

Please do mind **the order** of parsing rules you specify.

#### Elasticsearch mappings

Elasticsearch mappings can be customized via `elasticsearch_config.templates` property of `maintenance` job:

```yaml
- name: maintenance
  templates:
  - (( merge ))
  - {name: elasticsearch-config-lfc, release: logsearch-for-cloudfoundry}
  properties:
    elasticsearch_config:
      templates:
      - index_template: /var/vcap/packages/logsearch-config-es-mappings/logs-template.json
      - my_custom_mappings_template: /path/to/my-template.json
```

Please pay attention that [_Elasticsearch mappings ordering_](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-templates.html#multiple-templates) is resolved by `order` attribute.

#### Kibana saved objects

To disable upload of predefined Kibana objects, provided by the job, set `kibana_objects.upload_predefined_kibana_objects: false`. To make Logsearch-for-cloudfoundry upload your custom Kibana saved objects (searches, visualizations, dashboards etc.) use `kibana_objects.upload_data_files` property of `upload-kibana-objects` job:

```yaml
- name: upload-kibana-objects
  ...
  properties:
    ...
    kibana_objects:
      upload_predefined_kibana_objects: false # Whether to upload Kibana objects predefined in this job or not.
      upload_data_files: # List of text files to put in API endpoint /_bulk
        - /path/to/your/custom/kibana/objects/bulk/json/file
```

#### Kibana plugins

If you want some additional plugins to be installed to Kibana then specify them as the following:

1) in case of Kibana deployed to CloudFoundry
```yaml
properties:
  cf-kibana:
    plugins:
      - my-plugin-1: /path/to/my-plugin-1
      - my-plugin-2: /path/to/my-plugin-2
```

2) in case of standalone Kibana instance
```yaml
- name: kibana
  properties:
    kibana:
      plugins:
      - auth: /var/vcap/packages/kibana-auth-plugin/kibana-auth-plugin.tar.gz
      - my-plugin-1: /path/to/my-plugin-1
      - my-plugin-2: /path/to/my-plugin-2
```

</br>[<- prev page](logs-parsing.md) | [next page ->](troubleshooting.md)
