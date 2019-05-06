## exercise 1: parameters, templates, tasks, and plans in action

### parameters

```yaml
parameters:
  - name: WHO
    default: "Ravi"
```

### templates

```yaml
templates:

  config.yaml: |
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: config
      namespace: {{NAMESPACE}}
    data:
      index.html: |
        Hello {{WHO}} !!!

  pod.yaml: |
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod
      namespace: {{NAMESPACE}}
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html
      volumes:
        - name: config
          configMap:
            name: {{NAME}}-config
```

### tasks

```yaml
tasks:
  deploy-task:
    resources:
      - config.yaml
      - pod.yaml
```

### plans

```yaml
plans:
  deploy:
    strategy: serial
    phases:
      - name: deploy-phase
        strategy: parallel
        steps:
          - name: deploy-setp
            tasks:
            - deploy-task
```
