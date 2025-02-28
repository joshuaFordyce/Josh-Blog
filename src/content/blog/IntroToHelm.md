---
title: 'Helm: Your Kubernetes Package Manager - A Friendly Intro'
description: 'In this article we take a gentle stroll in the land of Helm '
pubDate: 'February 28 2025'
category: 'Data Engineering and Data Infrastructure'
heroImage: '../../assets/images/bg.jpg'
tags: ['ML']
---


Helm: Your Kubernetes Package Manager - A Friendly Intro
Hey everyone! This month, we're diving into Helm, the package manager that makes Kubernetes deployments a whole lot easier. Think of it as the "apt" or "brew" for your Kubernetes clusters. This is going to be a great foundation before we start our Starburst Enterprise on Helm series!
Why Helm?
If you've ever wrestled with complex Kubernetes deployments, you know how much YAML configuration files you need to manage. Helm helps us tame that complexity.
The Helm Essentials: Charts, Repositories, and Releases
To understand Helm, we need to talk about three key concepts:
Helm Charts: These are like packages or recipes for your Kubernetes applications. They bundle all the YAML files and configuration you need to deploy something. Think of it as a blueprint for your application.
Helm Repositories: These are where you store and share your Helm charts. They're like app stores for Kubernetes. You can have public or private repositories, making it easy to share your work with your team or the world.
Helm Releases: When you install a Helm chart, you create a "release." This is a running instance of your application on your Kubernetes cluster. Each time you install a chart, you get a new release.
Let's Look at a Helm Chart Example
Here's a simplified example of what a Helm chart might look like:
YAML
apiVersion: v2
name: my-app
description: My simple application
type: application
version: 0.1.0
appVersion: 1.0

maintainers:
  - name: Your Name
    email: your.email@example.com

values:
  replicaCount: 1
  image:
    repository: nginx
    tag: latest
  service:
    type: LoadBalancer
    port: 80

templates:
  - deployment.yaml
  - service.yaml
The values.yaml file is where you set default configurations, and users can override them.
The templates directory holds your Kubernetes YAML files.
Helm Repositories: Sharing Your Charts
A Helm repository is simply an HTTP server that stores your charts. It has an index.yaml file that lists all the charts available. It's really useful for sharing charts within your team, or even publicly.
Getting Started: Installing Helm
Let's get Helm installed! We'll use the official installation script:
Bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
And to make sure your Kubernetes cluster is running:
Bash
kubectl cluster-info
Creating Your First Helm Chart
Let's make a "hello-world" chart:
Bash
helm create hello-world
This creates a basic chart structure:
hello-world/
  Chart.yaml
  values.yaml
  templates/
  charts/
  .helmignore
Customizing Your Deployment and Service
Now, let's edit the deployment.yaml and service.yaml files in the templates directory. These files define your Kubernetes deployment and service.
deployment.yaml manages your application's pods.
service.yaml exposes your application to the network.
Passing Values with values.yaml
The values.yaml file lets you customize your chart. For example:
YAML
replicaCount: 1
image:
  repository: "nginx"
  tag: "latest"
service:
  type: NodePort
  port: 80
Previewing Your Chart with helm template
Before installing, you can preview your chart:
Bash
helm template hello-world .
This renders your templates with the values, so you can see what will be deployed.
Installing Your Chart with helm install
Now, let's install your chart:
Bash
helm install my-release hello-world .
You can check your releases with:
Bash
helm list
Upgrading and Rolling Back
To update your chart:
Bash
helm upgrade my-release hello-world .
And to rollback to a previous version:
Bash
helm rollback my-release 1
Uninstalling Your Chart
To remove your chart:
Bash
helm uninstall my-release
Wrapping Up
Helm makes Kubernetes deployments much simpler and more manageable. Hopefully, this introduction has given you a solid foundation to start using Helm in your own projects. Stay tuned for our Starburst Enterprise on Helm series!
Thanks for reading!



