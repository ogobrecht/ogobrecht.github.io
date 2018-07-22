---
title: APEX_EXPORT and Version Control
subtitle: How to export the splitted APEX app definition with SQL*Plus
tags: [oracle, apex, version-control, sqlplus]
---

Since years it is possible to export an APEX app definition with the help of APEXExport, a Java utility delivered within the APEX install zip file. There is also the possibility to split the file into its components like pages, plugins and so on. There are some blog postings available how to do this - simply ask Google. Also the Java based SQLcl has the capability to do the export of an APEX app directly.

So, why bother with a different way to export and split an APEX application?

Since APEX 5.1.4 there is a new PL/SQL package APEX_EXPORT, which can be used to get a file collection of the application - one big file or the splitted ones. Unfortunately in the API docs there is (as of this writing) only one example available how to export the single file within SQL*Plus - no example to handle the splitted files. Sidenote: Despite the subtitle of this post all examples should also run in SQLcl...

But again, why discuss this if there are already options to do it?

Because of the possibility to modifiy the APEX_EXPORT file collection before fetch it into the file system. Imagine you have a different repository structure and the delivered file structure of the splitted files does not match your needs or you want to have all install files in one scripts directory of your repo and need therefore a relocation of the app install script. Another use case is to enrich the file collection with additional data or objects. This was possible in the past also with some postprocessing outside the database, but now we are able to do this wthin our DB session and PL/SQL. I started already an open source project to leaverage these possibilities - more about this in my next post...

So, and how to do it?

First, I have to describe, what the sctructure of each file in the collection is. Dead simple: a record type with two columns: `name` of type VARCHAR2(255) which is in fact the file path and `contents` of type CLOB.

The desired file structure:

```
- app_backend
- app_frontend (for the splitted files without subfolder fxxx)
  - pages
  - shared_components
  - ...
- docs
- scripts
  - logs
    - temp_export_files.sql (our intermediate script file)
    - ...
  - export_frontend.bat (our os shell script to start the export)
  - export_frontend.sql (our export script)
  - install_frontend.sql (the generated install file from apex_export)
- tests
```

Here the idea: 

- We create a script file to get the file collection, iterate over the collection and modify the content regarding our needs
- To unload the files with the spool command in SQL*Plus we need it accessable via SQL - therefore we put the files into a global temporary table
- We need to create an intermediate script file to unload the files (select the clob content)
- We need also to create host commands for the needed directories because the spool command does NOT create missing directories
- We spool our progress to a log file, so the caller of the script can see what is going on during runtime

```sql
-- file: export_frontend.sql
set verify off feedback off heading off 
set trimout on trimspool on pagesize 0 linesize 5000 long 100000000 longchunksize 32767
whenever sqlerror exit sql.sqlcode rollback


-- https://blogs.oracle.com/opal/sqlplus-101-substitution-variables
define logfile = "logs/export_frontend_from_&1._&2._&3..log"
spool "&logfile." replace


prompt
prompt Start frontend export on &1.
prompt ==================================================
prompt Create global temporary table temp_export_files if not exist
BEGIN
  FOR i IN (SELECT 'TEMP_EXPORT_FILES' AS object_name FROM dual 
            MINUS
            SELECT object_name FROM user_objects) LOOP
    EXECUTE IMMEDIATE q'[
--------------------------------------------------------------------------------
CREATE GLOBAL TEMPORARY TABLE temp_export_files (
  name       VARCHAR2(255),
  contents   CLOB
) ON COMMIT DELETE ROWS
--------------------------------------------------------------------------------
    ]';
  END LOOP;
END;
/


prompt Do the frontend export, relocate files and save to temporary table
DECLARE
  l_app_id  pls_integer;
  l_files   apex_t_export_files;
BEGIN
  l_files   := apex_export.get_application(
    p_application_id          => l_app_id,
    p_split                   => true,
    p_with_date               => true,
    p_with_ir_public_reports  => true,
    p_with_ir_private_reports => false,
    p_with_ir_notifications   => false,
    p_with_translations       => true,
    p_with_pkg_app_mapping    => false,
    p_with_original_ids       => true,
    p_with_no_subscriptions   => false,
    p_with_comments           => true,
    p_with_supporting_objects => 'Y');
  FOR i IN 1..l_files.count LOOP
    -- relocate files to own project structure
    l_files(i).name := replace(
      l_files(i).name, 
      'f'||l_app_id||'/application/',
      '../app_frontend/'
    );
    -- correct prompts for relocation
    l_files(i).contents := replace(
      l_files(i).contents, 
      'prompt --application/',
      'prompt --app_frontend/'
    );
    -- special handling for install file
    IF l_files(i).name = 'f'||l_app_id||'/install.sql' THEN
      l_files(i).name := 'install_frontend.sql';
      l_files(i).contents := replace(
        replace(
          l_files(i).contents,
          '@application/',
          '@../app_frontend/'
        ),
        'prompt --install',
        'prompt --install_frontend'
      );
    END IF;
  END LOOP;
  FORALL i IN 1..l_files.count
    INSERT INTO temp_export_files VALUES (
      l_files(i).name,
      l_files(i).contents
    );
END;
/


prompt Create intermediate script file to unload the table content into files
spool off
set termout off serveroutput on
spool "logs/temp_export_files.sql"
BEGIN
  -- create host commands for the needed directories (spool does NOT create missing directories)
  FOR i IN (
    WITH t1 AS (
      SELECT regexp_substr(name, '^((\w|\.)+\/)+') AS dir
        FROM temp_export_files
    )
    SELECT DISTINCT
            dir,
            -- This is for Windows to create a directory and suppress warning if it exist.
            -- Align the command to your operating system:
            'host mkdir "' || replace(dir,'/','\') || '" 2>NUL' AS mkdir
      FROM t1
      WHERE dir IS NOT NULL
  ) LOOP
    dbms_output.put_line('set termout on');
    dbms_output.put_line('spool "&logfile." append');
    dbms_output.put_line('prompt --create directory if not exist: ' || i.dir);
    dbms_output.put_line('spool off');
    dbms_output.put_line('set termout off');
    dbms_output.put_line(i.mkdir);
    dbms_output.put_line('-----');
  END LOOP;

  -- create the spool calls for unload the files
  FOR i IN (SELECT * FROM temp_export_files) LOOP
    dbms_output.put_line('set termout on');
    dbms_output.put_line('spool "&logfile." append');
    dbms_output.put_line('prompt --' || i.name);
    dbms_output.put_line('spool off');
    dbms_output.put_line('set termout off');
    dbms_output.put_line('spool "' || i.name || '"');
    dbms_output.put_line('select contents from temp_export_files where name = ''' || i.name || ''';');
    dbms_output.put_line('spool off');
    dbms_output.put_line('-----');
  END LOOP;
END;
/
spool off
set termout on serveroutput off
spool "&logfile." append


prompt Call the intermediate script file to save the files
spool off
@logs/temp_export_files.sql
set termout on serveroutput off
spool "&logfile." append


prompt Delete files from the global temporary table
COMMIT;


prompt ==================================================
prompt Export DONE :-) 
prompt
```

To run this script file on your operating system you need some shell script to call it. Here an example for Windows:

```bat
echo off
setlocal
set areyousure = N

rem align delimiters to your os locale
for /f "tokens=1-3 delims=. " %%a in ('date /t') do (set mydate=%%c%%b%%a)
for /f "tokens=1-2 delims=:"  %%a in ('time /t') do (set mytime=%%a%%b)

:PROMPT
echo.
echo. 
set /p areyousure=Export HR App/Schema from DEV (Y/N)?

if /i %areyousure% neq y goto END
set NLS_LANG=AMERICAN_AMERICA.AL32UTF8
set /p password_db_user=Please enter password for HR on DEV:
echo exit | sqlplus -S HR/%password_db_user%@localhost:1521/orcl @export_frontend.sql DEV %mydate% %mytime%

:END
pause
```

Hope this helps someone.

Happy exporting :-)  
Ottmar

