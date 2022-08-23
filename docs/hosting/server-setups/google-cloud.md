# Hosting n8n on Google Cloud

This hosting guide shows you how to self-host n8n on Google Cloud (GCP). It uses n8n with Postgres as a database backend using Kubernetes to manage the necessary resources and reverse proxy.

## Prerequisites

- [The gcloud command line tool](https://cloud.google.com/sdk/gcloud/){:target="_blank" .external-link}

## Hosting options

Google Cloud offers several ways suitable for hosting n8n, including Cloud Run (optimized for running containers), Compute Engine (VMs), and Kubernetes Engine (containers running with Kubernetes).

This guide uses the Google Kubernetes Engine (GKE) as the hosting option. Using Kubernetes requires some additional complexity and configuration, but is the best method for scaling n8n as demand changes.

Most of the steps in this guide use the Google Cloud UI, but you can also use the [gcloud command line tool](https://cloud.google.com/sdk/gcloud/){:target="_blank" .external-link} instead to undertake all the steps.

## Create project

GCP encourages you to create projects to logically organize resources and configuration. Create a new project for your n8n deployment by clicking the project dropdown menu and then the _NEW PROJECT_ button. Then select the newly created project and as you follow other steps in this guide, make sure you have the correct project selected.

## Enable the Kubernetes Engine API

GKE isn't enabled by default, search for "Kubernetes" in the top search bar and select "Kubernetes Engine" from the results.

Enable the Kubernetes Engine API by clicking the **Enable** button.

## Create a cluster

From the GKE service page, click the **Clusters** menu item and then the **CREATE** button. Make sure you select the "Standard" cluster option, n8n doesn't work with an "Autopilot" cluster.

## Login to instance

The remainder of the steps in this guide require you to login to the instance via an SSH connection. You can find the connection details for a cluster instance by opening its details page and then the **CONNECT** button. The resulting code snippet shows a connection string for the gcloud CLI tool. Paste and run that code snippet into a terminal to change your local Kubernetes settings to use the new gcloud cluster.

## Setup DNS

n8n typically operates on a subdomain. Create a DNS record with your provider for the subdomain and point it to a static IP address of the instance.

If the instance doesn't already have a static IP address, you can assign one to it by editing the instance, and changing the network interface from "Ephemeral" to "Static".

<!-- TODO: Kubernetes handles? -->
<!-- ## Open ports

To set up http connections to the instance, you need to open Firewall rules. You can do this when creating or editing an instance from the _Firewall_ section, or you can [use the gcloud CLI tool to open the ports](https://cloud.google.com/vpc/docs/firewall-rules-logging). -->

## Clone repo

The next few steps walk through the important parts of what the manifests configure.

## Configure Postgres

For larger scale n8n deployments, Postgres provides a more robust database backend than SQLite.

### Create a volume for persistent storage

To maintain data between pod restarts, the Postgres deployment needs a persistent volume. Running Postgres on GCP requires a specific Kubernetes Storage Class, [you can read this guide for specifics](https://cloud.google.com/architecture/deploying-highly-available-postgresql-with-gke){:target="_blank" .external-link}, but the `storage.yaml` manifest creates it for you. You may want to change the regions to create the storage in under the `allowedTopologies` > `matchedLabelExpressions` > `values` key. By default, they're set to "us-central".

```yaml
…
allowedTopologies:
  - matchLabelExpressions:
      - key: failure-domain.beta.kubernetes.io/zone
        values:
          - us-central1-b
          - us-central1-c
```

### Environment variables

Postgres needs some environment variables set to pass to the application running in the containers.

The example `postgres-secret.yaml` file contains placeholders you need to replace with values of your own for user details and the database to use.

The `postgres-deployment.yaml` manifest then uses the values from this manifest file to send to the application pods.

## Configure n8n

### Create a volume for file storage

During initial setup and certain workflows, n8n needs to write files to disk and needs a persistent volume to do so.

The `n8n-claim0-persistentvolumeclaim.yaml` manifest creates this, and the n8n Deployment mounts that claim in the `volumes` section of the `n8n-deployment.yaml` manifest.

```yaml
…
volumes:
  - name: n8n-claim0
    persistentVolumeClaim:
      claimName: n8n-claim0
…
```

### Environment variables

n8n needs some environment variables set to pass to the application running in the containers.

The example `n8n-secret.yaml` file contains placeholders you need to replace with values of your own for authentication details.

## Deployments

The two deployment manifests (`n8n-deployment.yaml` and `postgres-deployment.yaml`) define the n8n and Postgres applications to Kubernetes.

The manifests define the following:

- Send the environment variables defined to each application pod
- Define the container image to use
- Set resource consumption limits. This is left empty in the example manifests, but you should set them to something appropriate for your deployment.
- The `volumes` defined earlier and `volumeMounts` to define the path in the container to mount volumes.
- Scaling and restart policies. The example manifests define only one instance of each pod, you should change this to meet your needs.

## Services

The two service manifests (`postgres-service.yaml` and `n8n-service.yaml`) expose the services to the outside world using the Kubernetes load balancer using ports 5432 and 5678 respectively by default.

## Send to Kubernetes cluster

Send all the manifests to the cluster with the following command:

```shell
kubectl apply -f .
```

!!! note "Namespace error"
    You may see an error message about not finding an "n8n" namespace as that resources isn't ready yet. You can run the same command again, or apply the namespace manifest first with the following command:

    ```shell
    kubectl apply -f namespace.yaml
    ```

Remove the resources created by the manifests with the following command:

```shell
kubectl delete -f .
```
