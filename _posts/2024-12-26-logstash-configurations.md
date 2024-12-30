---
title: "Logstash Configurations file"
date: 2024-12-26 14:56:24 +0800
categories: [SIEM]
tags: [siem, logstash, ELK]
---

# Mastering Logstash Configuration for Efficient Log Processing

Logs are the lifeblood of system monitoring and troubleshooting. Imagine a scenario where a server experiences a critical failure—without logs, identifying the root cause becomes nearly impossible. Logs serve as the foundational records that enable teams to diagnose issues, optimize performance, and ensure operational stability. Whether you're dealing with application logs, firewall logs, or server events, managing them effectively is crucial. In this article, we dive deep into configuring Logstash to seamlessly receive, process, and store logs from various sources. By the end, you'll be armed with the knowledge to optimize your Logstash setup and unlock powerful log management capabilities.

---

## Setting Up Logstash Configuration

Logstash relies on the configuration file located at `/etc/logstash/conf.d/logstash.conf` to define how logs are handled. This file consists of three main sections:

- **Input:** Specifies the sources of incoming logs.
- **Filter:** Processes and transforms logs to extract meaningful information.
- **Output:** Defines where processed logs will be sent.

Let’s explore each section in detail to understand how you can streamline log processing, transform data into actionable insights, and securely store it.

---

### 1. Configuring the Input Section

The input section defines how Logstash receives logs. Below is a sample configuration:

```plaintext
input {
    beats {
        port => 5044 # Default port for receiving logs from Beats
    }
    syslog {
        port => 514 # Default port for receiving logs from syslog agents
        add_field => { "source_type" => "syslog" } # Tag logs for filtering
    }
}
```

In this setup:
- **Beats input** listens on port 5044 for logs sent by Beats agents.
- **Syslog input** listens on port 514 for logs sent by syslog agents and adds a custom field (`source_type`) for easy identification.

---

### 2. Filtering Logs with Precision

Filters are where the magic happens. They transform raw log messages into structured, queryable data by applying parsing rules, enriching the logs with additional metadata, and formatting them for downstream processing. They transform raw logs into structured data that’s easy to query. For example, consider firewall logs:

#### Raw Syslog Example

```plaintext
<NUM>date=<DATE> time=<TIME> devname="<DEVICE-NAME>" srcip=<IP> action="<ACTION>"
```

This raw format is hard to work with directly. Using the **Grok** filter, we can parse it into structured fields:

#### Filter Example

```plaintext
filter {
    if [source_type] == "syslog" {
        grok {
            patterns_dir => ["/etc/logstash/patterns"]
            match => { "message" => "<%{NUMBER:syslog_priority}>%{GREEDYDATA:firewall_message}" }
        }
        kv {
            source => "firewall_message"
            field_split => " "
            value_split => "="
        }
        mutate {
            add_field => { "@timestamp" => "%{+YYYY-MM-dd HH:mm:ss}" }
        }
    }
}
```

#### Key Highlights
- **Grok Filter:** Extracts structured fields from raw log messages. For instance, it can parse unstructured log messages into key-value pairs, making them usable for advanced queries. [Learn more about Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html).
- **KV Filter:** Splits `key=value` pairs into individual fields for easy querying. This is particularly useful for logs already structured with such pairs but lacking detailed parsing.
- **Mutate Filter:** Adds or modifies fields, such as `@timestamp` for time-based searches or adding descriptive metadata, enabling more context-aware log analysis.
- **/etc/logstash/patterns**: this is the directory of the file that contains the pattern to apply
    ```
    FORTINET_KEYWORD [a-zA-Z0-9_-]+
    FORTINET_VALUE [^ ]+
    FORTINET_PAIR %{FORTINET_KEYWORD:key}=%{FORTINET_VALUE:value}
    ```

---

### 3. Enhancing Logs with Custom Fields

To make logs more insightful, you can add custom fields based on conditions:

#### Example: Detecting Source Device

```plaintext
mutate {
    add_field => { "source_dev" => "firewall" }
}
```

Or dynamically assign a value:

```plaintext
if [host][name] =~ /^gnbr/ {
    mutate {
        add_field => { "source_dev" => "servers" }
    }
}
```

#### Example: Detecting Windows Events

```plaintext
if [event_id] in [1074, 6008, 41] {
    mutate {
        add_field => { "source_dev" => "windows_restart" }
    }
}
```

These customizations make it easier to query and analyze logs by adding context-specific metadata.

---

### 4. Sending Logs to Elasticsearch

The output section defines where processed logs are sent. Below is an example configuration for Elasticsearch:

```plaintext
output {
    elasticsearch {
        data_stream => true
        data_stream_type => "logs"
        data_stream_dataset => "beats"
        data_stream_namespace => "testenv"
        hosts => ["https://localhost:9200"]
        user => "<USERNAME>"
        password => "<PASSWORD>"
        ssl_enabled => true
        ssl_certificate_authorities => "/etc/elasticsearch/certs/http_ca.crt"
        ssl_verification_mode => "full"
    }
}
```

#### Highlights
- **Data Streams:** Organizes logs into logical datasets.
- **Secure Communication:** SSL ensures logs are transmitted securely.

---

### Conclusion

Configuring Logstash effectively can transform your log management process. For example, consider a scenario where raw syslog entries from firewalls are parsed and transformed into structured, human-readable fields. This allows system administrators to quickly identify issues, streamline debugging, and gain insights that would otherwise remain hidden in unstructured data. By carefully structuring the input, filter, and output sections, you can:
- Simplify log ingestion.
- Extract actionable insights.
- Store logs securely and efficiently.

Dive into your Logstash setup today and unlock its full potential!

