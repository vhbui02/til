# ArgoCD Learning

## Introduction

- GitOps tool to manage repetitive configs across clusters/microservices/environments.
- Before: 10 microservices, 4 K8s clusters, 3 envs (dev, test, prod) => 120 app manifests
- Now: fit into a single template blueprint

## Generator

- Data gathering engine for ApplicationSet
- Produce a set of KV pairs that are injected into application template to create the final ArgoCD Application.

Most common types of ArgoCD generator are List, Cluster, Git, Matrix.

For Git generator, each file found will correspond to a cluster.

## Standard ArgoCD Application vs ApplicationSet

Standard ArgoCD Application:

- Relationship: 1-to-1 (connect a single Git source to a single destination namespace/cluster)
- Management: manual, create a new YAML file for every new app/env/cluster
- DRY Principle: low, repetitive boilerplate code

ArgoCD ApplicationSet:

- Relationship: 1-to-Many (dynamically generate ArgoCD Applications)
- Management: automated, changes are reflected automatically
- DRY Principle: high, utilizes templating and parameters

