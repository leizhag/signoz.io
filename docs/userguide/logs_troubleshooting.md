---
id: logs_troubleshooting
title: Troubleshooting
description: Instructions that should resolve most issues with logs
---
This troubleshooting guide includes step-by-step instructions that should resolve most issues with logs.


## K8s Attribute Filtering Issue in Logs

In the SigNoz charts releases `v0.9.1`, `v0.10.0` and `0.10.1`, some users who are facing
issues querying the following selected fields.

- `k8s_container_name`
- `k8s_namespace_name`
- `k8s_pod_name`
- `k8s_container_restart_count`
- `k8s_pod_uid`


If you have included any of the above to `selected` fields, and you get empty data
when you filter using those fields then you will have to perform the following steps.

- Exec into your clickhouse container
    ```bash
    kubectl exec -n platform -it chi-my-release-clickhouse-cluster-0-0-0 -- sh

    clickhouse client
    ```
- Run the following queries
    
    ```
    use signoz_logs;
    
    show create table logs;
    ```
    
- For the corresponding field, you will find a materialized column and an index.
For example: `k8s_namespace_name` you will have `k8s_namespace_name String
MATERIALIZED attributes_string_value[indexOf(attributes_string_key,
'k8s_namespace_name')] CODEC(LZ4)` and index `INDEX k8s_namespace_name_idx
k8s_namespace_name TYPE bloom_filter(0.01) GRANULARITY 64`
- You will have to delete the index and remove the materialized column
    
    ```
    alter table logs drop column k8s_namespace_name;
    alter table logs drop index k8s_namespace_name_idx;
    ```
    
- Perform the above steps for all the k8s fields listed.
- Once done truncate the attribute keys table
    
    ```
    truncate table logs_atrribute_keys;
    ```
    
- Now you can go back to the UI and convert them back to the selected field, and filters will work as expected.