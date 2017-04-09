---
title: "Cheat Sheet: Oracle APEX"
subtitle: My personal notes around APEX
tags: [cheatsheet, oracle, apex]
lang: en
---

This is a living document with my personal notes around APEX.

## Base Links

- [apex.oracle.com](https://apex.oracle.com/)
- [Statement of direction](http://www.oracle.com/technetwork/testcontent/apex-sod-087560.html)
- [Vote for features](https://apex.oracle.com/vote)
- [German APEX and PL/SQL Community](https://blogs.oracle.com/apexcommunity_deutsch/)


## Usual Editors for PL/SQL Development

- [Oracle SQL Developer](http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html)
- [Allround Automations PL/SQL Developer](https://www.allroundautomations.com/plsqldev.html)
- [Toad for Oracle](https://www.quest.com/de-de/products/toad-for-oracle/)


## Unusual Editors for PL/SQL Development

- [Atom - Trent Schafer](http://apextips.blogspot.de/2017/03/my-atom-editor-oracle-plugins.html)
- [Visual Studio Code - Morten Braten](http://ora-00001.blogspot.de/2017/03/using-vs-code-for-plsql-development.html)


## Update APEX XE Dev Instance

My usual steps in a [fresh Docker container](2017-03-21-pitfalls-with-oracle-11g-xe-and-docker-on-mac-os) with 11G XE - I don't need to install the images into the EPG because I use a [Node.js based proxy server](/posts/2017-03-25-cheatsheet-nodejs) in front of the Docker container which also hosts the images:

```sql
-- pimp EPG a little bit
alter system set shared_servers=10 scope=both;
alter system set max_shared_servers=20 scope=both;
alter system set dispatchers='(PROTOCOL=TCP) (SERVICE=xeXDB) (DISPATCHERS=2)' scope=both;

-- unlock APEX public user
alter user apex_public_user account unlock;

-- install APEX
@/Users/ottmar/Documents/nodejs-proxy/apex510en/apexins.sql SYSAUX SYSAUX TEMP /i/;

-- change password of instance admin (e.g. Oracle1! because of complexity rules)
@/Users/ottmar/Documents/nodejs-proxy/apex510en/apxchpwd.sql;

-- login into internal workspace and change security settings & passwords
```

## APEX Link Syntax

```html
<a href="f?p=1_app_id:
             2_app_page_id:
             3_session:
             4_request:
             5_debug:
             6_clear_cache:
             7_item_names:
             8_item_values:
             9_printer_friendly">
```

A common example:

```html
<a href="f?p=&APP_ID.:123:&SESSION.:::123:item1,item2:value1,value2">
```

## Predifined Variables

-  APP_USER
-  APP_ID
-  APP_PAGE_ID
-  APP_UNIQUE_PAGE_ID
-  APP_SESSION or SESSION
-  APP_VERSION
-  APP_IMAGES
-  WORKSPACE_IMAGES
-  IMAGE_PREFIX

-  REQUEST
-  DEBUG
-  PRINTER_FRIENDLY

-  BROWSER_LANGUAGE
-  PUBLIC_URL_PREFIX
-  AUTHENTICATED_URL_PREFIX
-  HOME_LINK
-  LOGIN_URL
-  LOGOUT_URL
-  CURRENT_PARENT_TAB_TEXT
-  PROXY_SERVER
-  SQLERRM
-  SYSDATE_YYYYMMDD

-  #SQLERRM# (error message page process)
-  #TIME# (timing information report footer)

Usage examples:

- In a region: `Hello &APP_USER. - Welcome to Application Express`
- In PL/SQL: `htp.p('You are logged in as: ' || V('APP_USER'));`
- As bind variable: `SELECT * FROM tab WHERE user_id = :APP_USER`
