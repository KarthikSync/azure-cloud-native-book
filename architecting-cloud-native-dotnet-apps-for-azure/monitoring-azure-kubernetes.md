---
title: Monitoring in Azure Kubernetes Services
description: Architecting Cloud Native .NET Apps for Azure | Monitoring in Azure Kubernetes Services
ms.date: 06/30/2019
---
# Monitoring in Azure Kubernetes Services

Kubernetes is an orchestration engine which provides for running Docker images on both Linux and [Windows](https://kubernetes.io/docs/setup/windows/). It is immensely popular and is likely to remain an excellent candidate for hosting a wide variety of applications. The learning curve is quite steep but the advantages are too. 

Out of the box the logging in Kubernetes is somewhat primitive. However there are some great options for getting the logs out of Kubernetes and into a place where they can be properly analyzed. 

## Elastic Stack

The Elastic Stack is a powerful option for gathering information from a Kubernetes cluster. Support for sending logs to an Elasticsearch endpoint is built into Kubernetes and for the [most part](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/) all that is needed to get started is to set the environmental variable

```
KUBE_LOGGING_DESTINATION=elasticsearch
KUBE_ENABLE_NODE_LOGGING=true
```

This will install Elasticsearch on the cluster and target sending all the cluster logs to it. 

![An example of a Kibana dashboard showing the results of a query against logs ingested from Kubernetes](media/kibana-dashboard.png)

## Azure Container Monitoring

Azure Container Monitoring supports consuming logs from not just Kubernetes but also from other orchestration engines such as DC/OS, Docker Swarm and Red Hat OpenShift.

![Consuming logs from various containers](media/containers-diagram.png)

Log and metric information is gathered not just from the containers running in the cluster but also from the cluster hosts themselves. This allows correlating log information from the two making it much easier to track down an error. 

Installing the log collectors differs on [Windows](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/containers#configure-a-log-analytics-windows-agent-for-kubernetes) and [Linux](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/containers#configure-a-log-analytics-linux-agent-for-kubernetes)
 clusters. But in both cases the log collection is implemented as a Kubernetes [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), meaning that the log collector is actually run as a container on each of the various nodes. 

No matter which orchestrator or operating system running the Azure Monitor daemon the log information is forwared to the same Azure Monitor tools with which users are familiar. This ensures a parallel experience in environments which mix a variety of log sources such as a hybrid Kubernetes/Azure Functions environment. 

![A sample dashboard showing logging and metric information from a number of running containers.](media/containers-dashboard.png)

>[!div class="step-by-step"]
>[Previous](azure-monitor.md)
>[Next](identity.md)
