# Alfresco Content Services Helm Deployment

Alfresco Content Services (ACS) is an Enterprise Content Management (ECM) system that is used for document and case management, project collaboration, web content publishing, and compliant records management.  The flexible compute, storage, and database services that Kubernetes offers make it an ideal platform for Alfresco Content Services. This helm chart presents an enterprise-grade Alfresco Content Services configuration that you can adapt to virtually any scenario with the ability to scale up, down or out, depending on your use case.

The Helm chart in this repository supports deploying the Enterprise or Community Edition of ACS.

The Enterprise configuration will deploy the following system:

![Helm Deployment Enterprise](./diagrams/helm-enterprise.png)

The Community configuration will deploy the following system:

![Helm Deployment Community](./diagrams/helm-community.png)

## Overview

Alfresco provides tested Helm charts as a "template" to accelerate deployment
and configuration for customers who want to take advantage of Kubernetes
orchestration capabilities. Please remember that as stated in our [support
policies](https://docs.alfresco.com/support/latest/policies/deployment/),
similarly to other deployment artifacts, they are not meant to be used 'as-is'
in production. Our goal is to save you time and effort deploying Alfresco
Content Services for your organization.

## Using the Helm charts

Out of the box we provide a working ACS installation by configuring a PostgreSQL
database via [Bitnami charts][bitnami-repo] and an ActiveMQ message broker with
a simple [activemq subchart][activemq-readme], both with basic authentication
and without any kind of redundancy.

[activemq-readme]: ../../helm/alfresco-content-services/charts/activemq/README.md
[bitnami-repo]: https://github.com/bitnami/charts

You typically want to disable the embedded postgres and activemq broker and
connect directly to managed instances of those services. Another typical change
would be the integration of your company-wide monitoring, logging and backup
solutions.

If you want to have additional JDBC drivers available or to extend the default
ACS functionalities, you are required to build [custom Docker
images][docker-customization], deploy them on a public/private registry and set
the appropriate values in the charts.

[docker-customization]: ../docker-compose/examples/customisation-guidelines.md

For example, you can override the ACS repository image by specifying in the
[values](../../helm/alfresco-content-services/README.md):

```yaml
repository:
  image:
    repository: registry.example.org/my-custom-alfresco-content-repository
    tag: 7.3.0
```

See the [registry authentication](registry-authentication.md) page to configure
credentials for your private registry.

Helm charts values contains secrets to be set. For deployment in sensitive
environments please see the [Security](security.md) page before proceeding with
the installation.

## Deploy

For the best results we recommend [deploying ACS to AWS EKS](./eks-deployment.md). If you have a machine with at least 16GB of memory you can also [deploy using Docker for Desktop](./docker-desktop-deployment.md).

The recommended cluster resources for the Enterprise version with the components enabled by default are:
at least 3 nodes with 12 cpu cores and 32 GB of memory in total. You can install with lower
requirements by fine tuning the [resource requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes)
available in the [values](../../helm/alfresco-content-services/values.yaml) for each component.

There are also several [examples](./examples) showing how to deploy with various configurations:

* [Deploy with AWS Services (S3, RDS and MQ)](./examples/with-aws-services.md)
* [Deploy with Intelligence Services](./examples/with-ai.md)
* [Deploy with Microsoft 365 Connector (Office Online Integration)](./examples/with-ooi.md)
* [Enable access to Search Services](./examples/search-services.mdi#enable-alfresco-search-services-external-access)
* [Enable Email Services](./examples/email-enabled.md)
* [Use a custom metadata keystore](./examples/custom-metadata-keystore.md)
* [Install ACS license as part of the deployment](./examples/alf_license.md)

## Upgrade

You can use the standard `helm upgrade acs ./alfresco/alfresco-content-services
--reuse-values` command, but make sure you read the [upgrades page](upgrades.md)
to learn about breaking changes that may have been introduced since previous
helm charts versions.

## Configure

An autogenerated list of helm values used in the chart and their default values can be found here: [Alfresco Content Services Helm Chart](./../../helm/alfresco-content-services/README.md)

Since the Alfresco Content Services chart also has local chart dependencies you can find the list of values that can be configured for these charts in their respective base folder:

* [Alfresco ActiveMQ Helm Chart](./../../helm/alfresco-content-services/charts/activemq/README.md)
* [Alfresco Search Helm Chart](./../../helm/alfresco-content-services/charts/alfresco-search/README.md)
* [Alfresco Insight Zeppelin Helm Chart](./../../helm/alfresco-content-services/charts/alfresco-search/charts/alfresco-insight-zeppelin/README.md)
* [Alfresco Sync Service Helm Chart](./../../helm/alfresco-content-services/charts/alfresco-sync-service/README.md)

## Customise

To customise the Helm deployment, for example applying AMPs, we recommend following the best practice of creating your own custom Docker image(s). The [Customisation Guide](./examples/customisation-guidelines.md) walks you through this process.

## Troubleshooting

### Lens

The easiest way to troubleshoot issues on a Kubernetes deployment is to use the [Lens](https://k8slens.dev) desktop application, which is available for Mac, Windows and Linux. Follow the [getting started guide](https://docs.k8slens.dev/v4.0.3/getting-started) to configure your environment.

![Lens Application](./diagrams/k8s-lens.png)

### Kubernetes Dashboard

Alternatively, the traditional Kubernetes dashboard can also be used. Presuming you have deployed the dashboard in the cluster you can use the following steps to explore your deployment:

1. Retrieve the service account token with the following command:

    ```bash
    kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
    ```

2. Run the kubectl proxy:

    ```bash
    kubectl proxy &
    ```

3. Open a browser and navigate to: `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login`

4. Select "Token", enter the token retrieved in step 1 and press the "Sign in" button

5. Select "alfresco" from the "Namespace" drop-down menu, click the "Pods" link and click on a pod name. To view the logs press the Menu icon in the toolbar as highlighted in the screenshot below:

    ![Kubernetes Dashboard](./diagrams/k8s-dashboard.png)

### Port-Forwarding To A Pod

This approach allows to connect to a specific application in the cluster.
See [kubernetes documentation](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster) for details.

Any component of the deployment that is not exposed via ingress rules can be accessed in this way, for example Alfresco Search, DB or individual transformers.

### Viewing Log Files Via Command Line

Log files for individual pods can also be viewed from the command line using the kubectl utility.

Retrieve the list of pods in the alfresco namespace by using the following command:

```bash
kubectl get pods -n alfresco
```

Then to retrieve the logs for a pod using the following command (replacing the pod name appropriately):

```bash
kubectl logs acs-alfresco-cs-repository-69545958df-6wzl6 -n alfresco
```

To continually follow the log file for a pod use the `-f` options as follows:

```bash
kubectl logs -f acs-alfresco-cs-repository-69545958df-6wzl6 -n alfresco
```

### Changing Log Levels

If you wish to change the log levels for a specific Java package or class in a running content-repository these can be changed via the Admin Console. However, please be aware that the changes are applied only to one content-repository node, the one from which the Admin console is launched. Use the following URL to access the log settings page: `https://<host>/alfresco/service/enterprise/admin/admin-log-settings`

To add additional repository log statements across the whole cluster use the `extraLogStatements` property via the values file as shown in the example below:

```yaml
repository:
  ...
  extraLogStatements:
    org.alfresco.repo.content.transform.TransformerDebug: debug
    org.alfresco.repo.security.authentication.identityservice: debug
```

**NOTE:** ACS deployment does not include any log aggregation tools. The logs generated by pods will be lost once the pods are terminated.

### JMX Dump

This tool allows you to download a ZIP file containing information useful for troubleshooting and supporting your system. Issue a GET request (Admin only) to: `https://<host>/alfresco/service/api/admin/jmxdump`
