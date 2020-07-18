# Simple helm chart for flask application
simple helm chart for flask. this chart set autoscaling and public url. 
### add nodeport on service configuration  `service.yaml`
```sh
apiVersion: v1
kind: Service
metadata:
  name: {{ include "simple-chart-flask.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "simple-chart-flask.name" . }}
    helm.sh/chart: {{ include "simple-chart-flask.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.port }}
      protocol: TCP
      name: {{ include "simple-chart-flask.fullname" . }}
      nodePort: {{ .Values.service.nodePort }}      # public port 
  selector:
    app.kubernetes.io/name: {{ include "simple-chart-flask.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
```

### add autoscaling config
add horizontal pod autoscaling on `deployment.yaml` for enable this feature.
```sh
--- # sparator config

kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2beta1
metadata:
  name: {{ include "simple-chart-flask.fullname" . }}
spec:
  maxReplicas: {{ .Values.autoMaxReplicas }}
  minReplicas: {{ .Values.autoMinReplicas }}
  scaleTargetRef:
    apiVersion: app/v1
    kind: Deployment
    name: {{ include "simple-chart-flask.fullname" . }}
  metrics: 
  - type: Resource
    resource:
      name: {{ .Values.resourceName }}
      targetAverageUtilization: {{ .Values.Treshold }}
```

### custom value 
```sh
replicaCount: 1     # set first replication count
autoMaxReplicas: 3  # set maximum autoscaling replica
autoMinReplicas: 1  # set minimum autoscaling replica
resourceName: cpu   # set matric resource for parameter autoscaling
Treshold: "70"      # set treshold usage resource


image:
  repository: <private repository>  # set your private repository
  tag: <tag-version>        # set tag version of repository
  pullPolicy: Always

spec: 
  containerPort: 5001   # containr port base on application

livenessProbe:
  httpGet:
    path: / # path url for health checking
readlinessProbe:
  httpGet:
    path: /


nameOverride: ""
fullnameOverride: ""

service:
  type: NodePort    # set service type
  port: 5001    # set port of service
  targetPort: 5001  # set target port of service
  nodePort: 30301   # set public port for service

```

### sample commmand to install chart
```sh
helm install --name simpleflask ./simple-chart-flask/
```

