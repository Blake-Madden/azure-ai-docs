---
title: How to use chat completions with Azure AI model inference
titleSuffix: Azure AI Foundry
description: Learn how to generate chat completions with Azure AI model inference
manager: scottpolly
author: mopeakande
reviewer: santiagxf
ms.service: azure-ai-model-inference
ms.topic: how-to
ms.date: 1/21/2025
ms.author: mopeakande
ms.reviewer: fasantia
ms.custom: references_regions, tool_generated
zone_pivot_groups: azure-ai-inference-samples
---

[!INCLUDE [Feature preview](~/reusable-content/ce-skilling/azure/includes/ai-studio/includes/feature-preview.md)]

This article explains how to use chat completions API with models deployed to Azure AI model inference in Azure AI services.

## Prerequisites

To use chat completion models in your application, you need:

[!INCLUDE [how-to-prerequisites](../how-to-prerequisites.md)]

* A chat completions model deployment. If you don't have one read [Add and configure models to Azure AI services](../../how-to/create-model-deployments.md) to add a chat completions model to your resource.

* Add the [Azure AI inference package](https://aka.ms/azsdk/azure-ai-inference/java/reference) to your project:

  ```xml
  <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-ai-inference</artifactId>
      <version>1.0.0-beta.1</version>
  </dependency>
  ```
  
* If you are using Entra ID, you also need the following package:

  ```xml
  <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-identity</artifactId>
      <version>1.13.3</version>
  </dependency>
  ```

* Import the following namespace:
  
  ```java
  package com.azure.ai.inference.usage;
  
  import com.azure.ai.inference.EmbeddingsClient;
  import com.azure.ai.inference.EmbeddingsClientBuilder;
  import com.azure.ai.inference.models.EmbeddingsResult;
  import com.azure.ai.inference.models.EmbeddingItem;
  import com.azure.core.credential.AzureKeyCredential;
  import com.azure.core.util.Configuration;
  
  import java.util.ArrayList;
  import java.util.List;
  ```

## Use chat completions

First, create the client to consume the model. The following code uses an endpoint URL and key that are stored in environment variables.

If you have configured the resource to with **Microsoft Entra ID** support, you can use the following code snippet to create a client.

### Create a chat completion request

The following example shows how you can create a basic chat completions request to the model.
> [!NOTE]
> Some models don't support system messages (`role="system"`). When you use the Azure AI model inference API, system messages are translated to user messages, which is the closest capability available. This translation is offered for convenience, but it's important for you to verify that the model is following the instructions in the system message with the right level of confidence.

The response is as follows, where you can see the model's usage statistics:

```console
Response: As of now, it's estimated that there are about 7,000 languages spoken around the world. However, this number can vary as some languages become extinct and new ones develop. It's also important to note that the number of speakers can greatly vary between languages, with some having millions of speakers and others only a few hundred.
Model: mistral-large-2407
Usage: 
  Prompt tokens: 19
  Total tokens: 91
  Completion tokens: 72
```

Inspect the `usage` section in the response to see the number of tokens used for the prompt, the total number of tokens generated, and the number of tokens used for the completion.

#### Stream content

By default, the completions API returns the entire generated content in a single response. If you're generating long completions, waiting for the response can take many seconds.

You can _stream_ the content to get it as it's being generated. Streaming content allows you to start processing the completion as content becomes available. This mode returns an object that streams back the response as [data-only server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html#server-sent-events). Extract chunks from the delta field, rather than the message field.

You can visualize how streaming generates content:

#### Explore more parameters supported by the inference client

Explore other parameters that you can specify in the inference client. For a full list of all the supported parameters and their corresponding documentation, see [Azure AI Model Inference API reference](https://aka.ms/azureai/modelinference).
Some models don't support JSON output formatting. You can always prompt the model to generate JSON outputs. However, such outputs are not guaranteed to be valid JSON.

If you want to pass a parameter that isn't in the list of supported parameters, you can pass it to the underlying model using *extra parameters*. See [Pass extra parameters to the model](#pass-extra-parameters-to-the-model).

#### Create JSON outputs

Some models can create JSON outputs. Set `response_format` to `json_object` to enable JSON mode and guarantee that the message the model generates is valid JSON. You must also instruct the model to produce JSON yourself via a system or user message. Also, the message content might be partially cut off if `finish_reason="length"`, which indicates that the generation exceeded `max_tokens` or that the conversation exceeded the max context length.

### Pass extra parameters to the model

The Azure AI Model Inference API allows you to pass extra parameters to the model. The following code example shows how to pass the extra parameter `logprobs` to the model. 

Before you pass extra parameters to the Azure AI model inference API, make sure your model supports those extra parameters. When the request is made to the underlying model, the header `extra-parameters` is passed to the model with the value `pass-through`. This value tells the endpoint to pass the extra parameters to the model. Use of extra parameters with the model doesn't guarantee that the model can actually handle them. Read the model's documentation to understand which extra parameters are supported.

### Use tools

Some models support the use of tools, which can be an extraordinary resource when you need to offload specific tasks from the language model and instead rely on a more deterministic system or even a different language model. The Azure AI Model Inference API allows you to define tools in the following way.

The following code example creates a tool definition that is able to look from flight information from two different cities.

In this example, the function's output is that there are no flights available for the selected route, but the user should consider taking a train.

> [!NOTE]
> Cohere models require a tool's responses to be a valid JSON content formatted as a string. When constructing messages of type *Tool*, ensure the response is a valid JSON string.

Prompt the model to book flights with the help of this function:

You can inspect the response to find out if a tool needs to be called. Inspect the finish reason to determine if the tool should be called. Remember that multiple tool types can be indicated. This example demonstrates a tool of type `function`.

To continue, append this message to the chat history:

Now, it's time to call the appropriate function to handle the tool call. The following code snippet iterates over all the tool calls indicated in the response and calls the corresponding function with the appropriate parameters. The response is also appended to the chat history.

View the response from the model:

### Apply content safety

The Azure AI model inference API supports [Azure AI content safety](https://aka.ms/azureaicontentsafety). When you use deployments with Azure AI content safety turned on, inputs and outputs pass through an ensemble of classification models aimed at detecting and preventing the output of harmful content. The content filtering system detects and takes action on specific categories of potentially harmful content in both input prompts and output completions.

The following example shows how to handle events when the model detects harmful content in the input prompt and content safety is enabled.

> [!TIP]
> To learn more about how you can configure and control Azure AI content safety settings, check the [Azure AI content safety documentation](https://aka.ms/azureaicontentsafety).

## Use chat completions with images

Some models can reason across text and images and generate text completions based on both kinds of input. In this section, you explore the capabilities of Some models for vision in a chat fashion:

> [!IMPORTANT]
> Some models support only one image for each turn in the chat conversation and only the last image is retained in context. If you add multiple images, it results in an error.

To see this capability, download an image and encode the information as `base64` string. The resulting data should be inside of a [data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URLs):

Visualize the image:

:::image type="content" source="../../../../ai-foundry/media/how-to/sdks/small-language-models-chart-example.jpg" alt-text="A chart displaying the relative capabilities between large language models and small language models." lightbox="../../../../ai-foundry/media/how-to/sdks/small-language-models-chart-example.jpg":::

Now, create a chat completion request with the image:

The response is as follows, where you can see the model's usage statistics:

```console
ASSISTANT: The chart illustrates that larger models tend to perform better in quality, as indicated by their size in billions of parameters. However, there are exceptions to this trend, such as Phi-3-medium and Phi-3-small, which outperform smaller models in quality. This suggests that while larger models generally have an advantage, there might be other factors at play that influence a model's performance.
Model: mistral-large-2407
Usage: 
  Prompt tokens: 2380
  Completion tokens: 126
  Total tokens: 2506
```
