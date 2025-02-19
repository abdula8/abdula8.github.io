# Managing ILM Policies in ELK Stack: Retention and Deletion of Data Streams

## Introduction
Managing log retention is crucial in an **ELK Stack (Elasticsearch, Logstash, Kibana)** setup, especially when dealing with large volumes of data. In this post, I will walk through how to create an **Index Lifecycle Management (ILM) policy** to automatically delete data after 7 days or when reaching a size of 25GB.

Additionally, I will discuss **issues encountered with built-in policies**, how to override them, and how to apply ILM policies to **existing indices and data streams**.

---

## Step 1: Creating an ILM Policy to Delete Data After 7 Days or 25GB

To ensure indices do not grow indefinitely, we will create an **ILM policy** that:
- **Rolls over** indices when they reach 25GB.
- **Deletes** indices that are older than 7 days.

### **Create the ILM Policy**
Run the following command in Kibana Dev Tools or via `curl`:

```json
PUT _ilm/policy/my_custom_ilm
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "25GB",
            "max_age": "7d"
          }
        }
      },
      "delete": {
        "min_age": "7d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

## Step 2: Applying the ILM Policy to an Index Template
To make sure **new indices** follow this policy, attach it to an **index template**:

```json
PUT _index_template/packetbeat_template
{
  "index_patterns": ["packetbeat-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "my_custom_ilm"
    }
  }
}
```

This ensures all future `packetbeat-*` indices use the ILM policy automatically.

---

## Step 3: Applying ILM to Existing Indices
If you already have indices that were created **before** this policy, they will still use the old built-in ILM settings. You need to manually assign them to `my_custom_ilm`:

```json
PUT packetbeat-*/_settings
{
  "index.lifecycle.name": "my_custom_ilm"
}
```

To verify the changes, run:

```json
GET packetbeat-*/_ilm/explain?human=true&pretty=true
```

This should show `"policy": "my_custom_ilm"` for your indices.

---

## Step 4: Applying ILM Policy to All Data Streams
If you have multiple data streams (e.g., logs, metrics), apply the ILM policy across all of them:

### **List All Data Streams**
```json
GET _data_stream/*
```

### **Apply ILM Policy to All Data Streams**
```json
PUT _index_template/all_data_streams
{
  "index_patterns": ["*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "my_custom_ilm"
    }
  }
}
```

### **Apply to Existing Indices**
```json
PUT */_settings
{
  "index.lifecycle.name": "my_custom_ilm"
}
```

This will ensure **both old and new indices** follow the ILM policy.

---

## Step 5: Fixing ILM Conflicts with Existing Policies
### **Error: ILM Policy Conflicts with Built-In Templates**
If you encounter an error like this:

```json
{
  "error": {
    "reason": "index template [logs*] has index patterns [logs-*] matching existing templates with the same priority."
  }
}
```

It means **two templates** are trying to manage the same indices with the **same priority**.

### **Solution: Adjust the Template Priority**
Increase the priority of your ILM policy template:

```json
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "priority": 100,
  "template": {
    "settings": {
      "index.lifecycle.name": "my_custom_ilm"
    }
  }
}
```

### **Reapply ILM to Existing Logs Indices**
```json
PUT logs-*/_settings
{
  "index.lifecycle.name": "my_custom_ilm"
}
```

---

## **Final Verification**
After completing all steps, confirm that ILM is working correctly:

```json
GET */_ilm/explain?human=true&pretty=true
```

Look for:
- `"policy": "my_custom_ilm"` → Ensures the new policy is applied.
- `"phase": "hot"` or `"phase": "delete"` → Confirms ILM is active.

---

## **Conclusion**
By implementing this ILM policy:
- **New indices automatically follow the deletion policy**.
- **Old indices are updated to use the new ILM policy**.
- **Storage is efficiently managed by deleting indices older than 7 days or exceeding 25GB**.
- **Conflicts with existing templates are resolved using priority adjustments**.

This ensures a **well-optimized ELK Stack**, preventing unnecessary storage consumption while keeping logs available for analysis.
