---
title: Marketplace Order Management feature integration
last_updated: Jun 25, 2021
description: This document describes how to integrate the Marketplace Order Management feature into a Spryker project.
template: feature-integration-guide-template
---

This document describes how to integrate the Marketplace Order Management feature into a Spryker project.

## Install feature core

Follow the steps below to install the Marketplace Order Management feature core.

### Prerequisites

To start feature integration, integrate the required features:

| NAME | VERSION | INTEGRATION GUIDE |
| --------- | ------ | ---------------|
| Spryker Core | {{page.version}} | [Spryker Core feature integration](https://documentation.spryker.com/docs/spryker-core-feature-integration) |
| Order Management | {{page.version}} | [Order Management feature integration](https://documentation.spryker.com/docs/order-management-feature-integration) |
| Shipment | {{page.version}} | [Shipment feature integration](https://documentation.spryker.com/docs/shipment-feature-integration) |
| State Machine | {{page.version}} | [State Machine feature integration](https://github.com/spryker-feature/state-machine) |
| Marketplace Merchant | {{page.version}} | [Marketplace Merchant feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/marketplace-merchant-feature-integration.html) |

### 1) Install required modules using Сomposer

Install the required modules:

```bash
composer require spryker-feature/marketplace-order-management --update-with-dependencies
```

{% info_block warningBox "Verification" %}

Make sure that the following modules have been installed:

| MODULE  | EXPECTED DIRECTORY |
| -------- | ------------------- |
| MerchantOms | spryker/merchant-oms |
| MerchantOmsDataImport | spryker/merchant-oms-data-import |
| MerchantOmsGui | spryker/merchant-oms-gui |
| MerchantSalesOrder | spryker/merchant-sales-order |
| MerchantSalesOrderMerchantUserGui | spryker/merchant-sales-order-merchant-user-gui |
| MerchantSalesOrderDataExport | spryker/merchant-sales-order-data-export |
| ProductOfferSales | spryker/product-offer-sales |

{% endinfo_block %}

### 2) Set up configuration

<!--Describe system and module configuration changes. If the default configuration is enough for a primary behavior, skip this step.-->

Add the following configuration:

| CONFIGURATION | SPECIFICATION | NAMESPACE |
| ------------- | ------------ | ------------ |
| MainMerchantStateMachine | Introduce `MainMerchantStateMachine` configuration. | config/Zed/StateMachine/Merchant/MainMerchantStateMachine.xml |
| MerchantDefaultStateMachine | Introduce `MerchantDefaultStateMachine` configuration. | config/Zed/StateMachine/Merchant/MerchantDefaultStateMachine.xml |
| MarketplacePayment  | Introduce `MarketplacePayment` order management system. | config/Zed/oms/MarketplacePayment01.xml |
| MarketplacePayment  | Introduce `MarketplacePayment` order management system. | config/Zed/oms/MarketplacePayment01.xml |

<details>
<summary markdown='span'>config/Zed/StateMachine/Merchant/MainMerchantStateMachine.xml</summary>

```xml
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:state-machine-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:state-machine-01 http://static.spryker.com/state-machine-01.xsd"
>
    <process name="MainMerchantStateMachine" main="true">

        <states>
            <state name="created"/>
            <state name="new"/>
            <state name="canceled"/>
            <state name="left the merchant location"/>
            <state name="arrived at distribution center"/>
            <state name="shipped"/>
            <state name="delivered"/>
            <state name="closed"/>
        </states>

        <transitions>
            <transition happy="true">
                <source>created</source>
                <target>new</target>
                <event>initiate</event>
            </transition>

            <transition>
                <source>new</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition>
                <source>new</source>
                <target>canceled</target>
                <event>cancel</event>
            </transition>

            <transition>
                <source>canceled</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition happy="true">
                <source>new</source>
                <target>left the merchant location</target>
                <event>send to distribution</event>
            </transition>

            <transition happy="true">
                <source>left the merchant location</source>
                <target>arrived at distribution center</target>
                <event>confirm at center</event>
            </transition>

            <transition happy="true">
                <source>arrived at distribution center</source>
                <target>shipped</target>
                <event>ship</event>
            </transition>

            <transition happy="true">
                <source>shipped</source>
                <target>delivered</target>
                <event>deliver</event>
            </transition>

            <transition happy="true">
                <source>delivered</source>
                <target>closed</target>
                <event>close</event>
            </transition>
        </transitions>

        <events>
            <event name="initiate" onEnter="true"/>
            <event name="send to distribution" manual="true"/>
            <event name="confirm at center" manual="true"/>
            <event name="ship" manual="true" command="DummyMarketplacePayment/ShipOrderItem"/>
            <event name="deliver" manual="true" command="DummyMarketplacePayment/DeliverOrderItem"/>
            <event name="close"/>
            <event name="cancel" manual="true"/>
        </events>

    </process>

</statemachine>

```

</details>

<details>
<summary markdown='span'>config/Zed/StateMachine/Merchant/MerchantDefaultStateMachine.xml</summary>

```xml
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:state-machine-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:state-machine-01 http://static.spryker.com/state-machine-01.xsd"
>

    <process name="MerchantDefaultStateMachine" main="true">

        <states>
            <state name="created"/>
            <state name="new"/>
            <state name="canceled by merchant"/>
            <state name="shipped"/>
            <state name="delivered"/>
            <state name="closed"/>
        </states>

        <transitions>
            <transition happy="true">
                <source>created</source>
                <target>new</target>
                <event>initiate</event>
            </transition>

            <transition happy="true">
                <source>new</source>
                <target>shipped</target>
                <event>ship</event>
            </transition>

            <transition>
                <source>new</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition>
                <source>new</source>
                <target>canceled by merchant</target>
                <event>cancel by merchant</event>
            </transition>

            <transition>
                <source>canceled by merchant</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition happy="true">
                <source>shipped</source>
                <target>delivered</target>
                <event>deliver</event>
            </transition>

            <transition happy="true">
                <source>delivered</source>
                <target>closed</target>
                <event>close</event>
            </transition>
        </transitions>

        <events>
            <event name="initiate" onEnter="true"/>
            <event name="ship" manual="true" command="DummyMarketplacePayment/ShipOrderItem"/>
            <event name="deliver" manual="true" command="DummyMarketplacePayment/DeliverOrderItem"/>
            <event name="close"/>
            <event name="cancel by merchant" manual="true"/>
        </events>

    </process>

</statemachine>

```
</details>

<details>
<summary markdown='span'>config/Zed/oms/MarketplacePayment01.xml</summary>

```xml
<?xml version="1.0"?>
<statemachine
    xmlns="spryker:oms-01"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="spryker:oms-01 http://static.spryker.com/oms-01.xsd"
>

    <process name="MarketplacePayment01" main="true">
        <states>
            <state name="new" reserved="true" display="oms.state.new"/>
            <state name="paid" reserved="true" display="oms.state.paid"/>
            <state name="canceled" display="oms.state.canceled"/>
            <state name="refunded" display="oms.state.refunded"/>
            <state name="merchant split pending" reserved="true" display="oms.state.merchant-split-pending"/>
            <state name="sent to merchant" reserved="true" display="oms.state.sent-to-merchant"/>
            <state name="shipped by merchant" reserved="true" display="oms.state.shipped-by-merchant"/>
            <state name="delivered" reserved="true" display="oms.state.delivered"/>
            <state name="closed" display="oms.state.closed"/>
        </states>

        <transitions>
            <transition happy="true">
                <source>new</source>
                <target>paid</target>
                <event>pay</event>
            </transition>

            <transition happy="true" condition="MerchantSalesOrder/IsOrderPaid">
                <source>paid</source>
                <target>merchant split pending</target>
            </transition>

            <transition>
                <source>paid</source>
                <target>paid</target>
            </transition>

            <transition happy="true">
                <source>merchant split pending</source>
                <target>sent to merchant</target>
                <event>send to merchant</event>
            </transition>

            <transition>
                <source>sent to merchant</source>
                <target>canceled</target>
                <event>cancel</event>
            </transition>

            <transition>
                <source>canceled</source>
                <target>refunded</target>
                <event>refund</event>
            </transition>

            <transition>
                <source>refunded</source>
                <target>closed</target>
                <event>close</event>
            </transition>

            <transition happy="true">
                <source>sent to merchant</source>
                <target>shipped by merchant</target>
                <event>ship by merchant</event>
            </transition>

            <transition happy="true">
                <source>shipped by merchant</source>
                <target>delivered</target>
                <event>deliver</event>
            </transition>

            <transition happy="true">
                <source>delivered</source>
                <target>closed</target>
                <event>close</event>
            </transition>

        </transitions>

        <events>
            <event name="pay" manual="true"/>
            <event name="cancel" manual="true"/>
            <event name="refund" manual="true"/>
            <event name="send to merchant" onEnter="true" command="MerchantSalesOrder/CreateOrders"/>
            <event name="ship by merchant"/>
            <event name="deliver"/>
            <event name="close" manual="true" command="MerchantOms/CloseOrderItem"/>
        </events>
    </process>

</statemachine>
```

</details>


### 3) Set up database schema and transfer objects
Apply database changes and generate entity and transfer changes:

```bash
console transfer:generate
console propel:install
console transfer:generate
```

{% info_block warningBox "Verification" %}

Check your database to make sure that the following changes have been applied:

| DATABASE ENTITY | TYPE | EVENT |
| --------------- | ---- | ------ |
|spy_merchant.fk_state_machine_process |column |created  |
|spy_merchant_sales_order_item.fk_state_machine_item_state | column|created  |
|spy_merchant_sales_order | table |created |
|spy_merchant_sales_order_item | table |created |
|spy_merchant_sales_order_totals | table |created |
|spy_sales_expense.merchant_reference | column |created |
|spy_sales_order_item.merchant_reference | column |created  |

{% endinfo_block %}

{% info_block warningBox "Verification" %}

Make sure that the following changes have been triggered in transfer objects:

| TRANSFER | TYPE | EVENT  | PATH  |
| --------- | ------- | ----- | ------------- |
| MerchantOrderCriteria.idMerchant | attribute | created | src/Generated/Shared/Transfer/MerchantOrderCriteriaTransfer |
| DataImporterConfiguration | class | created | src/Generated/Shared/Transfer/DataImporterConfigurationTransfer |
| StateMachineItem.stateName | attribute | created | src/Generated/Shared/Transfer/StateMachineItemTransfer |
| Merchant.merchantReference | attribute | created | src/Generated/Shared/Transfer/MerchantTransfer |
| MerchantOrderItem.idMerchantOrderItem | attribute | created | src/Generated/Shared/Transfer/MerchantOrderItemTransfer |
| Item.productOfferReference | attribute | created| src/Generated/Shared/Transfer/ItemTransfer |

{% endinfo_block %}

### 4) Add translations

Generate a new translation cache for Zed:

```bash
console translator:generate-cache
```

### 5) Import data

Import data as follows:

1. Prepare your data according to your requirements using the demo data:

**data/import/common/common/marketplace/merchant_oms_process.csv**
```csv
merchant_reference,merchant_oms_process_name
MER000001,MainMerchantStateMachine
MER000002,MerchantDefaultStateMachine
MER000006,MerchantDefaultStateMachine
MER000004,MerchantDefaultStateMachine
MER000003,MerchantDefaultStateMachine
MER000007,MerchantDefaultStateMachine
MER000005,MerchantDefaultStateMachine
```


|PAREMETER |REQUIRED?  |TYPE  |DATA EXAMPLE | DESCRIPTION |
|---------|---------|---------|---------| ---------|
|merchant_reference     |  &check;       |  string       | spryker        |String identifier for merchant in the Spryker system. |
|merchant_oms_process_name     |    &check;     |     string   |  MainMerchantStateMachine       | String identifier for the State Machine processes.|

2. Register the following plugin to enable data import:

|PLUGIN  |SPECIFICATION  |PREREQUISITES  |NAMESPACE  |
|---------|---------|---------|---------|
|MerchantOmsProcessDataImportPlugin |  Imports Merchant State Machine data  |  |   Spryker\Zed\MerchantOmsDataImport\Communication\Plugin\DataImport  |

**src/Pyz/Zed/DataImport/DataImportDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\DataImport;

use Spryker\Zed\DataImport\DataImportDependencyProvider as SprykerDataImportDependencyProvider;
use Spryker\Zed\MerchantOmsDataImport\Communication\Plugin\DataImport;

class DataImportDependencyProvider extends SprykerDataImportDependencyProvider
{
    protected function getDataImporterPlugins(): array
    {
        return [
            new MerchantOmsProcessDataImportPlugin(),
        ];
    }
}

```


3. Import data:

```bash
console data:import merchant-oms-process
```

{% info_block warningBox "Verification" %}

Make sure that in the `spy_merchant` table, merchants have correct `fk_process id` in their columns.

{% endinfo_block %}

### 6) Export data

Export data as follows:

1. Create and prepare your `data/export/config/merchant_order_export_config.yml` file  according to your requirements using our demo config template:

<details>
<summary markdown='span'>data/export/config/merchant_order_export_config.yml</summary>

```xml
version: 1

defaults:
    filter_criteria: &default_filter_criteria
        merchant_order_created_at:
            type: between
            from: '2020-05-01 00:00:00+09:00'
            to: '2021-12-31 23:59:59+09:00'
        merchant_order_updated_at:
            type: between
            from: '2021-01-08 09:00:12+12:00'
            to: '2021-12-31 23:59:59+09:00'

actions:
#Merchant orders data export
    - data_entity: merchant-order-expense
      destination: 'merchants/{merchant_name}/merchant-orders/{data_entity}s_{store_name}_{timestamp}.csv'
      filter_criteria:
          <<: *default_filter_criteria
          store_name: [DE]

    - data_entity: merchant-order-expense
      destination: 'merchants/{merchant_name}/merchant-orders/{data_entity}s_{store_name}_{timestamp}.csv'
      filter_criteria:
          <<: *default_filter_criteria
          store_name: [US]

    - data_entity: merchant-order-item
      destination: 'merchants/{merchant_name}/merchant-orders/{data_entity}s_{store_name}_{timestamp}.csv'
      filter_criteria:
          <<: *default_filter_criteria
          store_name: [DE]

    - data_entity: merchant-order-item
      destination: 'merchants/{merchant_name}/merchant-orders/{data_entity}s_{store_name}_{timestamp}.csv'
      filter_criteria:
          <<: *default_filter_criteria
          store_name: [US]

    - data_entity: merchant-order
      destination: 'merchants/{merchant_name}/merchant-orders/{data_entity}s_{store_name}_{timestamp}.csv'
      filter_criteria:
          <<: *default_filter_criteria
          store_name: [DE]

    - data_entity: merchant-order
      destination: 'merchants/{merchant_name}/merchant-orders/{data_entity}s_{store_name}_{timestamp}.csv'
      filter_criteria:
          <<: *default_filter_criteria
          store_name: [US]

```
</details>


| PARAMETER |  |  | REQUIRED | POSSIBLE VALUES | DESCRIPTION |
|-|-|-|-|-|-|
| data_entity |  |  | &check; | merchant-order merchant-order-item merchant-order-expense | String identifier for data entity that is expected to be  exported. |
| filter_criteria | store_name |  | &check; | All existing store names. | An existing store name for the data to filter on. |
|  | merchant_order_created_at | from |  | Date in format 'YYYY-MM-DD HH:mm:ss HH24:MI' | Date of merchant order creation from which the data needs to be filtered. |
|  |  | to |  | Date in format 'YYYY-MM-DD HH:mm:ss HH24:MI' | Date of merchant order creation up to  which the data needs to be filtered. |
|  | merchant_order_updated_at | from |  | Date in format 'YYYY-MM-DD HH:mm:ss HH24:MI' | Date of merchant order update from which the data needs to be filtered. |
|  |  | to |  | Date in format 'YYYY-MM-DD HH:mm:ss HH24:MI' | Date of merchant order update up to  which the data needs to be filtered. |

2. Register the following plugins to enable data export:

 PLUGIN | SPECIFICATION | PREREQUISITES| NAMESPACE|
| --------------- | -------------- | ------ | -------------- |
| MerchantOrderDataEntityExporterPlugin | Exports merchant order data |   | Spryker\Zed\MerchantSalesOrderDataExport\Communication\Plugin\DataExport|
| MerchantOrderItemDataEntityExporterPlugin | Exports merchant order Items data |     | Spryker\Zed\MerchantSalesOrderDataExport\Communication\Plugin\DataExport |
|MerchantOrderExpenseDataEntityExporterPlugin  | Exports merchant order Expense data |     |Spryker\Zed\MerchantSalesOrderDataExport\Communication\Plugin\DataExport |

**src/Pyz/Zed/DataExport/DataExportDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\DataExport;

use Spryker\Zed\DataExport\DataExportDependencyProvider as SprykerDataExportDependencyProvider;
use Spryker\Zed\MerchantSalesOrderDataExport\Communication\Plugin\DataExport\MerchantOrderDataEntityExporterPlugin;
use Spryker\Zed\MerchantSalesOrderDataExport\Communication\Plugin\DataExport\MerchantOrderExpenseDataEntityExporterPlugin;
use Spryker\Zed\MerchantSalesOrderDataExport\Communication\Plugin\DataExport\MerchantOrderItemDataEntityExporterPlugin;

class DataExportDependencyProvider extends SprykerDataExportDependencyProvider
{
    /**
     * @return \Spryker\Zed\DataExportExtension\Dependency\Plugin\DataEntityExporterPluginInterface[]
     */
    protected function getDataEntityExporterPlugins(): array
    {
        return [
            new MerchantOrderDataEntityExporterPlugin(),
            new MerchantOrderItemDataEntityExporterPlugin(),
            new MerchantOrderExpenseDataEntityExporterPlugin(),
        ];
    }
}
```


3. Export data:

```bash
console data:export --config=merchant_order_export_config.yml
```

### 7) Set up behavior

Enable the following behaviors by registering the plugins:

| PLUGIN  | SPECIFICATION | PREREQUISITES | NAMESPACE |
| ------------ | ----------- | ----- | ------------ |
| TriggerEventFromCsvFileConsole |Allows for updating merchant order status via CSV input.  |  |Spryker\Zed\MerchantOms\Communication\Console |
|EventTriggerMerchantOrderPostCreatePlugin  | Triggers new events for the newly created merchant orders | |Spryker\Zed\MerchantOms\Communication\Plugin\MerchantSalesOrder  |
| MerchantOmsMerchantOrderExpanderPlugin |Expands merchant order with merchant Oms data (item state and manual events)  | | Spryker\Zed\MerchantOms\Communication\Plugin\MerchantSalesOrder |
| MerchantStateMachineHandlerPlugin | Wires merchant order updates in the State Machine module | |Spryker\Zed\MerchantOms\Communication\Plugin\StateMachine |
|MerchantOmsStateOrderItemsTableExpanderPlugin  |Expands the order item table with merchant order item state  | | Spryker\Zed\MerchantOmsGui\Communication\Plugin\Sales |
|MerchantOrderDataOrderExpanderPlugin  | Expands order data with merchant order details | | Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Sales |
|MerchantReferenceOrderItemExpanderPreSavePlugin  | Expands order item with merchant reference before saving an order item to the database | | Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Sales |
|MerchantReferencesOrderExpanderPlugin  |Expands order with merchant references from order items  | |	Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Sales  |
| MerchantReferenceShipmentExpenseExpanderPlugin | Expands expense transfer with merchant reference from items | | Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Shipment |
| ProductOfferReferenceOrderItemExpanderPreSavePlugin |Expands order item with product offer reference before saving the order item to the database  | | Spryker\Zed\ProductOfferSales\Communication\Plugin\Sales |

<details>
<summary markdown='span'>src/Pyz/Zed/Sales/SalesDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\Sales;

use Spryker\Zed\MerchantOmsGui\Communication\Plugin\Sales\MerchantOmsStateOrderItemsTableExpanderPlugin;
use Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Sales\MerchantOrderDataOrderExpanderPlugin;
use Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Sales\MerchantReferenceOrderItemExpanderPreSavePlugin;
use Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Sales\MerchantReferencesOrderExpanderPlugin;
use Spryker\Zed\ProductOfferSales\Communication\Plugin\Sales\ProductOfferReferenceOrderItemExpanderPreSavePlugin;
use Spryker\Zed\Sales\SalesDependencyProvider as SprykerSalesDependencyProvider;

class SalesDependencyProvider extends SprykerSalesDependencyProvider
{
    /**
     * @return \Spryker\Zed\SalesExtension\Dependency\Plugin\OrderExpanderPluginInterface[]
     */
    protected function getOrderHydrationPlugins(): array
    {
        return [
            new MerchantOrderDataOrderExpanderPlugin(),
            new MerchantReferencesOrderExpanderPlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\SalesExtension\Dependency\Plugin\OrderItemExpanderPreSavePluginInterface[]
     */
    protected function getOrderItemExpanderPreSavePlugins(): array
    {
        return [
            new MerchantReferenceOrderItemExpanderPreSavePlugin(),
            new ProductOfferReferenceOrderItemExpanderPreSavePlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\SalesExtension\Dependency\Plugin\OrderItemsTableExpanderPluginInterface[]
     */
    protected function getOrderItemsTableExpanderPlugins(): array
    {
        return [
            new MerchantOmsStateOrderItemsTableExpanderPlugin(),
        ];
    }
}
```
</details>

<details>
<summary markdown='span'>src/Pyz/Zed/Console/ConsoleDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\Console;

use Spryker\Zed\Kernel\Container;
use Spryker\Zed\Console\ConsoleDependencyProvider as SprykerConsoleDependencyProvider;
use Spryker\Zed\MerchantOms\Communication\Console\TriggerEventFromCsvFileConsole;

class ConsoleDependencyProvider extends SprykerConsoleDependencyProvider
{
    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return \Symfony\Component\Console\Command\Command[]
     */
    protected function getConsoleCommands(Container $container): array
    {
        return [
            new TriggerEventFromCsvFileConsole(),
        ];
    }
}
```
</details>

<details>
<summary markdown='span'>src/Pyz/Zed/MerchantSalesOrder/MerchantSalesOrderDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\MerchantSalesOrder;

use Spryker\Zed\MerchantOms\Communication\Plugin\MerchantSalesOrder\EventTriggerMerchantOrderPostCreatePlugin;
use Spryker\Zed\MerchantOms\Communication\Plugin\MerchantSalesOrder\MerchantOmsMerchantOrderExpanderPlugin;
use Spryker\Zed\MerchantSalesOrder\MerchantSalesOrderDependencyProvider as SprykerMerchantSalesOrderDependencyProvider;

class MerchantSalesOrderDependencyProvider extends SprykerMerchantSalesOrderDependencyProvider
{
    /**
     * @return \Spryker\Zed\MerchantSalesOrderExtension\Dependency\Plugin\MerchantOrderPostCreatePluginInterface[]
     */
    protected function getMerchantOrderPostCreatePlugins(): array
    {
        return [
            new EventTriggerMerchantOrderPostCreatePlugin(),
        ];
    }

    /**
     * @return \Spryker\Zed\MerchantSalesOrderExtension\Dependency\Plugin\MerchantOrderExpanderPluginInterface[]
     */
    protected function getMerchantOrderExpanderPlugins(): array
    {
        return [
            new MerchantOmsMerchantOrderExpanderPlugin(),
        ];
    }
}
```
</details>

<details>
<summary markdown='span'>src/Pyz/Zed/StateMachine/StateMachineDependencyProvider.php</summary>

```php
<?php

/**
 * This file is part of the Spryker Suite.
 * For full license information, view the LICENSE file that was distributed with this source code.
 */

namespace Pyz\Zed\StateMachine;

use Spryker\Zed\MerchantOms\Communication\Plugin\StateMachine\MerchantStateMachineHandlerPlugin;
use Spryker\Zed\StateMachine\StateMachineDependencyProvider as SprykerStateMachineDependencyProvider;

class StateMachineDependencyProvider extends SprykerStateMachineDependencyProvider
{
    /**
     * @return \Spryker\Zed\StateMachine\Dependency\Plugin\StateMachineHandlerInterface[]
     */
    protected function getStateMachineHandlers()
    {
        return [
            new MerchantStateMachineHandlerPlugin(),
        ];
    }
```
</details>

<details>
<summary markdown='span'>src/Pyz/Zed/Shipment/ShipmentDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\Shipment;

use Spryker\Zed\MerchantSalesOrder\Communication\Plugin\Shipment\MerchantReferenceShipmentExpenseExpanderPlugin;
use Spryker\Zed\Shipment\ShipmentDependencyProvider as SprykerShipmentDependencyProvider;

class ShipmentDependencyProvider extends SprykerShipmentDependencyProvider
{
    /**
     * @return \Spryker\Zed\ShipmentExtension\Dependency\Plugin\ShipmentExpenseExpanderPluginInterface[]
     */
    protected function getShipmentExpenseExpanderPlugins(): array
    {
        return [
            new MerchantReferenceShipmentExpenseExpanderPlugin(),
        ];
    }
}

```
</details>

{% info_block warningBox "Verification" %}

Make sure that the Merchant State Machine is executed on merchant orders after the order has been split.

Make sure that when retrieving an order in the *Sales* module, it is split by the merchant order and that the Order state is derived from the Merchant State Machine.

{% endinfo_block %}

## Install feature front end

Follow the steps below to install the Marketplace Order Management feature front end.

### Prerequisites

To start feature integration, integrate the required features:

| NAME | VERSION | INTEGRATION GUIDE |
| --------- | ------ | --------------|
| Spryker Core | {{page.version}} | [Spryker Core feature integration](https://documentation.spryker.com/docs/spryker-core-feature-integration) |

### 1) Install the required modules using Сomposer

If installed before, not needed.

Make sure that the following modules have been installed:

| MODULE  | EXPECTED DIRECTORY <!--for public Demo Shops--> |
| -------- | ------------------- |
| SalesMerchantPortalGui | spryker/sales-merchant-portal-gui |

### 2) Set up transfers

Apply database changes and to generate entity and transfer changes:

```bash
console transfer:generate
```

{% info_block warningBox "Verification" %}

Ensure the following transfers have been created:

| TRANSFER | TYPE | EVENT  | PATH  |
| --------- | ------- | ----- | ------------- |
| MerchantOrderTableCriteria | class | created | src/Generated/Shared/Transfer/MerchantOrderCriteriaTransfer |
| MerchantOrderItemTableCriteria | class | created | src/Generated/Shared/Transfer/DataImporterConfigurationTransfer |

{% endinfo_block %}

### 3) Set up plugins

Register the following plugins to enable widgets:

| PLUGIN | SPECIFICATION | PREREQUISITES   | NAMESPACE   |
| --------------- | -------------- | ------ | -------------- |
| OrdersMerchantDashboardCardPlugin | Adds the Sales widget to MerchantDashboard |  |   Spryker\Zed\SalesMerchantPortalGui\Communication\Plugin |

<details>
<summary markdown='span'>src/Pyz/Zed/DashboardMerchantPortalGui/DashboardMerchantPortalGuiDependencyProvider.php</summary>

```php
<?php

namespace Pyz\Zed\DashboardMerchantPortalGui;

use Spryker\Zed\DashboardMerchantPortalGui\DashboardMerchantPortalGuiDependencyProvider as SprykerDashboardMerchantPortalGuiDependencyProvider;
use Spryker\Zed\SalesMerchantPortalGui\Communication\Plugin\DashboardMerchantPortalGui\OrdersMerchantDashboardCardPlugin;

class DashboardMerchantPortalGuiDependencyProvider extends SprykerDashboardMerchantPortalGuiDependencyProvider
{
    protected function getDashboardCardPlugins(): array
    {
        return [
            new OrdersMerchantDashboardCardPlugin(),
        ];
    }
}

```

</details>

{% info_block warningBox "Verification" %}

Make sure that the following widgets have been registered by adding the respective code snippets to a Twig template:

| WIDGET | VERIFICATION |
| ----------- | ---------- |
| SalesMerchantPortalGui| Open MerchantDashboard at `http://mysprykershop.com/dashboard-merchant-portal-gui` and check that the Sales widget is available. |

{% endinfo_block %}

## Related features
Integrate the following related features:

| FEATURE | REQUIRED FOR THE CURRENT FEATURE |INTEGRATION GUIDE |
| --- | --- | --- |
| Marketplace Order Management + Order Threshold |  |[Marketplace Order Management + Order Threshold feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/marketplace-order-management-order-threshold-feature-integration.html) |
| Marketplace Order Management + Cart |  | [Marketplace Order Management + Cart feature integration](/docs/marketplace/dev/feature-integration-guides/{{page.version}}/marketplace-order-management-cart-feature-integration.html)|