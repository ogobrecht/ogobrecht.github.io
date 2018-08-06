---
title: Quickstart Version Control for Oracle Developers
subtitle: Bridging the gap between low code and the files first approach
tags: [project, plsql, apex]
---

At the APEX connect 2018 in Düsseldorf, Germany, I had an interesting discussion mit my collegue André Borngräber and Martin D'Souza about our table API generator project. André and me built a PL/SQL package to generate our API's - that means code first. Although our generator is capable to produce only the code without generating the API at runtime the latter is the normal case for me. Martin has built also a table API generator - but he does this with Node.js to create only the code in his version controlled code repository and run then a script to compile the API's in the database - this is the files first approach. The advantage of the files first approach is clear - you can't forget to put the code into your version control system and it is overwrite save when more then one developer is working on the same schema. And by the way: for the rest of my daily coding I use also the files first approach - except for the API generation.

It can be discussed, if generated code should be version controlled or not - at least the generator package itself should. Maybe you want to put the generated code into a specific directory to separate the hand crafted one from the generated one. But this discussion is not the focus of this post.

Low code is currently a hype and you can save much time by using for example [Quick SQL][1] and [Blueprint][2] in an APEX project. Quick SQL follows the files first approach and Blueprint is normally a code first approach. Not using code first approaches is no solution - they are ok and can save you much time. The question is here how can we bridge the gap when using code first techniques? 

[1]: fixme
[2]: fixme

I came up with the idea to have a tool, which is capable to export in one go an APEX app defifnition and all schema objects in a nice directory structure ready for use with version control. The tool generates also basic script templates to be able to automate the export of an app from your DEV system and to install it into your TEST or PROD system. The export format can be either a zip file or an file collection for further prozessing before unloading the files into your version control repository. The name of the tool is PLEX - it stands for PL/SQL Export Utilities and is available under the MIT license at [GitHub][3].

[3]: fixme

How to start with the tool? And for some projects not using version control the question could be - How to start at all?

The little Problem here is, that the tool delivers script templates to do the export with SQL*Plus but to use it you have to export the first time your code in a different way to get the script templates. That is the reason why plex has the ability to also export a zip file (BLOB).

- Install the PLEX package itself - it depends on APEX version 5.1.4 or later (we use APEX_EXPORT and APEX_ZIP)
- Startup SQL Developer, connect with your desired schema and fire up this query:

```sql
select plex.backapp_to_zip(p_app_id => yourAppId) from dual;
```

Save the resulting BLOB file under a name with the extension `.zip` and extract it to a local directory of your choice. You will find this directory structure and files:

```
- app_backend (only, when p_include_object_ddl is set to true)
- app_frontend (for the splitted files without subfolder fxxx)
  - pages
  - shared_components
  - ...
- docs
- scripts
  - logs
  - templates
    - 1_xxx.bat
    - 2_xxx.bat
    - 3_xxx.bat
    - xxx.sql
    - xxx.sql
- tests
- plex_README.md
- plex_runtime_log.md
```

If you like, you could fully configure your first export into the zip file. The plex.backapp method has boolean parameters, so you need to use an inline function or create a small wrapper for the plex.backapp method:

```sql
-- Inline function (needs Oracle 12c)
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

Or you use some PL/SQL logic, save it to a table, whatever you like... Please pay attention, exporting all database objects can take some time. I have seen huge runtime differences from 6 Seconds for a small app up to several hundred seconds for big apps or slower databases. This is normally not the problem of PLEX. If you interested in runtime statistics of plex you can inspect the delivered plex_runtime_log.md in the root of the zip file.

... fixme

Happy exporting :-)  
Ottmar

[1]: /posts/2017-03-05-dokuwiki-plugin-revealjs
[2]: https://jekyllrb.com
[3]: https://github.com/nijikokun/minami
[4]: https://shopify.github.io/liquid/
[5]: https://2017.doag.org/en/home/
[6]: https://ogobrecht.github.io/hydebar/features#quickstart-online-in-5-minutes
[7]: https://ogobrecht.github.io/hydebar
