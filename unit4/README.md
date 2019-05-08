

## raw story

add data
```
curl -X POST "myes-node-0.myes-hs:9200/twitter/_doc/" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

search it
```
curl -X GET "myes-node-0.myes-hs:9200/twitter/_search?q=user:kimchy&pretty"
```



create backup repo
```
curl -X PUT "myes-node-0.myes-hs:9200/_snapshot/my_backup" -H 'Content-Type: application/json' -d'
{
  "type": "s3",
  "settings": {
    "bucket": "es-bucket",
    "endpoint": "a8667773a71ce11e988bb0ab3e78e711-57018547.us-west-2.elb.amazonaws.com:9000",
    "protocol": "http"
  }
}
'
```

snapshot
```
curl -X PUT "myes-node-0.myes-hs:9200/_snapshot/my_backup/snapshot_1?wait_for_completion=true"
```

look in minio

delete the index
```
curl -X DELETE "myes-node-0.myes-hs:9200/twitter"
```

restore it
```
curl -X POST "myes-node-0.myes-hs:9200/_snapshot/my_backup/snapshot_1/_restore"

```

check that the data is back
