# Training prometheus stack
## Index

## Introduction to Prometheus
Para comenzar, crearemos nuestro escenario utilizando minikube.
```sh
# Minikube start
$ minikube start --network-plugin=cni --cni=calico -p prometheus
```

### Prometheus instalation
### Basic Concepts
### Prometheus configuration
### Prometheus Config File
### Monitoring Nodes with Prometheus
### Node Exporter
### Prometheus Arquitecture
### Extras

## Monitoring
### Introduction
### Client Libraries
### Pushing Metrics
### Querying
### Service Discovery
### Exporters
### Extras

## Alerting
### Introduction to Alerting
### Setting Up Alerts
### Extras

## Internals
### Prometheus Storage
### Prometheus Security
### TLS & Authentication on Prometheus Server
### Extras

## Prometheus client implementations
### Monitoring a web application - Python Flask
### Calculating Apdex score
### Monitoring a wb application - Java Spring
### Extras

## Another Use Cases
### AWS Cloudwatch Exporter
### Grafana Provisioning
### Scraping Kubernetes with Prometheus
### Consul integration with Prometheus
### AWS EC2 Auto Discovery
### Extras