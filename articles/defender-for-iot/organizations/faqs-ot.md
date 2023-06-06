---
title: FAQs for OT networks - Microsoft Defender for IoT
description: Find answers to the most frequently asked questions about Microsoft Defender for IoT OT networks.
ms.topic: faq
ms.date: 05/17/2023
---

# Operational Technology (OT) networks frequently asked questions

This article provides a list of frequently asked questions and answers about OT networks in Defender for IoT.

## Our organization uses proprietary non-standard industrial protocols. Are they supported?

Microsoft Defender for IoT provides comprehensive protocol support. In addition to embedded protocol support, you can secure IoT and OT devices running proprietary and custom protocols, or protocols that deviate from any standard. Use the Horizon Open Development Environment (ODE) SDK, to create dissector plugins that decode network traffic based on defined protocols. Traffic is analyzed by services to provide complete monitoring, alerting, and reporting. Use Horizon to:

- Expand visibility and control without the need to upgrade to new versions.
- Secure proprietary information by developing on-site as an external plugin.
- Localize text for alerts, events, and protocol parameters.

This unique solution for developing protocols as plugins, doesn't require dedicated developer teams or version releases in order to support a new protocol. Developers, partners, and customers can securely develop protocols and share insights and knowledge using Horizon.

## Do I have to purchase hardware appliances from Microsoft partners?

Microsoft Defender for IoT sensor runs on specific hardware specs as described in the [Hardware Specifications Guide](./how-to-identify-required-appliances.md), customers can purchase certified hardware from Microsoft partners or use the supplied bill of materials  (BOM) and purchase it on their own.

Certified hardware has been tested in our labs for driver stability, packet drops and network sizing.

## Regulation doesn't allow us to connect our system to the Internet. Can we still use Defender for IoT?

Yes you can! The Microsoft Defender for IoT platform on-premises solution is deployed as a physical or virtual sensor appliance that passively ingests network traffic, such as via SPAN, RSPAN, or TAP, to analyze, discover, and continuously monitor IT, OT, and IoT networks. 

For larger enterprises, multiple sensors can aggregate their data to an on-premises management console.

## Do I need to purchase new licenses when my site grows?

When your Defender for IoT license expires, you'll need to purchase a new license from the Microsoft 365 admin center. If your site has outgrown your previous license size, you'll be prompted to purchase a larger license.

## Are changes automatically synchronized between the Microsoft 365 admin center and the Azure portal?

Any new licenses that you purchase in the Microsoft 365 admin center are automatically reflected in your OT plan on the Azure portal.

However, canceling an OT plan from the Azure portal does *not* cancel your license in the Microsoft 365 admin center. If you cancel your OT plan in the Azure portal, make sure to also cancel your license in the Microsoft 365 admin center.

All license management procedures are done from the Microsoft 365 admin center, including buying, canceling, renewing, setting to auto-renew, auditing, and more. 

For more information, see [Manage OT plans and licenses](how-to-manage-subscriptions.md) and the [Microsoft 365 admin center help](/microsoft-365/admin/).

## I have a legacy OT plan on my Azure subscription. When do I need to take action?

Starting June 1, 2023, Microsoft Defender for IoT licenses for OT monitoring are available for purchase only in the [Microsoft 365 admin center](https://admin.microsoft.com/Adminportal/Home).

Existing customers can continue to use any legacy OT plan already onboarded to an Azure subscription, with no changes in functionality. However, if you need to add a new plan, for a new subscription, you'll need to purchase a license for that plan from the Microsoft 365 admin center.

For more information, see [OT plans billed by site-based licenses](whats-new.md#ot-plans-billed-by-site-based-licenses).


## Where in the network should I connect monitoring ports?

The Microsoft Defender for IoT sensor connects to a SPAN port or network TAP and immediately begins collecting ICS network traffic via passive (agentless) monitoring. It has zero effect on OT networks since it isn’t placed in the data path and doesn’t actively scan OT devices.

For example:

- A single appliance (virtual of physical) can be in the Shop Floor DMZ layer, having all Shop Floor cell traffic routed to this layer.
- Alternatively, locate small mini-sensors in each Shop Floor cell with either cloud or local management that will reside in the Shop Floor DMZ layer. Another appliance (virtual or physical) can monitor the traffic in the Shop Floor DMZ layer (for SCADA, Historian, or MES).

For more information, see [Prepare an OT site deployment](best-practices/plan-prepare-deploy.md).

## How can I change a user's passwords

You can change user passwords or recover access to privileged users on both the OT network sensor and the on-premises management console. For more information, see:

- [Create and manage users on an OT network sensor](manage-users-sensor.md)
- [Create and manage users on an on-premises management console](manage-users-on-premises-management-console.md)

## How do I activate the sensor and on-premises management console

For information on how to activate your on-premises management console, see [Activate and set up an on-premises management console](ot-deploy/activate-deploy-management.md).

## How to change the network configuration

Change network configuration settings before or after you activate your sensor using either of the following options:

- **From the sensor UI**: [Update the OT sensor network configuration](how-to-manage-individual-sensors.md#update-the-ot-sensor-network-configuration)
- **From the sensor CLI**: [Network configuration](cli-ot-sensor.md#network-configuration)

For more information, see [Activate and set up your OT network sensor](ot-deploy/activate-deploy-sensor.md), [Getting started with advanced CLI commands](references-work-with-defender-for-iot-cli-commands.md), and [CLI command reference from OT network sensors](cli-ot-sensor.md).

## How do I check the sanity of my deployment

After installing the software for your sensor or on-premises management console, you'll want to perform the [Post-installation validation](ot-deploy/post-install-validation-ot-software.md).

You can also use our [UI and CLI tools](how-to-troubleshoot-sensor.md#check-system-health) to check system health and review your overall system statistics.

For more information, see [Troubleshoot the sensor](how-to-troubleshoot-sensor.md) and [Troubleshoot the on-premises management console](how-to-troubleshoot-on-premises-management-console.md).

## Next steps

- [Tutorial: Get started with Microsoft Defender for IoT for OT security](tutorial-onboarding.md)
