---
title: "SIEM problems"
date: 2024-10-17 16:56:24 +0800
categories: [problems]
tags: [siem problems]
---

# Problems I Faced in Installing ELK Stack on CentOS 9

## General Configurations
We use `/etc/profile` for permanent storage of environmental variables working after a restart.

Selinux file config: `/etc/sysconfig/selinux`

## Firewall Commands
```bash
sudo firewall-cmd --permanent --add-port=514/udp
sudo firewall-cmd --permanent --add-port=514/tcp
sudo firewall-cmd --reload
```

## ElasticSearch
The problem that lasted for about 2 days was related to certificates. After installing Elasticsearch on CentOS 9, I got this error when visiting [https://localhost:9200](https://localhost:9200):

```json
{
  "error": {
    "root_cause": [
      {
        "type": "status_exception",
        "reason": "Cluster state has not been recovered yet, cannot write to the [null] index"
      }
    ],
    "type": "authentication_processing_error",
    "reason": "failed to promote the auto-configured elastic password hash",
    "caused_by": {
      "type": "status_exception",
      "reason": "Cluster state has not been recovered yet, cannot write to the [null] index"
    }
  },
  "status": 503
}
```

The solution was changing the following parameter:

```yaml
cluster.initial_master_nodes: ["localhost.localdomain"]
```

## Kibana
I faced issues when trying to connect Kibana to Elasticsearch using a token. The error I received was due to a mismatch between the certificate's alternative names and the server IP. The solution was to manually configure the network host:

```yaml
network.publish_host: 127.0.0.1
```

## Logstash
Running Logstash as a service resulted in certificate errors with Elasticsearch. I resolved this by running Logstash from its own `bin` directory and manually copying configuration files.

```bash
mkdir /usr/share/logstash/config
cp /etc/logstash/pipelines.yml /usr/share/logstash/config
cp /etc/logstash/log4j2.properties /usr/share/logstash/config
/usr/share/logstash/bin/logstash
```

### Key Notes:
1. **Headings**: In Markdown, `#` is used for headings. One `#` is for `h1`, two `##` for `h2`, and so on.
2. **Code Blocks**: Use triple backticks (```) to create code blocks for commands, JSON, YAML, etc.
3. **Links Between Pages**: You can link to other pages like `index.md`, `about.md`, and `projects.md` using the `[text](link)` format.
4. **Footer and Navigation**: Markdown doesn’t support advanced HTML-like navigation bars, but you can add simple links for navigation between pages at the bottom.

### Steps to Implement:
1. **Create a New Markdown File**: Save this code as `problems-elk.md`.
   
2. **Update Links in Other Pages**: If your portfolio uses Markdown files (e.g., `index.md`, `about.md`), update the links to include your new page:
   ```md
   [Problems I Faced](problems-elk.md)
   ```

3. **Push Changes to GitHub**: Commit the `problems-elk.md` file to your GitHub Pages repository and update the links in your `README.md` or other Markdown files.

Let me know if you need further guidance on setting up your portfolio!