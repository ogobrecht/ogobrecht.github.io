---
title: PLEX - PL/SQL Export Utilities
subtitle: Export Oracle APEX app, schema object DDL and table data in one go
tags: [project, oracle, apex, plsql, version-control]
---

FIXME: CREATE PLEX VERSION 1.0.0 AND CORRECT DOWNLOAD LINK


I created this tool to be able to quickstart version control for existing (APEX) apps. It has currently one main function called `BackApp` with the follwing features:

- Export the app definition of an APEX app (splitted files and optional single SQL file)
- Export all object DDL from the current schema
- Export table data into CSV files
- Provide basic script templates for export/import of whole app for DEV, TEST and PROD 
- Everything in a (hopefully) nice directory structure ready for use with version control
- Return value is a file collection of type apex_t_export_files for further processing
  - Each file in the collection is represented by a record with two columns
    - `name` of type VARCHAR2(255) which is in fact the file path
    - `contents` of type CLOB
  - You can optionally zip the file collection with the helper function `to_zip`
  - See also the [APEX_EXPORT][apex_export] package and my previous post on [how to handle the apex_t_export_files type returned by the APEX_EXPORT package with SQL*Plus][prev_post]



Quick Start
------------

1. [Download the latest code][plex_download] - depends on APEX 5.1.4 or later because we use APEX_EXPORT and APEX_ZIP
1. Compile these two files in your desired schema - could also be a central tools schema, don't forget then  `grant execute on plex to public`
  - `PLEX.pks`
  - `PLEX.pkb`
1. Startup your favorite SQL Tool, connect to your app schema and fire up the following query

```sql
select plex.to_zip(plex.backapp(p_app_id => yourAppId)) from dual;
```

Save the resulting BLOB file under a name with the extension `.zip` and extract it to a local directory of your choice. You will find this directory structure and files:

FIXME: PROVIDE SCREENSHOT HERE INSTEAD OF LIST

```
- app_backend (only, when p_include_object_ddl is set to true, see next example)
  - package_bodies
  - packages
  - tables
  - ref_constraints
  - ...
- app_frontend (for the apex app files without subfolder fxxx)
  - pages
  - shared_components
  - ...
- docs
- scripts
  - logs
  - templates
    - 1_export_app_from_DEV.bat
    - 2_install_app_into_TEST.bat
    - 3_install_app_into_PROD.bat
    - export_app_custom_code.sql
    - install_app_custom_code.sql
  - install_backend_generated_by_plex.sql
  - install_frontend_generated_by_apex.sql
- tests
- plex_README.md
- plex_backapp_log.md
```

If you like, you can fully configure your first export into the zip file. The `PLEX.BackApp` method has boolean parameters, so you need to use an inline function in a pure SQL context. You can also use an anonymous PL/SQL block or you create a small SQL wrapper for the method like the inline function of the example. All parameters are optional and listet here with their default values:

```sql
-- Inline function (needs Oracle 12c or higher)
with
  function backapp return blob is 
  begin
    return plex.to_zip(plex.backapp(
      p_app_id                    => null,  -- If null, we simply skip the APEX app export.
      p_app_date                  => true,  -- If true, include export date and time in the result.
      p_app_public_reports        => true,  -- If true, include public reports that a user saved.
      p_app_private_reports       => false, -- If true, include private reports that a user saved.
      p_app_notifications         => false, -- If true, include report notifications.
      p_app_translations          => true,  -- If true, include application translation mappings and all text from the translation repository.
      p_app_pkg_app_mapping       => false, -- If true, export installed packaged applications with references to the packaged application definition. If FALSE, export them as normal applications.
      p_app_original_ids          => true,  -- If true, export with the IDs as they were when the application was imported.
      p_app_subscriptions         => true,  -- If true, components contain subscription references.
      p_app_comments              => true,  -- If true, include developer comments.
      p_app_supporting_objects    => null,  -- If 'Y', export supporting objects. If 'I', automatically install on import. If 'N', do not export supporting objects. If null, the application's include in export deployment value is used.
      p_app_include_single_file   => false, -- If true, the single sql install file is also included beside the splitted files.
      p_app_build_status_run_only => false, -- If true, the build status of the app will be overwritten to RUN_ONLY.
      -----
      p_include_object_ddl        => false, -- If true, include DDL of current user/schema and all its objects.
      p_object_filter_regex       => null,  -- Filter the schema objects with the provided object prefix.
      -----
      p_include_data              => false, -- If true, include CSV data of each table.
      p_data_as_of_minutes_ago    => 0,     -- Read consistent data with the resulting timestamp(SCN).
      p_data_max_rows             => 1000,  -- Maximum number of rows per table.
      p_data_table_filter_regex   => null,  -- Filter user_tables with the given regular expression.
      -----
      p_include_templates         => true,  -- If true, include templates for README.md, export and install scripts.
      p_include_runtime_log       => true   -- If true, generate file plex_backapp_log.md with runtime statistics.
    ));
  end backapp;
select backapp from dual;
```

ATTENTION: Exporting all database objects can take some time. I have seen huge runtime differences from 6 seconds for a small app up to several hundred seconds for big apps and/or slow databases. This is normally not the problem of PLEX. If you are interested in runtime statistics of PLEX, you can inspect the delivered `plex_backapp_log.md` in the root of the zip file.<br>
Also, the possibility to export the data of your tables into CSV files does not mean that you should do this without thinking about it. The main reason for me to implement this feature was to track changes on catalog tables by regularly calling this export feature with a sensitive table filter and max rows parameter as catalog data is often relevant in business logic.

If you have organized your app into multiple schemas like described in [The Pink Database Paradigm][pinkdb] you may need to export database objects from more then one schema. This is no problem for PLEX.BackApp as all parameters are optional - you can simply logon to your second or third schema and extract only the DDL for these schemas by omitting the `p_app_id` parameter and setting `p_include_object_ddl` to `true`. Then unload the DDL files into a different directory - for example app_backend_schemaName.

A last word: you should inspect all the exported files and scripts and check if this solution can work for you. If not, please let me know what is missing or what should be done in a different way...

Feedback is welcome - simply create a [new issue][plex_issue] at the [GitHub project page][plex_project]



Inspirations
-------------

Thanks are going to:

- Martin D'Souza for his blog post [Exporting APEX Application in SQLcl with Build Status Override][post_martin] - PLEX.BackApp has now a parameter for this ;-)
- Tim Hall for his article about [Generating CSV Files][article_tim]

If you want to hear more about the story behind this tool see also [this post][plex_story].



Hope this helps someone.

Happy coding, apexing, version controlling :-)<br>
Ottmar



[apex_export]: https://docs.oracle.com/database/apex-18.1/AEAPI/APEX_EXPORT.htm
[article_tim]: https://oracle-base.com/articles/9i/generating-csv-files
[pinkdb]: https://www.salvis.com/blog/2018/07/18/the-pink-database-paradigm-pinkdb/
[plex_download]: https://github.com/ogobrecht/plex/archive/master.zip
[plex_issue]: https://github.com/ogobrecht/plex/issues/new
[plex_project]: https://github.com/ogobrecht/plex
[plex_story]: FIXME
[post_martin]: https://www.talkapex.com/2018/07/exporting-apex-application-in-sqlcl-with-build-status-override/
[prev_post]: https://ogobrecht.github.io/posts/2018-07-25-apex-export-and-version-control
