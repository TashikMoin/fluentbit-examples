# Example Configurations for Fluent Bit

## Service

There are some elements of Fluent Bit that are configured for the entire service; use this to set global configurations like the flush interval or troubleshooting mechanisms like the [HTTP server]((https://docs.fluentbit.io/manual/configuration/monitoring)). You can find an example in our Kubernetes Fluent Bit daemonset configuration [found here](https://github.com/newrelic/kubernetes-logging/blob/master/fluent-conf.yml).

```
[SERVICE]
    Flush         1
    Log_File      /var/log/fluentbit.log
    Log_Level     error
    Daemon        off
    Parsers_File  parsers.conf
    HTTP_Server   On
    HTTP_Listen   0.0.0.0
    HTTP_Port     2020
    
@INCLUDE input-kubernetes.conf
@INCLUDE output-newrelic.conf
@INCLUDE filter-kubernetes.conf
```

You will notice in the example below that we are making use of the [@INCLUDE](https://fluentbit.io/documentation/0.13/configuration/file.html#config_include_file) configuration command. This allows you to break your configuration up into different modular files and include them as well. The Log_File and Log_Level are used to set how Fluent Bit creates diagnostic logs for itself; this does not have any impact on the logs you monitor.

## Routing

Routing is a core feature that allows to route your data through Filters and finally to one or multiple destinations. Please take the time to read the [official documentation](https://docs.fluentbit.io/manual/getting_started/routing) on the subject. You will learn how the Tag value you set on an input relates to what filtering and outputs will match the data.

## Input

Input plugins are how logs are read or accepted into Fluent Bit. Common examples are syslog or tail. Syslog listens on a port for syslog messages, and tail follows a log file and forwards logs as they are added. A list of available input plugins can be [found here](https://docs.fluentbit.io/manual/input).

### File Input

One of the most common types of log input is tailing a file. The in_tail input plugin allows you to read from a text log file as though you were running the _tail -f_ command. Full documentation on this plugin can be found [here](https://fluentbit.io/documentation/0.13/input/tail.html).

```
[INPUT]
    Name              tail
    Tag               test.file
    Path              /var/log/apache2/error.log
    DB                /var/log/apache2_error.db
    Path_Key          filename
    Parser            apache2
    Mem_Buf_Limit     8MB
    Skip_Long_Lines   On
    Refresh_Interval  30
```

In this tail example we are tailing an Apache error log and parsing it. This is using the pre-defined parser for apache2; you can also define custom parsers. We recommend using the DB option to keep track of what you have monitored, and to set the Path_Key so that an attribute is populated in the output that will help you differentiate the file source of the logs you aggregate.

## Parser

## Filter

Filter plugins transform the data generated by the input plugins. This transformation can be "parsing" of the data, modification of the data or filtering (excluding) data. 

A list of available filter plugins can be [found here](https://docs.fluentbit.io/manual/filter).

### Record Modifier

We can use the [Record Modifier](https://fluentbit.io/documentation/0.12/filter/record_modifier.html) filter to add brand new attributes and values to the log entry. The example below matches to any input; all entries will have the hostname and service_name added to them. The hostname record is using an environment variable to get the hostname value. Service_name is a standard field in New Relic Logs that can be used to indicate what application is generating the log data. It is always optimal to match this value up with the same application name you are using in your New Relic APM configuration.

```
[FILTER]
    Name record_modifier
    Match *
    Record hostname ${HOSTNAME}
    Record service_name Sample-App-Name
```

## Output

Once you have input log data and filtered it, you will want to send it someplace. That is what an [output plugin](https://docs.fluentbit.io/manual/output) is for; hopefully you have already installed [New Relic's output plugin for Fluent Bit](https://docs.newrelic.com/docs/logs/new-relic-logs/enable-logs/enable-new-relic-logs-fluent-bit). The example below will match on everything; when testing be careful. Once you match on an entry it will not be in the pipeline anymore; if the newrelic output plugin is after your test output no logs will be sent to New Relic.

It is recommended to use an API_KEY if rotating or changing the keys will ever be necessary; alternatively a license key can be used. maxBufferSize and maxRecords are optional and defined in the documentation. In the examples below there are references to environment variables; you can simply put the values right into the configuration as well.

```
[OUTPUT]
    Name  newrelic
    Match *
    licenseKey ${LICENSE_KEY}
    maxBufferSize ${BUFFER_SIZE}
    maxRecords ${MAX_RECORDS}
```
