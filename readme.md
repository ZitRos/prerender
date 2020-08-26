A containerized, production-ready [prerender.io](https://github.com/prerender/prerender). Exposes prerender on port `80`.

**Usage example within Kubernetes**:

+ GET `http://prerender/render?url=https%3A%2F%2Fwww.google.com`
   + response: `<plain html>`
+ POST `http://prerender/render`
   + --data `{ "url":"https://www.google.com", "javascript":"window.prerenderData = window.localStorage", "renderType":"har" }`
   + response: `{ "prerenderData": { "aaa": "[[],{\"q\":\"xxx\"}]", content: {"...": "..."} }`

See the full prerender's API [here](https://github.com/prerender/prerender#prerendercom).

**Kubernetes config example**:

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prerender
  labels:
    app: prerender
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prerender
  template:
    metadata:
      labels:
        app: prerender
    spec:
      containers:
      - name: prerender
        image: zitros/prerender:latest
        resources:
          limits:
            memory: 512Mi
            cpu: 500m
          requests:
            memory: 128Mi
            cpu: 100m
        ports:
        - containerPort: 80
          protocol: TCP
        livenessProbe:
          initialDelaySeconds: 30
          timeoutSeconds: 7
          periodSeconds: 60
          httpGet:
            path: /render?url=https%3A%2F%2Fwww.google.com%2Frobots.txt
            port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: prerender
  labels:
    app: prerender
spec: 
  type: NodePort
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
  selector:
    app: prerender
```