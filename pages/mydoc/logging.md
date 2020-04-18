# Logging
Logging if done well helps in debugging problems in production.  It helps in understanding the flow of the logic and point at right piece of code that is causing the issue. 

We have seen many codebases in the past, where there is an huge amount of unnecessary logging which becomes an operational overhead.  

We suggest thinking of the following aspects from the beginning of the project:

1. What should we log and How much?
2. Where do we want to store the logs?
3. How do you want to search the logs?

## Just Enough Logging
Most of the logging frameworks provide various log levels for different purposes.  For example, log4j provides TRACE, DEBUG, WARN, INFO, ERROR .  But in our experience, we find so many levels are not required.  Just stick on to the below levels embracing [KISS](https://en.wikipedia.org/wiki/KISS_principle)

### DEBUG
Log useful debug messages.  When switched on, it should help you with tracing the flow of the program.  

### INFO
Log useful informational messages.  These will always be 

### ERROR - 
Log errors and exceptions.  Add more contextual information to the error logs

## Best Practices
1. Never log any Secrets & PII information
2. Be aware of compliance requirements while writing to logs

## Log Formats
We strongly suggest Structured Logging so the the log analysis tools can make sense of the data that is logged.  

### Structured Logging
Structured logging is an approach to dump log data in a structured format (preferably in JSON format) so that log processing tools (like [splunk](https://www.splunk.com/), [elastic stack](https://www.elastic.co/products/)) can make better sense of it.   The following is an example of a structured log
```json
{
  “@timestamp”: “2020-04-15T11:44:45.589+05:30”,
  “@version”: “1”,
  “message”: “No active profile set, falling back to default profiles: default”,
  “logger_name”: “com.example.demo.DemoApplication”,
  “thread_name”: “main”,
  “level”: “INFO”,
  “level_value”: 20000
}
```

## Centralised Logging with Elastic Stack
We can leverage Elastic Stack for performing centralised logging, log aggregation and analysis.  It is an open source stack and we strongly recommend to incorporate it into your development process starting Iteration 0.  We will discuss Filebeat, Elastic Search and Kibana in this document.

### Filebeat
[Filebeat](https://www.elastic.co/beats/filebeat) is a lightweight shipper for logs.  Helps in transporting the log files from application to Elasticsearch.  We can use the following filebeat configuration

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    json.message_key: message
    json.keys_under_root: true
    json.add_error_key: true
    paths:
      - “/applogs/app*”
output.elasticsearch:
  hosts: [“elasticsearch:9200”]
``` 

We are configuring the logs to be in JSON format

### Elasticsearch
[Elasticsearch](https://www.elastic.co/) is a software for searching and analyzing your data in realtime.  We are storing our logs in elasticsearch database so that it can be useful for debugging purposes.

It can be accessed at [this url](http://localhost:9200/) 

### Kibana
[Kibana](https://www.elastic.co/kibana) lets you visualize your elasticsearch data.  It helps you build visuals and explore the data available in Elasticsearch.

## Structured logging in Java based applications
We strongly recommend using Slf4j API in your application for logging.  Logback is an excellent implementation that is available and is supported out-of-the-box with spring boot applications.
Implementing JSON based logs is very easy.  Create a file appender as shown: 
```xml
    <appender name=“file-appender”
              class=“ch.qos.logback.core.rolling.RollingFileAppender”>
        <file>${LOG_LOCATION}</file>
        <encoder class=“net.logstash.logback.encoder.LogstashEncoder”/>
        <rollingPolicy
                class=“ch.qos.logback.core.rolling.TimeBasedRollingPolicy”>
            rollover daily and when the file reaches 10 MegaBytes
            <fileNamePattern>logs/logs.%d{yyyy-MM-dd}.%i.log
            </fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class=“ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP”>
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>

``` 

We are using [LogstashEncoder](https://github.com/logstash/logstash-logback-encoder) for converting the log messages into JSON format.  The following is an example log entry.

```json
{
  “@timestamp”: “2020-04-15T11:44:45.589+05:30”,
  “@version”: “1”,
  “message”: “No active profile set, falling back to default profiles: default”,
  “logger_name”: “com.example.demo.DemoApplication”,
  “thread_name”: “main”,
  “level”: “INFO”,
  “level_value”: 20000
}
```

### Logging entries
Nothing special.  Just log it using standard [Slf4j](http://www.slf4j.org/) interface
```java
log.info(“logging demo!”);
```

It is possible to customise the logging and publish custom fields in the JSON.
```java
log.info(“Audit”,StructuredArguments.value(“who”, audit.getWho()), 
    StructuredArguments.value(“app”,audit.getApp()), StructuredArguments.v(“operation”, audit.getOperation()));
```

Adding custom entries using [StructuredArguments](https://github.com/logstash/logstash-logback-encoder/blob/master/src/main/java/net/logstash/logback/argument/StructuredArguments.java)  helps in adding elements to JSON file.  For more information, refer to this amazing blog [here](https://www.innoq.com/en/blog/structured-logging/)

The above code will generate a JSON like this
```json
{
  “@timestamp”: “2020-04-15T11:44:49.762+05:30”,
  “@version”: “1”,
  “message”: “Audit”,
  “logger_name”: “com.example.demo.AuditService”,
  “thread_name”: “http-nio-8080-exec-3”,
  “level”: “INFO”,
  “level_value”: 20000,
  “who”: “bob”,
  “app”: “consent-service”,
  “operation”: “USER_REGISTER”
}
```
Note that who, app & operation are custom fields added in log statement above.

### Configuring Kibana
Once filebeat starts publishing the messages to Elasticsearch, you can see a new index getting created with a name filebeat-7.6.2-xxxxxx.  You can configure this Index Pattern as mentioned [here](https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html)
Then you can start searching for the fields and interact with it in Kibana dashboard.

## Setting up logging infra in your developer machine
The following docker-compose.yaml file demonstrates setting up filebeat, elasticsearch and kibana containers.  You can see that the filebeat container is sharing the volume with the application (demo in this example) so that the filbeat container can share the log and send it to Elasticsearch for indexing.

```yaml
—
version: ‘3.5’
services:
  filebeat:
    image: docker.elastic.co/beats/filebeat:7.6.2
    container_name: filebeat
    hostname: filebeat
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml
      - demo_logs:/tmp1

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.6.2
    container_name: elasticsearch
    hostname: elasticsearch
    ports:
      - “9200:9200”
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - “ES_JAVA_OPTS=-Xms1g -Xmx1g”

  kibana:
    image: docker.elastic.co/kibana/kibana:7.6.2
    container_name: kibana
    hostname: kibana
    ports:
      - “5601:5601”
    depends_on:
      - elasticsearch
  demo:
    image: bharatak/demo
    container_name: demo
    hostname: demo
    ports:
      - “8081:8080”
    volumes:
    - demo_logs:/tmp1
    environment:
      - LOG_LOCATION=/tmp1/audit.log
      - TEST_A_B=hi1234

volumes:
  demo_logs:
    driver: local
networks:
  default:
    name: eka_network


```



 