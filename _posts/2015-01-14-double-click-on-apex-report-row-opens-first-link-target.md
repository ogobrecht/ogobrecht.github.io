---
title: Double click on APEX report row opens first link target
subtitle: A page zero dynamic action for your whole application
tags: [oracle, apex]
lang: en
last_modified_at: 2017-08-12
---

_Do you like scanning a report for certain information and then moving the mouse to the first column to click the small link for details or a edit form? Would it be nice to double click anywhere on the row to open the first link?_

With a little help of a dynamic action on page zero you have the solution for all your interactive and standard reports in your application. The only additional to do is, to find out the right selectors for your reports. Here some usual ones:

- APEX 5: `table.a-IRR-table tr,table.uReportStandard tr`
- APEX 4: `table.apexir_WORKSHEET_DATA tr,table.uReportStandard tr`

I give you now guarantee that this will also working with your theme. Please ask your browsers developer tools and inspect your report layouts ;-)

Create a dynamic action with the following attributes:

- Identification
  - **Name**: On double click report row - open first link (usually edit/details)
- When
  - **Event**: Double Click
  - **Selection Type**: jQuery Selector
  - **jQuery Selector** (edit to your needs): `table.apexir_WORKSHEET_DATA tr,table.uReportStandard tr`
- True Actions
  - **Action**: Execute JavaScript Code
  - **Code**: `$(this.triggeringElement).find('td:first>a')[0].click();`
- Advanced
  - **Event Scope**: Dynamic
