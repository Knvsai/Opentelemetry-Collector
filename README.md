# Opentelemetry-Collector

1. Create a new Kubernetes namespace to isolate the OpenTelemetry Collector deployment:
kubectl create namespace otel

2. Create a YAML file named otel-collector-config.yaml to define the configuration for the OpenTelemetry Collector:
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
  namespace: otel
data:
  config.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
      zipkin:
        endpoint: 0.0.0.0:9411
    exporters:
      logging:
      otlp:
        endpoint: YOUR_EXPORTER_ENDPOINT
    processors:
      batch:
    extensions:
      health_check:
      zpages:
    service:
      extensions: [health_check, zpages]
      pipelines:
        traces:
          receivers: [otlp, zipkin]
          processors: [batch]
          exporters: [logging, otlp]
# Replace YOUR_EXPORTER_ENDPOINT with the actual endpoint where you want to send your telemetry data

3. Create the ConfigMap to store the OpenTelemetry Collector configuration:
kubectl create -f otel-collector-config.yaml

4. Create a YAML file named otel-collector-deployment.yaml to define the deployment for the OpenTelemetry Collector:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
  namespace: otel
spec:
  replicas: 1
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      containers:
        - name: otel-collector
          image: otel/opentelemetry-collector:latest
          volumeMounts:
            - name: config
              mountPath: /conf
              readOnly: true
          args: ["--config=/conf/config.yaml"]
      volumes:
        - name: config
          configMap:
            name: otel-collector-config

5. Deploy the OpenTelemetry Collector:
kubectl create -f otel-collector-deployment.yaml

6. Verify that the deployment is running:
kubectl get pods -n otel

