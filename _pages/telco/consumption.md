---
title: Consumption
---

{% capture version_note %}
{{ site.version_note_part1 }} 1.1 {{ site.version_note_forTUA }}
{% endcapture %}

{% include docs/feature_version.html content=version_note %}

## Contents

- [Overview](#overview)
- [Business Use Cases](#business-use-cases)
- [Frontend and Backend Dependencies](#frontend-and-backend-dependencies)
- [Components](#components)
- [TM Forum APIs](#tm-forum-apis)
- [Further Reading](#further-reading)

## Overview

Customers can view their current consumption for their subscribed offerings based on the frequency of the current contract term (that is, monthly consumption). By understanding usage, the customers can manage their subscriptions with proper authorization. Entries for the usage consumption are created and updated from backend systems by average service usage for subscribed product. 

## Business Use Cases

### Viewing Consumption Information

From subscriptions, customers can drill down on subscribed offerings to view consumption data as a pie chart, a table format, or both.

### Usage Consumption

If enabled, consumption information for each subscribed product can be viewed in either grid format, pie chart format, or both.


## Frontend and Backend Dependencies

| Dependency                                	| Detail                                                 	|
|--------------------------------------------	|--------------------------------------------------------	|
| Recipe                                     	| b2c_telco_spa                                          	|
| Minimum version of backend TUA             	| TUA Release 2003 (latest patch is required)           	|
| Minimum   version of core commerce backend 	| SAP Commerce release 1905 (latest patch is recommended) 	|

## Components

The following components must be enabled in the TUA backoffice to appear on the Spartacus TUA storefront:

| Component   Name                  | Status | Description                                                                                                                           |
|-----------------------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------|
| UsageConsumptionGridComponent     | New    | Displays usage consumption for   subscriptionBase Ids in a grid format                                                                |
| UsageConsumptionPieChartComponent | New    | Displays usage consumption for   subscriptionBase Ids in a Pie chart format with usage in percentage is also   displayed.             |
| UsageConsumptionHeaderComponent   | New    | Displays usage consumption heading and   navigation button to navigate to the subscription list page from subscription   details page |

## TM Forum APIs

| Entity Exposed for   CPI 	| TUA API                                    	| Description                                                              	|
|--------------------------	|--------------------------------------------	|--------------------------------------------------------------------------	|
| TmaSubscriptionBase      	| GET /subscriptionBase                      	| Shows a list of of subscription base in the Subscription Details screen  	|
| TmaSubscribedProduct     	| GET/product/{productId}                    	| Shows a list of subscription products in the Subscription Details screen 	|
| TmaSubscriptionAccess    	| GET /subscriptionbase/{subscriptionBaseId} 	| Shows details of subscription base                                       	|
| TmaSubscriptionUsage     	| GET/usageConsumptionReport                 	| Shows the usage consumption for a subscriptionBase Id                    	|

For more information, see [TM Forum APIs](https://help.sap.com/viewer/f59b0ac006d746caaa5fb599b4270151/2007/en-US/d46b30b30eca4d4d8ddd20ad833d77f9.html).

## Further Reading

- [Create Average Service Usage for Subscribed Product](https://help.sap.com/viewer/32f0086927f44c9ab1199f1dab8833cd/2007/en-US/ba5f222fb5814829bd74eaf6e6505a9f.html)
- [Making Components Visible](https://help.sap.com/viewer/9d346683b0084da2938be8a285c0c27a/2005/en-US/1cea3b2cb3334fc085dda9cc070ad6ac.html)