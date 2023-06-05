---
title: Access connection telemetry
titleSuffix: Azure Peering Service
description: Learn how to access Azure Peering Service connection telemetry.
services: peering-service
author: halkazwini
ms.service: peering-service
ms.topic: how-to
ms.date: 05/31/2023
ms.author: halkazwini
ms.custom: template-how-to, engagement-fy23
# Customer intent: Customer wants to access their connection telemetry per prefix to Microsoft services with Azure Peering Service.
---

# Access Peering Service connection telemetry
 
Connection telemetry provides insights collected for the connectivity between the customer's location and the Microsoft network. In this article, you learn how to access your Peering Service connection telemetry to view the latency report and prefix states for the Peering Service connection. 

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- A Peering Service connection. To learn how to create a connection, see [Create, change, or delete a Peering Service connection using the Azure portal](azure-portal.md).

## View a latency report

To view a latency report for a specific Peering Service connection, follow these steps.

1. Select **All resources** in the left pane, and select the Peering Service connection. Then select **Open** under **Prefixes**. 

   ![Select the Peering Service connection](./media/peering-service-measure/peering-service-measure-menu.png)

2. A latency report page for all the prefixes associated with that Peering Service connection appears. **The Peering service connections supports latency data for /24 or bigger prefixes only.**

      ![Latency report page](./media/peering-service-measure/peering-service-latency-report.png)

3. By default, the report is updated for every hour that's displayed on this page. To view the report for different timelines, choose the appropriate option from **Show data for last**. 

## View Prefix state report

1. To view events for a specific prefix, select the prefix name and select **Prefix Events** in the left pane. The events that are captured are displayed.


   ![Prefix Events](./media/peering-service-measure/peering-service-prefix-event.png)

 Some of the possible events that are captured in the **Prefix Events** list are shown here.

| **Prefix events** | **Event type**|**Reasoning**|
|-----------|---------|---------|
| PrefixAnnouncementEvent |Information|Prefix announcement was received|
| PrefixWithdrawalEvent|Warning| Prefix withdrawal was received |
| PrefixBackupRouteAnnouncementEvent |Information|Prefix backup route announcement was received |
| PrefixBackupRouteWithdrawalEvent|Warning|Prefix backup route withdrawal was received |
| PrefixActivePath |Information| Current prefix active route   |
| PrefixBackupPath | Information|Current prefix backup route   |
| PrefixOriginAsChangeEvent|Critical| Exact prefix received with different origin autonomous system number (for active route)| 
| PrefixBackupRouteOriginAsChangeEvent  | Error|Prefix received with different origin autonomous system number (for backup route)  |

## Next steps

- To learn about Peering Service connection, see [Peering Service connection](connection.md).
- To learn about Peering Service connection telemetry, see [Peering Service connection telemetry](connection-telemetry.md).
