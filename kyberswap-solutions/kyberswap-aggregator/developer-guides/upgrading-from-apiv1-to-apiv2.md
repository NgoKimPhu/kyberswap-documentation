---
description: More Performant Routing And Encoding
---

# Upgrading From APIv1 to APIv2

## Overview

Following feedback on the initial APIv1, KyberSwap has implemented a more performant APIv2 which improves the response time for getting a route via offloading encoding requirements to a separate post method. With this upgrade, APIv2 brings the following improvements:

* Reduced latency which enables gas estimation calculations to be more accurate
* Reduces the risk of stale encoded data which could potentially lead to failed transactions on the UI
* Functional separation which enables more precise data handling and compute

## Swap flow: V1 vs V2

As part of the APIv2 upgrade, querying of the route data and the encoded data has been separated into their own respective calls. This stands in contrast to APIv1 whereby both the route and encoded data are returned in a single [`GET`](../aggregator-api-specification/evm-swaps.md#chain-route-encode) query.

Due to this functional separation, applications have more flexibility when handling route refreshes prior to user confirming the swap route. For example, if rates need to be refreshed, the application will just have to query the APIv2 [`GET`](../aggregator-api-specification/evm-swaps.md#chain-api-v1-routes) endpoint which will return just the route summary. As the encoded data does not have to be processed at this stage, responses are much faster and redundant encodings are avoided.

Moreover, given the reduced latency for each call in APIv2, this reduces the likelihood that the returned data will be stale hence mitigating transaction failure risks. Routes are secured (i.e. "locked-in") via the APIv2 [`POST`](../aggregator-api-specification/evm-swaps.md#chain-api-v1-route-build) which returns the encoded data to be sent to the [`MetaAggregationRouterV2`](../contracts/aggregator-contract-addresses.md) contract.

### APIv1 swap flow

Route and encoded data returned in single `GET` call.

<figure><img src="../../../.gitbook/assets/Aggregator APIv2-APIv1.drawio (3).png" alt=""><figcaption><p>APIv1 sequence diagram</p></figcaption></figure>

### APIv2 swap flow&#x20;

Route summary returned in `GET` response and encoded data returned in `POST` response.

<figure><img src="../../../.gitbook/assets/Aggregator APIv2-APIv2.drawio.png" alt=""><figcaption><p>APIv2 sequence diagram</p></figcaption></figure>

## Field mappings

As part of this upgrade, some of the parameters have also been modified to make it more intuitive. The table below maps the relevant changes for avoidance of doubt.

* APIv1 `GET`: [`/{chain}/route/encode`](../aggregator-api-specification/evm-swaps.md#chain-route-encode)``
* APIv2 `GET`: [`/{chain}/api/v1/routes`](../aggregator-api-specification/evm-swaps.md#chain-api-v1-routes)``
* APIv2 `POST`: [`/{chain}/api/v1/route/build`](../aggregator-api-specification/evm-swaps.md#chain-api-v1-route-build)``

### Request

| APIv1 GET         | APIv2 GET       | APIv2 POST              |
| ----------------- | --------------- | ----------------------- |
| tokenIn\*         | tokenIn\*       | -                       |
| tokenOut\*        | tokenOut\*      | -                       |
| amountIn\*        | amountIn\*      | -                       |
| saveGas           | saveGas         | -                       |
| slippageTolerance | -               | slippageTolerance       |
| isInBps           | isInBps         | -                       |
| chargeFeeBy       | chargeFeeBy     | -                       |
| feeReceiver       | feeReceiver     | -                       |
| feeAmount         | feeAmount       | -                       |
| deadline          | -               | deadline                |
| to\*              | -               | recipient\*             |
| clientData        | -               | source                  |
| -                 | includedSources | -                       |
| -                 | excludedSources | -                       |
| -                 | gasInclude      | -                       |
| -                 | gasPrice        | -                       |
| -                 | -               | routeSummary\*          |
| -                 | -               | sender\*                |
| -                 | -               | skipSimulateTransaction |

### Response

| APIv1 GET         | APIv2 GET         | APIv2 POST    |
| ----------------- | ----------------- | ------------- |
| inputAmount       | amountIn          | amountIn      |
| outputAmount      | amountOut         | amountOut     |
| totalGas          | gas               | gas           |
| gasPriceGwei      | gasPrice          |               |
| gasUsd            | gasUsd            | gasUsd        |
| amountInUsd       | amountInUsd       | amountInUsd   |
| amountOutUsd      | amountOutUsd      | amountOutUsd  |
| receiveUsd        | -                 | -             |
| swaps             | route             | -             |
| pool              | pool              | -             |
| tokenIn           | tokenIn           | -             |
| tokenOut          | tokenOut          | -             |
| swapAmount        | swapAmount        | -             |
| amountOut         | amountOut         | -             |
| limitReturnAmount | limitReturnAmount | -             |
| maxPrice          | -                 | -             |
| exchange          | exchange          | -             |
| poolLength        | poolLength        | -             |
| poolType          | poolType          | -             |
| tokens            | -                 | -             |
| address           | -                 | -             |
| symbol            | -                 | -             |
| name              | -                 | -             |
| price             | -                 | -             |
| decimals          | -                 | -             |
| encodedSwapData   | -                 | data          |
| routerAddress     | routerAddress     | routerAddress |
| -                 | code              | code          |
| -                 | message           | message       |
| -                 | data              | data          |
| -                 | routeSummary      | -             |
| -                 | extraFee          | -             |
| -                 | feeAmount         | -             |
| -                 | chargeFeeBy       | -             |
| -                 | isInBps           | -             |
| -                 | feeReceiver       | -             |