---
layout: post
title: Configuring Kube Prometheus Stack Helm Chart using Terraform to ingest metrics to AWS Managed Prometheus (AMP)
---

I have recently worked on getting the Prometheus Community - Kube Prometheus Stack [Helm Chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) to work with the [AWS Managed Prometheus Service](https://aws.amazon.com/prometheus/) (AMP).

The kube-prometheus-stack provisions many builtin [mixins](https://github.com/monitoring-mixins/docs) (*A mixin is a set of Grafana dashboards and Prometheus rules and alerts, packaged together in a reuseable and extensible bundle* )

 Particularly, the kube-prometheus-stack Chart provisions  a collection of Kubernetes manifests, Grafana dashboards, and Prometheus rules combined with documentation and scripts to provide easy to operate end-to-end Kubernetes cluster monitoring and ingesting the metrics to AMP and visualising the metrics from AMP in Grafana.
 

Please find the project source code and deployment details on my Github Repository -> [kube-prometheus-stack-helm-aws-amp](https://github.com/msharma24/kube-prom-stack-helm-aws-amp)