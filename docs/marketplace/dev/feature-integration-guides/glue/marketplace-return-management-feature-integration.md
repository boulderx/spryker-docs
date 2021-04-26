---
title: Glue API - Marketplace Return Management feature integration
last_updated: Apr 8, 2021
description: This document describes the process how to integrate the Marketplace Return Management API feature into a Spryker project.
---

This document describes how to integrate the [Marketplace Return Management API](https://github.com/spryker-feature/marketplace-return-management) feature into a Spryker project.

## Install feature core

Follow the steps below to install the Marketplace Return Management Glue API feature core.

### Prerequisites
<!-- List the features a project must have before they can integrate the current feature. -->

To start feature integration, integrate the required features:
<!--See feature mapping at [Features](https://release.spryker.com/features). -->

| NAME | VERSION | INTEGRATION GUIDE |
| --------- | ------ | --------|
| Marketplace Merchant | dev-master  | [Marketplace Merchant feature integration](docs/marketplace/dev/feature-integration-guides/marketplace-merchants-feature-integration.html) |
| Marketplace Return Management | dev-master | [Marketplace Return Management feature integration](docs/marketplace/dev/feature-integration-guides/marketplace-return-management-feature-integration.html) |

### 1) Install the required modules using Сomposer
<!--Provide one or more console commands with the exact latest version numbers of all required modules. If the Composer command contains the modules that are not related to the current feature, move them to the [prerequisites](#prerequisites).-->

Install the required modules:

```bash
composer require spryker/merchant-sales-returns-rest-api:"^0.2.0" --update-with-dependencies
```

---
**Verification**
<!--Describe how a developer can check they have completed the step correctly.-->

Make sure that the following modules have been installed:

| MODULE  | EXPECTED DIRECTORY <!--for public Demo Shops--> |
| -------- | ------------------- |
|MerchantSalesReturnsRestApi | spryker/merchant-sales-returns-rest-api |

---


### 2) Set up transfer objects
<!--If the feature has database definition changes, merge the steps as described in [Set up database schema and transfer objects](#set-up-database-schema-and-transfer-objects). Provide code snippet with transfer schema changes, describing the changes before each code snippet. Provide the console commands to apply the changes in project and core.-->

Generate transfers:

```bash
console transfer:generate
```

---
**Verification**
<!--Describe how a developer can check they have completed the step correctly.-->

Ensure the following transfers have been created:

| TRANSFER | TYPE | EVENT  | PATH  |
| --------- | ------- | ----- | ------------- |
| Return.merchantReference | attribute | created | src/Generated/Shared/Transfer/ReturnTransfer |
| ReturnCollection | class | created | src/Generated/Shared/Transfer/ReturnCollectionTransfer |
| RestReturnsAttributes | class | created | src/Generated/Shared/Transfer/RestReturnsAttributesTransfer |
| RestOrderItemsAttributes | class | created | src/Generated/Shared/Transfer/RestOrderItemsAttributesTransfer |
| ReturnResponse.messages | attribute | created | src/Generated/Shared/Transfer/ReturnResponseTransfer |
---

### 3) Set up behavior
<!--This is a comment, it will not be included -->
Enable the following behaviors by registering the plugins:

| PLUGIN  | SPECIFICATION | PREREQUISITES | NAMESPACE |
| ------------ | ----------- | ----- | ------------ |
| MerchantByMerchantReferenceResourceRelationshipPlugin | Adds `merchants` resources as relationship by merchant references in the attributes |  |  Spryker\Glue\MerchantsRestApi\Plugin\GlueApplication     |

<details>
<summary markdown='span'>src/Pyz/Glue/GlueApplication/GlueApplicationDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Glue\GlueApplication;

use Spryker\Glue\MerchantsRestApi\Plugin\GlueApplication\MerchantByMerchantReferenceResourceRelationshipPlugin;
use Spryker\Glue\GlueApplicationExtension\Dependency\Plugin\ResourceRelationshipCollectionInterface;

class GlueApplicationDependencyProvider extends SprykerGlueApplicationDependencyProvider
{
  protected function getResourceRelationshipPlugins(
          ResourceRelationshipCollectionInterface $resourceRelationshipCollection
      ): ResourceRelationshipCollectionInterface {
          $resourceRelationshipCollection->addRelationship(
                SalesReturnsRestApiConfig::RESOURCE_RETURNS,
                new MerchantByMerchantReferenceResourceRelationshipPlugin()
            );

            return $resourceRelationshipCollection;
      }

}
```

</details>

---
**Verification**

<!--Describe how a developer can check they have completed the step correctly.-->

Make sure that the `MerchantByMerchantReferenceResourceRelationshipPlugin`
plugin is set up by:
1. sending the request `GET http://glue.mysprykershop.com/returns/{% raw %}{{returnId}}{% endraw %}include=merchants`.

Verify that the return data includes `merchant` resource attributes.

2. Sending the request `GET http://glue.mysprykershop.com/returns`.

Verify that the return data includes the `merchantReference`.

---