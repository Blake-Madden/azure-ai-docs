---
title: Enable or disable semantic ranker
titleSuffix: Azure AI Search
description: Steps for turning semantic ranker on or off in Azure AI Search.

manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: azure-ai-search
ms.custom:
  - ignite-2023
ms.topic: how-to
ms.date: 09/24/2024
---

# Enable or disable semantic ranker

Semantic ranker is a premium feature billed by usage. By default, semantic ranker is turned off on a new search service, but it can be enabled by anyone with **Contributor** permissions. If you don't want anyone enabling it inadvertently, you can [disable it using the REST API](#disable-semantic-ranker-using-the-rest-api).

## Check availability

Check the [regions list](search-region-support.md) to see if your region is listed.

## Enable semantic ranker

Follow these steps to enable [semantic ranker](semantic-search-overview.md) at the service level. Once enabled, it's available to all indexes. You can't turn it on or off for specific indexes.

### [**Azure portal**](#tab/enable-portal)

1. Open the [Azure portal](https://portal.azure.com).

1. Navigate to your search service. On the **Overview** page, make sure the service is a billable tier, Basic or higher.

1. On the left-navigation pane, select **Settings** > **Semantic ranker**.

1. Select either the **Free plan** or the **Standard plan**. You can switch between the free plan and the standard plan at any time.

   :::image type="content" source="media/semantic-search-overview/semantic-search-billing.png" alt-text="Screenshot of enabling semantic ranking in the Azure portal." border="true":::

The free plan is capped at 1,000 queries per month. After the first 1,000 queries in the free plan, an error message indicates you exhausted your quota on the next semantic query. When quota is exhausted, you should upgrade to the standard plan to continue using semantic ranking.

### [**REST**](#tab/enable-rest)

To enable semantic ranker using the REST API, you can use the [Create or Update Service API](/rest/api/searchmanagement/services/create-or-update?view=rest-searchmanagement-2023-11-01&tabs=HTTP#searchsemanticsearch&preserve-view=true).

Management REST API calls are authenticated through Microsoft Entra ID. See [Manage your Azure AI Search service with REST APIs](search-manage-rest.md) for instructions on how to authenticate.

* Management REST API version 2023-11-01 provides the configuration property.

* Owner or Contributor permissions are required to enable or disable features. 

> [!NOTE]
> Create or Update supports two HTTP methods: PUT and PATCH. Both PUT and PATCH can be used to update existing services, but only PUT can be used to create a new service. If PUT is used to update an existing service, it replaces all properties in the service with their defaults if they are not specified in the request. When PATCH is used to update an existing service, it only replaces properties that are specified in the request. When using PUT to update an existing service, it's possible to accidentally introduce an unexpected scaling or configuration change. When enabling semantic ranking on an existing service, it's recommended to use PATCH instead of PUT.

```http
PATCH https://management.azure.com/subscriptions/{{subscriptionId}}/resourcegroups/{{resource-group}}/providers/Microsoft.Search/searchServices/{{search-service-name}}?api-version=2023-11-01
    {
      "properties": {
        "semanticSearch": "standard"
      }
    }
```

---

## Disable semantic ranker using the REST API

To reverse feature enablement, or for full protection against accidental usage and charges, you can disable semantic ranker using the [Create or Update Service API](/rest/api/searchmanagement/services/create-or-update#searchsemanticsearch) on your search service. After the feature is disabled, any requests that include the semantic query type will be rejected.

Management REST API calls are authenticated through Microsoft Entra ID. See [Manage your Azure AI Search service with REST APIs](search-manage-rest.md) for instructions on how to authenticate.

```http
PATCH https://management.azure.com/subscriptions/{{subscriptionId}}/resourcegroups/{{resource-group}}/providers/Microsoft.Search/searchServices/{{search-service-name}}?api-version=2023-11-01
    {
      "properties": {
        "semanticSearch": "disabled"
      }
    }
```

To re-enable semantic ranker, rerun the previous request, setting "semanticSearch" to either "free" (default) or "standard".

## Next steps

[Configure semantic ranker](semantic-how-to-configure.md) so that you can test out semantic ranking on your content.
