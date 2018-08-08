---
title: Quickstart Version Control for Oracle APEX Developers with PLEX.BackApp
subtitle: Bridging the gap between low code and the files first approach
tags: [project, oracle, apex, plsql, version-control]
---

Introduction
-------------

At the APEX connect 2018 in Düsseldorf, Germany, I had an interesting discussion with André Borngräber and Martin D'Souza about our table API generator projects. André and me built a PL/SQL package to generate our API's - that means code first. Although our generator is capable to produce only the code without generating the API at runtime, the latter is the normal case for me. Martin has also built a table API generator - but he does this with Node.js to create only the code in his code repository and run then a script to compile the API's in the database - this is the files first approach. The advantage of the files first approach is clear - you can't forget to put the code into your version control system and it is overwrite save when more than one developer is working on the same schema or app. And by the way: for the rest of my daily coding I also use the files first approach - except for the API generation.

It can be discussed if generated code should be version controlled or not - at least the generator code itself should. Maybe you want to put the generated code into a specific directory to separate the hand crafted from the generated one. But this discussion is not the focus of this post.

Lets have a look at the low code examples [Quick SQL][1] and [Blueprint][2] - both are usable in the APEX Application Builder. Quick SQL follows the files first approach and Blueprint normally is a code first approach. Not using code first approaches is no solution - they are ok and can save you a lot of time. The question here is how can we bridge the gap when using code first techniques? 

[1]: https://apex.oracle.com/en/quicksql/
[2]: https://docs.oracle.com/database/apex-18.1/HTMDB/using-blueprints.htm

-----

Current correction progress from Alex:

-----

The Idea
--------

I came up with the idea to have a tool, which is capable to export in one go an APEX app defifnition and all schema objects in a nice directory structure ready for use with version control. The tool generates also basic script templates to be able to automate the export of an app from your DEV system and to install it into your TEST or PROD system.

The export format can be either a zip file or an file collection for further prozessing before unloading the files into your version control repository. For the latter case see also my [previous post on how to handle the apex_t_export_files type returned by the APEX_EXPORT package with SQL*Plus][3]. The name of the tool is PLEX - it stands for PL/SQL Export Utilities and is available under the MIT license at [GitHub][4].

[3]: https://ogobrecht.github.io/posts/2018-07-25-apex-export-and-version-control
[4]: https://github.com/ogobrecht/plex

Quick Start
-----------

How to start with the tool? And for some projects not using version control the question could be - How to start at all?

For the initial app export you need first to install the package - you can download the latest code [here][5]. Compile these two files in your desired schema - the package depends on APEX version 5.1.4 or later because we reuse the packages APEX_EXPORT and APEX_ZIP:

[5]: https://github.com/ogobrecht/plex/archive/master.zip

- `PLEX.pks`
- `PLEX.pkb`

Then startup your favorite SQL Tool, connect to your app schema and fire up this query:

```sql
select plex.backapp_to_zip(p_app_id => yourAppId) from dual;
```

Save the resulting BLOB file under a name with the extension `.zip` and extract it to a local directory of your choice. You will find this directory structure and files:

```
- app_backend (only, when p_include_object_ddl is set to true)
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
- tests
- plex_README.md
- plex_runtime_log.md
```

If you like, you could fully configure your first export into the zip file. The `PLEX.BackApp` method has boolean parameters, so you need to use an inline function or create a small wrapper for the method:

```sql
-- Inline function (needs Oracle 12c or higher)
with
  function backapp return blob is 
  begin
    return plex.backapp_to_zip(
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
    );
  end backapp;
select backapp from dual;
/
```

ATTENTION: Exporting all database objects can take some time. I have seen huge runtime differences from 6 Seconds for a small app up to several hundred seconds for big apps and/or slow databases. This is normally not the problem of PLEX. If you interested in runtime statistics of plex you can inspect the delivered plex_runtime_log.md in the root of the zip file.\\
Also, the possibility to export the data of your tables into CSV files does not mean, that you should do this without thinking about it. The main reason for me to implement this feature was to track changes on catalog tables by regular calling this export feature with a sensitive table filter and max rows parameter.

I you have organized your app like described in [The Pink Database Paradigm][9] you may need to export database objects from more then one schema. This is no problem for PLEX.BackApp - all parameters are optional and you can simply logon to your second or third schema and extract only the DDL for these schemas by omitting the `p_app_id` parameter and setting `p_include_object_ddl` to `true`. Then copy the DDL files into a different directory - app_backend_schemaName?

It is up to you how you organize your version control repository and how often you export the DDL for your database objects. I personally would extract these objects only ones for existing apps and start then to follow the files first approach except for the APEX app definition. But all these thoughts on how to manage a version control repository for your app is a topic for an additional post - the target of this post is to quickstart version control as an Oracle (APEX) developer. And if you have done the installation of the tool and your first export you are almost finished - you need only to do the first commit (of course not in the database but in the version control system to ckeck-in your code ;-) 

A last word: you should inspect all the exported files and scripts and check, if this solution can work for you. If not, please let me know, what is missing, what should be done in a different way...

Feedback is highly appreciated - simply create a new [issue][6] at the [GitHub project page][7]: This is only the first (hopefully helpful) public version.

[6]: https://github.com/ogobrecht/plex/issues/new
[7]: https://github.com/ogobrecht/plex

Inspirations, Further Reading
-----------------------------

Thanks are going to:

- André Borngräber for his ability to think and discuss database topics in deep details
- Martin D'Souza for the interesting discussion about our table API generators and version control in general and for his blog post [Exporting APEX Application in SQLcl with Build Status Override][8] - PLEX.BackApp has now a parameter for this ;-)
- Philip Salvisberg for his thougts on [The Pink Database Paradigm (PinkDB)][9]
- Samual Nitsche for his thoughts on [There is no clean (database) development without Version Control][10]
- Blain Carter for his thoughts on [CI/CD for Database Developers – Export Database Objects into Version Control][11]
- Tim Hall for his article about [Generating CSV Files][12]

[8]: https://www.talkapex.com/2018/07/exporting-apex-application-in-sqlcl-with-build-status-override/
[9]: https://www.salvis.com/blog/2018/07/18/the-pink-database-paradigm-pinkdb/
[10]: https://cleandatabase.wordpress.com/2017/09/22/there-is-no-clean-database-development-without-version-control/
[11]: https://learncodeshare.net/2018/07/16/ci-cd-for-database-developers-export-database-objects-into-version-control/
[12]: https://oracle-base.com/articles/9i/generating-csv-files

Hope this helps someone.

Happy coding, apexing, version controlling :-)\\
Ottmar
