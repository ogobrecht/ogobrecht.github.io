---
title: PLEX - The Story behind The Project
subtitle: Bridging the gap between the code first and the files first approach
tags: [oracle, apex, plsql, version-control]
---

FIXME: CREATE PLEX VERSION 1.0.0 AND CORRECT DOWNLOAD LINK


Introduction
-------------

PLEX is a PL/SQL package and stands for "PL/SQL Export Utilities". The current main feature is the method `BackApp`, which is able to export an Oracle APEX app, all schema object DDL and optional table data in one go. The package itself and how to quickstart with it is described in detail in my [previous post][post_plex_project]. In this post I will discuss why this package was created. I will also show you the difference between the code first and files first approach and why it is not so easy to follow only the preferred way of the files first approach. And for the buzzword bashing: We have Low Code also in this post ;-)



Part 1 - The Early Days
------------------------

I the past I had to take over some APEX apps. To figure out which parts of an app are built in what way take often more time then you would expect. Also to setup a version control repository if none is existing takes time. The same is valid for building install/deployment scripts. And for each new app you will find better ways to organize the things. It is some kind of continuous improvement process. I thought often it would be nice to have some tool to shorten the time for doing the basic things.

And clear, here and there you have a little time to improve things. One of those improvements was a first approach by my collegue André Borngräber to provide an package with methods for extracting schema objects with dbms_metadata per object_type.



Part 2 - This Spring - A Totally Different Story
-------------------------------------------------

At the APEX connect 2018 in Düsseldorf, Germany, I had an interesting discussion with André Borngräber and Martin D'Souza about our table API generator projects. André and me built a [PL/SQL package to generate our table API's][om_tapigen] - that means code first. Although our generator is capable to produce only the code without generating the API at runtime, the latter is the normal case for me. Martin has also built a [table API generator][oos_tapi] - but he does this with Node.js to create only the code in his repository and then run a script to compile the API's in the database - this is the files first approach. The advantage of the files first approach is clear - you can't forget to put the code into your version control system and it is overwrite save when more than one developer is working on the same schema or app. And by the way: for the rest of my daily coding I also use the files first approach - except for the API generation.

It can be discussed if generated code should be version controlled or not - at least the generator itself should. Maybe you want to put the generated code into a specific directory to separate the hand crafted from the generated one. But this discussion is not the focus of this post.

Lets have a look at the low code examples [Quick SQL][quick_sql] and [Blueprint][blueprint] - both are usable in the APEX Application Builder. 

Quick SQL allows you to generate data models with a markdown like syntax - you can save the scripts (also in your version control system if you like) and run it directly in the Application Builder to create your data model. It can be used as a files first approach but I would say this is mainly a code first approach, specifically if you have no access to the system with a SQL command line tool to run the resulting scripts in a different way. You can bundle it with the application via the supporting objects feature - ok, ok - but I think you get my point ...

Blueprint is integrated in the App Creation Wizard and you can inspect and edit the configuration and therefore you can save it as a file in your VCS. But you need to use the Application Builder to generate your app. As a result, you probably have new objects in your database schema without the DDL scripts for this objects - this is clearly a code first approach.

With both tools you additionally have the problem that you don't have the generator code - you can't check-in the generator into your version control system.

Not using code first approaches is not a solution - they are ok and can save you a lot of time. The question here is how can we bridge the file (and documentation) gap when using code first techniques? 



Part 3 - Putting Things Together
---------------------------------

Can you see the similarities? In both Scenarios there is a need to extract (maybe unknown) objects (not created by yourself) into version control to understand and document what you got from other (people or generators).

It was time to sit down and bring the ideas together: APEX app export and schema object DDL together, optional some table data and everything in a nice directory structure ready to use with version control. On top it would be nice to have some basic script templates for standard tasks like export the app from the DEV system, import the app into the TEST/PROD system.<br>
For the directory structure it should be possible for the users to easily change directory names or content before unloading the files into their version control systems.

At the time I was starting with the implementation there was luckily a new package called APEX_EXPORT available (since APEX 5.1.4), which returns a file collection type - ideally for the last mentioned point to modify or enrich the output. PLEX is reusing this type and also APEX_EXPORT for the part of exporting the app definition - you will find all parameters from APEX_EXPORT also in PLEX - with a slightly different name, because PLEX has more parameters and I had to find a way to group parameter names together for the different export sections.



The End of The Story
---------------------

After some hot weeks (temperature and less freetime) PLEX is now available under the MIT license.

Feedback is welcome - simply create a [new issue][plex_issue] at the [GitHub project page][plex_project]



Inspirations, Further Reading
-----------------------------

Thanks are going to:

- André Borngräber for his ability to think and discuss database topics in deep details
- Martin D'Souza for the interesting discussion about our table API generators and version control in general
- Samual Nitsche for his thoughts on [There is no clean (database) development without Version Control][post-samuel]
- Blain Carter for his thoughts on [CI/CD for Database Developers – Export Database Objects into Version Control][post-blain]


Hope PLEX.BackApp helps someone else.

Happy coding, apexing, version controlling :-)<br>
Ottmar



[blueprint]: https://docs.oracle.com/database/apex-18.1/HTMDB/using-blueprints.htm
[om_tapigen]: https://github.com/OraMUC/table-api-generator
[oos_tapi]: https://github.com/OraOpenSource/oos-tapi
[plex_issue]: https://github.com/ogobrecht/plex/issues/new
[plex_project]: https://github.com/ogobrecht/plex
[post_plex_project]: FIXME
[post-blain]: https://learncodeshare.net/2018/07/16/ci-cd-for-database-developers-export-database-objects-into-version-control/
[post-samuel]: https://cleandatabase.wordpress.com/2017/09/22/there-is-no-clean-database-development-without-version-control/
[quick_sql]: https://apex.oracle.com/en/quick_sql/
