---
title: Linux部署ELK流程
date: 2021-10-13T15:28:13+08:00
hero: head.png
menu:
  sidebar:
    name: Linux部署ELK流程
    identifier: deployment-linux-elk
    parent: deployment
    weight: 3006
---

## Logstash

## Elasticsearch

### 开启试用

https://www.elastic.co/guide/en/elasticsearch/reference/current/start-trial.html

```json
{
  "license" : {
    "status" : "active",
    "uid" : "f9cd2e30-450e-4d54-a928-18d8f135c665",
    "type" : "trial",
    "issue_date" : "2021-10-14T05:20:55.621Z",
    "issue_date_in_millis" : 1634188855621,
    "expiry_date" : "2021-11-13T05:20:55.621Z",
    "expiry_date_in_millis" : 1636780855621,
    "max_nodes" : 1000,
    "issued_to" : "elasticsearch",
    "issuer" : "elasticsearch",
    "start_date_in_millis" : -1
  }
}
```


https://www.elastic.co/cn/subscriptions
