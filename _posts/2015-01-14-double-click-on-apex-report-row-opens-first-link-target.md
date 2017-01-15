---
title: Double click on APEX report row opens first link target
subtitle: A small jQuery report helper
tags: [oracle, apex, jquery]
lang: en
layout: post
---

_Do you like scanning a report for certain information and then moving the mouse to the first column to click the small link for details or a edit form?
Would it be nice to double click anywhere on the row to open the first link?_

With a little help of jQuery on page zero you have the solution for all your interactive and standard reports in your application. The only additional to do is, to find out the right selectors for your reports. In the code below you find often used classes for APEX 4 and 5 - but I give you now guarantee that this will also working with your theme. Please ask your browsers developer tools and inspect your report layouts ;-)

```javascript
// Selectors depending on your theme/template
// APEX 5: 'table.a-IRR-table tr,table.uReportStandard tr'
// APEX 4: 'table.apexir_WORKSHEET_DATA tr,table.uReportStandard tr'

$(document).ready(function() {
    $(document).on('dblclick', 'table.apexir_WORKSHEET_DATA tr,table.uReportStandard tr', function() {
        $(this).find('a:first').each(function() {
            var href = $(this).attr('href');
            var onclick = $(this).attr('onclick');
            var runCode;
            if (onclick != null) {
                runCode = new Function(onclick);
                runCode();
            } else if (href != null && href.indexOf('javascript:') >= 0) {
                runCode = new Function(href.replace("javascript:", ""));
                runCode();
            } else if (href != null) {
                apex.navigation.redirect(href);
            }
        });
    });
});
```

A last comment: I tried to use a dynamic action for this, but for some reasons I was not able to get it working. Also, a simple `$(this.triggeringElement).find('td:first>a').click()` is not working. If you get it working with a dynamic action please let me know your solution.
