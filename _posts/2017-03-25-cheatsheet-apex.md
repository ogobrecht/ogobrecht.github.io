---
title: "Cheat Sheet: Oracle APEX"
subtitle: My personal notes around APEX
tags: [cheatsheet, oracle, apex]
lang: en
---

This is a living document with my personal notes around APEX.

## Base links

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
