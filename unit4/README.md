

```
curl -X POST "myes-node-0.myes-hs:9200/twitter/_doc/" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```


```
curl -X GET "myes-node-0.myes-hs:9200/twitter/_search?q=user:kimchy&pretty"
```

```
bin/elasticsearch-keystore add s3.client.default.access_key
bin/elasticsearch-keystore add s3.client.default.secret_key
```

```
curl -X POST "myes-node-0.myes-hs:9200/_nodes/reload_secure_settings"
```

```
curl -X PUT "myes-node-0.myes-hs:9200/_snapshot/m" -H 'Content-Type: application/json' -d'
{
  "type": "s3",
  "settings": {
    "bucket": "es_bucket",
    "client": "default",
    "endpoint": "http://http://a8667773a71ce11e988bb0ab3e78e711-57018547.us-west-2.elb.amazonaws.com:9000"
  }
}
'
```
