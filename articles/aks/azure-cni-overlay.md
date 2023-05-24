---
title: Configure Azure CNI Overlay networking in Azure Kubernetes Service (AKS)
description: Learn how to configure Azure CNI Overlay networking in Azure Kubernetes Service (AKS), including deploying an AKS cluster into an existing virtual network and subnet.
author: asudbring
ms.author: allensu
ms.subservice: aks-networking
ms.topic: how-to
ms.custom: references_regions
ms.date: 05/10/2023
---

# Configure Azure CNI Overlay networking in Azure Kubernetes Service (AKS)

The traditional [Azure Container Networking Interface (CNI)](./configure-azure-cni.md) assigns a VNet IP address to every Pod, either from a pre-reserved set of IPs on every node, or from a separate subnet reserved for pods. This approach requires planning IP addresses and could lead to address exhaustion, which introduces difficulties scaling your clusters as your application demands grow.

With Azure CNI Overlay, the cluster nodes are deployed into an Azure Virtual Network (VNet) subnet, whereas pods are assigned IP addresses from a private CIDR logically different from the VNet hosting the nodes. Pod and node traffic within the cluster use an Overlay network, and Network Address Translation (using the node's IP address) is used to reach resources outside the cluster. This solution saves a significant amount of VNet IP addresses and enables you to seamlessly scale your cluster to very large sizes. An added advantage is that the private CIDR can be reused in different AKS clusters, truly extending the IP space available for containerized applications in AKS.

## Overview of Overlay networking

In Overlay networking, only the Kubernetes cluster nodes are assigned IPs from a subnet. Pods receive IPs from a private CIDR that is provided at the time of cluster creation. Each node is assigned a `/24` address space carved out from the same CIDR. Additional nodes that are created when you scale out a cluster automatically receive `/24` address spaces from the same CIDR. Azure CNI assigns IPs to pods from this `/24` space.

A separate routing domain is created in the Azure Networking stack for the pod's private CIDR space, which creates an Overlay network for direct communication between pods. There is no need to provision custom routes on the cluster subnet or use an encapsulation method to tunnel traffic between pods. This provides connectivity performance between pods on par with VMs in a VNet.

:::image type="content" source="media/azure-cni-Overlay/azure-cni-overlay.png" alt-text="A diagram showing two nodes with three pods each running in an Overlay network. Pod traffic to endpoints outside the cluster is routed via NAT.":::

Communication with endpoints outside the cluster, such as on-premises and peered VNets, happens using the node IP through Network Address Translation. Azure CNI translates the source IP (Overlay IP of the pod) of the traffic to the primary IP address of the VM, which enables the Azure Networking stack to route the traffic to the destination. Endpoints outside the cluster can't connect to a pod directly. You will have to publish the pod's application as a Kubernetes Load Balancer service to make it reachable on the VNet.

Outbound (egress) connectivity to the internet for Overlay pods can be provided using a [Standard SKU Load Balancer](./egress-outboundtype.md#outbound-type-of-loadbalancer) or [Managed NAT Gateway](./nat-gateway.md). You can also control egress traffic by directing it to a firewall using [User Defined Routes on the cluster subnet](./egress-outboundtype.md#outbound-type-of-userdefinedrouting).

Ingress connectivity to the cluster can be achieved using an ingress controller such as Nginx or [HTTP application routing](./http-application-routing.md).

## Difference between Kubenet and Azure CNI Overlay

Like Azure CNI Overlay, Kubenet assigns IP addresses to pods from an address space logically different from the VNet but has scaling and other limitations. The below table provides a detailed comparison between Kubenet and Azure CNI Overlay. If you do not want to assign VNet IP addresses to pods due to IP shortage, then Azure CNI Overlay is the recommended solution.

| Area                         | Azure CNI Overlay                                                | Kubenet                                                                       |
|------------------------------|------------------------------------------------------------------|-------------------------------------------------------------------------------|
| Cluster scale                | 1000 nodes and 250 pods/node                                     | 400 nodes and 250 pods/node                                                   |
| Network configuration        | Simple - no additional configuration required for pod networking | Complex - requires route tables and UDRs on cluster subnet for pod networking |
| Pod connectivity performance | Performance on par with VMs in a VNet                            | Additional hop adds minor latency                                             |
| Kubernetes Network Policies  | Azure Network Policies, Calico, Cilium                           | Calico                                                                        |
| OS platforms supported       | Linux and Windows Server 2022(Preview)                           | Linux only                                                                    |

## IP address planning

- **Cluster Nodes**: When setting up your AKS cluster, make sure your VNet subnet has enough room to grow for future scaling. Keep in mind that clusters can't scale across subnets, but you can always add new node pools in another subnet within the same VNet for extra space. Note that a `/24`subnet can fit up to 251 nodes since the first three IP addresses are reserved for management tasks.

- **Pods**: The Overlay solution assigns a `/24` address space for pods on every node from the private CIDR that you specify during cluster creation. The `/24` size is fixed and can't be increased or decreased. You can run up to 250 pods on a node. When planning the pod address space, ensure that the private CIDR is large enough to provide `/24` address spaces for new nodes to support future cluster expansion.

The following are additional factors to consider when planning pods IP address space:

   - Pod CIDR space must not overlap with the cluster subnet range.
   - Pod CIDR space must not overlap with IP ranges used in on-premises networks and peered networks.
   - The same pod CIDR space can be used on multiple independent AKS clusters in the same VNet.

- **Kubernetes service address range**: The size of the service address CIDR depends on the number of cluster services you plan to create. It must be smaller than `/12`. This range should also not overlap with the pod CIDR range, cluster subnet range, and IP range used in peered VNets and on-premises networks.

- **Kubernetes DNS service IP address**: This is an IP address within the Kubernetes service address range that's used by cluster service discovery. Don't use the first IP address in your address range, as this address is used for the `kubernetes.default.svc.cluster.local` address.

## Network security groups

Pod to pod traffic with Azure CNI Overlay is not encapsulated and subnet [network security group][nsg] rules are applied. If the subnet NSG contains deny rules that would impact the pod CIDR traffic, make sure the following rules are in place to ensure proper cluster functionality (in addition to all [AKS egress requirements][aks-egress]):

- Traffic from the node CIDR to the node CIDR on all ports and protocols
- Traffic from the node CIDR to the pod CIDR on all ports and protocols (required for service traffic routing)
- Traffic from the pod CIDR to the pod CIDR on all ports and protocols (required for pod to pod and pod to service traffic, including DNS)

Traffic from a pod to any destination outside of the pod CIDR block will utilize SNAT to set the source IP to the IP of the node where the pod is running.

If you wish to restrict traffic between workloads in the cluster, [network policies][aks-network-policies] are the recommended solution.

## Maximum pods per node

You can configure the maximum number of pods per node at the time of cluster creation or when you add a new node pool. The default for Azure CNI Overlay is 30. The maximum value that you can specify in Azure CNI Overlay is 250, and the minimum value is 10. The maximum pods per node value configured during creation of a node pool applies to the nodes in that node pool only.

## Choosing a network model to use

Azure CNI offers two IP addressing options for pods - the traditional configuration that assigns VNet IPs to pods, and Overlay networking. The choice of which option to use for your AKS cluster is a balance between flexibility and advanced configuration needs. The following considerations help outline when each network model may be the most appropriate.

Use Overlay networking when:

- You would like to scale to a large number of pods, but have limited IP address space in your VNet.
- Most of the pod communication is within the cluster.
- You don't need advanced AKS features, such as virtual nodes.

Use the traditional VNet option when:

- You have available IP address space.
- Most of the pod communication is to resources outside of the cluster.
- Resources outside the cluster need to reach pods directly.
- You need AKS advanced features, such as virtual nodes.

## Limitations with Azure CNI Overlay

Azure CNI Overlay has the following limitations:

- You can't use Application Gateway as an Ingress Controller (AGIC) for an Overlay cluster.
- Windows support is still in Preview
    - Windows Server 2019 node pools are **not** supported for Overlay
    - Traffic from host network pods is not able to reach Windows Overlay pods.
- Virtual Machine Availability Sets (VMAS) are not supported for Overlay
- Dualstack networking is not supported in Overlay
- You can't use [DCsv2-series](/azure/virtual-machines/dcv2-series) virtual machines in node pools. To meet Confidential Computing requirements, consider using [DCasv5 or DCadsv5-series confidential VMs](/azure/virtual-machines/dcasv5-dcadsv5-series) instead.

## Set up Overlay clusters

>[!NOTE]
> You must have CLI version 2.47.0 or later to use the `--network-plugin-mode` argument. For Windows, you must have the latest aks-preview Azure CLI extension installed and can follow the instructions below.

Create a cluster with Azure CNI Overlay. Use the argument `--network-plugin-mode` to specify that this is an overlay cluster. If the pod CIDR is not specified then AKS assigns a default space, viz. 10.244.0.0/16. Replace the values for the variables `clusterName`, `resourceGroup`, and `location`.

```azurecli-interactive
clusterName="myOverlayCluster"
resourceGroup="myResourceGroup"
location="westcentralus"
az aks create -n $clusterName -g $resourceGroup --location $location --network-plugin azure --network-plugin-mode overlay --pod-cidr 192.168.0.0/16
```

## Install the aks-preview Azure CLI extension - Windows only

[!INCLUDE [preview features callout](includes/preview/preview-callout.md)]

To install the aks-preview extension, run the following command:

```azurecli
az extension add --name aks-preview
```

Run the following command to update to the latest version of the extension released:

```azurecli
az extension update --name aks-preview
```

## Register the 'AzureOverlayPreview' feature flag

Register the `AzureOverlayPreview` feature flag by using the [az feature register][az-feature-register] command, as shown in the following example:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "AzureOverlayPreview"
```

It takes a few minutes for the status to show *Registered*. Verify the registration status by using the [az feature show][az-feature-show] command:

```azurecli-interactive
az feature show --namespace "Microsoft.ContainerService" --name "AzureOverlayPreview"
```

When the status reflects *Registered*, refresh the registration of the *Microsoft.ContainerService* resource provider by using the [az provider register][az-provider-register] command:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## Upgrade an existing cluster to CNI Overlay (Preview)

> [!NOTE]
> The upgrade capability is still in preview and requires the preview AKS Azure CLI extension.
You can update an existing Azure CNI cluster to Overlay if the cluster meets certain criteria. A cluster must:
> - be on Kubernetes version 1.22+
> - **not** be using the dynamic pod IP allocation feature
> - **not** have network policies enabled
> - **not** be using any Windows node pools with docker as the container runtime

The upgrade process will trigger each node pool to be re-imaged simultaneously (i.e. upgrading each node pool separately to Overlay is not supported). Any disruptions to cluster networking will be similar to a node image upgrade or Kubernetes version upgrade where each node in a node pool is re-imaged.

To update and existing Azure CNI cluster to use overlay, run the following CLI command:

```azurecli-interactive
clusterName="myOverlayCluster"
resourceGroup="myResourceGroup"
location="westcentralus"
az aks update --name $clusterName \
--group $resourceGroup \
--network-plugin-mode overlay \
--pod-cidr 192.168.0.0/16
```

The `--pod-cidr` parameter is required when upgrading from legacy CNI because the pods will need to get IPs from a new overlay space which does not overlap with the existing node subnet. The pod CIDR also cannot overlap with any VNet address of the node pools. For example if your VNet address is 10.0.0.0/8 and your nodes are in the subnet 10.240.0.0/16, then the `--pod-cidr` cannot overlap with 10.0.0.0/8 or the existing service CIDR on the cluster.

> [!WARNING] 
> Prior to Windows OS Build 20348.1668, there was a limitation around Windows Overlay pods incorrectly SNATing packets from host network pods, this had a more detrimental effect for clusters upgrading to Overlay. To avoid this issue, **use Windows OS Build 20348.1668**. 


## Next steps

To learn how to utilize AKS with your own Container Network Interface (CNI) plugin, see [Bring your own Container Network Interface (CNI) plugin](use-byo-cni.md).

<!-- LINKS - internal -->
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-show]: /cli/azure/feature#az-feature-show
[aks-egress]: limit-egress-traffic.md
[aks-network-policies]: use-network-policies.md
[nsg]: ../virtual-network/network-security-groups-overview.md
