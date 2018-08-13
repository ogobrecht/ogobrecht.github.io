---
title: PLEX.BackApp - Quickstart Version Control for Oracle APEX Developers
subtitle: Bridging the gap between the code first and the files first approach
tags: [project, oracle, apex, plsql, version-control]
---

FIXME: CREATE PLEX VERSION 1.0.0 AND CORRECT DOWNLOAD LINK

Introduction
-------------

At the APEX connect 2018 in Düsseldorf, Germany, I had an interesting discussion with André Borngräber and Martin D'Souza about our table API generator projects. André and me built a [PL/SQL package to generate our table API's][omtapigen] - that means code first. Although our generator is capable to produce only the code without generating the API at runtime, the latter is the normal case for me. Martin has also built a [table API generator][oostapi] - but he does this with Node.js to create only the code in his repository and then run  a script to compile the API's in the database - this is the files first approach. The advantage of the files first approach is clear - you can't forget to put the code into your version control system and it is overwrite save when more than one developer is working on the same schema or app. And by the way: for the rest of my daily coding I also use the files first approach - except for the API generation.

It can be discussed if generated code should be version controlled or not - at least the generator itself should. Maybe you want to put the generated code into a specific directory to separate the hand crafted from the generated one. But this discussion is not the focus of this post.

Lets have a look at the low code examples [Quick SQL][quicksql] and [Blueprint][blueprint] - both are usable in the APEX Application Builder. 

Quick SQL allows you to generate data models with a markdown like syntax - you can save the scripts (also in your version control system if you like) and run it directly in the Application Builder to create your data model. It can be used as a files first approach but I would say this is mainly a code first approach, specifically if you have no access to the system with a SQL command line tool to run the resulting scripts in a different way. You can bundle it with the application via the supporting objects feature - ok, ok - but I think you get my point ...

Blueprint is integrated in the App Creation Wizard and you can inspect and edit the configuration and therefore you can save it as a file in your VCS. But you need to use the Application Builder to generate your app. As a result, you probably have new objects in your database schema without the DDL scripts for this objects - this is clearly a code first approach.

With both tools you additionally have the problem that you don't have the generator code - you can't check-in the generator into your version control system.

Not using code first approaches is not a solution - they are ok and can save you a lot of time. The question here is how can we bridge the file (and documentation) gap when using code first techniques? 



The Idea
---------

I came up with the idea to have a tool, which is capable to export an APEX app defifnition and all schema objects in one go in a nice directory structure ready to use with version control. The tool also generates basic script templates to be able to automate the export of an app from your DEV system and to install it into your TEST or PROD system.

The export format is a file collection of type `apex_t_export_files` - we reuse the output type of the [APEX_EXPORT package][apexexport], which is used in the background to export the app definition. Each file in the collection is represented by a record with two columns: `name` of type VARCHAR2(255) which is in fact the file path and `contents` of type CLOB.

You can do your own processing on this file collection before you unload the files into your version control repository. For example change file (directory) names to align it to your local repository structure or add new files with some sort of reports - whatever you like and fits into a CLOB. Also see my previous post on [how to handle the apex_t_export_files type returned by the APEX_EXPORT package with SQL*Plus][prevpost]. Alternatively, you can convert the file collection to a zip file with a helper function - see the code examples in the next section.

I called the tool PLEX - a short name for PL/SQL Export Utilities. It is available under the MIT license at [GitHub][plex].



Quick Start
------------

How to start with the tool? And for some projects not using version control the question could be: How to start at all?

For the initial app export you first need to install the package - you can download the latest code [here][plexdownload]. The package depends on APEX version 5.1.4 or later because we use the packages APEX_EXPORT and APEX_ZIP. Compile these two files in your desired schema (could also be a central tools schema, don't forget the `grant execute on plex to public`):

- `PLEX.pks`
- `PLEX.pkb`

Then startup your favorite SQL Tool, connect to your app schema and fire up this query:

```sql
select plex.to_zip(plex.backapp(p_app_id => yourAppId)) from dual;
```

Save the resulting BLOB file under a name with the extension `.zip` and extract it to a local directory of your choice. You will find this directory structure and files:

```
- app_backend (only, when p_include_object_ddl is set to true)
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

If you like, you can fully configure your first export into the zip file. The `PLEX.BackApp` method has boolean parameters, so you need to use an inline function or create a small wrapper for the method:

```sql
-- Inline function (needs Oracle 12c or higher)
with
  function backapp return blob is 
  begin
    return plex.to_zip(plex.backapp(
      p_app_id                    => 100,  -- If null, we simply skip the APEX app export.
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

      p_include_object_ddl        => false, -- If true, include DDL of current user/schema and all its objects.
      p_object_filter_regex       => null,  -- Filter the schema objects with the provided object prefix.

      p_include_data              => false, -- If true, include CSV data of each table.
      p_data_as_of_minutes_ago    => 0,     -- Read consistent data with the resulting timestamp(SCN).
      p_data_max_rows             => 1000,  -- Maximum number of rows per table.
      p_data_table_filter_regex   => null,  -- Filter user_tables with the given regular expression.

      p_include_templates         => true,  -- If true, include templates for README.md, export and install scripts.
      p_include_runtime_log       => true   -- If true, generate file plex_backapp_log.md with runtime statistics.
    ));
  end backapp;
select backapp from dual;
```

ATTENTION: Exporting all database objects can take some time. I have seen huge runtime differences from 6 seconds for a small app up to several hundred seconds for big apps and/or slow databases. This is normally not the problem of PLEX. If you are interested in runtime statistics of PLEX, you can inspect the delivered `plex_backapp_log.md` in the root of the zip file.\\
Also, the possibility to export the data of your tables into CSV files does not mean that you should do this without thinking about it. The main reason for me to implement this feature was to track changes on catalog tables by regularly calling this export feature with a sensitive table filter and max rows parameter.

If you have organized your app like described in [The Pink Database Paradigm][pinkdb] you may need to export database objects from more then one schema. This is no problem for PLEX.BackApp - all parameters are optional and you can simply logon to your second or third schema and extract only the DDL for these schemas by omitting the `p_app_id` parameter and setting `p_include_object_ddl` to `true`. Then copy the DDL files into a different directory - for example app_backend_schemaName

It is up to you how you organize your version control repository and how often you export the DDL for your database objects. I personally would extract these objects only once for existing apps and start then to follow the files first approach except for the APEX app definition. But all these thoughts on how to manage a version control repository for your app is a topic for an additional post - the target of this post is to quickstart version control as an Oracle (APEX) developer. And if you have done the installation of the tool and your first export you are almost finished - you only need to do the first commit (of course not in the database but in the version control system to ckeck-in your code ;-) 

A last word: you should inspect all the exported files and scripts and check if this solution can work for you. If not, please let me know what is missing or what should be done in a different way...

Feedback is welcome - simply create a [new issue][plexissue] at the [GitHub project page][plex]



Inspirations, Further Reading
-----------------------------

Thanks are going to:

- André Borngräber for his ability to think and discuss database topics in deep details
- Martin D'Souza for the interesting discussion about our table API generators and version control in general and for his blog post [Exporting APEX Application in SQLcl with Build Status Override][post-martin] - PLEX.BackApp has now a parameter for this ;-)
- Philip Salvisberg for his thougts on [The Pink Database Paradigm (PinkDB)][pinkdb]
- Samual Nitsche for his thoughts on [There is no clean (database) development without Version Control][post-samuel]
- Blain Carter for his thoughts on [CI/CD for Database Developers – Export Database Objects into Version Control][post-blain]
- Tim Hall for his article about [Generating CSV Files][article-tim]



Hope this helps someone.

Happy coding, apexing, version controlling :-)\\
Ottmar

[omtapigen]: https://github.com/OraMUC/table-api-generator
[oostapi]: https://github.com/OraOpenSource/oos-tapi
[quicksql]: https://apex.oracle.com/en/quicksql/
[blueprint]: https://docs.oracle.com/database/apex-18.1/HTMDB/using-blueprints.htm
[apexexport]: https://docs.oracle.com/database/apex-18.1/AEAPI/APEX_EXPORT.htm
[prevpost]: https://ogobrecht.github.io/posts/2018-07-25-apex-export-and-version-control
[plex]: https://github.com/ogobrecht/plex
[plexissue]: https://github.com/ogobrecht/plex/issues/new
[plexdownload]: https://github.com/ogobrecht/plex/archive/master.zip
[pinkdb]: https://www.salvis.com/blog/2018/07/18/the-pink-database-paradigm-pinkdb/
[post-martin]: https://www.talkapex.com/2018/07/exporting-apex-application-in-sqlcl-with-build-status-override/
[post-samuel]: https://cleandatabase.wordpress.com/2017/09/22/there-is-no-clean-database-development-without-version-control/
[post-blain]: https://learncodeshare.net/2018/07/16/ci-cd-for-database-developers-export-database-objects-into-version-control/
[article-tim]: https://oracle-base.com/articles/9i/generating-csv-files
