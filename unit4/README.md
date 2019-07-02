## unit 4: custom plans

This unit builds on `unit 2` showcasing custom backup and restore plans using `elasticsearch` [snapshot and restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html). There are various repository plugins for elasticsearch snapshot restore, in this unit we are using the [repository-s3 plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/7.0/repository-s3.html). The unit requires an S3 compatible store, we use `MinIO`. The `MinIO operator` install you can find [here](https://github.com/minio/minio-operator).


### [params.yaml](operator/params.yaml)

The sample has four additional configuration parameters. The `S3_ACCESS_KEY`, `S3_SECRET_KEY`, and `S3_ENDPOINT` the defaults are set for the `MinIO` instance. There is also the `RESTORE_SNAPSHOT_ID` that we will have to set before running the restore plan.

```yaml
NODE_COUNT:
  default: "3"
S3_ACCESS_KEY:
  default: "minio"
S3_SECRET_KEY:
  default: "minio123"
S3_ENDPOINT:
  default: "minio:9000"
RESTORE_SNAPSHOT_ID:
  default: ""
```

### [templates](operator/templates)

The `container` section in the `node.yaml` template now has an explicit command setting that we need to install the `repository-s3` plugin, and to add `ACCESS_KEY` and `SECRET_KEY` to the `elasticsearch-keystore`.

[node.yaml](operator/node.yaml)
```yaml
...
containers:
  - name: elastic
    image: elasticsearch:7.0.0
    command:
      - sh
      - -c
      - |
        /usr/share/elasticsearch/bin/elasticsearch-plugin install repository-s3 -b;
        /usr/share/elasticsearch/bin/elasticsearch-keystore create
        echo {{ .Params.S3_ACCESS_KEY }} | /usr/share/elasticsearch/bin/elasticsearch-keystore add --stdin s3.client.default.access_key;
        echo {{ .Params.S3_SECRET_KEY }} | /usr/share/elasticsearch/bin/elasticsearch-keystore add --stdin s3.client.default.secret_key;
        /usr/local/bin/docker-entrypoint.sh eswrapper
...        
```


There are two additional resource templates in the framework implementation, they are both of type `Job`.

The `backup` template does the elasticsearch `snapshot`.

[backup.yaml](operator/backup.yaml)
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .PlanName }}-job
  namespace: default
spec:
  template:
    metadata:
      name: {{ .PlanName }}-job
    spec:
      restartPolicy: OnFailure
      containers:
        - name: backup
          image: centos:7
          command:
            - sh
            - -c
            - |
              curl -X PUT "myes-node-0.myes-hs:9200/_snapshot/my_s3_repository" -H 'Content-Type: application/json' -d'
              {
               "type": "s3",
               "settings": {
                 "bucket": "es-bucket",
                 "endpoint": "{{ .Params.S3_ENDPOINT }}",
                 "protocol": "http"
               }
              }
              ';
              TS=$(date +%s);
              curl -X PUT "myes-node-0.myes-hs:9200/_snapshot/my_s3_repository/snapshot_$TS?wait_for_completion=true&pretty"
```

The `restore` template does the elasticsearch `restore`.

[restore.yaml](operator/restore.yaml)
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .PlanName }}-job
  namespace: default
spec:
  template:
    metadata:
      name: {{ .PlanName }}-job
    spec:
      restartPolicy: OnFailure
      containers:
        - name: restore
          image: centos:7
          command:
            - sh
            - -c
            - |
              curl -X POST "myes-node-0.myes-hs:9200/_snapshot/my_s3_repository/{{ .Params.RESTORE_SNAPSHOT_ID }}/_restore?pretty"
```

### [operator.yaml](operator/operator.yaml)

#### tasks

The `backup-task` applies the backup template, and the `restore-task` applies the restore template.

```yaml
tasks:
  ...
  backup-task:
    resources:
      - backup.yaml
  restore-task:
    resources:
      - restore.yaml
```

#### plans

This sample has two custom plans. The `backup` and a `restore` plan.

```yaml
backup:
  strategy: serial
  phases:
    - name: backup-phase
      strategy: serial
      steps:
        - name: backup-step
          tasks:
            - backup-task
restore:
  strategy: serial
  phases:
    - name: restore-phase
      strategy: serial
      steps:
        - name: restore-step
          tasks:
            - restore-task
```


### Run the framework instance

If you haven't already then clone the `kudo-tutorial` repository.

```
git clone https://github.com/realmbgl/kudo-tutorial.git
```

From the `unit4/operator` folder use the following command to run the instance.

```
kubectl kudo install . --instance myes
```


### Add some data to the elasticsearch cluster

Exec into one of the POD's.

```
kubectl exec -ti myes-node-2 bash
```

Lets add some data.

```
curl -X POST "myes-node-0.myes-hs:9200/twitter/_doc/" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

Lets search for it.

```
curl -X GET "myes-node-0.myes-hs:9200/twitter/_search?q=user:kimchy&pretty"
```

You should see the following output.

```
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "twitter",
        "_type" : "_doc",
        "_id" : "n18aemoBCj0qv5VrMWv2",
        "_score" : 0.2876821,
        "_source" : {
          "user" : "kimchy",
          "post_date" : "2009-11-15T14:12:12",
          "message" : "trying out Elasticsearch"
        }
      }
    ]
  }
}
```


### Run the backup plan

Before doing this the first time you need to go to your `MinIO` instance and create a bucket named `es-bucket` with policy `Read and Write`. You can do this via the `MinIO` console.

Enable localhost access to the `MinIO` service.

```
kubectl port-forward service/minio 9000
```

[Click](http://localhost:9000/) to access the `MinIO console` from your browser.

Next apply the backup plan from the `unit4` folder.

```
kubectl apply -f ../backup-restore/backup.yaml
```

Check the logs of the backup job for the id of the snapshot that got created. Should look like follows, so the id for this backup is `snapshot_1557432189`.

```
{
  "snapshot" : {
    "snapshot" : "snapshot_1557432189",
    "uuid" : "3klj2YoBToiGmskPsKukuA",
    "version_id" : 7000099,
    "version" : "7.0.0",
    "indices" : [
      "twitter"
    ],
    "include_global_state" : true,
    "state" : "SUCCESS",
    "start_time" : "2019-05-09T20:03:09.810Z",
    "start_time_in_millis" : 1557432189810,
    "end_time" : "2019-05-09T20:03:10.053Z",
    "end_time_in_millis" : 1557432190053,
    "duration_in_millis" : 243,
    "failures" : [ ],
    "shards" : {
      "total" : 1,
      "failed" : 0,
      "successful" : 1
    }
  }
}
```


### Run the restore plan

Before doing this lets delete the data that we stored earlier.

Exec into one of the POD's.

```
kubectl exec -ti myes-node-2 bash
```

From there we delete the whole index.

```
curl -X DELETE "myes-node-0.myes-hs:9200/twitter"
```

Back to the `unit4/operator` folder.

Since you can't pass input at the moment when applying a PlanExecution, we need to update the instance configuration with the snapshot id to use for restore.

```
kubectl patch instance myes -p '{"spec":{"parameters":{"RESTORE_SNAPSHOT_ID":"snapshot_1562101091"}}}' --type=merge
```

Next apply the restore plan.

```
kubectl apply -f ../backup-restore/restore.yaml
```

Lets check that the data is back.

Exec into one of the POD's.

```
kubectl exec -ti myes-node-2 bash
```

Lets search for the data. You should see the same JSON document that we saw earlier.

```
curl -X GET "myes-node-0.myes-hs:9200/twitter/_search?q=user:kimchy&pretty"
```
