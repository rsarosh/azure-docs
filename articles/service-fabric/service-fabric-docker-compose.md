---
title: Azure Service Fabric Docker Compose Deployment Preview
description: Azure Service Fabric accepts Docker Compose format to make it easier to orchestrate existing containers using Service Fabric. This support is currently in preview.
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''

ms.assetid: ab49c4b9-74a8-4907-b75b-8d2ee84c6d90
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/25/2017
ms.author: subramar
---
# Docker Compose deployment support in Azure Service Fabric (Preview)

Docker uses the [docker-compose.yml](https://docs.docker.com/compose) file for defining multi-container applications. To make it easy for customers familiar with Docker to orchestrate existing container applications on Azure Service Fabric, we have included preview support for Docker Compose deployment natively in the platform. Service Fabric can accept version 3 and later of `docker-compose.yml` files. 

Because this support is in preview, only a subset of Compose directives is supported. For example, application upgrades are not supported. However, you can always remove and deploy applications instead of upgrading them.

To use this preview, create your cluster with version 5.7 or greater of the Service Fabric runtime through the Azure portal along with the corresponding SDK. 

> [!NOTE]
> This feature is in preview and is not supported in production.
> The examples below are based on runtime version 6.0 and SDK version 2.8.

## Deploy a Docker Compose file on Service Fabric

The following commands create a Service Fabric application (named `fabric:/TestContainerApp`), which you can monitor and manage like any other Service Fabric application. You can use the specified application name for health queries.
Service Fabric recognizes "DeploymentName" as the identifier of the Compose deployment.

### Use PowerShell

Create a Service Fabric Compose deployment from a docker-compose.yml file by running the following command in PowerShell:

```powershell
New-ServiceFabricComposeDeployment -DeploymentName TestContainerApp -Compose docker-compose.yml [-RegistryUserName <>] [-RegistryPassword <>] [-PasswordEncrypted]
```

`RegistryUserName` and `RegistryPassword` refer to the container registry username and password. After you've completed the deployment, you can check its status by using the following command:

```powershell
Get-ServiceFabricComposeDeploymentStatus -DeploymentName TestContainerApp
```

To delete the Compose deployment through PowerShell, use the following command:

```powershell
Remove-ServiceFabricComposeDeployment  -DeploymentName TestContainerApp
```

To start a Compose deployment upgrade through PowerShell, use the following command:

```powershell
Start-ServiceFabricComposeDeploymentUpgrade -DeploymentName TestContainerApp -Compose docker-compose-v2.yml -Monitored -FailureAction Rollback
```

After upgrade is accepted, the upgrade progress could be tracked using the following command:

```powershell
Get-ServiceFabricComposeDeploymentUpgrade -Deployment TestContainerApp
```

### Use Azure Service Fabric CLI (sfctl)

Alternatively, you can use the following Service Fabric CLI command:

```azurecli
sfctl compose create --name TestContainerApp --file-path docker-compose.yml [ [ --user --encrypted-pass ] | [ --user --has-pass ] ] [ --timeout ]
```

After you've created the deployment, you can check its status by using the following command:

```azurecli
sfctl compose status --deployment-name TestContainerApp [ --timeout ]
```

To delete the compose deployment, use the following command:

```azurecli
sfctl compose remove  --deployment-name TestContainerApp [ --timeout ]
```

To start a Compose deployment upgrade, use the following command:

```powershell
sfctl compose upgrade --name TestContainerApp --file-path docker-compose-v2.yml [ [ --user --encrypted-pass ] | [ --user --has-pass ] ] [--upgrade-mode Monitored] [--failure-action Rollback] [ --timeout ]
```

After upgrade is accepted, the upgrade progress could be tracked using the following command:

```powershell
sfctl compose upgrade-status --deployment-name TestContainerApp
```

## Supported Compose directives

This preview supports a subset of the configuration options from the Compose version 3 format, including the following primitives:

* Services > Deploy > Replicas
* Services > Deploy > Placement > Constraints
* Services > Deploy > Resources > Limits
    * -cpu-shares
    * -memory
    * -memory-swap
* Services > Commands
* Services > Environment
* Services > Ports
* Services > Image
* Services > Isolation (only for Windows)
* Services > Logging > Driver
* Services > Logging > Driver > Options
* Volume & Deploy > Volume

Set up the cluster for enforcing resource limits, as described in [Service Fabric resource governance](service-fabric-resource-governance.md). All other Docker Compose directives are unsupported for this preview.

## ServiceDnsName computation

If the service name that you specify in a Compose file is a fully qualified domain name (that is, it contains a dot [.]), the DNS name registered by Service Fabric is `<ServiceName>` (including the dot). If not, each path segment in the application name becomes a domain label in the service DNS name, with the first path segment becoming the top-level domain label.

For example, if the specified application name is `fabric:/SampleApp/MyComposeApp`, `<ServiceName>.MyComposeApp.SampleApp` would be the registered DNS name.

## Differences between Compose deployment (instance definition) and Service Fabric application model (type definition)

A docker-compose.yml file describes a deployable set of containers, including their properties and configurations.
For example, the file can contain environment variables and ports. You can also specify deployment parameters, such as placement constraints, resource limits, and DNS names, in the docker-compose.yml file.

The [Service Fabric application model](service-fabric-application-model.md) uses service types and application types, where you can have many application instances of the same type. For example, you can have one application instance per customer. This type-based model supports multiple versions of the same application type that's registered with the runtime.

For example, customer A can have an application instantiated with type 1.0 of AppTypeA, and customer B can have another application instantiated with the same type and version. You define the application types in the application manifests, and you specify the application name and deployment parameters when you create the application.

Although this model offers flexibility, we are also planning to support a simpler, instance-based deployment model where types are implicit from the manifest file. In this model, each application gets its own independent manifest. We are previewing this effort by adding support for docker-compose.yml, which is an instance-based deployment format.

## Next steps

* Read up on the [Service Fabric application model](service-fabric-application-model.md)
* [Get started with Service Fabric CLI](service-fabric-cli.md)
