---
permalink: /nodes/n8n-nodes-base.spontit
description: Learn how to use the Spontit node in n8n
---

# Spontit

[Spontit](https://www.spontit.com/) enables you to send push notifications without your own app, or even website. You can create different channels and even send notifications to specific followers.

::: tip 🔑 Credentials
You can find authentication information for this node [here](../../../credentials/Spontit/README.md).
:::

## Basic Operations

::: details Push
- Create a push notification
:::

## Example Usage

This workflow allows you to send daily weather updates via a push notification using the Spontit node. You can also find the [workflow](https://n8n.io/workflows/740) on n8n.io. This example usage workflow uses the following nodes.
- [Cron](../../core-nodes/Cron/README.md)
- [OpenWeatherMap](../../nodes/OpenWeatherMap/README.md)
- [Spontit]()

The final workflow should look like the following image.

![A workflow with the Spontit node](./workflow.png)

### 1. Cron node

The Cron node will trigger the workflow daily at 9 AM.

1. Click on ***Add Cron Time***.
2. Set hours to 9 in the ***Hour*** field.
3. Click on ***Execute Node*** to run the node.

In the screenshot below, you will notice that the Cron node is configured to trigger the workflow every day at 9 AM.

![Using the Cron node to trigger the workflow daily at 9 am](./Cron_node.png)

### 2. OpenWeatherMap node (Current Weather)

This node will return data about the current weather in Berlin. To get the weather updates for your city, you can enter the name of your city instead.

1. First of all, you'll have to enter credentials for the OpenWeatherMap node. You can find out how to do that [here](../../../credentials/OpenWeatherMap/README.md). 
2. Enter `berlin` in the ***City*** field.
3. Click on ***Execute Node*** to run the node.

In the screenshot below, you will notice that the node returns data about the current weather in Berlin.

![Using the OpenWeatherMap node to get weather updates for Berlin](./OpenWeatherMap_node.png)

### 3. Spontit node (create: push)

This node will send a push notification with the weather update.

1. First of all, you'll have to enter credentials for the Spontit node. You can find out how to do that [here](../../../credentials/Spontit/README.md).
2. Enter `Today's Weather Update` in the ***Title*** field.
3. Click on the gears icon next to the ***Body*** field and click on ***Add Expression***.
::: v-pre
4. Enter the following message in the ***Expression*** field: `Hey! The temperature outside is {{$node["OpenWeatherMap"].json["main"]["temp"]}}°C.`.
5. Click on ***Execute Node*** to run the node.
:::

In the screenshot below, you will notice that the node sends a push notification to the default device with the weather update.

![Using the Spontit node to send weather updates via a push notification](./Spontit_node.png)
