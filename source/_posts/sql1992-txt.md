---
title: sql1992.txt
toc: true
date: 2021-04-11 15:34:55
tags:
categories:
---

源地址：http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt

(This is stale and you may wish to find a more up-to-date copy, but it is preserved here for posterity. Enjoy -- Daria 24 Nov 2017)
 









                                                    Information Technology -

                                                       Database Language SQL

                                         (Proposed revised text of DIS 9075)






                                                                   July 1992

 


















         (Second Informal Review Draft) ISO/IEC 9075:1992, Database
         Language SQL- July 30, 1992








































         Digital Equipment Corporation
         Maynard, Massachusetts

 












        Contents                                                       Page


        Foreword.........................................................xi

        Introduction.....................................................xiii

        1  Scope ........................................................ 1

        2  Normative references ......................................... 3

        3  Definitions, notations, and conventions ...................... 5

        3.1  Definitions ................................................ 5

        3.1.1Definitions taken from ISO/IEC 10646 ....................... 5

        3.1.2Definitions taken from ISO 8601 ............................ 5

        3.1.3Definitions provided in this International Standard ........ 5

        3.2  Notation ................................................... 7

        3.3  Conventions ................................................ 9

        3.3.1Informative elements ....................................... 9

        3.3.2Specification of syntactic elements ........................ 9

        3.3.3Specification of the Information Schema ....................10

        3.3.4Use of terms ...............................................10

        3.3.4Exceptions .................................................10

        3.3.4Syntactic containment ......................................11

        3.3.4Terms denoting rule requirements ...........................12

        3.3.4Rule evaluation order ......................................12

        3.3.4Conditional rules ..........................................13

        3.3.4Syntactic substitution .....................................13

        3.3.4Other terms ................................................14

        3.3.5Descriptors ................................................14

        3.3.6Index typography ...........................................15

        3.4  Object identifier for Database Language SQL ................16

        4  Concepts .....................................................19

        4.1  Data types .................................................19

        4.2  Character strings ..........................................20

        4.2.1Character strings and collating sequences ..................20

        4.2.2Operations involving character strings .....................22

        4.2.2Operators that operate on character strings and return char-
             acter strings...............................................22

        4.2.2Other operators involving character strings ................23

        4.2.3Rules determining collating sequence usage .................23

        4.3  Bit strings ................................................26

        4.3.1Bit string comparison and assignment .......................27

        4.3.2Operations involving bit strings ...........................27

        4.3.2Operators that operate on bit strings and return bit strings
             ............................................................27

        4.3.2Other operators involving bit strings ......................27

        ii  Database Language SQL

 









         4.4  Numbers ....................................................27

         4.4.1Characteristics of numbers .................................28

         4.4.2Operations involving numbers ...............................29

         4.5  Datetimes and intervals ....................................29

         4.5.1Datetimes ..................................................30

         4.5.2Intervals ..................................................32

         4.5.3Operations involving datetimes and intervals ...............34

         4.6  Type conversions and mixing of data types ..................34

         4.7  Domains ....................................................35

         4.8  Columns ....................................................36

         4.9  Tables .....................................................37

         4.10 Integrity constraints ......................................40

         4.10.Checking of constraints ....................................41

         4.10.Table constraints ..........................................41

         4.10.Domain constraints .........................................43

         4.10.Assertions .................................................43

         4.11 SQL-schemas ................................................44

         4.12 Catalogs ...................................................45

         4.13 Clusters of catalogs .......................................45

         4.14 SQL-data ...................................................45

         4.15 SQL-environment ............................................46

         4.16 Modules ....................................................46

         4.17 Procedures .................................................47

         4.18 Parameters .................................................47

         4.18.Status parameters ..........................................47

         4.18.Data parameters ............................................48

         4.18.Indicator parameters .......................................48

         4.19 Diagnostics area ...........................................48

         4.20 Standard programming languages .............................49

         4.21 Cursors ....................................................49

         4.22 SQL-statements .............................................51

         4.22.Classes of SQL-statements ..................................51

         4.22.SQL-statements classified by function ......................52

         4.22.Embeddable SQL-statements ..................................55

         4.22.Preparable and immediately executable SQL-statements .......56

         4.22.Directly executable SQL-statements .........................58

         4.22.SQL-statements and transaction states ......................59

         4.23 Embedded syntax ............................................61

         4.24 SQL dynamic statements .....................................61

         4.25 Direct invocation of SQL ...................................64

         4.26 Privileges .................................................64

         4.27 SQL-agents .................................................66

         4.28 SQL-transactions ...........................................67

         4.29 SQL-connections ............................................70

         4.30 SQL-sessions ...............................................72

                                                      Table of Contents  iii

 









         4.31 Client-server operation ....................................74

         4.32 Information Schema .........................................75

         4.33 Leveling ...................................................75

         4.34 SQL Flagger ................................................76

         5  Lexical elements .............................................79

         5.1  <SQL terminal character> ...................................79

         5.2  <token> and <separator> ....................................82

         5.3  <literal> ..................................................89

         5.4  Names and identifiers ......................................98

         6  Scalar expressions ...........................................107

         6.1  <data type> ................................................107

         6.2  <value specification> and <target specification> ...........114

         6.3  <table reference> ..........................................118

         6.4  <column reference> .........................................121

         6.5  <set function specification> ...............................124

         6.6  <numeric value function> ...................................128

         6.7  <string value function> ....................................132

         6.8  <datetime value function> ..................................139

         6.9  <case expression> ..........................................141

         6.10 <cast specification> .......................................144

         6.11 <value expression> .........................................155

         6.12 <numeric value expression> .................................157

         6.13 <string value expression> ..................................160

         6.14 <datetime value expression> ................................165

         6.15 <interval value expression> ................................168

         7  Query expressions ............................................173

         7.1  <row value constructor> ....................................173

         7.2  <table value constructor> ..................................176

         7.3  <table expression> .........................................177

         7.4  <from clause> ..............................................178

         7.5  <joined table> .............................................180

         7.6  <where clause> .............................................185

         7.7  <group by clause> ..........................................187

         7.8  <having clause> ............................................189

         7.9  <query specification> ......................................191

         7.10 <query expression> .........................................196

         7.11 <scalar subquery>, <row subquery>, and <table subquery> ....203

         8  Predicates ...................................................205

         8.1  <predicate> ................................................205

         8.2  <comparison predicate> .....................................207

         8.3  <between predicate> ........................................211

         8.4  <in predicate> .............................................212

         iv  Database Language SQL

 









         8.5  <like predicate> ...........................................214

         8.6  <null predicate> ...........................................218

         8.7  <quantified comparison predicate> ..........................220

         8.8  <exists predicate> .........................................222

         8.9  <unique predicate> .........................................223

         8.10 <match predicate> ..........................................224

         8.11 <overlaps predicate> .......................................227

         8.12 <search condition> .........................................229

         9  Data assignment rules ........................................231

         9.1  Retrieval assignment .......................................231

         9.2  Store assignment ...........................................234

         9.3  Set operation result data types ............................237

         10 Additional common elements ...................................239

         10.1 <interval qualifier> .......................................239

         10.2 <language clause> ..........................................243

         10.3 <privileges> ...............................................245

         10.4 <character set specification> ..............................248

         10.5 <collate clause> ...........................................251

         10.6 <constraint name definition> and <constraint attributes> ...252

         11 Schema definition and manipulation ...........................255

         11.1 <schema definition> ........................................255

         11.2 <drop schema statement> ....................................258

         11.3 <table definition> .........................................260

         11.4 <column definition> ........................................262

         11.5 <default clause> ...........................................266

         11.6 <table constraint definition> ..............................270

         11.7 <unique constraint definition> .............................272

         11.8 <referential constraint definition> ........................274

         11.9 <check constraint definition> ..............................281

         11.10<alter table statement> ....................................283

         11.11<add column definition> ....................................284

         11.12<alter column definition> ..................................286

         11.13<set column default clause> ................................287

         11.14<drop column default clause> ...............................288

         11.15<drop column definition> ...................................289

         11.16<add table constraint definition> ..........................291

         11.17<drop table constraint definition> .........................292

         11.18<drop table statement> .....................................294

         11.19<view definition> ..........................................296

         11.20<drop view statement> ......................................300

         11.21<domain definition> ........................................301

         11.22<alter domain statement> ...................................304

         11.23<set domain default clause> ................................305

                                                        Table of Contents  v

 









         11.24<drop domain default clause> ...............................306

         11.25<add domain constraint definition> .........................307

         11.26<drop domain constraint definition> ........................308

         11.27<drop domain statement> ....................................309

         11.28<character set definition> .................................311

         11.29<drop character set statement> .............................313

         11.30<collation definition> .....................................314

         11.31<drop collation statement> .................................318

         11.32<translation definition> ...................................320

         11.33<drop translation statement> ...............................323

         11.34<assertion definition> .....................................325

         11.35<drop assertion statement> .................................328

         11.36<grant statement> ..........................................329

         11.37<revoke statement> .........................................333

         12 Module .......................................................341

         12.1 <module> ...................................................341

         12.2 <module name clause> .......................................344

         12.3 <procedure> ................................................346

         12.4 Calls to a <procedure> .....................................352

         12.5 <SQL procedure statement> ..................................368

         13 Data manipulation ............................................371

         13.1 <declare cursor> ...........................................371

         13.2 <open statement> ...........................................375

         13.3 <fetch statement> ..........................................377

         13.4 <close statement> ..........................................381

         13.5 <select statement: single row> .............................382

         13.6 <delete statement: positioned> .............................384

         13.7 <delete statement: searched> ...............................386

         13.8 <insert statement> .........................................388

         13.9 <update statement: positioned> .............................391

         13.10<update statement: searched> ...............................394

         13.11<temporary table declaration> ..............................397

         14 Transaction management .......................................399

         14.1 <set transaction statement> ................................399

         14.2 <set constraints mode statement> ...........................401

         14.3 <commit statement> .........................................403

         14.4 <rollback statement> .......................................405

         15 Connection management ........................................407

         15.1 <connect statement> ........................................407

         15.2 <set connection statement> .................................410

         15.3 <disconnect statement> .....................................412

         vi  Database Language SQL

 









         16 Session management ...........................................415

         16.1 <set catalog statement> ....................................415

         16.2 <set schema statement> .....................................417

         16.3 <set names statement> ......................................419

         16.4 <set session authorization identifier statement> ...........420

         16.5 <set local time zone statement> ............................422

         17 Dynamic SQL ..................................................425

         17.1 Description of SQL item descriptor areas ...................425

         17.2 <allocate descriptor statement> ............................431

         17.3 <deallocate descriptor statement> ..........................433

         17.4 <get descriptor statement> .................................434

         17.5 <set descriptor statement> .................................438

         17.6 <prepare statement> ........................................442

         17.7 <deallocate prepared statement> ............................449

         17.8 <describe statement> .......................................450

         17.9 <using clause> .............................................451

         17.10<execute statement> ........................................459

         17.11<execute immediate statement> ..............................462

         17.12<dynamic declare cursor> ...................................464

         17.13<allocate cursor statement> ................................465

         17.14<dynamic open statement> ...................................467

         17.15<dynamic fetch statement> ..................................469

         17.16<dynamic close statement> ..................................471

         17.17<dynamic delete statement: positioned> .....................472

         17.18<dynamic update statement: positioned> .....................474

         17.19<preparable dynamic delete statement: positioned> ..........476

         17.20<preparable dynamic update statement: positioned> ..........477

         18 Diagnostics management .......................................479

         18.1 <get diagnostics statement> ................................479

         19 Embedded SQL .................................................489

         19.1 <embedded SQL host program> ................................489

         19.2 <embedded exception declaration> ...........................497

         19.3 <embedded SQL Ada program> .................................500

         19.4 <embedded SQL C program> ...................................504

         19.5 <embedded SQL COBOL program> ...............................508

         19.6 <embedded SQL Fortran program> .............................512

         19.7 <embedded SQL MUMPS program> ...............................515

         19.8 <embedded SQL Pascal program> ..............................517

         19.9 <embedded SQL PL/I program> ................................520

         20 Direct invocation of SQL .....................................525

         20.1 <direct SQL statement> .....................................525

         20.2 <direct select statement: multiple rows> ...................530

                                                      Table of Contents  vii

 









         21 Information Schema and Definition Schema .....................533

         21.1 Introduction ...............................................533

         21.2 Information Schema .........................................535

         21.2.INFORMATION_SCHEMA Schema ..................................535

         21.2.INFORMATION_SCHEMA_CATALOG_NAME base table .................536

         21.2.INFORMATION_SCHEMA_CATALOG_NAME_CARDINALITY assertion ......537

         21.2.SCHEMATA view ..............................................538

         21.2.DOMAINS view ...............................................539

         21.2.DOMAIN_CONSTRAINTS view ....................................541

         21.2.TABLES view ................................................542

         21.2.VIEWS view .................................................543

         21.2.COLUMNS view ...............................................544

         21.2.TABLE_PRIVILEGES view ......................................546

         21.2.COLUMN_PRIVILEGES view .....................................547

         21.2.USAGE_PRIVILEGES view ......................................548

         21.2.TABLE_CONSTRAINTS view .....................................549

         21.2.REFERENTIAL_CONSTRAINTS view ...............................550

         21.2.CHECK_CONSTRAINTS view .....................................551

         21.2.KEY_COLUMN_USAGE view ......................................552

         21.2.ASSERTIONS view ............................................553

         21.2.CHARACTER_SETS view ........................................554

         21.2.COLLATIONS view ............................................555

         21.2.TRANSLATIONS view ..........................................556

         21.2.VIEW_TABLE_USAGE view ......................................557

         21.2.VIEW_COLUMN_USAGE view .....................................558

         21.2.CONSTRAINT_TABLE_USAGE view ................................559

         21.2.CONSTRAINT_COLUMN_USAGE view ...............................561

         21.2.COLUMN_DOMAIN_USAGE view ...................................562

         21.2.SQL_LANGUAGES view .........................................563

         21.2.SQL_IDENTIFIER domain ......................................564

         21.2.CHARACTER_DATA domain ......................................564

         21.2.CARDINAL_NUMBER domain .....................................565

         21.3 Definition Schema ..........................................566

         21.3.Introduction ...............................................566

         21.3.DEFINITION_SCHEMA Schema ...................................567

         21.3.USERS base table ...........................................568

         21.3.SCHEMATA base table ........................................569

         21.3.DATA_TYPE_DESCRIPTOR base table ............................570

         21.3.DOMAINS base table .........................................573

         21.3.DOMAIN_CONSTRAINTS base table ..............................574

         21.3.TABLES base table ..........................................576

         21.3.VIEWS base table ...........................................578

         21.3.COLUMNS base table .........................................580

         21.3.VIEW_TABLE_USAGE base table ................................583

         21.3.VIEW_COLUMN_USAGE base table ...............................584

         viii  Database Language SQL

 









         21.3.TABLE_CONSTRAINTS base table ...............................585

         21.3.KEY_COLUMN_USAGE base table ................................588

         21.3.REFERENTIAL_CONSTRAINTS base table .........................590

         21.3.CHECK_CONSTRAINTS base table ...............................593

         21.3.CHECK_TABLE_USAGE base table ...............................595

         21.3.CHECK_COLUMN_USAGE base table ..............................596

         21.3.ASSERTIONS base table ......................................598

         21.3.TABLE_PRIVILEGES base table ................................600

         21.3.COLUMN_PRIVILEGES base table ...............................602

         21.3.USAGE_PRIVILEGES base table ................................604

         21.3.CHARACTER_SETS base table ..................................606

         21.3.COLLATIONS base table ......................................608

         21.3.TRANSLATIONS base table ....................................610

         21.3.SQL_LANGUAGES base table ...................................612

         21.4 Assertions on the base tables ..............................616

         21.4.UNIQUE_CONSTRAINT_NAME assertion ...........................616

         21.4.EQUAL_KEY_DEGREES assertion ................................617

         21.4.KEY_DEGREE_GREATER_THAN_OR_EQUAL_TO_1 assertion ............618

         22 Status codes .................................................619

         22.1 SQLSTATE ...................................................619

         22.2 SQLCODE ....................................................624

         23 Conformance ..................................................625

         23.1 Introduction ...............................................625

         23.2 Claims of conformance ......................................625

         23.3 Extensions and options .....................................626

         23.4 Flagger requirements .......................................626

         23.5 Processing methods .........................................627

         Annex A   Leveling the SQL Language..............................629

         A.1  Intermediate SQL Specifications ............................629

         A.2  Entry SQL Specifications ...................................640

         Annex B   Implementation-defined elements........................653

         Annex C   Implementation-dependent elements......................667

         Annex D   Deprecated features....................................675

         Annex E   Incompatibilities with ISO/IEC 9075:1989...............677

         Annex F   Maintenance and interpretation of SQL..................685

         Index

                                                       Table of Contents  ix

 










                                        TABLES

         Table                                                          Page

         1    Collating coercibility rules for monadic operators .........24

         2    Collating coercibility rules for dyadic operators ..........24

         3    Collating sequence usage for comparisons ...................25

         4    Fields in datetime items ...................................30

         5    Fields in year-month INTERVAL items ........................32

         6    Fields in day-time INTERVAL items ..........................32

         7    Valid values for fields in INTERVAL items ..................33

         8    Valid operators involving datetimes and intervals ..........34

         9    SQL-transaction isolation levels and the three phenomena ...69

         10   Valid values for fields in datetime items ..................112

         11   Valid values for fields in INTERVAL items ..................113

         12   <null predicate> semantics .................................218

         13   Truth table for the AND boolean ............................230

         14   Truth table for the OR boolean .............................230

         15   Truth table for the IS boolean .............................230

         16   Standard programming languages .............................243

         17   Data types of <key word>s used in SQL item descriptor areas
              ............................................................427

         18   Codes used for SQL data types in Dynamic SQL ...............429

         19   Codes associated with datetime data types in Dynamic SQL ...429

         20   Codes used for <interval qualifier>s in Dynamic SQL ........430

         21   <identifier>s for use with <get diagnostics statement> .....481

         22   SQL-statement character codes for use in the diagnostics
              area........................................................482

         23   SQLSTATE class and subclass values .........................619

         24   SQLCODE values .............................................624

         x  Database Language SQL

 





                                                     X3H2-92-154/DBL CBR-002





         Foreword



         ISO (the International Organization for Standardization) is a
         worldwide federation of national standards bodies (ISO member
         bodies). The work of preparing International Standards is nor-
         mally carried out through ISO technical committees. Each member
         body interested in a subject for which a technical committee has
         been established has the right to be represented on that committee.
         International organizations, governmental and non-governmental,
         in liaison with ISO, also take part in the work. ISO collaborates
         closely with the International Electrotechnical Commission (IEC) on
         all matters of electrotechnical standardization.

         Draft International Standards adopted by the technical committees
         are circulated to the member bodies for approval before their ac-
         ceptance as International Standards by the ISO Council. They are
         approved in accordance with ISO procedures requiring at least 75%
         approval by the member bodies voting.

         International Standard ISO/IEC 9075:1992 was prepared by Joint
         Technical Committee ISO/IEC JTC1, Information Processing Systems.

         It cancels and replaces International Standard ISO/IEC 9075:1989,
         Database Language-SQL, of which it constitutes a technical revi-
         sion.

         This International Standard contains seven informative annexes:

         -  Annex A (informative): Leveling the SQL Language;

         -  Annex B (informative): Implementation-defined elements;

         -  Annex C (informative): Implementation-dependent elements;

         -  Annex D (informative): Deprecated features;

         -  Annex E (informative): Incompatibilities with ISO/IEC 9075:1989;
            and

         -  Annex F (informative): Maintenance and interpretation of SQL.










                                                                Foreword  xi

 





                                                     X3H2-92-154/DBL CBR-002





         Introduction



         This International Standard was approved in 1992.

         This International Standard was developed from ISO/IEC 9075:1989,
         Information Systems, Database Language SQL with Integrity
         Enhancements, and replaces that International Standard. It adds
         significant new features and capabilities to the specifications.
         It is generally compatible with ISO/IEC 9075:1989, in the sense
         that, with very few exceptions, SQL language that conforms to
         ISO/IEC 9075:1989 also conforms to this International Standard,
         and will be treated in the same way by an implementation of this
         International Standard as it would by an implementation of ISO/IEC
         9075:1989. The known incompatibilities between ISO/IEC 9075:1989
         and this International Standard are stated in informative Annex E,
         "Incompatibilities with ISO/IEC 9075:1989".

         Technical changes between ISO/IEC 9075:1989 and this International
         Standard include both improvements or enhancements to existing fea-
         tures and the definition of new features. Significant improvements
         in existing features include:

         -  A better definition of direct invocation of SQL language;

         -  Improved diagnostic capabilities, especially a new status param-
            eter (SQLSTATE), a diagnostics area, and supporting statements.

         Significant new features are:

         1) Support for additional data types (DATE, TIME, TIMESTAMP,
            INTERVAL, BIT string, variable-length character and bit strings,
            and NATIONAL CHARACTER strings),

         2) Support for character sets beyond that required to express SQL
            language itself and support for additional collations,

         3) Support for additional scalar operations, such as string opera-
            tions for concatenate and substring, date and time operations,
            and a form for conditional expressions,

         4) Increased generality and orthogonality in the use of scalar-
            valued and table-valued query expressions,

         5) Additional set operators (for example, union join, natural join,
            set difference, and set intersection),

         6) Capability for domain definitions in the schema,

         7) Support for Schema Manipulation capabilities (especially DROP
            and ALTER statements),

                                                          Introduction  xiii

 





         X3H2-92-154/DBL CBR-002



         8) Support for bindings (modules and embedded syntax) in the Ada,
            C, and MUMPS languages,

         9) Additional privilege capabilities,

         10)Additional referential integrity facilities, including ref-
            erential actions, subqueries in CHECK constraints, separate
            assertions, and user-controlled deferral of constraints,

         11)Definition of an Information Schema,

         12)Support for dynamic execution of SQL language,

         13)Support for certain facilities required for Remote Database
            Access (especially connection management statements and quali-
            fied schema names),

         14)Support for temporary tables,

         15)Support for transaction consistency levels,

         16)Support for data type conversions (CAST expressions among data
            types),

         17)Support for scrolled cursors, and

         18)A requirement for a flagging capability to aid in portability of
            application programs.

         The organization of this International Standard is as follows:

         1) Clause 1, "Scope", specifies the scope of this International
            Standard.

         2) Clause 2, "Normative references", identifies additional stan-
            dards that, through reference in this International Standard,
            constitute provisions of this International Standard.

         3) Clause 3, "Definitions, notations, and conventions", defines the
            notations and conventions used in this International Standard.

         4) Clause 4, "Concepts", presents concepts used in the definition
            of SQL.

         5) Clause 5, "Lexical elements", defines the lexical elements of
            the language.

         6) Clause 6, "Scalar expressions", defines the elements of the
            language that produce scalar values.

         7) Clause 7, "Query expressions", defines the elements of the lan-
            guage that produce rows and tables of data.

         8) Clause 8, "Predicates", defines the predicates of the language.

         xiv  Database Language SQL

 





                                                     X3H2-92-154/DBL CBR-002



         9) Clause 9, "Data assignment rules", specifies the rules for
            assignments that retrieve data from or store data into the
            database, and formation rules for set operations.

         10)Clause 10, "Additional common elements", defines additional lan-
            guage elements that are used in various parts of the language.

         11)Clause 11, "Schema definition and manipulation", defines facili-
            ties for creating and managing a schema.

         12)Clause 12, "Module", defines modules and procedures.

         13)Clause 13, "Data manipulation", defines the data manipulation
            statements.

         14)Clause 14, "Transaction management", defines the SQL-transaction
            management statements.

         15)Clause 15, "Connection management" defines the SQL-connection
            management statements.

         16)Clause 16, "Session management", defines the SQL-session manage-
            ment statements.

         17)Clause 17, "Dynamic SQL", defines the facilities for executing
            SQL-statements dynamically.

         18)Clause 18, "Diagnostics management", defines the diagnostics
            management facilities.

         19)Clause 19, "Embedded SQL", defines syntax for embedding SQL in
            certain standard programming languages.

         20)Clause 20, "Direct invocation of SQL", defines the direct invo-
            cation of SQL language.

         21)Clause 21, "Information Schema and Definition Schema", defines
            viewed tables that contain schema information.

         22)Clause 22, "Status codes", defines values that identify the
            status of the execution of SQL-statements and the mechanisms by
            which those values are returned.

         23)Clause 23, "Conformance", defines the criteria for conformance
            to this International standard.

         24)Annex A, "Leveling the SQL Language", is an informative
            Annex. It lists the leveling rules defining the Entry SQL and
            Intermediate SQL subset levels of the SQL language.

         25)Annex B, "Implementation-defined elements", is an informa-
            tive Annex. It lists those features for which the body of the
            International Standard states that the syntax or meaning or ef-
            fect on the database is partly or wholly implementation-defined,
            and describes the defining information that an implementor shall
            provide in each case.

                                                            Introduction  xv

 





         X3H2-92-154/DBL CBR-002



         26)Annex C, "Implementation-dependent elements", is an informa-
            tive Annex. It lists those features for which the body of the
            International Standard states explicitly that the meaning or
            effect on the database is implementation-dependent.

         27)Annex D, "Deprecated features", is an informative Annex. It
            lists features that the responsible Technical Committee in-
            tends will not appear in a future revised version of this
            International Standard.

         28)Annex E, "Incompatibilities with ISO/IEC 9075:1989", is an in-
            formative Annex. It lists the incompatibilities between this
            version of this International Standard and ISO/IEC 9075:1989.

         29)Annex F, "Maintenance and interpretation of SQL", is an infor-
            mative Annex. It identifies SQL interpretations and corrections
            that have been processed by ISO/IEC JTC1/SC21 since adoption of
            ISO/IEC 9075:1989.

         In the text of this International Standard, Clauses begin a new
         odd-numbered page, and in Clause 5, "Lexical elements", through
         Clause 22, "Status codes", Subclauses begin a new page. Any result-
         ing blank space is not significant.































         xvi  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002








         Information Technology - Database Language SQL



         1  Scope


         This International Standard defines the data structures and basic
         operations on SQL-data. It provides functional capabilities for
         creating, accessing, maintaining, controlling, and protecting SQL-
         data.

         Note: The framework for this International Standard is described by
         the Reference Model of Data Management (ISO/IEC DIS 10032:1991).

         This International Standard specifies the syntax and semantics of a
         database language

         -  for specifying and modifying the structure and the integrity
            constraints of SQL-data,

         -  for declaring and invoking operations on SQL-data and cursors,
            and

         -  for declaring database language procedures and embedding them
            into a standard programming language.

         It also specifies an Information Schema that describes the struc-
         ture and the integrity constraints of SQL-data.

         This International Standard

         -  provides a vehicle for portability of data definitions and com-
            pilation units between SQL-implementations,

         -  provides a vehicle for interconnection of SQL-implementations,

         -  specifies syntax for embedding SQL-statements in a compilation
            unit that otherwise conforms to the standard for a particular
            programming language. It defines how an equivalent compilation
            unit may be derived that conforms to the particular programming
            language standard. In that equivalent compilation unit, each
            embedded SQL-statement has been replaced by statements that
            invoke a database language procedure that contains the SQL-
            statement, and

         -  specifies syntax for direct invocation of SQL-statements.


                                                                   Scope   1

 





          X3H2-92-154/DBL CBR-002



         This International Standard does not define the method or time of
         binding between any of:

         -  database management system components,

         -  SQL data definition declarations,

         -  SQL procedures, or

         -  compilation units, including those containing embedded SQL.

         Implementations of this International Standard may exist in en-
         vironments that also support application programming languages,
         end-user query languages, report generator systems, data dictionary
         systems, program library systems, and distributed communication
         systems, as well as various tools for database design, data admin-
         istration, and performance optimization.





































         2  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002






         2  Normative references


         The following standards contain provisions that, through reference
         in this text, constitute provisions of this International Standard.
         At the time of publication, the editions indicated were valid.
         All standards are subject to revision, and parties to agreements
         based on this International Standard are encouraged to investigate
         the possibility of applying the most recent editions of the stan-
         dards listed below. Members of IEC and ISO maintain registers of
         currently valid International Standards.

         -  ISO/IEC 646:1991, Information technology-ISO 7-bit coded charac-
            ter set for information interchange.

         -  ISO/IEC 1539:1991, Information technology-Programming languages-
            Fortran.

         -  ISO 1989:1985, Programming languages-COBOL.
            (Endorsement of ANSI X3.23-1985).

         -  ISO 2022:1986, Information technology-ISO 7-bit and 8-bit coded
            character sets-code extension techniques.

         -  ISO 6160:1979, Programming languages-PL/I
            (Endorsement of ANSI X3.53-1976).

         -  ISO 7185:1990, Information technology-Programming languages-
            Pascal.

         -  ISO 8601:1988, Data elements and interchange formats - Information
            interchange-Representation of dates and times.

         -  ISO 8652:1987, Programming languages-Ada.
            (Endorsement of ANSI/MIL-STD-1815A-1983).

         -  ISO/IEC 8824:1990, Information technology-Open Systems Interconnection-
            Specification of Abstract Syntax Notation One (ASN.1).

         -  ISO/IEC 9579-2:[1], Information technology - Open Systems
            Interconnection - Remote Database Access, Part 2: SQL special-
            ization.

         -  ISO/IEC 9899:1990, Programming languages - C.

         -  ISO/IEC 10206:1991, Information technology-Programming languages-
            Extended Pascal.

         -  ISO/IEC 10646:[1], Information technology-Multiple-octet coded
            character set.

         ____________________

           [1] To be published

                                                    Normative references   3

 





          X3H2-92-154/DBL CBR-002



         -  ISO/IEC 11756:[1], Information technology-Programming languages-
            MUMPS.




















































         4  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002






         3  Definitions, notations, and conventions



         3.1  Definitions

         For the purposes of this International Standard, the following
         definitions apply.


         3.1.1  Definitions taken from ISO/IEC 10646

         This International Standard makes use of the following terms de-
         fined in ISO/IEC 10646:

            a) character

            b) octet

            c) variable-length coding

            d) fixed-length coding

         3.1.2  Definitions taken from ISO 8601

         This International Standard makes use of the following terms de-
         fined in ISO 8601:

            a) Coordinated Universal Time (UTC)

            b) date ("date, calendar" in ISO 8601)

         3.1.3  Definitions provided in this International Standard

         This International Standard defines the following terms:

         a) assignable: The characteristic of a value or of a data type
            that permits that value or the values of that data type to be
            assigned to data instances of a specified data type.

         b) cardinality (of a collection): The number of objects in that
            collection. Those objects need not necessarily have distinct
            values.

         c) character repertoire; repertoire: A set of characters used for a
            specific purpose or application. Each character repertoire has
            an implied default collating sequence.

         d) coercibility: An attribute of character string data items that
            governs how a collating sequence for the items is determined.

                                 Definitions, notations, and conventions   5

 





          X3H2-92-154/DBL CBR-002
         3.1 Definitions


         e) collation; collating sequence: A method of ordering two com-
            parable character strings. Every character set has a default
            collation.

         f) comparable: The characteristic of two data objects that per-
            mits the value of one object to be compared with the value of
            the other object. Also said of data types: Two data types are
            comparable if objects of those data types are comparable.

         g) descriptor: A coded description of an SQL object. It includes
            all of the information about the object that a conforming SQL-
            implementation requires.

         h) distinct: Two values are said to be not distinct if either:
            both are the null value, or they compare equal according to
            Subclause 8.2, "<comparison predicate>". Otherwise they are
            distinct. Two rows (or partial rows) are distinct if at least
            one of their pairs of respective values is distinct. Otherwise
            they are not distinct. The result of evaluating whether or not
            two values or two rows are distinct is never unknown.

         i) duplicate: Two or more values or rows are said to be duplicates
            (of each other) if and only if they are not distinct.

         j) dyadic operator: An operator having two operands: a left operand
            and a right operand. An example of a dyadic arithmetic operator
            in this International Standard is "-", specifying the subtrac-
            tion of the right operand from the left operand.

         k) form-of-use: A convention (or encoding) for representing charac-
            ters (in character strings). Some forms-of-use are fixed-length
            codings and others are variable-length codings.

         l) form-of-use conversion: A method of converting character strings
            from one form-of-use to another form-of-use.

         m) implementation-defined: Possibly differing between SQL-
            implementations, but specified by the implementor for each
            particular SQL-implementation.

         n) implementation-dependent: Possibly differing between SQL-
            implementations, but not specified by this International
            Standard and not required to be specified by the implementor
            for any particular SQL-implementations.

         o) monadic operator: An operator having one operand. An example of
            a monadic arithmetic operator in this International Standard is
            "-", specifying the negation of the operand.

         p) multiset: An unordered collection of objects that are not neces-
            sarily distinct. The collection may be empty.

         q) n-adic operator: An operator having a variable number of
            operands (informally: n operands). An example of an n-adic
            operator in this International Standard is COALESCE.

         6  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             3.1 Definitions


         r) null value (null): A special value, or mark, that is used to
            indicate the absence of any data value.

         s) persistent: Continuing to exist indefinitely, until destroyed
            deliberately. Referential and cascaded actions are regarded
            as deliberate. Actions incidental to the ending of an SQL-
            transaction (see Subclause 4.28, "SQL-transactions") or an SQL-
            session (see Subclause 4.30, "SQL-sessions") are not regarded as
            deliberate.

         t) redundant duplicates: All except one of any multiset of dupli-
            cate values or rows.

         u) repertoire: See character repertoire.

         v) sequence: An ordered collection of objects that are not neces-
            sarily distinct.

         w) set: An unordered collection of distinct objects. The collection
            may be empty.

         x) SQL-implementation: A database management system that conforms
            to this International Standard.

         y) translation: A method of translating characters in one character
            repertoire into characters of the same or a different character
            repertoire.

         3.2  Notation

         The syntactic notation used in this International Standard is
         an extended version of BNF ("Backus Naur Form" or "Backus Normal
         Form").

         In BNF, each syntactic element of the language is defined by means
         of a production rule. This defines the element in terms of a for-
         mula consisting of the characters, character strings, and syntactic
         elements that can be used to form an instance of it.

         The version of BNF used in this International Standard makes use of
         the following symbols:

         SymbolMeaning

         < >   Angle brackets delimit character strings that are the names
               of syntactic elements, the non-terminal symbols of the SQL
               language.

         ::=   The definition operator. This is used in a production rule to
               separate the element defined by the rule from its definition.
               The element being defined appears to the left of the opera-
               tor and the formula that defines the element appears to the
               right.

                                 Definitions, notations, and conventions   7

 





          X3H2-92-154/DBL CBR-002
         3.2 Notation



         [ ]   Square brackets indicate optional elements in a formula. The
               portion of the formula within the brackets may be explicitly
               specified or may be omitted.

         { }   Braces group elements in a formula. The portion of the for-
               mula within the braces shall be explicitly specified.

         |     The alternative operator. The vertical bar indicates that
               the portion of the formula following the bar is an alterna-
               tive to the portion preceding the bar. If the vertical bar
               appears at a position where it is not enclosed in braces
               or square brackets, it specifies a complete alternative for
               the element defined by the production rule. If the vertical
               bar appears in a portion of a formula enclosed in braces or
               square brackets, it specifies alternatives for the contents
               of the innermost pair of such braces or brackets.

          . . . The ellipsis indicates that the element to which it applies
               in a formula may be repeated any number of times. If the el-
               lipsis appears immediately after a closing brace "}", then it
               applies to the portion of the formula enclosed between that
               closing brace and the corresponding opening brace "{". If
               an ellipsis appears after any other element, then it applies
               only to that element.

         !!    Introduces ordinary English text. This is used when the defi-
               nition of a syntactic element is not expressed in BNF.

         Spaces are used to separate syntactic elements. Multiple spaces and
         line breaks are treated as a single space. Apart from those symbols
         to which special functions were given above, other characters and
         character strings in a formula stand for themselves. In addition,
         if the symbols to the right of the definition operator in a produc-
         tion consist entirely of BNF symbols, then those symbols stand for
         themselves and do not take on their special meaning.

         Pairs of braces and square brackets may be nested to any depth,
         and the alternative operator may appear at any depth within such a
         nest.

         A character string that forms an instance of any syntactic element
         may be generated from the BNF definition of that syntactic element
         by application of the following steps:

         1) Select any one option from those defined in the right hand side
            of a production rule for the element, and replace the element
            with this option.

         2) Replace each ellipsis and the object to which it applies with
            one or more instances of that object.

         3) For every portion of the string enclosed in square brackets,
            either delete the brackets and their contents or change the
            brackets to braces.

         8  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                                3.2 Notation


         4) For every portion of the string enclosed in braces, apply steps
            1 through 5 to the substring between the braces, then remove the
            braces.

         5) Apply steps 1 through 5 to any non-terminal syntactic element
            (i.e., name enclosed in angle brackets) that remains in the
            string.

         The expansion or production is complete when no further non-
         terminal symbols remain in the character string.

         3.3  Conventions

         3.3.1  Informative elements

         In several places in the body of this International Standard, in-
         formative notes appear. For example:

         Note: This is an example of a note.
         Those notes do not belong to the normative part of this International
         Standard and conformance to material specified in those notes shall
         not be claimed.

         3.3.2  Specification of syntactic elements

         Syntactic elements are specified in terms of:

         -  Function: A short statement of the purpose of the element.

         -  Format: A BNF definition of the syntax of the element.

         -  Syntax Rules: A specification of the syntactic properties of the
            element, or of additional syntactic constraints, not expressed
            in BNF, that the element shall satisfy, or both.

         -  Access Rules: A specification of rules governing the accessibil-
            ity of schema objects that shall hold before the General Rules
            may be successfully applied.

         -  General Rules: A specification of the run-time effect of the
            element. Where more than one General Rule is used to specify the
            effect of an element, the required effect is that which would be
            obtained by beginning with the first General Rule and applying
            the Rules in numerical sequence unless a Rule is applied that
            specifies or implies a change in sequence or termination of the
            application of the Rules. Unless otherwise specified or implied
            by a specific Rule that is applied, application of General Rules
            terminates when the last in the sequence has been applied.

         -  Leveling Rules: A specification of how the element shall be
            supported for each of the levels of SQL.



                                 Definitions, notations, and conventions   9

 





          X3H2-92-154/DBL CBR-002
         3.3 Conventions


         The scope of notational symbols is the Subclause in which those
         symbols are defined. Within a Subclause, the symbols defined in
         Syntax Rules, Access Rules, or General Rules can be referenced in
         other rules provided that they are defined before being referenced.

         3.3.3  Specification of the Information Schema

         The objects of the Information Schema in this International
         Standard are specified in terms of:

         -  Function: A short statement of the purpose of the definition.

         -  Definition: A definition, in SQL, of the object being defined.

         -  Description: A specification of the run-time value of the ob-
            ject, to the extent that this is not clear from the definition.

         The definitions used to define the views in the Information Schema
         are used only to specify clearly the contents of those viewed
         tables. The actual objects on which these views are based are
         implementation-dependent.

         3.3.4  Use of terms

         3.3.4.1  Exceptions

         The phrase "an exception condition is raised:", followed by the
         name of a condition, is used in General Rules and elsewhere to
         indicate that the execution of a statement is unsuccessful, ap-
         plication of General Rules, other than those of Subclause 12.3,
         "<procedure>", and Subclause 20.1, "<direct SQL statement>", may
         be terminated, diagnostic information is to be made available,
         and execution of the statement is to have no effect on SQL-data or
         schemas. The effect on <target specification>s and SQL descriptor
         areas of an SQL-statement that terminates with an exception condi-
         tion, unless explicitly defined by this International Standard, is
         implementation-dependent.

         The phrase "a completion condition is raised:", followed by the
         name of a condition, is used in General Rules and elsewhere to
         indicate that application of General Rules is not terminated and
         diagnostic information is to be made available; unless an excep-
         tion condition is also raised, the execution of the statement is
         successful.

         If more than one condition could have occurred as a result of a
         statement, it is implementation-dependent whether diagnostic infor-
         mation pertaining to more than one condition is made available.






         10  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             3.3 Conventions


         3.3.4.2  Syntactic containment

         In a Format, a syntactic element <A> is said to immediately contain
         a syntactic element <B> if <B> appears on the right-hand side of
         the BNF production rule for <A>. A syntactic element <A> is said
         to contain or specify a syntactic element <C> if <A> immediately
         contains <C> or if <A> immediately contains a syntactic element <B>
         that contains <C>.

         In SQL language, an instance A1 of <A> is said to immediately con-
         tain an instance B1 of <B> if <A> immediately contains <B> and the
         text of B1 is part of the text of A1. An instance A1 of <A> is said
         to contain or specify an instance C1 of <C> if A1 immediately con-
         tains C1 or if A1 immediately contains an instance B1 of <B> that
         contains C1.

         An instance A1 of <A> is said to contain an instance B1 of <B> with
         an intervening <C> if A1 contains B1 and A1 contains an instance C1
         of <C> that contains B1. An instance A1 of <A> is said to contain
         an instance B1 of <B> without an intervening <C> if A1 contains B1
         and A1 does not contain an instance C1 of <C> that contains B1.

         An instance A1 of <A> simply contains an instance B1 of <B> if
         A1 contains B1 without an intervening instance A2 of <A> or an
         intervening instance B2 of <B>.

         If <A> contains <B>, then <B> is said to be contained in <A> and
         <A> is said to be a containing production symbol for <B>. If <A>
         simply contains <B>, then <B> is said to be simply contained in
         <A> and <A> is said to be a simply containing production symbol for
         <B>.

         Let A1 be an instance of <A> and let B1 be an instance of <B>. If
         <A> contains <B>, then A1 is said to contain B1 and B1 is said to
         be contained in A1. If <A> simply contains <B>, then A1 is said to
         simply contain B1 and B1 is said to be simply contained in A1.

         An instance A1 of <A> is the innermost <A> satisfying a condition
         C if A1 satisfies C and A1 does not contain an instance A2 of <A>
         that satisfies C. An instance A1 of <A> is the outermost <A> satis-
         fying a condition C if A1 satisfies C and A1 is not contained in an
         instance A2 of <A> that satisfies C.

         If <A> contains a <table name> that identifies a view that is
         defined by a <view definition> V, then <A> is said to generally
         contain the <query expression> contained in V. If <A> contains <B>,
         then <A> generally contains <B>. If <A> generally contains <B> and
         <B> generally contains <C>, then <A> generally contains <C>.

         An instance A1 of <A> directly contains an instance B1 of <B> if A1
         contains B1 without an intervening <set function specification> or
         <subquery>.


                                Definitions, notations, and conventions   11

 





          X3H2-92-154/DBL CBR-002
         3.3 Conventions


         3.3.4.3  Terms denoting rule requirements

         In the Syntax Rules, the term shall defines conditions that are
         required to be true of syntactically conforming SQL language. When
         such conditions depend on the contents of the schema, then they
         are required to be true just before the actions specified by the
         General Rules are performed. The treatment of language that does
         not conform to the SQL Formats and Syntax Rules is implementation-
         dependent. If any condition required by Syntax Rules is not sat-
         isfied when the evaluation of Access or General Rules is attempted
         and the implementation is neither processing non-conforming SQL
         language nor processing conforming SQL language in a non-conforming
         manner, then an exception condition is raised: syntax error or
         access rule violation (if this situation occurs during dynamic ex-
         ecution of an SQL-statement, then the exception that is raised is
         syntax error or access rule violation in dynamic SQL statement; if
         the situation occurs during direct invocation of an SQL-statement,
         then the exception that is raised is syntax error or access rule
         violation in direct SQL statement).

         In the Access Rules, the term shall defines conditions that are
         required to be satisfied for the successful application of the
         General Rules. If any such condition is not satisfied when the
         General Rules are applied, then an exception condition is raised:
         syntax error or access rule violation (if this situation occurs
         during dynamic execution of an SQL-statement, then the exception
         that is raised is syntax error or access rule violation in dynamic
         SQL statement; if the situation occurs during direct invocation of
         an SQL-statement, then the exception that is raised is syntax error
         or access rule violation in direct SQL statement).

         In the Leveling Rules, the term shall defines conditions that are
         required to be true of SQL language for it to syntactically conform
         to the specified level of conformance.

         3.3.4.4  Rule evaluation order

         A conforming implementation is not required to perform the exact
         sequence of actions defined in the General Rules, but shall achieve
         the same effect on SQL-data and schemas as that sequence. The term
         effectively is used to emphasize actions whose effect might be
         achieved in other ways by an implementation.

         The Syntax Rules and Access Rules for contained syntactic elements
         are effectively applied at the same time as the Syntax Rules and
         Access Rules for the containing syntactic elements. The General
         Rules for contained syntactic elements are effectively applied be-
         fore the General Rules for the containing syntactic elements. Where
         the precedence of operators is determined by the Formats of this
         International Standard or by parentheses, those operators are ef-
         fectively applied in the order specified by that precedence. Where
         the precedence is not determined by the Formats or by parentheses,
         effective evaluation of expressions is generally performed from

         12  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             3.3 Conventions


         left to right. However, it is implementation-dependent whether ex-
         pressions are actually evaluated left to right, particularly when
         operands or operators might cause conditions to be raised or if
         the results of the expressions can be determined without completely
         evaluating all parts of the expression. In general, if some syn-
         tactic element contains more than one other syntactic element, then
         the General Rules for contained elements that appear earlier in the
         production for the containing syntactic element are applied before
         the General Rules for contained elements that appear later.

         For example, in the production:

            <A> ::= <B> <C>

         the Syntax Rules and Access Rules for <A>, <B>, and <C> are ef-
         fectively applied simultaneously. The General Rules for <B> are
         applied before the General Rules for <C>, and the General Rules for
         <A> are applied after the General Rules for both <B> and <C>.

         If the result of an expression or search condition can be deter-
         mined without completely evaluating all parts of the expression or
         search condition, then the parts of the expression or search condi-
         tion whose evaluation is not necessary are called the inessential
         parts. If the Access Rules pertaining to inessential parts are not
         satisfied, then the syntax error or access rule violation exception
         condition is raised regardless of whether or not the inessential
         parts are actually evaluated. If evaluation of the inessential
         parts would cause an exception condition to be raised, then it is
         implementation-dependent whether or not that exception condition is
         raised.

         3.3.4.5  Conditional rules

         Conditional rules are specified with "If" or "Case" conventions.
         Rules specified with "Case" conventions include a list of con-
         ditional sub-rules using "If" conventions. The first such "If"
         sub-rule whose condition is true is the effective sub-rule of
         the "Case" rule. The last sub-rule of a "Case" rule may specify
         "Otherwise". Such a sub-rule is the effective sub-rule of the
         "Case" rule if no preceding "If" sub-rule in the "Case" rule has
         a true condition.

         3.3.4.6  Syntactic substitution

         In the Syntax and General Rules, the phrase "X is implicit" indi-
         cates that the Syntax and General Rules are to be interpreted as if
         the element X had actually been specified.

         In the Syntax and General Rules, the phrase "the following <X> is
         implicit: Y" indicates that the Syntax and General Rules are to be
         interpreted as if a syntactic element <X> containing Y had actually
         been specified.


                                Definitions, notations, and conventions   13

 





          X3H2-92-154/DBL CBR-002
         3.3 Conventions


         In the Syntax Rules and General Rules, the phrase "former is equiv-
         alent to latter" indicates that the Syntax Rules and General Rules
         are to be interpreted as if all instances of former in the element
         had been instances of latter.

         If a BNF nonterminal is referenced in a Subclause without speci-
         fying how it is contained in a BNF production that the Subclause
         defines, then

         Case:

         -  If the BNF nonterminal is itself defined in the Subclause, then
            the reference shall be assumed to be the occurrence of that BNF
            nonterminal on the left side of the defining production.

         -  Otherwise, the reference shall be assumed to be to a BNF pro-
            duction in which the particular BNF nonterminal is immediately
            contained.

         3.3.4.7  Other terms

         Some Syntax Rules define terms, such as T1, to denote named or
         unnamed tables. Such terms are used as table names or correlation
         names. Where such a term is used as a correlation name, it does
         not imply that any new correlation name is actually defined for
         the denoted table, nor does it affect the scopes of any actual
         correlation names.

         An SQL-statement S1 is said to be executed as a direct result of
         executing an SQL-statement if S1 is the SQL-statement contained
         in a <procedure> that has been executed, or if S1 is the value of
         an <SQL statement variable> referenced by an <execute immediate
         statement> contained in a <procedure> that has been executed, or if
         S1 was the value of the <SQL statement variable> that was associ-
         ated with an <SQL statement name> by a <prepare statement> and that
         same <SQL statement name> is referenced by an <execute statement>
         contained in a <procedure> that has been executed.

         3.3.5  Descriptors

         A descriptor is a conceptual structured collection of data that
         defines the attributes of an instance of an object of a specified
         type. The concept of descriptor is used in specifying the seman-
         tics of SQL. It is not necessary that any descriptor exist in any
         particular form in any database or environment.

         Some SQL objects cannot exist except in the context of other SQL
         objects. For example, columns cannot exist except in tables. Those
         objects are independently described by descriptors, and the de-
         scriptors of enabling objects (e.g., tables) are said to include
         the descriptors of enabled objects (e.g., columns or table con-
         straints). Conversely, the descriptor of an enabled object is said
         to be included in the descriptor of an enabling object.

         14  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             3.3 Conventions


         In other cases, certain SQL objects cannot exist unless some other
         SQL object exists, even though there is not an inclusion relation-
         ship. For example, SQL does not permit an assertion to exist if the
         tables referenced by the assertion do not exist. Therefore, an as-
         sertion descriptor is dependent on or depends on zero or more table
         descriptors (equivalently, an assertion is dependent on or depends
         on zero or more tables). In general, a descriptor D1 can be said to
         depend on, or be dependent on, some descriptor D2.

         There are two ways of indicating dependency of one construct on
         another. In many cases, the descriptor of the dependent construct
         is said to "include the name of" the construct on which it is de-
         pendent. In this case "the name of" is to be understood as meaning
         "sufficient information to identify the descriptor of"; thus an
         implementor might choose to use a pointer or a concatenation of
         <catalog name>, <schema name>, etc. Alternatively, the descrip-
         tor may be said to include text (e.g., <query expression>, <search
         condition>). In such cases, whether the implementation includes ac-
         tual text (with defaults and implications made explicit) or its own
         style of parse tree is irrelevant; the validity of the descriptor
         is clearly "dependent on" the existence of descriptors for objects
         that are referred to in it.

         The statement that a column "is based on" a domain, is equivalent
         to a statement that a column "is dependent on" that domain.

         An attempt to destroy a descriptor may fail if other descriptors
         are dependent on it, depending on how the destruction is specified.
         Such an attempt may also fail if the descriptor to be destroyed
         is included in some other descriptor. Destruction of a descriptor
         results in the destruction of all descriptors included in it, but
         has no effect on descriptors on which it is dependent.

         3.3.6  Index typography

         In the Index to this International Standard, the following conven-
         tions are used:

         -  Index entries appearing in boldface indicate the page where the
            word, phrase, or BNF nonterminal was defined;

         -  Index entries appearing in italics indicate a page where the BNF
            nonterminal was used in a Format; and

         -  Index entries appearing in roman type indicate a page where
            the word, phrase, or BNF nonterminal was used in a heading,
            Function, Syntax Rule, Access Rule, General Rule, Leveling Rule,
            Table, or other descriptive text.






                                Definitions, notations, and conventions   15

 





          X3H2-92-154/DBL CBR-002
         3.4 Object identifier for Database Language SQL


         3.4  Object identifier for Database Language SQL

         Function

         The object identifier for Database Language SQL identifies the
         characteristics of an SQL-implementation to other entities in an
         open systems environment.

         Format

         <SQL object identifier> ::=
              <SQL provenance> <SQL variant>

         <SQL provenance> ::= <arc1> <arc2> <arc3>

         <arc1> ::= iso | 1 | iso <left paren> 1 <right paren>

         <arc2> ::= standard | 0 | standard <left paren> 0 <right paren>

         <arc3> ::= 9075

         <SQL variant> ::= <SQL edition> <SQL conformance>

         <SQL edition> ::= <1987> | <1989> | <1992>

         <1987> ::= 0 | edition1987 <left paren> 0 <right paren>

         <1989> ::= <1989 base> <1989 package>

         <1989 base> ::= 1 | edition1989 <left paren> 1 <right paren>

         <1989 package> ::= <integrity no> | <integrity yes>

         <integrity no> ::= 0 | IntegrityNo <left paren> 0 <right paren>

         <integrity yes> ::= 1 | IntegrityYes <left paren> 1 <right paren>

         <1992> ::= 2 | edition1992 <left paren> 2 <right paren>

         <SQL conformance> ::= <low> | <intermediate> | <high>

         <low> ::= 0 | Low <left paren> 0 <right paren>

         <intermediate> ::= 1 | Intermediate <left paren> 1 <right paren>

         <high> ::= 2 | High <left paren> 2 <right paren>








         16  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                             3.4 Object identifier for Database Language SQL


         Syntax Rules

         1) An <SQL conformance> of <high> shall not be specified unless the
            <SQL edition> is specified as <1992>.

         2) The value of <SQL conformance> identifies the level at which
            conformance is claimed as follows:

            a) If <SQL edition> specifies <1992>, then

              Case:

              i) <low>, then Entry SQL level.

             ii) <intermediate>, then Intermediate SQL level.

            iii) <high>, then Full SQL level.

            b) Otherwise:

              i) <low>, then level 1.

             ii) <intermediate>, then level 2.

         3) A specification of <1989 package> as <integrity no> implies
            that the integrity enhancement feature is not implemented. A
            specification of <1989 package> as <integrity yes> implies that
            the integrity enhancement feature is implemented.


























                                Definitions, notations, and conventions   17

 





          X3H2-92-154/DBL CBR-002

























































         18  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002






         4  Concepts



         4.1  Data types

         A data type is a set of representable values. The logical represen-
         tation of a value is a <literal>. The physical representation of a
         value is implementation-dependent.

         A value is primitive in that it has no logical subdivision within
         this International Standard. A value is a null value or a non-null
         value.

         A null value is an implementation-dependent special value that
         is distinct from all non-null values of the associated data type.
         There is effectively only one null value and that value is a member
         of every SQL data type. There is no <literal> for a null value,
         although the keyword NULL is used in some places to indicate that a
         null value is desired.

         SQL defines distinct data types named by the following <key word>s:
         CHARACTER, CHARACTER VARYING, BIT, BIT VARYING, NUMERIC, DECIMAL,
         INTEGER, SMALLINT, FLOAT, REAL, DOUBLE PRECISION, DATE, TIME,
         TIMESTAMP, and INTERVAL.

         Subclause 6.1, "<data type>", describes the semantic properties of
         each data type.

         For reference purposes, the data types CHARACTER and CHARACTER
         VARYING are collectively referred to as character string types.
         The data types BIT and BIT VARYING are collectively referred to
         as bit string types. Character string types and bit string types
         are collectively referred to as string types and values of string
         types are referred to as strings. The data types NUMERIC, DECIMAL,
         INTEGER, and SMALLINT are collectively referred to as exact numeric
         types. The data types FLOAT, REAL, and DOUBLE PRECISION are col-
         lectively referred to as approximate numeric types. Exact numeric
         types and approximate numeric types are collectively referred to as
         numeric types. Values of numeric type are referred to as numbers.
         The data types DATE, TIME, and TIMESTAMP are collectively referred
         to as datetime types. Values of datetime types are referred to as
         datetimes. The data type INTERVAL is referred to as an interval
         type. Values of interval types are called intervals.

         Each data type has an associated data type descriptor. The contents
         of a data type descriptor are determined by the specific data type
         that it describes. A data type descriptor includes an identifica-
         tion of the data type and all information needed to characterize an
         instance of that data type.

                                                               Concepts   19

 





          X3H2-92-154/DBL CBR-002
         4.1 Data types


         Each host language has its own data types, which are separate and
         distinct from SQL data types, even though similar names may be
         used to describe the data types. Mappings of SQL data types to data
         types in host languages are described in Subclause 12.3, "<pro-
         cedure>", and Subclause 19.1, "<embedded SQL host program>". Not
         every SQL data type has a corresponding data type in every host
         language.

         4.2  Character strings

         A character string data type is described by a character string
         data type descriptor. A character string data type descriptor con-
         tains:

         -  the name of the specific character string data type (CHARACTER
            or CHARACTER VARYING; NATIONAL CHARACTER and NATIONAL CHARACTER
            VARYING are represented as CHARACTER and CHARACTER VARYING,
            respectively);

         -  the length or maximum length in characters of the character
            string data type;

         -  the catalog name, schema name, and character set name of the
            character set of the character string data type; and

         -  the catalog name, schema name, and collation name of the colla-
            tion of the character string data type.

         Character sets fall into three categories: those defined by na-
         tional or international standards, those provided by implemen-
         tations, and those defined by applications. All character sets,
         however defined, always contain the <space> character. Character
         sets defined by applications can be defined to "reside" in any
         schema chosen by the application. Character sets defined by stan-
         dards or by implementations reside in the Information Schema (named
         INFORMATION_SCHEMA) in each catalog, as do collations defined by
         standards and collations and form-of-use conversions defined by
         implementations.

         The <implementation-defined character repertoire name> SQL_TEXT
         specifies the name of a character repertoire and implied form-of-
         use that can represent every character that is in <SQL language
         character> and all other characters that are in character sets
         supported by the implementation.

         4.2.1  Character strings and collating sequences

         A character string is a sequence of characters chosen from the
         same character repertoire. The character repertoire from which
         the characters of a particular string are chosen may be specified
         explicitly or implicitly. A character string has a length, which
         is the number of characters in the sequence. The length is 0 or a
         positive integer.

         20  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       4.2 Character strings


         All character strings of a given character repertoire are mutu-
         ally comparable, subject to the restrictions specified in Table 3,
         "Collating sequence usage for comparisons".

         A collating sequence, also known as a collation, is a set of rules
         determining comparison of character strings in a particular char-
         acter repertoire. There is a default collating sequence for each
         character repertoire, but additional collating sequences can be
         defined for any character repertoire.

         Note: A column may be defined as having a default collating se-
         quence. This default collating sequence for the column may be
         different from the default collating sequence for its character
         repertoire, e.g., if the <collate clause> is specified in the
         <column reference>. It will be clear from context when the term
         "default collating sequence" is used whether it is meant for a
         column or for a character repertoire.

         Given a collating sequence, two character strings are identical if
         and only if they are equal in accordance with the comparison rules
         specified in Subclause 8.2, "<comparison predicate>". The collat-
         ing sequence used for a particular comparison is determined as in
         Subclause 4.2.3, "Rules determining collating sequence usage".

         The <key word>s NATIONAL CHARACTER are used to specify a character
         string data type with a particular implementation-defined character
         repertoire. Special syntax (N'string') is provided for representing
         literals in that character repertoire.

         A character set is described by a character set descriptor. A char-
         acter set descriptor includes:

         -  the name of the character set or character repertoire,

         -  if the character set is a character repertoire, then the name of
            the form-of-use,

         -  an indication of what characters are in the character set, and

         -  the name of the default collation of the character set.

         For every character set, there is at least one collation. A colla-
         tion is described by a collation descriptor. A collation descriptor
         includes:

         -  the name of the collation,

         -  the name of the character set on which the collation operates,

         -  whether the collation has the NO PAD or the PAD SPACE attribute,
            and

         -  an indication of how the collation is performed.

                                                               Concepts   21

 





          X3H2-92-154/DBL CBR-002
         4.2 Character strings


         4.2.2  Operations involving character strings

         4.2.2.1  Operators that operate on character strings and return
                  character strings

         <concatenation operator> is an operator, |, that returns the char-
         acter string made by joining its character string operands in the
         order given.

         <character substring function> is a triadic function, SUBSTRING,
         that returns a string extracted from a given string according
         to a given numeric starting position and a given numeric length.
         Truncation occurs when the implied starting and ending positions
         are not both within the given string.

         <fold> is a pair of functions for converting all the lower case
         characters in a given string to upper case (UPPER) or all the upper
         case ones to lower case (LOWER), useful only in connection with
         strings that may contain <simple Latin letter>s.

         <form-of-use conversion> is a function that invokes an installation-
         supplied form-of-use conversion to return a character string S2
         derived from a given character string S1. It is intended, though
         not enforced by this International Standard, that S2 be exactly the
         same sequence of characters as S1, but encoded according some dif-
         ferent form-of-use. A typical use might be to convert a character
         string from two-octet UCS to one-octet Latin1 or vice versa.

         <trim function> is a function that returns its first string ar-
         gument with leading and/or trailing pad characters removed. The
         second argument indicates whether leading, or trailing, or both
         leading and trailing pad characters should be removed. The third
         argument specifies the pad character that is to be removed.

         <character translation> is a function for changing each charac-
         ter of a given string according to some many-to-one or one-to-one
         mapping between two not necessarily distinct character sets. The
         mapping, rather than being specified as part of the function, is
         some external function identified by a <translation name>.

         For any pair of character sets, there are zero or more translations
         that may be invoked by a <character translation>. A translation
         is described by a translation descriptor. A translation descriptor
         includes:

         -  the name of the translation,

         -  the name of the character set from which it translates,

         -  the name of the character set to which it translates, and

         -  an indication of how the translation is performed.


         22  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       4.2 Character strings


         4.2.2.2  Other operators involving character strings

         <length expression> returns the length of a given character string,
         as an integer, in characters, octets, or bits according to the
         choice of function.

         <position expression> determines the first position, if any, at
         which one string, S1, occurs within another, S2. If S1 is of length
         zero, then it occurs at position 1 for any value of S2. If S1 does
         not occur in S2, then zero is returned.

         <like predicate> uses the triadic operator LIKE (or the inverse,
         NOT LIKE), operating on three character strings and returning
         a Boolean. LIKE determines whether or not a character string
         "matches" a given "pattern" (also a character string). The char-
         acters '%' (percent) and '_' (underscore) have special meaning when
         they occur in the pattern. The optional third argument is a charac-
         ter string containing exactly one character, known as the "escape
         character", for use when a percent or underscore is required in the
         pattern without its special meaning.

         4.2.3  Rules determining collating sequence usage

         The rules determining collating sequence usage for character
         strings are based on the following:

         -  Expressions where no columns are involved (e.g., literals, host
            variables) are by default compared using the default collating
            sequence for their character repertoire.

            Note: The default collating sequence for a character repertoire
            is defined in Subclause 10.4, "<character set specification>",
            and Subclause 11.28, "<character set definition>".

         -  When columns are involved (e.g., comparing two columns, or com-
            paring a column to a literal), by default the default collating
            sequence of the columns involved is used so long as the columns
            have the same default collating sequence.

         -  When columns are involved having different default collating
            sequences, explicit specification of the collating sequence in
            the expression is required via the <collate clause> when the
            expression participates in a comparison.

         -  Any explicit specification of collating sequence in an expres-
            sion overrides any default collating sequence.

         To formalize this, <character value expression>s effectively have
         a coercibility attribute. This attribute has the values Coercible,
         Implicit, No collating sequence, and Explicit. <character value
         expression>s with the Coercible, Implicit, or Explicit attributes
         have a collating sequence.


                                                               Concepts   23

 





          X3H2-92-154/DBL CBR-002
         4.2 Character strings


         A <character value expression> consisting of a column reference has
         the Implicit attribute, with collating sequence as defined when the
         column was created. A <character value expression> consisting of a
         value other than a column (e.g., a host variable or a literal) has
         the Coercible attribute, with the default collation for its char-
         acter repertoire. A <character value expression> simply containing
         a <collate clause> has the Explicit attribute, with the collating
         sequence specified in the <collate clause>.

         Note: When the coercibility attribute is Coercible, the collating
         sequence is uniquely determined as specified in Subclause 8.2,
         "<comparison predicate>".

         The tables below define how the collating sequence and the co-
         ercibility attribute is determined for the result of any monadic
         or dyadic operation. Table 1, "Collating coercibility rules for
         monadic operators", shows the collating sequence and coercibility
         rules for monadic operators, and Table 2, "Collating coercibil-
         ity rules for dyadic operators", shows the collating sequence and
         coercibility rules for dyadic operators. Table 3, "Collating se-
         quence usage for comparisons", shows how the collating sequence is
         determined for a particular comparison.

         _____Table_1-Collating_coercibility_rules_for_monadic_operators____

                Operand Coercibility              Result Coercibility
          _____and_Collating_Sequence_____  _____and_Collating_Sequence___

        |                   Collating     |                   Collating    |
        |_Coercibility______Sequence______|_Coercibility______Sequence_____|
        |                                 |                                |
        | Coercible       | default       | Coercible       | default      |
        |                 |               |                 |              |
        | Implicit        | X             | Implicit        | X            |
        |                 |               |                 |              |
        | Explicit        | X             | Explicit        | X            |
        |                 |               |                 |              |
        |_______No_collati|g_sequence_____|______No_collatin|_sequence_____|
        |                 |               |                 |              |
         _____Table_2-Collating_coercibility_rules_for_dyadic_operators_____

                                                                 Result
                                                              Coercibility
           Operand 1 Coercibility   Operand 2 Coercibility   and Collating
          _and_Collating_Sequence  _and_Collating_Sequence   ___Sequence___

        |              Collating |              Collating |             Col|ating
        |_Coercibility_Sequence__|_Coercibility_Sequence__|__CoercibilitySe|uence
        |                        |                        |                |
        | Coercible  | default   | Coercible  | default   |  Coercible| def|ult
        |            |           |            |           |           |    |
        | Coercible  | default   | Implicit   | Y         |  Implicit | Y  |
        |            |           |            |           |           |    |
        | Coercible  | default   |  No collati|g sequence |   No colla|ing |
                                                                sequence

         24  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       4.2 Character strings


         _Table_2-Collating_coercibility_rules_for_dyadic_operators_(Cont.)_

                                                                 Result
                                                              Coercibility
           Operand 1 Coercibility   Operand 2 Coercibility   and Collating
          _and_Collating_Sequence  _and_Collating_Sequence   ___Sequence___

        |              Collating |              Collating |             Col|ating
        |_Coercibility_Sequence__|_Coercibility_Sequence__|__CoercibilitySe|uence
        |                        |                        |                |
        | Coercible  | default   | Explicit   | Y         |  Explicit | Y  |
        |            |           |            |           |           |    |
        | Implicit   | X         | Coercible  | default   |  Implicit | X  |
        |            |           |            |           |           |    |
        | Implicit   | X         | Implicit   | X         |  Implicit | X  |
        |            |           |            |           |           |    |
        | Implicit   | X         | Implicit   | Y /= X    |   No colla|ing |
                                                                sequence

        | Implicit   | X         |  No collati|g sequence |   No collating |
        |            |           |            |           |     sequence   |
        |            |           |            |           |                |
        | Implicit   | X         | Explicit     Y         |  Explicit   Y  |
        |            |           |                        |                |
        |  No collati|g sequence | Any,       | Any       |   No colla|ing |
                                   except                       sequence
                                   Explicit

        |  No collating sequence | Explicit   | X         |  Explicit   X  |
        |                        |            |           |                |
        | Explicit     X         | Coercible  | default   |  Explicit | X  |
        |                        |            |           |           |    |
        | Explicit   | X         | Implicit   | Y         |  Explicit | X  |
        |            |           |            |           |           |    |
        | Explicit   | X         |  No collati|g sequence |  Explicit | X  |
        |            |           |            |           |           |    |
        | Explicit   | X         | Explicit     X         |  Explicit | X  |
        |            |           |                        |           |    |
        | Explicit   | X         | Explicit   | Y /= X    |  Not permi|ted:|
         ____________________________________________________invalid_syntax_

        |__________Ta|le_3-Collat|ng_sequence_|sage_for_co|parisons________|

              Comparand 1          Comparand 2
            Coercibility and     Coercibility and
          _Collating_Sequence  _Collating_Sequence

        |                    |                    |  Collating Sequence    |
        |            Collatin|            Collatin|  Used For The          |
        |_CoercibilitSequence|_CoercibilitSequence|__Comparison____________|
        |                    |                    |                        |
        | Coercible| default | Coercible| default |  default               |
        |          |         |          |         |                        |
        | Coercible| default | Implicit | Y       |  Y                     |
        |          |         |          |         |                        |
        | Coercible| default |     No co|lating   |  Not permitted: invalid|
                                     sequence        syntax

        | Coercible| default | Explicit   Y       |  Y                     |
        |          |         |                    |                        |
                                                               Concepts   25

 





          X3H2-92-154/DBL CBR-002
         4.2 Character strings


         ______Table_3-Collating_sequence_usage_for_comparisons_(Cont.)_____

              Comparand 1          Comparand 2
            Coercibility and     Coercibility and
          _Collating_Sequence  _Collating_Sequence

        |                    |                    |  Collating Sequence    |
        |            Collatin|            Collatin|  Used For The          |
        |_CoercibilitSequence|_CoercibilitSequence|__Comparison____________|
        |                    |                    |                        |
        | Implicit | X       | Coercible| default |  X                     |
        |          |         |          |         |                        |
        | Implicit | X       | Implicit | X       |  X                     |
        |          |         |          |         |                        |
        | Implicit | X       | Implicit | Y /= X  |  Not permitted: invalid|
                                                     syntax

        | Implicit | X       |     No co|lating   |  Not permitted: invalid|
        |          |         |       seq|ence     |  syntax                |
        |          |         |          |         |                        |
        | Implicit | X       | Explicit   Y       |  Y                     |
        |          |         |                    |                        |
        |     No co|lating   | Any      | Any     |  Not permitted: invalid|
                sequence       except                syntax
                               Explicit

        |     No collating   | Explicit | X       |  X                     |
        |       sequence     |          |         |                        |
        |                    |          |         |                        |
        | Explicit   X       | Coercible| default |  X                     |
        |                    |          |         |                        |
        | Explicit | X       | Implicit | Y       |  X                     |
        |          |         |          |         |                        |
        | Explicit | X       |     No co|lating   |  X                     |
                                     sequence

        | Explicit | X       | Explicit   X       |  X                     |
        |          |         |                    |                        |
        | Explicit | X       | Explicit | Y /= X  |  Not permitted: invalid|
         ____________________________________________syntax_________________

        |For n-adic|operation| (e.g., <c|se expres|ion>) with operands X1, |
         X2, . . . , n , the collating sequence is effectively determined by
         considering X1 and X2, then combining this result with X3, and so
         on.

         4.3  Bit strings

         A bit string is a sequence of bits, each having the value of 0 or
         1. A bit string has a length, which is the number of bits in the
         string. The length is 0 or a positive integer.

         A bit string data type is described by a bit string data type de-
         scriptor. A bit string data type descriptor contains:

         -  the name of the specific bit string data type (BIT or BIT
            VARYING); and

         26  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             4.3 Bit strings


         -  the length of the bit string data type (in bits).

         4.3.1  Bit string comparison and assignment

         All bit strings are mutually comparable. A bit string is identical
         to another bit string if and only if it is equal to that bit string
         in accordance with the comparison rules specified in Subclause 8.2,
         "<comparison predicate>".

         Assignment of a bit string to a bit string variable is performed
         from the most significant bit to the least significant bit in the
         source string to the most significant bit in the target string, one
         bit at a time.


         4.3.2  Operations involving bit strings

         4.3.2.1  Operators that operate on bit strings and return bit
                  strings

         <bit concatenation> is an operator, |, that returns the bit string
         made by concatenating the two bit string operands in the order
         given.

         <bit substring function> is a triadic function identical in syntax
         and semantics to <character substring function> except that the
         first argument and the returned value are both bit strings.

         4.3.2.2  Other operators involving bit strings

         <length expression> returns the length (as an integer number of
         octets or bits according to the choice of function) of a given bit
         string.

         <position expression> determines the first position, if any, at
         which one string, S1, occurs within another, S2. If S1 is of length
         zero, then it occurs at position 1 for any value of S2. If S1 does
         not occur in S2, then zero is returned.

         4.4  Numbers

         A number is either an exact numeric value or an approximate numeric
         value. Any two numbers are mutually comparable to each other.

         A numeric data type is described by a numeric data type descriptor.
         A numeric data type descriptor contains:

         -  the name of the specific numeric data type (NUMERIC, DECIMAL,
            INTEGER, SMALLINT, FLOAT, REAL, or DOUBLE PRECISION);

         -  the precision of the numeric data type;



                                                               Concepts   27

 





          X3H2-92-154/DBL CBR-002
         4.4 Numbers


         -  the scale of the numeric data type, if it is an exact numeric
            data type; and

         -  an indication of whether the precision (and scale) are expressed
            in decimal or binary terms.

         4.4.1  Characteristics of numbers

         An exact numeric value has a precision and a scale. The precision
         is a positive integer that determines the number of significant
         digits in a particular radix (binary or decimal). The scale is a
         non-negative integer. A scale of 0 indicates that the number is an
         integer. For a scale of S, the exact numeric value is the integer
         value of the significant digits multiplied by 10-S.

         An approximate numeric value consists of a mantissa and an expo-
         nent. The mantissa is a signed numeric value, and the exponent is
         a signed integer that specifies the magnitude of the mantissa. An
         approximate numeric value has a precision. The precision is a posi-
         tive integer that specifies the number of significant binary digits
         in the mantissa. The value of an approximate numeric value is the
         mantissa multiplied by 10exponent.

         Whenever an exact or approximate numeric value is assigned to a
         data item or parameter representing an exact numeric value, an
         approximation of its value that preserves leading significant dig-
         its after rounding or truncating is represented in the data type
         of the target. The value is converted to have the precision and
         scale of the target. The choice of whether to truncate or round is
         implementation-defined.

         An approximation obtained by truncation of a numerical value N
         for an <exact numeric type> T is a value V representable in T such
         that N is not closer to zero than the numerical value of V and such
         that the absolute value of the difference between N and the numer-
         ical value of V is less than the absolute value of the difference
         between two successive numerical values representable in T.

         An approximation obtained by rounding of a numerical value N for
         an <exact numeric type> T is a value V representable in T such
         that the absolute value of the difference between N and the nu-
         merical value of V is not greater than half the absolute value
         of the difference between two successive numerical values repre-
         sentable in T. If there are more than one such values V, then it is
         implementation-defined which one is taken.

         All numerical values between the smallest and the largest value,
         inclusive, representable in a given exact numeric type have an
         approximation obtained by rounding or truncation for that type; it
         is implementation-defined which other numerical values have such
         approximations.



         28  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                                 4.4 Numbers


         An approximation obtained by truncation or rounding of a numerical
         value N for an <approximate numeric type> T is a value V repre-
         sentable in T such that there is no numerical value representable
         in T and distinct from that of V that lies between the numerical
         value of V and N, inclusive.

         If there are more than one such values V then it is implementation-
         defined which one is taken. It is implementation-defined which
         numerical values have approximations obtained by rounding or trun-
         cation for a given approximate numeric type.

         Whenever an exact or approximate numeric value is assigned to a
         data item or parameter representing an approximate numeric value,
         an approximation of its value is represented in the data type of
         the target. The value is converted to have the precision of the
         target.

         Operations on numbers are performed according to the normal rules
         of arithmetic, within implementation-defined limits, except as
         provided for in Subclause 6.12, "<numeric value expression>".

         4.4.2  Operations involving numbers

         As well as the usual arithmetic operators, plus, minus, times,
         divide, unary plus, and unary minus, there are the following func-
         tions that return numbers:

         -  <position expression> (see Subclause 4.2.2, "Operations involv-
            ing character strings", and Subclause 4.3.2, "Operations involv-
            ing bit strings") takes two strings as arguments and returns an
            integer;

         -  <length expression> (see Subclause 4.2.2, "Operations involving
            character strings", and Subclause 4.3.2, "Operations involv-
            ing bit strings") operates on a string argument and returns an
            integer;

         -  <extract expression> (see Subclause 4.5.3, "Operations involving
            datetimes and intervals") operates on a datetime or interval
            argument and returns an integer.

         4.5  Datetimes and intervals

         A datetime data type is described by a datetime data type descrip-
         tor. An interval data type is described by an interval data type
         descriptor.

         A datetime data type descriptor contains:

         -  the name of the specific datetime data type (DATE, TIME,
            TIMESTAMP, TIME WITH TIME ZONE, or TIMESTAMP WITH TIME ZONE);
            and


                                                               Concepts   29

 





          X3H2-92-154/DBL CBR-002
         4.5 Datetimes and intervals


         -  the value of the <time fractional seconds precision>, if it is
            a TIME, TIMESTAMP, TIME WITH TIME ZONE, or TIMESTAMP WITH TIME
            ZONE type.

         An interval data type descriptor contains:

         -  the name of the interval data type (INTERVAL);

         -  an indication of whether the interval data type is a year-month
            interval or a day-time interval; and

         -  the <interval qualifier> that describes the precision of the
            interval data type.

         Every datetime or interval data type has an implied length in po-
         sitions. Let D denote a value in some datetime or interval data
         type DT. The length in positions of DT is constant for all D. The
         length in positions is the number of characters from the character
         set SQL_TEXT that it would take to represent any value in a given
         datetime or interval data type.

         4.5.1  Datetimes

         Table 4, "Fields in datetime items", specifies the fields that can
         make up a date time value; a datetime value is made up of a subset
         of those fields. Not all of the fields shown are required to be in
         the subset, but every field that appears in the table between the
         first included primary field and the last included primary field
         shall also be included. If either timezone field is in the subset,
         then both of them shall be included.

         __________________Table_4-Fields_in_datetime_items_________________

         _Keyword____________Meaning________________________________________

        |__________________|___Primary_datetime_fields_____________________|
        |                  |                                               |
        | YEAR               Year                                          |
        |                                                                  |
        | MONTH            | Month within year                             |
        |                  |                                               |
        | DAY              | Day within month                              |
        |                  |                                               |
        | HOUR             | Hour within day                               |
        |                  |                                               |
        | MINUTE           | Minute within hour                            |
        |                  |                                               |
        | SECOND           | Second and possibly fraction of a second      |
         ____________________within_minute__________________________________

        |__________________|___Timezone_datetime_fields____________________|
        |                  |                                               |
        | TIMEZONE_HOUR    | Hour value of time zone displacement          |
        |                                                                  |
        |_TIMEZONE_MINUTE__|_Minute_value_of_time_zone_displacement________|
        |                  |                                               |
         30  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 4.5 Datetimes and intervals


         There is an ordering of the significance of <datetime field>s. This
         is, from most significant to least significant: YEAR, MONTH, DAY,
         HOUR, MINUTE, and SECOND.

         The <datetime field>s other than SECOND contain non-negative in-
         teger values, constrained by the natural rules for dates using the
         Gregorian calendar. SECOND, however, can be defined to have a <time
         fractional seconds precision> that indicates the number of decimal
         digits maintained following the decimal point in the seconds value,
         a non-negative exact numeric value.

         There are three classes of datetime data types defined within this
         International Standard:

         -  DATE - contains the <datetime field>s YEAR, MONTH, and DAY;

         -  TIME - contains the <datetime field>s HOUR, MINUTE, and SECOND;
            and

         -  TIMESTAMP - contains the <datetime field>s YEAR, MONTH, DAY,
            HOUR, MINUTE, and SECOND.

         Items of type datetime are mutually comparable only if they have
         the same <datetime field>s.

         Datetimes only have absolute meaning in the context of additional
         information. Time zones are political divisions of the earth's
         surface that allow the convention that time is measured the same
         at all locations within the time zone, regardless of the precise
         value of "sun time" at specific locations. Political entities often
         change the "local time" within a time zone for certain periods of
         the year, e.g., in the summer. However, different political enti-
         ties within the same time zone are not necessarily synchronized in
         their local time changes. When a datetime is specified (in SQL-data
         or elsewhere) it has an implied or explicit time zone specifier as-
         sociated with it. Unless that time zone specifier, and its meaning,
         is known, the meaning of the datetime value is ambiguous.

         Therefore, datetime data types that contain time fields (TIME and
         TIMESTAMP) are maintained in Universal Coordinated Time (UTC), with
         an explicit or implied time zone part.

         The time zone part is an interval specifying the difference between
         UTC and the actual date and time in the time zone represented by
         the time or timestamp data item. The time zone displacement is
         defined as

              INTERVAL HOUR TO MINUTE

         A TIME or TIMESTAMP that does not specify WITH TIME ZONE has an im-
         plicit time zone equal to the local time zone for the SQL-session.
         The value of time represented in the data changes along with the
         local time zone for the SQL-session. However, the meaning of the
         time does not change because it is effectively maintained in UTC.

                                                               Concepts   31

 





          X3H2-92-154/DBL CBR-002
         4.5 Datetimes and intervals


         Note: On occasion, UTC is adjusted by the omission of a second or
         the insertion of a "leap second" in order to maintain synchro-
         nization with sidereal time. This implies that sometimes, but very
         rarely, a particular minute will contain exactly 59, 61, or 62
         seconds.

         4.5.2  Intervals

         There are two classes of intervals. One class, called year-month
         intervals, has an express or implied datetime precision that in-
         cludes no fields other than YEAR and MONTH, though not both are
         required. The other class, called day-time intervals, has an ex-
         press or implied interval precision that can include any fields
         other than YEAR or MONTH.

         Table 5, "Fields in year-month INTERVAL items", specifies the
         fields that make up a year-month interval. A year-month interval
         is made up of a contiguous subset of those fields.

         ____________Table_5-Fields_in_year-month_INTERVAL_items____________

         _Keyword______Meaning______________________________________________

        | YEAR       | Years                                               |
        |            |                                                     |
        |_MONTH______|_Months______________________________________________|
        |            |                                                     |
         Table 6, "Fields in day-time INTERVAL items", specifies the fields
         that make up a day-time interval. A day-time interval is made up of
         a contiguous subset of those fields.

         _____________Table_6-Fields_in_day-time_INTERVAL_items_____________

         _Keyword______Meaning______________________________________________

        | DAY        | Days                                                |
        |            |                                                     |
        | HOUR       | Hours                                               |
        |            |                                                     |
        | MINUTE     | Minutes                                             |
        |            |                                                     |
        |_SECOND_____|_Seconds_and_possibly_fractions_of_a_second__________|
        |            |                                                     |
         The actual subset of fields that comprise an item of either type of
         interval is defined by an <interval qualifier> and this subset is
         known as the precision of the item.

         Within an item of type interval, the first field is constrained
         only by the <interval leading field precision> of the associated
         <interval qualifier>. Table 7, "Valid values for fields in INTERVAL
         items", specifies the constraints on subsequence field values.



         32  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 4.5 Datetimes and intervals


         _________Table_7-Valid_values_for_fields_in_INTERVAL_items_________

         _Keyword______Valid_values_of_INTERVAL_fields______________________

        | YEAR       | Unconstrained except by <interval leading field     |
                       precision

        | MONTH      | Months (within years) (0-11)                        |
        |            |                                                     |
        | DAY        | Unconstrained except by <interval leading field     |
                       precision

        | HOUR       | Hours (within days) (0-23)                          |
        |            |                                                     |
        | MINUTE     | Minutes (within hours) (0-59)                       |
        |            |                                                     |
        |_SECOND_____|_Seconds_(within_minutes)_(0-59.999...)______________|
        |            |                                                     |
         Values in interval fields other than SECOND are integers. SECOND,
         however, can be defined to have an <interval fractional seconds
         precision> that indicates the number of decimal digits maintained
         following the decimal point in the seconds value.

         Fields comprising an item of type interval are also constrained by
         the definition of the Gregorian calendar.

         Year-month intervals are mutually comparable only with other year-
         month intervals. If two year-month intervals have different inter-
         val precisions, they are, for the purpose of any operations between
         them, effectively converted to the same precision by appending new
         <datetime field>s to either the most significant end or the least
         significant end of one or both year-month intervals. New least sig-
         nificant <datetime field>s are assigned a value of 0. When it is
         necessary to add new most significant date time fields, the as-
         sociated value is effectively converted to the new precision in
         a manner obeying the natural rules for dates and times associated
         with the Gregorian calendar.

         Day-time intervals are mutually comparable only with other day-
         time intervals. If two day-time intervals have different interval
         precisions, they are, for the purpose of any operations between
         them, effectively converted to the same precision by appending new
         <datetime field>s to either the most significant end or the least
         significant end of one or both day-time intervals. New least sig-
         nificant <datetime field>s are assigned a value of 0. When it is
         necessary to add new most significant datetime fields, the asso-
         ciated value is effectively converted to the new precision in a
         manner obeying the natural rules for dates and times associated
         with the Gregorian calendar.





                                                               Concepts   33

 





          X3H2-92-154/DBL CBR-002
         4.5 Datetimes and intervals


         4.5.3  Operations involving datetimes and intervals

         Table 8, "Valid operators involving datetimes and intervals", spec-
         ifies the results of arithmetic expressions involving datetime and
         interval operands.

         _____Table_8-Valid_operators_involving_datetimes_and_intervals_____

          Operand            Operand
         _1__________Operator_2_________Result_Type_________________________

        | Datetime | -     | Datetime | Interval                           |
        |          |       |          |                                    |
        | Datetime | + or -| Interval | Datetime                           |
        |          |       |          |                                    |
        | Interval | +     | Datetime | Datetime                           |
        |          |       |          |                                    |
        | Interval | + or -| Interval | Interval                           |
        |          |       |          |                                    |
        | Interval | * or /| Numeric  | Interval                           |
        |          |       |          |                                    |
        |_Numeric__|_*_____|_Interval_|_Interval___________________________|
        |          |       |          |                                    |
         Arithmetic operations involving items of type datetime or inter-
         val obey the natural rules associated with dates and times and
         yield valid datetime or interval results according to the Gregorian
         calendar.

         Operations involving items of type datetime require that the date-
         time items be mutually comparable. Operations involving items of
         type interval require that the interval items be mutually compara-
         ble.

         Operations involving a datetime and an interval preserve the time
         zone of the datetime operand. If the datetime operand does not
         include a time zone part, then the local time zone is effectively
         used.

         <overlaps predicate> uses the operator OVERLAPS to determine
         whether or not two chronological periods overlap in time. A chrono-
         logical period is specified either as a pair of datetimes (starting
         and ending) or as a starting datetime and an interval.

         <extract expression> operates on a datetime or interval and returns
         an exact numeric value representing the value of one component of
         the datetime or interval.

         4.6  Type conversions and mixing of data types

         Values of the data types NUMERIC, DECIMAL, INTEGER, SMALLINT,
         FLOAT, REAL, and DOUBLE PRECISION are numbers and are all mutually
         comparable and mutually assignable. If an assignment would result
         in a loss of the most significant digits, an exception condition
         is raised. If least significant digits are lost, implementation-
         defined rounding or truncating occurs with no exception condition
         being raised. The rules for arithmetic are generally governed by
         Subclause 6.12, "<numeric value expression>".

         34  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                               4.6 Type conversions and mixing of data types


         Values corresponding to the data types CHARACTER and CHARACTER
         VARYING are mutually assignable if and only if they are taken from
         the same character repertoire. If they are from different character
         repertoires, then the value of the source of the assignment shall
         be translated to the character repertoire of the target before an
         assignment is possible. If a store assignment would result in the
         loss of non-<space> characters due to truncation, then an exception
         condition is raised. The values are mutually comparable only if
         they are mutually assignable and can be coerced to have the same
         collation. The comparison of two character strings depends on the
         collating sequence used for the comparison (see Table 3, "Collating
         sequence usage for comparisons"). When values of unequal length
         are compared, if the collating sequence for the comparison has
         the NO PAD attribute and the shorter value is equal to a prefix of
         the longer value, then the shorter value is considered less than
         the longer value. If the collating sequence for the comparison has
         the PAD SPACE attribute, for the purposes of the comparison, the
         shorter value is effectively extended to the length of the longer
         by concatenation of <space>s on the right.

         Values corresponding to the data types BIT and BIT VARYING are al-
         ways mutually comparable and are mutually assignable. If a store
         assignment would result in the loss of bits due to truncation, then
         an exception condition is raised. When values of unequal length are
         to be compared, if the shorter is a prefix of the longer, then the
         shorter is less than the longer; otherwise, the longer is effec-
         tively truncated to the length of the shorter for the purposes of
         comparison. When values of equal length are to be compared, then a
         bit-by-bit comparison is made. A 0-bit less than a 1-bit.

         Values of type datetime are mutually assignable only if the source
         and target of the assignment have the same datetime fields.

         Values of type interval are mutually assignable only if the source
         and target of the assignment are both year-month intervals or if
         they are both day-time intervals.

         Implicit type conversion can occur in expressions, fetch opera-
         tions, single row select operations, inserts, deletes, and updates.
         Explicit type conversions can be specified by the use of the CAST
         operator.

         4.7  Domains

         A domain is a set of permissible values. A domain is defined in
         a schema and is identified by a <domain name>. The purpose of a
         domain is to constrain the set of valid values that can be stored
         in SQL-data by various operations.

         A domain definition specifies a data type. It may also specify a
         <domain constraint> that further restricts the valid values of the
         domain and a <default clause> that specifies the value to be used
         in the absence of an explicitly specified value or column default.

                                                               Concepts   35

 





          X3H2-92-154/DBL CBR-002
         4.7 Domains


         A domain is described by a domain descriptor. A domain descriptor
         includes:

         -  the name of the domain;

         -  the data type descriptor of the data type of the domain;

         -  the <collation name> from the <collate clause>, if any, of the
            domain;

         -  the value of <default option>, if any, of the domain; and

         -  the domain constraint descriptors of the domain constraints, if
            any, of the domain.

         4.8  Columns

         A column is a multiset of values that may vary over time. All val-
         ues of the same column are of the same data type or domain and are
         values in the same table. A value of a column is the smallest unit
         of data that can be selected from a table and the smallest unit of
         data that can be updated.

         Every column has a <column name>.

         Every column has a nullability characteristic of known not nullable
         or possibly nullable, defined as follows:

         A column has a nullability characteristic that indicates whether
         any attempt to store a null value into that column will inevitably
         raise an exception, or whether any attempt to retrieve a value
         from that column can ever result in a null value. A column C with
         <column name> CN of a base table T has a nullability characteristic
         that is known not nullable if and only if either:

         -  there exists at least one constraint that is not deferrable and
            that simply contains a <search condition> that contains CN IS
            NOT NULL or NOT CN IS NULL or RVE IS NOT NULL, where RVE is a
            <row value constructor> that contains a <row value constructor
            expression> that is simply CN without an intervening <search
            condition> that specifies OR and without an intervening <boolean
            factor> that specifies NOT.

         -  C is based on a domain that has a domain constraint that is
            not deferrable and that simply contains a <search condition>
            that contains VALUE IS NOT NULL or NOT VALUE IS NULL without an
            intervening <search condition> that specifies OR and without an
            intervening <boolean factor> that specifies NOT.

         -  CN is contained in a non-deferrable <unique constraint defi-
            nition> whose <unique specification> specifies PRIMARY KEY.

         Otherwise, a column C is possibly nullable.

         36  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                                 4.8 Columns


         A column is described by a column descriptor. A column descriptor
         includes:

         -  the name of the column;

         -  whether the name of the column is an implementation-dependent
            name;

         -  if the column is based on a domain, then the name of that do-
            main; otherwise, the data type descriptor of the data type of
            the column;

         -  the <collation name> from the <collate clause>, if any, of the
            column;

         -  the value of <default option>, if any, of the column;

         -  the nullability characteristic of the column; and

         -  the ordinal position of the column within the table that con-
            tains the column.

         4.9  Tables

         A table is a multiset of rows. A row is a nonempty sequence of
         values. Every row of the same table has the same cardinality and
         contains a value of every column of that table. The i-th value in
         every row of a table is a value of the i-th column of that table.
         The row is the smallest unit of data that can be inserted into a
         table and deleted from a table.

         The degree of a table is the number of columns of that table. At
         any time, the degree of a table is the same as the cardinality of
         each of its rows and the cardinality of a table is the same as the
         cardinality of each of its columns. A table whose cardinality is 0
         is said to be empty.

         A table is either a base table, a viewed table, or a derived table.
         A base table is either a persistent base table, a global tempo-
         rary table, a created local temporary table, or a declared local
         temporary table.

         A persistent base table is a named table defined by a <table defi-
         nition> that does not specify TEMPORARY.

         A derived table is a table derived directly or indirectly from one
         or more other tables by the evaluation of a <query expression>.
         The values of a derived table are derived from the values of the
         underlying tables when the <query expression> is evaluated.

         A viewed table is a named derived table defined by a <view defini-
         tion>. A viewed table is sometimes called a view.


                                                               Concepts   37

 





          X3H2-92-154/DBL CBR-002
         4.9 Tables


         The terms simply underlying table, underlying table, leaf underly-
         ing table, generally underlying table, and leaf generally underly-
         ing table define a relationship between a derived table or cursor
         and other tables.

         The simply underlying tables of derived tables and cursors are
         defined in Subclause 7.9, "<query specification>", Subclause 7.10,
         "<query expression>", and Subclause 13.1, "<declare cursor>". A
         viewed table has no simply underlying tables.

         The underlying tables of a derived table or cursor are the simply
         underlying tables of the derived table or cursor and the underlying
         tables of the simply underlying tables of the derived table or
         cursor.

         The leaf underlying tables of a derived table or cursor are the
         underlying tables of the derived table or cursor that do not them-
         selves have any underlying tables.

         The generally underlying tables of a derived table or cursor are
         the underlying tables of the derived table or cursor and, for those
         underlying tables of the derived table or cursor that are viewed
         tables, the <query expression> of each viewed table and the gen-
         erally underlying tables of the <query expression> of each viewed
         table.

         The leaf generally underlying tables of a derived table or cursor
         are the generally underlying tables of the derived table or cursor
         that do not themselves have any generally underlying tables.

         All base tables are updatable. Derived tables are either updatable
         or read-only. The operations of insert, update, and delete are
         permitted for updatable tables, subject to constraining Access
         Rules. The operations of insert, update, and delete are not allowed
         for read-only tables.

         A grouped table is a set of groups derived during the evaluation
         of a <group by clause> or a <having clause>. A group is a multiset
         of rows in which all values of the grouping column or columns are
         equal if a <group by clause> is specified, or the group is the
         entire table if no <group by clause> is specified. A grouped table
         may be considered as a collection of tables. Set functions may
         operate on the individual tables within the grouped table.

         A global temporary table is a named table defined by a <table defi-
         nition> that specifies GLOBAL TEMPORARY. A created local temporary
         table is a named table defined by a <table definition> that speci-
         fies LOCAL TEMPORARY. Global and created local temporary tables are
         effectively materialized only when referenced in an SQL-session.
         Every <module> in every SQL-session that references a created local
         temporary table causes a distinct instance of that created local
         temporary table to be materialized. That is, the contents of a
         global temporary table or a created local temporary table cannot
         be shared between SQL-sessions. In addition, the contents of a cre-
         ated local temporary table cannot be shared between <module>s of a
         single SQL-session. The definition of a global temporary table or a
         created local temporary table appears in a schema. In SQL language,

         38  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                                  4.9 Tables


         the name and the scope of the name of a global temporary table or
         a created local temporary table are indistinguishable from those
         of a persistent base table. However, because global temporary ta-
         ble contents are distinct within SQL-sessions, and created local
         temporary tables are distinct within <module>s within SQL-sessions,
         the effective <schema name> of the schema in which the global tem-
         porary table or the created local temporary table is instantiated
         is an implementation-dependent <schema name> that may be thought
         of as having been effectively derived from the <schema name> of
         the schema in which the global temporary table or created local
         temporary table is defined and the implementation-dependent SQL-
         session identifier associated with the SQL-session. In addition,
         the effective <schema name> of the schema in which the created
         local temporary table is instantiated may be thought of as being
         further qualified by a unique implementation-dependent name associ-
         ated with the <module> in which the created local temporary table
         is referenced.

         A declared local temporary table is a named table defined by a
         <temporary table declaration> that is effectively materialized
         the first time any <procedure> in the <module> that contains the
         <temporary table declaration> is executed. A declared local tem-
         porary table is accessible only by <procedure>s in the <module>
         that contains the <temporary table declaration>. The effective
         <schema name> of the <qualified name> of the declared local tem-
         porary table may be thought of as the implementation-dependent
         SQL-session identifier associated with the SQL-session and a unique
         implementation-dependent name associated with the <module> that
         contains the <temporary table declaration>. All references to a
         declared local temporary table are prefixed by "MODULE.".

         The materialization of a temporary table does not persist beyond
         the end of the SQL-session in which the table was materialized.
         Temporary tables are effectively empty at the start of an SQL-
         session.

         A table is described by a table descriptor. A table descriptor is
         either a base table descriptor, a view descriptor, or a derived
         table descriptor (for a derived table that is not a view).

         Every table descriptor includes:

         -  the degree of the table (the number of column descriptors); and

         -  the column descriptor of each column in the table.

         A base table descriptor describes a base table. In addition to
         the components of every table descriptor, a base table descriptor
         includes:

         -  the name of the base table;

         -  an indication of whether the table is a persistent base table,
            a global temporary table, a created local temporary table, or a
            declared local temporary table; and

                                                               Concepts   39

 





          X3H2-92-154/DBL CBR-002
         4.9 Tables


         -  the descriptor of each table constraint specified for the table.

         A derived table descriptor describes a derived table. In addi-
         tion to the components of every table descriptor, a derived table
         descriptor includes:

         -  if the table is named, then the name of the table;

         -  the <query expression> that defines how the table is to be de-
            rived; and

         -  an indication of whether the derived table is updatable or read-
            only (this is derived from the <query expression>);

         A view descriptor describes a view. In addition to the components
         of a derived table descriptor, a view descriptor includes:

         -  an indication of whether the view has the CHECK OPTION; if so,
            whether it is to be applied as CASCADED or LOCAL.

         4.10  Integrity constraints

         Integrity constraints, generally referred to simply as constraints,
         define the valid states of SQL-data by constraining the values
         in the base tables. A constraint is either a table constraint,
         a domain constraint or an assertion. A constraint is described
         by a constraint descriptor. A constraint descriptor is either a
         table constraint descriptor, a domain constraint descriptor or an
         assertion descriptor. Every constraint descriptor includes:

         -  the name of the constraint;

         -  an indication of whether or not the constraint is deferrable;

         -  an indication of whether the initial constraint mode is deferred
            or immediate;

         A <query expression> or <query specification> is possibly non-
         deterministic if an implementation might, at two different times
         where the state of the SQL-data is the same, produce results that
         differ by more than the order of the rows due to General Rules that
         specify implementation-dependent behavior.

         No integrity constraint shall be defined using a <query specifica-
         tion> or a <query expression> that is possibly non-deterministic.









         40  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                  4.10 Integrity constraints


         4.10.1  Checking of constraints

         Every constraint is either deferrable or non-deferrable. Within
         a transaction, every constraint has a constraint mode; if a con-
         straint is non-deferrable, then its constraint mode is always im-
         mediate, otherwise it is either or immediate or deferred. Every
         constraint has an initial constraint mode that specifies the
         constraint mode for that constraint at the start of each SQL-
         transaction and immediately after definition of that constraint.
         If a constraint is deferrable, then its constraint mode may be
         changed (from immediate to deferred, or from deferred to immediate)
         by execution of a <set constraints mode statement>.

         The checking of a constraint depends on its constraint mode within
         the current SQL-transaction. If the constraint mode is immedi-
         ate, then the constraint is effectively checked at the end of
         each SQL-statement. If the constraint mode is deferred, then the
         constraint is effectively checked when the constraint mode is
         changed to immediate either explicitly by execution of a <set con-
         straints mode statement>, or implicitly at the end of the current
         SQL-transaction.

         When a constraint is checked other than at the end of an SQL-
         transaction, if it is not satisfied, then an exception condition
         is raised and the SQL-statement that caused the constraint to be
         checked has no effect other than entering the exception information
         into the diagnostics area. When a <commit statement> is executed,
         all constraints are effectively checked and, if any constraint
         is not satisfied, then an exception condition is raised and the
         transaction is terminated by an implicit <rollback statement>.

         4.10.2  Table constraints

         A table constraint is either a unique constraint, a referential
         constraint or a table check constraint. A table constraint is de-
         scribed by a table constraint descriptor which is either a unique
         constraint descriptor, a referential constraint descriptor or a
         table check constraint descriptor.

         A unique constraint is described by a unique constraint descriptor.
         In addition to the components of every table constraint descriptor,
         a unique constraint descriptor includes:

         -  an indication of whether it was defined with PRIMARY KEY or
            UNIQUE, and

         -  the names and positions of the unique columns specified in the
            <unique column list>;

         A referential constraint is described by a referential constraint
         descriptor. In addition to the components of every table constraint
         descriptor, a referential constraint descriptor includes:


                                                               Concepts   41

 





          X3H2-92-154/DBL CBR-002
         4.10 Integrity constraints


         -  the names of the referencing columns specified in the <referenc-
            ing columns>,

         -  the names of the referenced columns and referenced table speci-
            fied in the <referenced table and columns>, and

         -  the value of the <match type>, if specified, and the <referen-
            tial triggered actions>, if specified.

         Note: If MATCH FULL or MATCH PARTIAL is specified for a referential
         constraint and if the referencing table has only one column spec-
         ified in <referential constraint definition> for that referential
         constraint, or if the referencing table has more than one specified
         column for that <referential constraint definition>, but none of
         those columns is nullable, then the effect is the same as if no
         <match option> were specified.

         A table check constraint is described by a table check constraint
         descriptor. In addition to the components of every table constraint
         descriptor, a table check constraint descriptor includes:

         -  the <search condition>.

         A unique constraint is satisfied if and only if no two rows in
         a table have the same non-null values in the unique columns. In
         addition, if the unique constraint was defined with PRIMARY KEY,
         then it requires that none of the values in the specified column or
         columns be the null value.

         In the case that a table constraint is a referential constraint,
         the table is referred to as the referencing table. The referenced
         columns of a referential constraint shall be the unique columns of
         some unique constraint of the referenced table.

         A referential constraint is satisfied if one of the following con-
         ditions is true, depending on the <match option> specified in the
         <referential constraint definition>:

         -  If no <match type> was specified then, for each row R1 of the
            referencing table, either at least one of the values of the
            referencing columns in R1 shall be a null value, or the value of
            each referencing column in R1 shall be equal to the value of the
            corresponding referenced column in some row of the referenced
            table.

         -  If MATCH FULL was specified then, for each row R1 of the refer-
            encing table, either the value of every referencing column in R1
            shall be a null value, or the value of every referencing column
            in R1 shall not be null and there shall be some row R2 of the
            referenced table such that the value of each referencing col-
            umn in R1 is equal to the value of the corresponding referenced
            column in R2.


         42  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                  4.10 Integrity constraints


         -  If MATCH PARTIAL was specified then, for each row R1 of the
            referencing table, there shall be some row R2 of the refer-
            enced table such that the value of each referencing column in
            R1 is either null or is equal to the value of the corresponding
            referenced column in R2.

         The referencing table may be the same table as the referenced ta-
         ble.

         A table check constraint is satisfied if and only if the specified
         <search condition> is not false for any row of a table.

         4.10.3  Domain constraints

         A domain constraint is a constraint that is specified for a domain.
         It is applied to all columns that are based on that domain, and to
         all values cast to that domain.

         A domain constraint is described by a domain constraint descriptor.
         In addition to the components of every constraint descriptor a
         domain constraint descriptor includes:

         -  the <search condition>.

         A domain constraint is satisfied by SQL-data if and only if, for
         any table T that has a column named C based on that domain, the
         specified <search condition>, with each occurrence of VALUE re-
         placed by C, is not false for any row of T.

         A domain constraint is satisfied by the result of a <cast specifi-
         cation> if and only if the specified <search condition>, with each
         occurrence of VALUE replaced by that result, is not false.

         4.10.4  Assertions

         An assertion is a named constraint that may relate to the content
         of individual rows of a table, to the entire contents of a table,
         or to a state required to exist among a number of tables.

         An assertion is described by an assertion descriptor. In addi-
         tion to the components of every constraint descriptor an assertion
         descriptor includes:

         -  the <search condition>.

         An assertion is satisfied if and only if the specified <search
         condition> is not false.







                                                               Concepts   43

 





          X3H2-92-154/DBL CBR-002
         4.11 SQL-schemas


         4.11  SQL-schemas

         An SQL-schema is a persistent descriptor that includes:

         -  the <schema name> of the SQL-schema;

         -  the <authorization identifier> of the owner of the SQL-schema;

         -  The <character set name> of the default character set for the
            SQL-schema; and

         -  the descriptor of every component of the SQL-schema.

         In this International Standard, the term "schema" is used only
         in the sense of SQL-schema. Each component descriptor is either
         a domain descriptor, a base table descriptor, a view descriptor,
         an assertion descriptor, a privilege descriptor, a character set
         descriptor, a collation descriptor, or a translation descriptor.
         The persistent objects described by the descriptors are said to be
         owned by or to have been created by the <authorization identifier>
         of the schema.

         A schema is created initially using a <schema definition> and may
         be subsequently modified incrementally over time by the execution
         of <SQL schema statement>s. <schema name>s are unique within a
         catalog.

         A <schema name> is explicitly or implicitly qualified by a <catalog
         name> that identifies a catalog.

         Base tables and views are identified by <table name>s. A <table
         name> consists of a <schema name> and an <identifier>. For a per-
         sistent table, the <schema name> identifies the schema in which
         the base table or view identified by the <table name> was de-
         fined. Base tables and views defined in different schemas can
         have <identifier>s that are equal according to the General Rules
         of Subclause 8.2, "<comparison predicate>".

         If a reference to a <table name> does not explicitly contain a
         <schema name>, then a specific <schema name> is implied. The par-
         ticular <schema name> associated with such a <table name> depends
         on the context in which the <table name> appears and is governed
         by the rules for <qualified name>. The default schema for <prepara-
         ble statement>s that are dynamically prepared in the current SQL-
         session through the execution of <prepare statement>s and <execute
         immediate statement>s is initially implementation-defined but may
         be changed by the use of <set schema statement>s.







         44  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                               4.12 Catalogs


         4.12  Catalogs

         Catalogs are named collections of schemas in an SQL-environment. An
         SQL-environment contains zero or more catalogs. A catalog con-
         tains one or more schemas, but always contains a schema named
         INFORMATION_SCHEMA that contains the views and domains of the
         Information Schema. The method of creation and destruction of
         catalogs is implementation-defined. The set of catalogs that
         can be referenced in any SQL-statement, during any particular
         SQL-transaction, or during the course of an SQL-session is also
         implementation-defined. The default catalog for a <module> whose
         <module authorization clause> does not specify an explicit <cata-
         log name> to qualify the <schema name> is implementation-defined.
         The default catalog for <preparable statement>s that are dynami-
         cally prepared in the current SQL-session through the execution
         of <prepare statement>s and <execute immediate statement>s is ini-
         tially implementation-defined but may be changed by the use of <set
         catalog statement>s.


         4.13  Clusters of catalogs


         A cluster is an implementation-defined collection of catalogs.
         Exactly one cluster is associated with an SQL-session and it
         defines the totality of the SQL-data that is available to that
         SQL-session.

         An instance of a cluster is described by an instance of a defi-
         nition schema. Given some SQL-data object, such as a view, a con-
         straint, a domain, or a base table, the definition of that object,
         and of all the objects that it directly or indirectly references,
         are in the same cluster of catalogs. For example, no <referential
         constraint definition> and no <joined table> can "cross" a cluster
         boundary.

         Whether or not any catalog can occur simultaneously in more than
         one cluster is implementation-defined.

         Within a cluster, no two catalogs have the same name.

         4.14  SQL-data

         SQL-data is any data described by schemas that is under the control
         of an SQL-implementation in an SQL-environment.









                                                               Concepts   45

 





          X3H2-92-154/DBL CBR-002
         4.15 SQL-environment


         4.15  SQL-environment

         An SQL-environment comprises the following:

         -  an SQL-implementation capable of processing some Level (Entry
            SQL, Intermediate SQL, or Full SQL) of this International
            Standard and at least one binding style; see Clause 23,
            "Conformance" for further information about binding styles;

         -  zero or more catalogs;

         -  zero or more <authorization identifier>s;

         -  zero or more <module>s; and

         -  the SQL-data described by the schemas in the catalogs.

         An SQL-environment may have other implementation-defined contents.

         The rules determining which <module>s are considered to be within
         an SQL-environment are implementation-defined.

         4.16  Modules

         A <module> is an object specified in the module language. A <mod-
         ule> is either a persistent <module> or an SQL-session <module>.
         The mechanisms by which <module>s are created or destroyed are
         implementation-defined. A <module> consists of an optional <module
         name>, a <language clause>, a <module authorization clause> with
         either or both of a <module authorization identifier> and a <schema
         name>, an optional <module character set specification> that iden-
         tifies the character repertoire used for expressing the names of
         schema objects used in the <module>, zero or more <temporary table
         declaration>s, zero or more cursors specified by <declare cur-
         sor>s, and one or more <procedure>s. All <identifier>s contained
         in the <module> are expressed in either <SQL language character>
         or the character repertoire indicated by <module character set
         specification> unless they are specified with "<introducer>".

         A compilation unit is a segment of executable code, possibly con-
         sisting of one or more subprograms. A <module> is associated with
         a compilation unit during its execution. A single <module> may be
         associated with multiple compilation units and multiple <module>s
         may be associated with a single compilation unit. The manner in
         which this association is specified, including the possible re-
         quirement for execution of some implementation-defined statement,
         is implementation-defined. Whether a compilation unit may invoke or
         transfer control to other compilation units, written in the same or
         a different programming language, is implementation-defined.





         46  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             4.17 Procedures


         4.17  Procedures

         A <procedure> consists of a <procedure name>, a sequence of <pa-
         rameter declaration>s, and a single <SQL procedure statement>.


         A <procedure> in a <module> is invoked by a compilation unit as-
         sociated with the <module> by means of a host language "call"
         statement that specifies the <procedure name> of the <procedure>
         and supplies a sequence of parameter values corresponding in number
         and in <data type> to the <parameter declaration>s of the <proce-
         dure>. A call of a <procedure> causes the <SQL procedure statement>
         that it contains to be executed.

         4.18  Parameters

         A parameter is declared in a <procedure> by a <parameter decla-
         ration>. The <parameter declaration> specifies the <data type>
         of its value. A parameter either assumes or supplies the value of
         the corresponding argument in the call of that <procedure>. These
         <data type>s map to host language types and are not nullable except
         through the use of additional indicator variables.


         4.18.1  Status parameters

         The SQLSTATE and SQLCODE parameters are status parameters. They
         are set to status codes that indicate either that a call of the
         <procedure> completed successfully or that an exception condition
         was raised during execution of the <procedure>.

         Note: The SQLSTATE parameter is the preferred status parameter. The
         SQLCODE parameter is a deprecated feature that is supported for
         compatibility with earlier versions of this International Standard.
         See Annex D, "Deprecated features".

         A <procedure> shall specify either the SQLSTATE parameter or the
         SQLCODE parameter or both. The SQLSTATE parameter is a charac-
         ter string parameter for which exception values are defined in
         Clause 22, "Status codes". The SQLCODE parameter is an integer pa-
         rameter for which the negative exception values are implementation-
         defined.

         If a condition is raised that causes a statement to have no effect
         other than that associated with raising the condition (that is,
         not a completion condition), then the condition is said to be an
         exception condition or exception. If a condition is raised that
         permits a statement to have an effect other than that associated
         with raising the condition (corresponding to an SQLSTATE class
         value of successful completion, warning, or no data), then the
         condition is said to be a completion condition.



                                                               Concepts   47

 





          X3H2-92-154/DBL CBR-002
         4.18 Parameters


         4.18.2  Data parameters

         A data parameter is a parameter that is used to either assume or
         supply the value of data exchanged between a host program and an
         SQL-implementation.

         4.18.3  Indicator parameters

         An indicator parameter is an integer parameter that is specified
         immediately following another parameter. Its primary use is to
         indicate whether the value that the other parameter assumes or
         supplies is a null value. An indicator parameter cannot immediately
         follow another indicator parameter.

         The other use for indicator parameters is to indicate whether
         string data truncation occurred during a transfer between a host
         program and an SQL-implementation in parameters or host variables.
         If a non-null string value is transferred and the length of the
         target data item is sufficient to accept the entire source data
         item, then the indicator parameter or variable is set to 0 to in-
         dicate that truncation did not occur. However, if the length of
         the target data item is insufficient, then the indicator parame-
         ter or variable is set to the length of the source data item (in
         characters or bits, as appropriate) to indicate that truncation
         occurred and to indicate the original length in characters or bits,
         as appropriate, of the source.

         4.19  Diagnostics area

         The diagnostics area is a place where completion and exception con-
         dition information is stored when an SQL-statement is executed.
         There is one diagnostics area associated with an SQL-agent, regard-
         less of the number of <module>s that the SQL-agent includes or the
         number of connections in use.

         At the beginning of the execution of any statement that is not an
         <SQL diagnostics statement>, the diagnostics area is emptied. An
         implementation shall place information about a completion condition
         or an exception condition reported by SQLCODE or SQLSTATE into this
         area. If other conditions are raised, an implementation may place
         information about them into this area.

         <procedure>s containing <SQL diagnostics statement>s return a code
         indicating completion or exception conditions for that statement
         via SQLCODE or SQLSTATE, but do not modify the diagnostics area.

         An SQL-agent may choose the size of the diagnostics area with the
         <set transaction statement>; if an SQL-agent does not specify the
         size of the diagnostics area, then the size of the diagnostics
         area is implementation-dependent, but shall always be able to hold
         information about at least one condition. An implementation may
         place information into this area about fewer conditions than are
         specified. The ordering of the information about conditions placed

         48  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       4.19 Diagnostics area


         into the diagnostics area is implementation-dependent, except that
         the first condition in the diagnostics area always corresponds to
         the condition specified by the SQLSTATE or SQLCODE value.

         4.20  Standard programming languages


         This International Standard specifies the actions of <procedure>s
         in <module>s when those <procedure>s are called by programs that
         conform to certain specified programming language standards. The
         term "standard PLN program", where PLN is the name of a program-
         ming language, refers to a program that conforms to the standard
         for that programming language as specified in Clause 2, "Normative
         references". This International Standard also specifies a mechanism
         whereby SQL language may be embedded in programs that otherwise
         conform to any of the same specified programming language stan-
         dards.

         Note: In this International Standard, for the purposes of inter-
         facing with programming languages, the data types DATE, TIME,
         TIMESTAMP, and INTERVAL shall be converted to or from character
         strings in those programming languages by means of a <cast speci-
         fication>. It is anticipated that future evolution of programming
         language standards will support data types corresponding to these
         four SQL data types; this standard will then be amended to reflect
         the availability of those corresponding data types. The data type
         CHARACTER is also mapped to character strings in the programming
         languages. However, because the facilities available in the pro-
         gramming languages do not provide the same capabilities as those
         available in SQL, there shall be agreement between the host pro-
         gram and SQL regarding the specific format of the character data
         being exchanged. Specific syntax for this agreement is provided
         in this International standard. For standard programming lan-
         guages, C, COBOL, Fortran, and Pascal, bit strings are mapped to
         character variables in the host language in a manner described in
         Subclause 19.1, "<embedded SQL host program>". For standard pro-
         gramming languages Ada and PL/I, bit string variables are directly
         supported.

         4.21  Cursors


         A cursor is specified by a <declare cursor>, <dynamic declare cur-
         sor>, or <allocate cursor statement>.

         For every <declare cursor> or <dynamic declare cursor> in a <mod-
         ule>, a cursor is effectively created when an SQL-transaction (see
         Subclause 4.28, "SQL-transactions") referencing the <module> is
         initiated, and destroyed when that SQL-transaction is terminated. A
         cursor is also effectively created when an <allocate cursor state-
         ment> is executed within a SQL-transaction and destroyed when that
         SQL-transaction is terminated. In addition, an extended dynamic


                                                               Concepts   49

 





          X3H2-92-154/DBL CBR-002
         4.21 Cursors


         cursor is destroyed when a <deallocate prepared statement> is exe-
         cuted that deallocates the prepared statement on which the extended
         dynamic cursor is based.

         A cursor is in either the open state or the closed state. The ini-
         tial state of a cursor is the closed state. A cursor is placed in
         the open state by an <open statement> or <dynamic open statement>
         and returned to the closed state by a <close statement> or <dynamic
         close statement>, a <commit statement>, or a <rollback statement>.

         A cursor in the open state identifies a table, an ordering of the
         rows of that table, and a position relative to that ordering. If
         the <declare cursor> does not include an <order by clause>, or
         includes an <order by clause> that does not specify the order of
         the rows completely, then the rows of the table have an order that
         is defined only to the extent that the <order by clause> specifies
         an order and is otherwise implementation-dependent.

         When the ordering of a cursor is not defined by an <order by
         clause>, the relative positions of two rows is implementation-
         dependent. When the ordering of a cursor is partially determined
         by an <order by clause>, then the relative positions of two rows
         are determined only by the <order by clause>; if the two rows have
         equal values for the purpose of evaluating the <order by clause>,
         then their relative positions are implementation-dependent.

         A cursor is either read-only or updatable. If the table identified
         by a cursor is not updatable or if INSENSITIVE is specified for
         the cursor, then the cursor is read-only; otherwise, the cursor is
         updatable. The operations of update and delete are not allowed for
         read-only cursors.

         The position of a cursor in the open state is either before a cer-
         tain row, on a certain row, or after the last row. If a cursor is
         on a row, then that row is the current row of the cursor. A cursor
         may be before the first row or after the last row of a table even
         though the table is empty. When a cursor is initially opened, the
         position of the cursor is before the first row.

         A <fetch statement> or <dynamic fetch statement> positions an open
         cursor on a specified row of the cursor's ordering and retrieves
         the values of the columns of that row. An <update statement: po-
         sitioned> or <dynamic update statement: positioned> updates the
         current row of the cursor. A <delete statement: positioned> or <dy-
         namic delete statement: positioned> deletes the current row of the
         cursor.

         If an error occurs during the execution of an SQL-statement that
         identifies an open cursor, then, except where otherwise explic-
         itly defined, the effect, if any, on the position or state of that
         cursor is implementation-dependent.

         If a cursor is open, and the current SQL-transaction makes a change
         to SQL-data other than through that cursor, and the <declare cur-
         sor> for that cursor specified INSENSITIVE, then the effect of
         that change will not be visible through that cursor before it is
         closed. Otherwise, whether the effect of such a change will be

         50  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                                4.21 Cursors


         visible through that cursor before it is closed is implementation-
         dependent.

         4.22  SQL-statements


         4.22.1  Classes of SQL-statements

         An SQL-statement is a string of characters that conforms to the
         format and syntax rules specified in this international standard.
         Most SQL-statements can be prepared for execution and executed in
         one of a number of ways. These are:

         -  in a <module>, in which case it is prepared when the <module>
            is created (see Subclause 4.16, "Modules") and executed when the
            containing procedure is called.

         -  in an embedded SQL host program, in which case it is pre-
            pared when the embedded SQL host program is preprocessed (see
            Subclause 4.23, "Embedded syntax").

         -  being prepared and executed by the use of SQL-dynamic statements
            (which are themselves executed in one of the foregoing two ways-
            see Subclause 4.24, "SQL dynamic statements").

         -  direct invocation, in which case it is effectively prepared
            immediately prior to execution (see Subclause 4.25, "Direct
            invocation of SQL").

         There are at least five ways of classifying SQL-statements:

         -  According to their effect on SQL objects, whether persistent
            objects, i.e., SQL-data and schemas, or transient objects, such
            as SQL-sessions and other SQL-statements.

         -  According to whether or not they start a transaction, or can, or
            must, be executed when no transaction is active.

         -  According to whether or not they may be embedded.

         -  According to whether they may be dynamically prepared and exe-
            cuted.

         -  According to whether or not they may be directly executed.

         This International Standard permits implementations to provide ad-
         ditional, implementation-defined, statements that may fall into any
         of these categories. This Subclause will not mention those state-
         ments again, as their classification is entirely implementation-
         defined.




                                                               Concepts   51

 





          X3H2-92-154/DBL CBR-002
         4.22 SQL-statements


         4.22.2  SQL-statements classified by function

         The following are the main classes of SQL-statements:

         -  SQL-schema statements; these may have a persistent effect on
            schemas

         -  SQL-data statements; some of these, the SQL-data change state-
            ments, may have a persistent effect on SQL-data

         -  SQL-transaction statements; except for the <commit statement>,
            these, and the following classes, have no effects that persist
            when a session is terminated

         -  SQL-connection statements

         -  SQL-session statements

         -  SQL-dynamic statements

         -  SQL-diagnostics statements

         -  SQL embedded exception declaration

         The following are the SQL-schema statements:

         -  <schema definition>

         -  <drop schema statement>

         -  <domain definition>

         -  <drop domain statement>

         -  <table definition>

         -  <drop table statement>

         -  <view definition>

         -  <drop view statement>

         -  <assertion definition>

         -  <drop assertion statement>

         -  <alter table statement>

         -  <alter domain statement>

         -  <grant statement>

         -  <revoke statement>

         -  <character set definition>

         -  <drop character set statement>

         52  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                         4.22 SQL-statements


         -  <collation definition>

         -  <drop collation statement>

         -  <translation definition>

         -  <drop translation statement>

         The following are the SQL-data statements:

         -  <temporary table declaration>

         -  <declare cursor>

         -  <dynamic declare cursor>

         -  <allocate cursor statement>

         -  <dynamic select statement>

         -  <open statement>

         -  <dynamic open statement>

         -  <close statement>

         -  <dynamic close statement>

         -  <fetch statement>

         -  <dynamic fetch statement>

         -  <select statement: single row>

         -  <direct select statement: multiple rows>

         -  <dynamic single row select statement>

         -  All SQL-data change statements

         The following are the SQL-data change statements:

         -  <insert statement>

         -  <delete statement: searched>

         -  <delete statement: positioned>

         -  <dynamic delete statement: positioned>

         -  <preparable dynamic delete statement: positioned>

         -  <update statement: searched>

         -  <update statement: positioned>

         -  <dynamic update statement: positioned>

                                                               Concepts   53

 





          X3H2-92-154/DBL CBR-002
         4.22 SQL-statements


         -  <preparable dynamic update statement: positioned>

         The following are the SQL-transaction statements:

         -  <set transaction statement>

         -  <set constraints mode statement>

         -  <commit statement>

         -  <rollback statement>

         The following are the SQL-connection statements:

         -  <connect statement>

         -  <set connection statement>

         -  <disconnect statement>

         The following are the SQL-session statements:

         -  <set catalog statement>

         -  <set schema statement>

         -  <set names statement>

         -  <set session authorization identifier statement>

         -  <set local time zone statement>

         The following are the SQL-dynamic statements:

         -  <execute immediate statement>

         -  <allocate descriptor statement>

         -  <deallocate descriptor statement>

         -  <get descriptor statement>

         -  <set descriptor statement>

         -  <prepare statement>

         -  <deallocate prepared statement>

         -  <describe input statement>

         -  <describe output statement>

         -  <execute statement>

         The following is the SQL-diagnostics statement:

         -  <get diagnostics statement>

         54  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                         4.22 SQL-statements


         The following is the SQL embedded exception declaration:

         -  <embedded exception declaration>

         4.22.3  Embeddable SQL-statements

         The following SQL-statements are embeddable in an embedded SQL host
         program, and may be the <SQL procedure statement> in a <procedure>
         in a <module>:

         -  All SQL-schema statements

         -  All SQL-transaction statements

         -  All SQL-connection statements

         -  All SQL-session statements

         -  All SQL-dynamic statements

         -  All SQL-diagnostics statements

         -  The following SQL-data statements:

            o <allocate cursor statement>

            o <open statement>

            o <dynamic open statement>

            o <close statement>

            o <dynamic close statement>

            o <fetch statement>

            o <dynamic fetch statement>

            o <select statement: single row>

            o <insert statement>

            o <delete statement: searched>

            o <delete statement: positioned>

            o <dynamic delete statement: positioned>

            o <update statement: searched>

            o <update statement: positioned>

            o <dynamic update statement: positioned>

                                                               Concepts   55

 





          X3H2-92-154/DBL CBR-002
         4.22 SQL-statements


         The following SQL-statements are embeddable in an embedded SQL host
         program, and may occur in a <module>, though not in a <procedure>:

         -  <temporary table declaration>

         -  <declare cursor>

         -  <dynamic declare cursor>

         The following SQL-statements are embeddable in an embedded SQL host
         program, but may not occur in a <module>:

         -  SQL embedded exception declarations

         Consequently, the following SQL-data statements are not embeddable
         in an embedded SQL host program, nor may they occur in a <mod-
         ule>, nor be the <SQL procedure statement> in a <procedure> in a
         <module>:

         -  <dynamic select statement>

         -  <dynamic single row select statement>

         -  <direct select statement: multiple rows>

         -  <preparable dynamic delete statement: positioned>

         -  <preparable dynamic update statement: positioned>

         4.22.4  Preparable and immediately executable SQL-statements

         The following SQL-statements are preparable:

         -  All SQL-schema statements

         -  All SQL-transaction statements

         -  All SQL-session statements

         -  The following SQL-data statements:

            o <delete statement: searched>

            o <dynamic select statement>

            o <dynamic single row select statement>

            o <insert statement>

            o <update statement: searched>

            o <preparable dynamic delete statement: positioned>

            o <preparable dynamic update statement: positioned>

            o <preparable implementation-defined statement>

         56  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                         4.22 SQL-statements


         Consequently, the following SQL-statements are not preparable:

         -  All SQL-connection statements

         -  All SQL-dynamic statements

         -  All SQL-diagnostics statements

         -  SQL embedded exception declarations

         -  The following SQL-data statements:

            o <allocate cursor statement>

            o <open statement>

            o <dynamic open statement>

            o <close statement>

            o <dynamic close statement>

            o <fetch statement>

            o <dynamic fetch statement>

            o <select statement: single row>

            o <delete statement: positioned>

            o <dynamic delete statement: positioned>

            o <update statement: positioned>

            o <dynamic update statement: positioned>

            o <direct select statement: multiple rows>

            o <temporary table declaration>

            o <declare cursor>

            o <dynamic declare cursor>

         Any preparable SQL-statement can be executed immediately, with the
         exception of:

         -  <dynamic select statement>

         -  <dynamic single row select statement>




                                                               Concepts   57

 





          X3H2-92-154/DBL CBR-002
         4.22 SQL-statements


         4.22.5  Directly executable SQL-statements

         The following SQL-statements may be executed directly:

         -  All SQL-schema statements

         -  All SQL-transaction statements

         -  All SQL-connection statements

         -  All SQL-session statements

         -  The following SQL-data statements:

            o <temporary table declaration>

            o <direct select statement: multiple rows>

            o <insert statement>

            o <delete statement: searched>

            o <update statement: searched>

         Consequently, the following SQL-statements may not be executed
         directly:

         -  All SQL-dynamic statements

         -  All SQL-diagnostics statements

         -  SQL embedded exception declarations

         -  The following SQL-data statements:

            o <declare cursor>

            o <dynamic declare cursor>

            o <allocate cursor statement>

            o <open statement>

            o <dynamic open statement>

            o <close statement>

            o <dynamic close statement>

            o <fetch statement>

            o <dynamic fetch statement>

            o <select statement: single row>

            o <dynamic select statement>

            o <dynamic single row select statement>

         58  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                         4.22 SQL-statements


            o <delete statement: positioned>

            o <dynamic delete statement: positioned>

            o <preparable dynamic delete statement: positioned>

            o <update statement: positioned>

            o <dynamic update statement: positioned>

            o <preparable dynamic update statement: positioned>

         4.22.6  SQL-statements and transaction states

         Whether an <execute immediate statement> starts a transaction de-
         pends on what SQL-statement is the value of <SQL statement vari-
         able>. Whether an <execute statement> starts a transaction depends
         on what SQL-statement was the value of <SQL statement variable>
         when the prepared statement identified by <SQL statement name> was
         prepared.

         The following SQL-statements are transaction initiating SQL-
         statements, i.e., if there is no current transaction, and a state-
         ment of this class is executed, a transaction is initiated:

         -  All SQL-schema statements

         -  The following SQL-data statements:

            o <allocate cursor statement>

            o <dynamic select statement>

            o <open statement>

            o <dynamic open statement>

            o <close statement>

            o <dynamic close statement>

            o <fetch statement>

            o <dynamic fetch statement>

            o <select statement: single row>

            o <direct select statement: multiple rows>

            o <dynamic single row select statement>

            o <insert statement>

            o <delete statement: searched>

            o <delete statement: positioned>

                                                               Concepts   59

 





          X3H2-92-154/DBL CBR-002
         4.22 SQL-statements


            o <dynamic delete statement: positioned>

            o <preparable dynamic delete statement: positioned>

            o <update statement: searched>

            o <update statement: positioned>

            o <dynamic update statement: positioned>

            o <preparable dynamic update statement: positioned>

         -  The following SQL-dynamic statements:

            o <describe input statement>

            o <describe output statement>

            o <allocate descriptor statement>

            o <deallocate descriptor statement>

            o <get descriptor statement>

            o <set descriptor statement>

            o <prepare statement>

            o <deallocate prepared statement>

         The following SQL-statements are not transaction initiating SQL-
         statements, i.e., if there is no current transaction, and a state-
         ment of this class is executed, no transaction is initiated.

         -  All SQL-transaction statements

         -  All SQL-connection statements

         -  All SQL-session statements

         -  All SQL-diagnostics statements

         -  SQL embedded exception declarations

         -  The following SQL-data statements:

            o <temporary table declaration>

            o <declare cursor>

            o <dynamic declare cursor>

            o <dynamic select statement>

         60  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                        4.23 Embedded syntax


         4.23  Embedded syntax

         An <embedded SQL host program> (<embedded SQL Ada program>, <em-
         bedded SQL C program>, <embedded SQL COBOL program>, <embedded
         SQL Fortran program>, <embedded SQL MUMPS program>, <embedded SQL
         Pascal program>, or <embedded SQL PL/I program>) is a compilation
         unit that consists of programming language text and SQL text. The
         programming language text shall conform to the requirements of a
         specific standard programming language. The SQL text shall con-
         sist of one or more <embedded SQL statement>s and, optionally,
         one or more <embedded SQL declare section>s, as defined in this
         International Standard. This allows database applications to be
         expressed in a hybrid form in which SQL-statements are embedded
         directly in a compilation unit. Such a hybrid compilation unit is
         defined to be equivalent to a standard compilation unit in which
         the SQL-statements have been replaced by standard procedure or
         subroutine calls of SQL <procedure>s in a separate SQL <module>,
         and in which each <embedded SQL begin declare> and each <embedded
         SQL end declare> has been removed and the declarations contained
         therein have been suitably transformed into standard host-language
         syntax.

         An implementation may reserve a portion of the name space in the
         <embedded SQL host program> for the names of procedures or subrou-
         tines that are generated to replace SQL-statements and for program
         variables and branch labels that may be generated as required to
         support the calling of these procedures or subroutines; whether
         this reservation is made is implementation-defined. They may sim-
         ilarly reserve name space for the <module name> and <procedure
         name>s of the generated <module> that may be associated with the
         resulting standard compilation unit. The portion of the name space
         to be so reserved, if any, is implementation-defined.


         4.24  SQL dynamic statements

         In many cases, the SQL-statement to be executed can be coded into
         a <module> or into a compilation unit using the embedded syntax.
         In other cases, the SQL-statement is not known when the program
         is written and will be generated during program execution. An
         <execute immediate statement> can be used for a one-time prepa-
         ration and execution of an SQL-statement. A <prepare statement>
         is used to prepare the generated SQL-statement for subsequent ex-
         ecution. A <deallocate prepared statement> is used to deallocate
         SQL-statements that have been prepared with a <prepare statement>.
         A description of the input parameters for a prepared statement
         can be obtained by execution of a <describe input statement>. A
         description of the resultant columns of a <dynamic select state-
         ment> or <dynamic single row select statement> can be obtained by
         execution of a <describe output statement>. For a statement other
         than a <dynamic select statement>, an <execute statement> is used
         to associate parameters with the prepared statement and execute


                                                               Concepts   61

 





          X3H2-92-154/DBL CBR-002
         4.24 SQL dynamic statements


         it as though it had been coded when the program was written. For a
         <dynamic select statement>, the prepared <cursor specification> is
         associated with a cursor via a <dynamic declare cursor> or <allo-
         cate cursor statement>. The cursor can be opened and parameters can
         be associated with the cursor with a <dynamic open statement>. A
         <dynamic fetch statement> positions an open cursor on a specified
         row and retrieves the values of the columns of that row. A <dynamic
         close statement> closes a cursor that was opened with a <dynamic
         open statement>. A <dynamic delete statement: positioned> is used
         to delete rows through a dynamic cursor. A <dynamic update state-
         ment: positioned> is used to update rows through a dynamic cursor.
         A <preparable dynamic delete statement: positioned> is used to
         delete rows through a dynamic cursor when the precise format of the
         statement isn't known until runtime. A <preparable dynamic update
         statement: positioned> is used to update rows through a dynamic
         cursor when the precise format of the statement isn't known until
         runtime.

         The interface for input parameters for a prepared statement and
         for the resulting values from a <dynamic fetch statement> or the
         execution of a prepared <dynamic single row select statement> can
         be either a list of parameters or embedded variables or an SQL
         descriptor area. An SQL descriptor area consists of zero or more
         item descriptor areas, together with a COUNT of the number of those
         item descriptor areas. Each item descriptor area consists of the
         fields specified in Table 17, "Data types of <key word>s used in
         SQL item descriptor areas", in Subclause 17.1, "Description of SQL
         item descriptor areas". The SQL descriptor area is allocated and
         maintained by the system with the following statements: <allocate
         descriptor statement>, <deallocate descriptor statement>, <set
         descriptor statement>, and <get descriptor statement>.

         An SQL descriptor area is identified by a <descriptor name>, which
         is a <simple value specification> whose value is an <identifier>.
         Two <descriptor name>s identify the same SQL descriptor area if
         their values, with leading and trailing <space>s removed, are
         equivalent according to the rules for <identifier> comparisons
         in Subclause 5.2, "<token> and <separator>".

         Dynamic statements can be identified by <statement name>s or by
         <extended statement name>s. Similarly, dynamic cursors can be
         identified by <cursor name>s and by <extended cursor name>s. The
         non-extended names are <identifier>s. The extended names are <tar-
         get specification>s whose values are <identifier>s used to iden-
         tify the statement or cursor. Two extended names are equivalent
         if their values, with leading and trailing <space>s removed, are
         equivalent according to the rules for <identifier> comparison in
         Subclause 5.2, "<token> and <separator>".

         An SQL descriptor area name may be defined as global or local.
         Similarly, an extended statement name or extended cursor name may
         be global or local. The scope of a global name is the SQL-session.
         The scope of a local name is the <module> in which it appears. A
         reference to an entity in which one specifies a global scope is
         valid only if the entity was defined as global and if the reference

         62  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 4.24 SQL dynamic statements


         is from the same SQL-session in which it was defined. A reference
         to an entity in which one specifies a local scope is valid only if
         the entity was defined as local and if the reference is from the
         same <module> in which it was defined. (The scope of non-extended
         statement names and non-extended cursor names is always local.)

         Within an SQL-session, all global prepared statements (prepared
         statements with global statement names) belong to the SQL-session
         <module>. Within an SQL-session, each local prepared statement
         (prepared statements with local statement names) belongs to the
         <module> that contains the <prepare statement> or <execute immedi-
         ate statement> with which it is prepared.

         Note: The SQL-session <module> is defined in Subclause 4.30, "SQL-
         sessions".

         Dynamic execution of SQL-statements can generally be accomplished
         in two different ways. Statements can be prepared for execution
         and then later executed one or more times; when the statement is
         no longer needed for execution, it can be released by the use of
         a <deallocate prepared statement>. Alternatively, a statement that
         is needed only once can be executed without the preparation step-it
         can be executed immediately (not all SQL-statements can be executed
         immediately).

         Many SQL-statements can be written to use "parameters" (which are
         manifested in static execution of SQL-statements as <parameters>
         in <SQL statement>s contained in <procedure>s in <module>s or as
         <embedded variable name>s in <SQL statement>s contained in <em-
         bedded SQL host program>s). In SQL-statements that are executed
         dynamically, the parameters are called dynamic parameters (<dynamic
         parameter specification>s) and are represented in SQL language by a
         <question mark> (?).

         In many situations, an application that generates an SQL-statement
         for dynamic execution knows in detail the required characteristics
         (e.g., <data type>, <length>, <precision>, <scale>, etc.) of each
         of the dynamic parameters used in the statement; similarly, the ap-
         plication may also know in detail the characteristics of the values
         that will be returned by execution of the statement. However, in
         other cases, the application may not know this information to the
         required level of detail; it is possible in some cases for the ap-
         plication to ascertain the information from the Information Schema,
         but in other cases (e.g., when a returned value is derived from
         a computation instead of simply from a column in a table, or when
         dynamic parameters are supplied) this information is not gener-
         ally available except in the context of preparing the statement for
         execution.

         To provide the necessary information to applications, SQL per-
         mits an application to request the database system to describe a
         prepared statement. The description of a statement identifies the
         number of dynamic parameters (describe input) and their data type
         information or it identifies the number of values to be returned

                                                               Concepts   63

 





          X3H2-92-154/DBL CBR-002
         4.24 SQL dynamic statements


         (describe output) and their data type information. The descrip-
         tion of a statement is placed into the SQL descriptor areas already
         mentioned.

         Many, but not all, SQL-statements can be prepared and executed
         dynamically.

         Note: The complete list of statements that may be dynamically pre-
         pared and executed is defined in Subclause 4.22.4, "Preparable and
         immediately executable SQL-statements".

         Certain "set statements" (<set catalog statement>, <set schema
         statement>, and <set names statement>) have no effect other than
         to set up default information (<catalog name>, <schema name>,
         and <character set>, respectively) to be applied to other SQL-
         statements that are prepared or executed immediately or that are
         invoked directly.

         Syntax errors and Access Rule violations caused by the preparation
         or immediate execution of <preparable statement>s are identi-
         fied when the statement is prepared (by <prepare statement>) or
         when it is executed (by <execute statement> or <execute immediate
         statement>); such violations are indicated by the raising of an
         exception condition.

         4.25  Direct invocation of SQL

         Direct invocation of SQL is a mechanism for executing direct SQL-
         statements, known as <direct SQL statement>s. In direct invocation
         of SQL, the method of invoking <direct SQL statement>s, the method
         of raising conditions that result from the execution of <direct SQL
         statement>s, the method of accessing the diagnostics information
         that results from the execution of <direct SQL statement>s, and the
         method of returning the results are implementation-defined.

         4.26  Privileges

         A privilege authorizes a given category of <action> to be per-
         formed on a specified base table, view, column, domain, character
         set, collation, or translation by a specified <authorization iden-
         tifier>. The mapping of <authorization identifier>s to operating
         system users is implementation-dependent. The <action>s that can be
         specified are:

         -  INSERT

         -  INSERT (<column name list>)

         -  UPDATE

         -  UPDATE (<column name list>)

         -  DELETE

         64  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             4.26 Privileges


         -  SELECT

         -  REFERENCES

         -  REFERENCES (<column name list>)

         -  USAGE

         An <authorization identifier> is specified for each <schema defini-
         tion> and <module> as well as for each SQL-session.

         A schema that is owned by a given <authorization identifier> may
         contain privilege descriptors that describe privileges granted to
         other <authorization identifier>s (grantees). The granted priv-
         ileges apply to objects defined in the current schema. The WITH
         GRANT OPTION clause of a <grant statement> specifies whether the
         recipient of a privilege (acting as a grantor) may grant it to
         others.

         When an SQL-session is initiated, the <authorization identifier>
         for the SQL-session, called the SQL-session <authorization identi-
         fier>, is determined in an implementation-dependent manner, unless
         the session is initiated using a <connect statement>. Subsequently,
         the SQL-session <authorization identifier> can be redefined by the
         successful execution of a <set session authorization identifier
         statement>.

         A <module> may specify an <authorization identifier>, called a
         <module authorization identifier>. If the <module authorization
         identifier> is specified, then that <module authorization iden-
         tifier> is used as the current <authorization identifier> for the
         execution of all <procedure>s in the <module>. If the <module au-
         thorization identifier> is not specified, then the SQL-session
         <authorization identifier> is used as the current <authorization
         identifier> for the execution of each <procedure> in the <module>.

         A <schema definition> may specify an <authorization identifier>,
         called a <schema authorization identifier>. If the <schema autho-
         rization identifier> is specified, then that is used as the current
         <authorization identifier> for the creation of the schema. If the
         <module authorization identifier> is not specified, then the <mod-
         ule authorization identifier> or the SQL-session <authorization
         identifier> is used as the current <authorization identifier> for
         the creation of the schema.

         The current <authorization identifier> determines the privileges
         for the execution of each SQL-statement. For direct SQL, the SQL-
         session <authorization identifier> is always the current <autho-
         rization identifier>.

         Each privilege is represented by a privilege descriptor. A privi-
         lege descriptor contains:

         -  the identification of the table, column, domain, character set,
            collation, or translation that the descriptor describes;

         -  the <authorization identifier> of the grantor of the privilege;

                                                               Concepts   65

 





          X3H2-92-154/DBL CBR-002
         4.26 Privileges


         -  the <authorization identifier> of the grantee of the privilege;

         -  identification of the action that the privilege allows; and

         -  an indication of whether or not the privilege is grantable.

         A privilege descriptor with an action of INSERT, UPDATE, DELETE,
         SELECT, or REFERENCES is called a table privilege descriptor and
         identifies the existence of a privilege on the table identified by
         the privilege descriptor.

         A privilege descriptor with an action of SELECT (<column name
         list>), INSERT (<column name list>), UPDATE (<column name list>),
         or REFERENCES (<column name list>) is called a column privilege de-
         scriptor and identifies the existence of a privilege on the column
         in the table identified by the privilege descriptor.

         Note: In this International Standard, a SELECT column privilege
         cannot be explicitly granted or revoked. However, for the sake
         of compatibility with planned future language extensions, SELECT
         column privilege descriptors will appear in the Information Schema.

         A table privilege descriptor specifies that the privilege iden-
         tified by the action (unless the action is DELETE) is to be au-
         tomatically granted by the grantor to the grantee on all columns
         subsequently added to the table.

         A privilege descriptor with an action of USAGE is called a usage
         privilege descriptor and identifies the existence of a privilege on
         the domain, character set, collation, or translation identified by
         the privilege descriptor.

         A grantable privilege is a privilege associated with a schema that
         may be granted by a <grant statement>.

         The phrase applicable privileges refers to the privileges defined
         by the privilege descriptors that define privileges granted to the
         current <authorization identifier>.

         The set of applicable privileges for the current <authorization
         identifier> consists of the privileges defined by the privilege
         descriptors associated with that <authorization identifier> and
         the privileges defined by the privilege descriptors associated with
         PUBLIC.

         Privilege descriptors that represent privileges for the owner of
         an object have a special grantor value, "_SYSTEM". This value is
         reflected in the Information Schema for all privileges that apply
         to the owner of the object.

         4.27  SQL-agents

         An SQL-agent is an implementation-dependent entity that causes the
         execution of SQL-statements.

         66  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       4.28 SQL-transactions


         4.28  SQL-transactions

         An SQL-transaction (sometimes simply called a "transaction") is
         a sequence of executions of SQL-statements that is atomic with
         respect to recovery. These operations are performed by one or more
         compilation units and <module>s or by the direct invocation of SQL.

         It is implementation-defined whether or not the non-dynamic or
         dynamic execution of an SQL-data statement or the execution of
         an <SQL dynamic data statement> is permitted to occur within the
         same SQL-transaction as the non-dynamic or dynamic execution of
         an SQL-schema statement. If it does occur, then the effect on any
         open cursor, prepared dynamic statement, or deferred constraint
         is implementation-defined. There may be additional implementation-
         defined restrictions, requirements, and conditions. If any such
         restrictions, requirements, or conditions are violated, then an
         implementation-defined exception condition or a completion con-
         dition warning with an implementation-defined subclass code is
         raised.

         Each <module> or direct invocation of SQL that executes an
         SQL-statement of an SQL-transaction is associated with that
         SQL-transaction. An SQL-transaction is initiated when no SQL-
         transaction is currently active and a <procedure> is called that
         results in the execution of a transaction-initiating SQL-statement
         or by direct invocation of SQL that results in the execution of a
         transaction-initiating <direct SQL statement>. An SQL-transaction
         is terminated by a <commit statement> or a <rollback statement>.
         If an SQL-transaction is terminated by successful execution of a
         <commit statement>, then all changes made to SQL-data or schemas by
         that SQL-transaction are made persistent and accessible to all con-
         current and subsequent SQL-transactions. If an SQL-transaction is
         terminated by a <rollback statement> or unsuccessful execution of
         a <commit statement>, then all changes made to SQL-data or schemas
         by that SQL-transaction are canceled. Committed changes cannot be
         canceled. If execution of a <commit statement> is attempted, but
         certain exception conditions are raised, it is unknown whether or
         not the changes made to SQL-data or schemas by that SQL-transaction
         are canceled or made persistent.

         An SQL-transaction has a constraint mode for each integrity con-
         straint. The constraint mode for an integrity constraint in an
         SQL-transaction is described in Subclause 4.10, "Integrity con-
         straints".

         An SQL-transaction has an access mode that is either read-only or
         read-write. The access mode may be explicitly set by a <set trans-
         action statement>; otherwise, it is implicitly set to read-write.
         The term read-only applies only to viewed tables and persistent
         base tables.

         An SQL-transaction has a diagnostics area limit, which is a pos-
         itive integer that specifies the maximum number of conditions
         that can be placed in the diagnostics area during execution of
         an SQL-statement in this SQL-transaction.

                                                               Concepts   67

 





          X3H2-92-154/DBL CBR-002
         4.28 SQL-transactions


         SQL-transactions initiated by different SQL-agents that access
         the same SQL-data or schemas and overlap in time are concurrent
         SQL-transactions.

         An SQL-transaction has an isolation level that is READ UNCOMMITTED,
         READ COMMITTED, REPEATABLE READ, or SERIALIZABLE. The isolation
         level of an SQL-transaction defines the degree to which the opera-
         tions on SQL-data or schemas in that SQL-transaction are affected
         by the effects of and can affect operations on SQL-data or schemas
         in concurrent SQL-transactions. The isolation level of a SQL-
         transaction is SERIALIZABLE by default. The level can be explicitly
         set by the <set transaction statement>.

         The execution of concurrent SQL-transactions at isolation level
         SERIALIZABLE is guaranteed to be serializable. A serializable exe-
         cution is defined to be an execution of the operations of concur-
         rently executing SQL-transactions that produces the same effect as
         some serial execution of those same SQL-transactions. A serial exe-
         cution is one in which each SQL-transaction executes to completion
         before the next SQL-transaction begins.

         The isolation level specifies the kind of phenomena that can occur
         during the execution of concurrent SQL-transactions. The following
         phenomena are possible:

         1) P1 ("Dirty read"): SQL-transaction T1 modifies a row. SQL-
            transaction T2 then reads that row before T1 performs a COMMIT.
            If T1 then performs a ROLLBACK, T2 will have read a row that was
            never committed and that may thus be considered to have never
            existed.

         2) P2 ("Non-repeatable read"): SQL-transaction T1 reads a row. SQL-
            transaction T2 then modifies or deletes that row and performs
            a COMMIT. If T1 then attempts to reread the row, it may receive
            the modified value or discover that the row has been deleted.

         3) P3 ("Phantom"): SQL-transaction T1 reads the set of rows N
            that satisfy some <search condition>. SQL-transaction T2 then
            executes SQL-statements that generate one or more rows that
            satisfy the <search condition> used by SQL-transaction T1. If
            SQL-transaction T1 then repeats the initial read with the same
            <search condition>, it obtains a different collection of rows.

         The four isolation levels guarantee that each SQL-transaction will
         be executed completely or not at all, and that no updates will be
         lost. The isolation levels are different with respect to phenomena
         P1, P2, and P3. Table 9, "SQL-transaction isolation levels and the
         three phenomena" specifies the phenomena that are possible and not
         possible for a given isolation level.





         68  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       4.28 SQL-transactions


         __Table_9-SQL-transaction_isolation_levels_and_the_three_phenomena_

         _Level__________________P1______P2_______P3________________________

        | READ UNCOMMITTED     | Possib|e Possib|e Possible                |
        |                      |       |        |                          |
        | READ COMMITTED       | Not   | Possibl| Possible                 |
                                 Possible

        | REPEATABLE READ      | Not   | Not    | Possible                 |
        |                      | Possib|e Possib|e                         |
        |                      |       |        |                          |
        | SERIALIZABLE         | Not   | Not    | Not Possible             |
        |______________________|_Possib|e_Possib|e_________________________|
        |                      |       |        |                          |
        |Note: The exclusion of|these p|enomena |or SQL-transactions ex-   |
         ecuting at isolation level SERIALIZABLE is a consequence of the
         requirement that such transactions be serializable.

         Changes made to SQL-data or schemas by an SQL-transaction in an
         SQL-session may be perceived by that SQL-transaction in that
         same SQL-session, and by other SQL-transactions, or by that same
         SQL-transaction in other SQL-sessions, at isolation level READ
         UNCOMMITTED, but cannot be perceived by other SQL-transactions at
         isolation level READ COMMITTED, REPEATABLE READ, or SERIALIZABLE
         until the former SQL-transaction terminates with a <commit state-
         ment>.

         Regardless of the isolation level of the SQL-transaction, phenomena
         P1, P2, and P3 shall not occur during the implied reading of schema
         definitions performed on behalf of executing an SQL-statement, the
         checking of integrity constraints, and the execution of referen-
         tial actions associated with referential constraints. The schema
         definitions that are implicitly read are implementation-dependent.
         This does not affect the explicit reading of rows from tables in
         the Information Schema, which is done at the isolation level of the
         SQL-transaction.

         The execution of a <rollback statement> may be initiated implicitly
         by an implementation when it detects the inability to guarantee the
         serializability of two or more concurrent SQL-transactions. When
         this error occurs, an exception condition is raised: transaction
         rollback-serialization failure.

         The execution of a <rollback statement> may be initiated implicitly
         by an implementation when it detects unrecoverable errors. When
         such an error occurs, an exception condition is raised: transaction
         rollback with an implementation-defined subclass code.

         The execution of an SQL-statement within an SQL-transaction has
         no effect on SQL-data or schemas other than the effect stated in
         the General Rules for that SQL-statement, in the General Rules
         for Subclause 11.8, "<referential constraint definition>", and
         in the General Rules for Subclause 12.3, "<procedure>". Together
         with serializable execution, this implies that all read opera-
         tions are repeatable within an SQL-transaction at isolation level
         SERIALIZABLE, except for:

                                                               Concepts   69

 





          X3H2-92-154/DBL CBR-002
         4.28 SQL-transactions


         1) the effects of changes to SQL-data or schemas and its contents
            made explicitly by the SQL-transaction itself,

         2) the effects of differences in parameter values supplied to pro-
            cedures, and

         3) the effects of references to time-varying system variables such
            as CURRENT_DATE and CURRENT_USER.

         In some environments (e.g., remote database access), an SQL-
         transaction can be part of an encompassing transaction that is
         controlled by an agent other than the SQL-agent. The encompass-
         ing transaction may involve different resource managers, the
         SQL-environment being just one instance of such a manager. In
         such environments, an encompassing transaction shall be ter-
         minated via that other agent, which in turn interacts with the
         SQL-environment via an interface that may be different from SQL
         (COMMIT or ROLLBACK), in order to coordinate the orderly termi-
         nation of the encompassing transaction. When an SQL-transaction
         is part of an encompassing transaction that is controlled by an
         agent other than an SQL-agent and a <rollback statement> is ini-
         tiated implicitly by an implementation, then the implementation
         will interact with that other agent to terminate that encompassing
         transaction. The specification of the interface between such agents
         and the SQL-environment is beyond the scope of this International
         Standard. However, it is important to note that the semantics of an
         SQL-transaction remain as defined in the following sense:

         -  When an agent that is different from the SQL-agent requests
            the SQL-environment to rollback an SQL-transaction, the General
            Rules of Subclause 14.4, "<rollback statement>", are performed.

         -  When such an agent requests the SQL-environment to commit an
            SQL-transaction, the General Rules of Subclause 14.3, "<commit
            statement>", are performed. To guarantee orderly termination
            of the encompassing transaction, this commit operation may be
            processed in several phases not visible to the application; not
            all the General Rules of Subclause 14.3, "<commit statement>",
            need to be executed in a single phase.

         However, even in such environments, the SQL-agent interacts di-
         rectly with the SQL-server to set attributes (such as read-only
         or read-write, isolation level, and constraints mode) that are
         specific to the SQL-transaction model.

         4.29  SQL-connections

         An SQL-connection is an association between an SQL-client and an
         SQL-server. An SQL-connection may be established and named by a
         <connect statement>, which identifies the desired SQL-server by
         means of an <SQL-server name>. A <connection name> is specified



         70  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                        4.29 SQL-connections


         as a <simple value specification> whose value is an <identi-
         fier>. Two <connection name>s identify the same SQL-connection
         if their values, with leading and trailing <space>s removed, are
         equivalent according to the rules for <identifier> comparison in
         Subclause 5.2, "<token> and <separator>". It is implementation-
         defined how an implementation uses <SQL-server name> to determine
         the location, identity, and communication protocol required to
         access the SQL-server and create an SQL-session.

         An SQL-connection is an active SQL-connection if any SQL-statement
         that initiates or requires an SQL-transaction has been executed at
         its SQL-server during the current SQL-transaction.

         An SQL-connection is either current or dormant. If the SQL-
         connection established by the most recently executed implicit
         or explicit <connect statement> or <set connection statement>
         has not been terminated, then that SQL-connection is the current
         SQL-connection; otherwise, there is no current SQL-connection. An
         existing SQL-connection that is not the current SQL-connection is a
         dormant SQL-connection.

         An SQL-implementation may detect the loss of the current SQL-
         connection during execution of any SQL-statement. When such a
         connection failure is detected, an exception condition is raised:
         transaction rollback-statement completion unknown. This excep-
         tion condition indicates that the results of the actions performed
         in the SQL-server on behalf of the statement are unknown to the
         SQL-agent.

         Similarly, an SQL-implementation may detect the loss of the current
         SQL-connection during the execution of a <commit statement>. When
         such a connection failure is detected, an exception condition is
         raised: connection exception-transaction resolution unknown. This
         exception condition indicates that the SQL-implementation cannot
         verify whether the SQL-transaction was committed successfully,
         rolled back, or left active.

         A user may initiate an SQL-connection between the SQL-client as-
         sociated with the SQL-agent and a specific SQL-server by executing
         a <connect statement>. Otherwise, an SQL-connection between the
         SQL-client and an implementation-defined default SQL-server is
         initiated when a <procedure> is called and no SQL-connection is
         current. The SQL-connection associated with an implementation-
         defined default SQL-server is called the default SQL-connection.
         An SQL-connection is terminated either by executing a <disconnect
         statement>, or following the last call to a <procedure> within the
         last active <module>, or by the last execution of a <direct SQL
         statement> through the direct invocation of SQL. The mechanism and
         rules by which an SQL-environment determines whether a call to a
         <procedure> is the last call within the last active <module> or
         the last execution of a <direct SQL statement> through the direct
         invocation of SQL are implementation-defined.


                                                               Concepts   71

 





          X3H2-92-154/DBL CBR-002
         4.29 SQL-connections


         An implementation shall support at least one SQL-connection and
         may require that the SQL-server be identified at the binding time
         chosen by the implementation. If an implementation permits more
         than one concurrent SQL-connection, then the SQL-agent may connect
         to more than one SQL-server and select the SQL-server by executing
         a <set connection statement>.

         4.30  SQL-sessions


         An SQL-session spans the execution of a sequence of consecutive
         SQL-statements invoked by a single user from a single SQL-agent or
         by the direct invocation of SQL.

         An SQL-session is associated with an SQL-connection. The SQL-
         session associated with the default SQL-connection is called the
         default SQL-session. An SQL-session is either current or dormant.
         The current SQL-session is the SQL-session associated with the cur-
         rent SQL-connection. A dormant SQL-session is an SQL-session that
         is associated with a dormant SQL-connection.

         An SQL-session has an SQL-session <module> that is different
         from any other <module> that exists simultaneously in the SQL-
         environment. The SQL-session <module> contains the global prepared
         SQL-statements that belong to the SQL-session. The SQL-session
         <module> contains a <module authorization clause> that speci-
         fies SCHEMA <schema name>, where the value of <schema name> is
         implementation-dependent.

         Within an SQL-session, declared local temporary tables are effec-
         tively created by <temporary table declaration>s. Declared local
         temporary tables are accessible only to invocations of <proce-
         dure>s in the <module> in which they are created. The definitions
         of declared local temporary tables persist until the end of the
         SQL-session.

         An SQL-session has a unique implementation-dependent SQL-session
         identifier. This SQL-session identifier is different from the SQL-
         session identifier of any other concurrent SQL-session. The SQL-
         session identifier is used to effectively define implementation-
         defined schemas that contain the instances of any global temporary
         tables, created local temporary tables, or declared local temporary
         tables within the SQL-session.

         An SQL-session has an <authorization identifier> that is initially
         set to an implementation-defined value when the SQL-session is
         started, unless the SQL-session is started as a result of suc-
         cessful execution of a <connect statement>, in which case the
         <authorization identifier> of the SQL-session is set to the value
         of the implicit or explicit <user name> contained in the <connect
         statement>.



         72  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                           4.30 SQL-sessions


         An SQL-session has a default catalog name that is used to effec-
         tively qualify unqualified <schema name>s that are contained in
         <preparable statement>s when those statements are prepared in the
         current SQL-session by either an <execute immediate statement>
         or a <prepare statement> or are contained in <direct SQL state-
         ment>s when those statements are invoked directly. The default
         catalog name is initially set to an implementation-defined value
         but can subsequently be changed by the successful execution of a
         <set catalog statement> or <set schema statement>.

         An SQL-session has a default unqualified schema name that is used
         to effectively qualify unqualified <qualified name>s that are con-
         tained in <preparable statement>s when those statements are pre-
         pared in the current SQL-session by either an <execute immediate
         statement> or a <prepare statement> or are contained in <direct SQL
         statement>s when those statements are invoked directly. The default
         unqualified schema name is initially set to an implementation-
         defined value but can subsequently be changed by the successful
         execution of a <set schema statement>.

         An SQL-session has a default character set name that is used to
         identify the character set implicit for <identifier>s and <charac-
         ter string literal>s that are contained in <preparable statement>s
         when those statements are prepared in the current SQL-session by
         either an <execute immediate statement> or a <prepare statement> or
         are contained in <direct SQL statement>s when those statements are
         invoked directly. The default character set name is initially set
         to an implementation-defined value but can subsequently be changed
         by the successful execution of a <set names statement>.

         An SQL-session has a default local time zone displacement, which is
         a value of data type INTERVAL HOUR TO MINUTE. The default local
         time zone displacement is initially set to an implementation-
         defined value but can subsequently be changed by successful exe-
         cution of a <set local time zone statement>.

         An SQL-session has context that is preserved when an SQL-session
         is made dormant and restored when the SQL-session is made current.
         This context comprises:

         -  the current SQL-session identifier,

         -  the current <authorization identifier>,

         -  the identities of all instances of temporary tables,

         -  the SQL-session <module>,

         -  the current default catalog name,

         -  the current default unqualified schema name,

         -  the current character set name substitution value,

         -  the current default time zone,

                                                               Concepts   73

 





          X3H2-92-154/DBL CBR-002
         4.30 SQL-sessions


         -  the current constraint mode for each integrity constraint,

         -  the current transaction access mode,

         -  the cursor position of all open cursors,

         -  the contents of all SQL dynamic descriptor areas,

         -  the current transaction isolation level, and

         -  the current transaction diagnostics area limit.

         4.31  Client-server operation

         Within an SQL-environment, an SQL-implementation may be considered
         to effectively contain an SQL-client component and one or more
         SQL-server components.

         When an SQL-agent is active, it is bound in some implementation-
         defined manner to a single SQL-client. That SQL-client processes
         the explicit or implicit <SQL connection statement> for the first
         call to a <procedure> by an SQL-agent. The SQL-client communicates
         with, either directly or possibly through other agents such as RDA,
         one or more SQL-servers. An SQL-session involves an SQL-agent, an
         SQL-client, and a single SQL-server.

         <module>s associated with the SQL-agent exist in the SQL-
         environment containing the SQL-client associated with the SQL-
         agent.

         Called <procedure>s (and, analogously, <direct SQL statement>s)
         containing an <SQL connection statement> or an <SQL diagnostics
         statement> are processed by the SQL-client. Following the suc-
         cessful execution of a <connect statement> or a <set connection
         statement>, the <module>s associated with the SQL-agent are ef-
         fectively materialized with an implementation-dependent <module
         name> in the SQL-server. Other called <procedure>s and <direct SQL
         statement>s are processed by the SQL-server.

         A call by the SQL-agent to a <procedure> containing an <SQL di-
         agnostics statement> fetches information from the diagnostics
         area associated with the SQL-client. Following the execution of
         an <SQL procedure statement> by an SQL-server, diagnostic in-
         formation is passed in an implementation-dependent manner into
         the SQL-agent's diagnostics area in the SQL-client. The effect
         on diagnostic information of incompatibilities between the char-
         acter repertoires supported by the SQL-client and SQL-server is
         implementation-dependent.






         74  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                     4.32 Information Schema


         4.32  Information Schema

         In each catalog in an SQL-environment, there is a schema, the
         Information Schema, with the name INFORMATION_SCHEMA, contain-
         ing a number of view descriptors, one base table descriptor ,
         and several domain descriptors. The data accessible through these
         views is a representation of all of the descriptors in all of the
         schemas in that catalog. The <query expression> of each view en-
         sures that a given user can access only those rows of the view
         that represent descriptors on which he has privileges. The rows
         of each view are required to represent correctly the descriptors
         in the catalog as they existed at the start of the current SQL-
         transaction, as modified subsequently by any changes made by the
         current SQL-transaction. The SELECT privilege is granted on each
         of the Information Schema views to PUBLIC WITH GRANT OPTION so they
         can be queried by any user and so that the SELECT privilege can
         be further granted on views that reference the Information Schema
         views. No further privilege is granted on them, so they cannot be
         updated.

         The viewed tables in INFORMATION_SCHEMA are defined in terms of
         a collection of base tables in a schema named DEFINITION_SCHEMA,
         the Definition Schema. The only purpose of the definition of these
         base tables is to provide a data model to support the Information
         Schema. An implementation need do no more than simulate the exis-
         tence of the base tables as viewed through the Information Schema
         views.

         The Information Schema describes itself. It does not describe the
         base tables or views of the Definition Schema. If an implemen-
         tation has defined additional objects that are associated with
         INFORMATION_SCHEMA, then those objects shall also be described in
         the Information Schema views.

         4.33  Leveling


         Three levels of conformance are specified in this International
         Standard.

         Entry SQL includes the statements for defining schemas, data ma-
         nipulation language, referential integrity, check constraints, and
         default clause from ISO/IEC 9075:1989, and options for module lan-
         guage and embedded SQL interfaces to seven different programming
         languages, as well as direct execution of the data manipulation
         statements. It also includes features related to deprecated fea-
         tures from ISO/IEC 9075:1989 (commas and parentheses in parameter
         lists, the SQLSTATE parameter, and renaming columns in the <se-
         lect list>), features related to incompatibilities with ISO/IEC
         9075:1989 (colons preceding <parameter name>s, WITH CHECK OPTION
         constraint clarifications), and aids for transitioning from ISO/IEC
         9075:1989 to this International Standard (<delimited identifier>s).


                                                               Concepts   75

 





          X3H2-92-154/DBL CBR-002
         4.33 Leveling


         Finally, it contains changes to correct defects found in ISO/IEC
         9075:1989 (see Annex F, "Maintenance and interpretation of SQL").

         Intermediate SQL includes major new facilities such as statements
         for changing schemas, dynamic SQL, and isolation levels for SQL-
         transactions. It also includes multiple-module support and cascade
         delete on referential actions, as well as numerous functional en-
         hancements such as row and table expressions, union join, character
         string operations, table intersection and difference operations,
         simple domains, the CASE expression, casting between data types,
         a diagnostics management capability for data administration and
         more comprehensive error analysis, multiple character repertoires,
         interval and simplified datetime data types, and variable-length
         character strings. It also includes a requirement for a flagger
         facility to aid in writing portable applications.

         Full SQL increases orthogonality and includes deferred constraint
         checking and named constraints. Other technical enhancements in-
         clude additional user options to define datetime data types,
         self-referencing updates and deletes, cascade update on referen-
         tial actions, subqueries in check constraints, scrolled cursors,
         character translations, a bit string data type, temporary tables,
         additional referential constraint options, and simple assertions.

         4.34  SQL Flagger

         An SQL Flagger is an implementation-provided facility that is able
         to identify SQL language extensions, or other SQL processing al-
         ternatives, that may be provided by a conforming SQL-implementation
         (see Subclause 23.3, "Extensions and options"). An SQL Flagger
         is intended to assist SQL programmers in producing SQL language
         that is both portable and interoperable among different conform-
         ing SQL-implementations operating under different levels of this
         International Standard.

         An SQL Flagger is intended to effect a static check of SQL lan-
         guage. There is no requirement to detect extensions that cannot be
         determined until the General Rules are evaluated.

         An SQL-implementation need only flag SQL language that is not oth-
         erwise in error as far as that implementation is concerned.

         Note: If a system is processing SQL language that contains er-
         rors, then it may be very difficult within a single statement to
         determine what is an error and what is an extension. As one pos-
         sibility, an implementation may choose to check SQL language in
         two steps; first through its normal syntax analyzer and secondly
         through the SQL Flagger. The first step produces error messages
         for nonstandard SQL language that the implementation cannot process
         or recognize. The second step processes SQL language that contains
         no errors as far as that implementation is concerned; it detects
         and flags at one time all nonstandard SQL language that could be
         processed by that implementation. Any such two-step process should
         be transparent to the end user.

         76  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                            4.34 SQL Flagger


         In order to provide upward compatibility for its own customer base,
         or to provide performance advantages under special circumstances, a
         conforming SQL-implementation may provide user options to process
         conforming SQL language in a nonconforming manner. If this is the
         case, then it is required that the implementation also provide a
         flagger option, or some other implementation-defined means, to
         detect SQL conforming language that may be processed differently
         under the various user options. This flagger feature allows an
         application programmer to identify conforming SQL language that may
         perform differently in alternative processing environments provided
         by a conforming SQL-implementation. It also provides a valuable
         tool in identifying SQL elements that may have to be modified if
         SQL language is to be moved from a nonconforming to a conforming
         SQL processing environment.

         An SQL Flagger provides one or more of the following "level of
         flagging" options:

         -  Entry SQL Flagging

         -  Intermediate SQL Flagging

         -  Full SQL Flagging

         An SQL Flagger that provides one of these options shall be able to
         identify SQL language constructs that violate the indicated level
         of SQL language as defined in Subclause 4.33, "Leveling".

         An SQL Flagger provides one or more of the following "extent of
         checking" options:

         -  Syntax Only

         -  Catalog Lookup

         Under the Syntax Only option, the SQL Flagger analyzes only the SQL
         language that is presented; it checks for violations of any Syntax
         Rules that can be determined without access to the Information
         Schema.

         Under the Catalog Lookup option, the SQL Flagger assumes the avail-
         ability of Definition Schema information and checks for violations
         of all Syntax Rules and Access Rules, except Access Rules that deal
         with privileges. For example, some Syntax Rules place restrictions
         on data types; this flagger option would identify extensions that
         relax such restrictions. In order to avoid security breaches, this
         option may view the Definition Schema only through the eyes of a
         specific Information Schema.






                                                               Concepts   77

 





          X3H2-92-154/DBL CBR-002

























































         78  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002






         5  Lexical elements



         5.1  <SQL terminal character>

         Function

         Define the terminal symbols of the SQL language and the elements of
         strings.

         Format

         <SQL terminal character> ::=
                <SQL language character>
              | <SQL embedded language character>

         <SQL embedded language character> ::=
                <left bracket>
              | <right bracket>

         <SQL language character> ::=
                <simple Latin letter>
              | <digit>
              | <SQL special character>

         <simple Latin letter> ::=
                <simple Latin upper case letter>
              | <simple Latin lower case letter>

         <simple Latin upper case letter> ::=
                    A | B | C | D | E | F | G | H | I | J | K | L | M | N | O
              | P | Q | R | S | T | U | V | W | X | Y | Z

         <simple Latin lower case letter> ::=
                    a | b | c | d | e | f | g | h | i | j | k | l | m | n | o
              | p | q | r | s | t | u | v | w | x | y | z

         <digit> ::=
              0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9

         <SQL special character> ::=
                <space>
              | <double quote>
              | <percent>
              | <ampersand>
              | <quote>
              | <left paren>
              | <right paren>
              | <asterisk>

                                                       Lexical elements   79

 





          X3H2-92-154/DBL CBR-002
         5.1 <SQL terminal character>


              | <plus sign>
              | <comma>
              | <minus sign>
              | <period>
              | <solidus>
              | <colon>
              | <semicolon>
              | <less than operator>
              | <equals operator>
              | <greater than operator>
              | <question mark>
              | <underscore>
              | <vertical bar>

         <space> ::= !! space character in character set in use

         <double quote> ::= "

         <percent> ::= %

         <ampersand> ::= &

         <quote> ::= '

         <left paren> ::= (

         <right paren> ::= )

         <asterisk> ::= *

         <plus sign> ::= +

         <comma> ::= ,

         <minus sign> ::= -

         <period> ::= .

         <solidus> ::= /

         <colon> ::= :

         <semicolon> ::= ;

         <less than operator> ::= <

         <equals operator> ::= =

         <greater than operator> ::= >

         <question mark> ::= ?

         <left bracket> ::= [

         80  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                5.1 <SQL terminal character>


         <right bracket> ::= ]

         <underscore> ::= _

         <vertical bar> ::= |


         Syntax Rules

            None.

         Access Rules

            None.

         General Rules

         1) There is a one-to-one correspondence between the symbols con-
            tained in <simple Latin upper case letter> and the symbols
            contained in <simple Latin lower case letter> such that, for
            all i, the symbol defined as the i-th alternative for <simple
            Latin upper case letter> corresponds to the symbol defined as
            the i-th alternative for <simple Latin lower case letter>.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

              None.




















                                                       Lexical elements   81

 





          X3H2-92-154/DBL CBR-002
         5.2 <token> and <separator>


         5.2  <token> and <separator>

         Function

         Specify lexical units (tokens and separators) that participate in
         SQL language.

         Format

         <token> ::=
                <nondelimiter token>
              | <delimiter token>

         <nondelimiter token> ::=
                <regular identifier>
              | <key word>
              | <unsigned numeric literal>
              | <national character string literal>
              | <bit string literal>
              | <hex string literal>

         <regular identifier> ::= <identifier body>

         <identifier body> ::=
              <identifier start> [ { <underscore> | <identifier part> }... ]


         <identifier start> ::= !! See the Syntax Rules

         <identifier part> ::=
                <identifier start>
              | <digit>

         <delimited identifier> ::=
              <double quote> <delimited identifier body> <double quote>

         <delimited identifier body> ::= <delimited identifier part>...

         <delimited identifier part> ::=
                <nondoublequote character>
              | <doublequote symbol>

         <nondoublequote character> ::= !! See the Syntax Rules

         <doublequote symbol> ::= <double quote><double quote>

         <delimiter token> ::=
                <character string literal>
              | <date string>
              | <time string>
              | <timestamp string>
              | <interval string>
              | <delimited identifier>

         82  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 5.2 <token> and <separator>


              | <SQL special character>
              | <not equals operator>
              | <greater than or equals operator>
              | <less than or equals operator>
              | <concatenation operator>
              | <double period>
              | <left bracket>
              | <right bracket>

         <not equals operator> ::= <>

         <greater than or equals operator> ::= >=

         <less than or equals operator> ::= <=

         <concatenation operator> ::= ||

         <double period> ::= ..

         <separator> ::= { <comment> | <space> | <newline> }...

         <comment> ::=
              <comment introducer> [ <comment character>... ] <newline>

         <comment character> ::=
                <nonquote character>
              | <quote>

         <comment introducer> ::= <minus sign><minus sign>[<minus sign>...]

         <newline> ::= !! implementation-defined end-of-line indicator

         <key word> ::=
                <reserved word>
              | <non-reserved word>

         <non-reserved word> ::=
                ADA
              | C | CATALOG_NAME | CHARACTER_SET_CATALOG | CHARACTER_SET_
              NAME
              | CHARACTER_SET_SCHEMA | CLASS_ORIGIN | COBOL | COLLATION_
              CATALOG
              | COLLATION_NAME | COLLATION_SCHEMA | COLUMN_NAME | COMMAND_
              FUNCTION | COMMITTED
              | CONDITION_NUMBER | CONNECTION_NAME | CONSTRAINT_CATALOG | CONSTRAINT_
              NAME
              | CONSTRAINT_SCHEMA | CURSOR_NAME
              | DATA | DATETIME_INTERVAL_CODE | DATETIME_INTERVAL_
              PRECISION | DYNAMIC_FUNCTION
              | FORTRAN
              | LENGTH
              | MESSAGE_LENGTH | MESSAGE_OCTET_LENGTH | MESSAGE_TEXT | MORE | MUMPS


                                                       Lexical elements   83

 





          X3H2-92-154/DBL CBR-002
         5.2 <token> and <separator>


              | NAME | NULLABLE | NUMBER
              | PASCAL | PLI
              | REPEATABLE | RETURNED_LENGTH | RETURNED_OCTET_LENGTH | RETURNED_
              SQLSTATE
              | ROW_COUNT
              | SCALE | SCHEMA_NAME | SERIALIZABLE | SERVER_NAME | SUBCLASS_
              ORIGIN
              | TABLE_NAME | TYPE
              | UNCOMMITTED | UNNAMED

         <reserved word> ::=
                ABSOLUTE | ACTION | ADD | ALL | ALLOCATE | ALTER | AND
              | ANY | ARE | AS | ASC
              | ASSERTION | AT | AUTHORIZATION | AVG
              | BEGIN | BETWEEN | BIT | BIT_LENGTH | BOTH | BY
              | CASCADE | CASCADED | CASE | CAST | CATALOG | CHAR | CHARACTER | CHAR_
              LENGTH
              | CHARACTER_LENGTH | CHECK | CLOSE | COALESCE | COLLATE | COLLATION

              | COLUMN | COMMIT | CONNECT | CONNECTION | CONSTRAINT
              | CONSTRAINTS | CONTINUE
              | CONVERT | CORRESPONDING | COUNT | CREATE | CROSS | CURRENT
              | CURRENT_DATE | CURRENT_TIME | CURRENT_TIMESTAMP | CURRENT_
              USER | CURSOR
              | DATE | DAY | DEALLOCATE | DEC | DECIMAL | DECLARE | DEFAULT | DEFERRABLE

              | DEFERRED | DELETE | DESC | DESCRIBE | DESCRIPTOR | DIAGNOSTICS

              | DISCONNECT | DISTINCT | DOMAIN | DOUBLE | DROP
              | ELSE | END | END-EXEC | ESCAPE | EXCEPT | EXCEPTION
              | EXEC | EXECUTE | EXISTS
              | EXTERNAL | EXTRACT
              | FALSE | FETCH | FIRST | FLOAT | FOR | FOREIGN | FOUND | FROM | FULL

              | GET | GLOBAL | GO | GOTO | GRANT | GROUP
              | HAVING | HOUR
              | IDENTITY | IMMEDIATE | IN | INDICATOR | INITIALLY | INNER | INPUT

              | INSENSITIVE | INSERT | INT | INTEGER | INTERSECT | INTERVAL | INTO | IS

              | ISOLATION
              | JOIN
              | KEY
              | LANGUAGE | LAST | LEADING | LEFT | LEVEL | LIKE | LOCAL | LOWER

              | MATCH | MAX | MIN | MINUTE | MODULE | MONTH
              | NAMES | NATIONAL | NATURAL | NCHAR | NEXT | NO | NOT | NULL

              | NULLIF | NUMERIC
              | OCTET_LENGTH | OF | ON | ONLY | OPEN | OPTION | OR
              | ORDER | OUTER
              | OUTPUT | OVERLAPS


         84  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 5.2 <token> and <separator>


              | PAD | PARTIAL | POSITION | PRECISION | PREPARE | PRESERVE | PRIMARY

              | PRIOR | PRIVILEGES | PROCEDURE | PUBLIC
              | READ | REAL | REFERENCES | RELATIVE | RESTRICT | REVOKE | RIGHT

              | ROLLBACK | ROWS
              | SCHEMA | SCROLL | SECOND | SECTION | SELECT | SESSION | SESSION_
              USER | SET
              | SIZE | SMALLINT | SOME | SPACE | SQL | SQLCODE | SQLERROR | SQLSTATE

              | SUBSTRING | SUM | SYSTEM_USER
              | TABLE | TEMPORARY | THEN | TIME | TIMESTAMP | TIMEZONE_
              HOUR | TIMEZONE_MINUTE
              | TO | TRAILING | TRANSACTION | TRANSLATE | TRANSLATION | TRIM | TRUE

              | UNION | UNIQUE | UNKNOWN | UPDATE | UPPER | USAGE | USER | USING

              | VALUE | VALUES | VARCHAR | VARYING | VIEW
              | WHEN | WHENEVER | WHERE | WITH | WORK | WRITE
              | YEAR
              | ZONE


         Note: The list of <reserved word>s is considerably longer than
         the analogous list of <key word>s in ISO/IEC 9075:1989. To assist
         users of this International Standard avoid such words in a possible
         future revision, the following list of potential <reserved word>s
         is provided. Readers must understand that there is no guarantee
         that all of these words will, in fact, become <reserved word>s
         in any future revision; furthermore, it is almost certain that
         additional words will be added to this list as any possible future
         revision emerges.

          The words are: AFTER, ALIAS, ASYNC, BEFORE, BOOLEAN, BREADTH,
         COMPLETION, CALL, CYCLE, DATA, DEPTH, DICTIONARY, EACH, ELSEIF,
         EQUALS, GENERAL, IF, IGNORE, LEAVE, LESS, LIMIT, LOOP, MODIFY,
         NEW, NONE, OBJECT, OFF, OID, OLD, OPERATION, OPERATORS, OTHERS,
         PARAMETERS, PENDANT, PREORDER, PRIVATE, PROTECTED, RECURSIVE, REF,
         REFERENCING, REPLACE, RESIGNAL, RETURN, RETURNS, ROLE, ROUTINE,
         ROW, SAVEPOINT, SEARCH, SENSITIVE, SEQUENCE, SIGNAL, SIMILAR,
         SQLEXCEPTION, SQLWARNING, STRUCTURE, TEST, THERE, TRIGGER, TYPE,
         UNDER, VARIABLE, VIRTUAL, VISIBLE, WAIT, WHILE, and WITHOUT.

         Syntax Rules

         1) An <identifier start> is one of:

            a) A <simple Latin letter>; or

            b) A character that is identified as a letter in the character
              repertoire identified by the <module character set specifica-
              tion> or by the <character set specification>; or


                                                       Lexical elements   85

 





          X3H2-92-154/DBL CBR-002
         5.2 <token> and <separator>


            c) A character that is identified as a syllable in the char-
              acter repertoire identified by the <module character set
              specification> or by the <character set specification>; or

            d) A character that is identified as an ideograph in the char-
              acter repertoire identified by the <module character set
              specification> or by the <character set specification>.

         2) With the exception of the <space> character explicitly contained
            in <timestamp string> and <interval string> and the permitted
            <separator>s in <bit string literal>s and <hex string literal>s,
            a <token>, other than a <character string literal>, a <national
            character string literal>, or a <delimited identifier>, shall
            not include a <space> character or other <separator>.

         3) A <nondoublequote character> is one of:

            a) Any <SQL language character> other than a <double quote>;

            b) Any character other than a <double quote> in the character
              repertoire identified by the <module character set specifica-
              tion>; or

            c) Any character other than a <double quote> in the character
              repertoire identified by the <character set specification>.

         4) The two <doublequote>s contained in a <doublequote symbol> shall
            not be separated by any <separator>.

         5) Any <token> may be followed by a <separator>. A <nondelimiter
            token> shall be followed by a <delimiter token> or a <separa-
            tor>. If the Format does not allow a <nondelimiter token> to be
            followed by a <delimiter token>, then that <nondelimiter token>
            shall be followed by a <separator>.

         6) There shall be no <space> nor <newline> separating the <minus
            sign>s of a <comment introducer>.

         7) SQL text containing one or more instances of <comment> is equiv-
            alent to the same SQL text with the <comment> replaced with
            <newline>.

         8) The sum of the number of <identifier start>s and the number
            of <identifier part>s in a <regular identifier> shall not be
            greater than 128.

         9) The <delimited identifier body> of a <delimited identifier>
            shall not comprise more than 128 <delimited identifier part>s.

         10)The <identifier body> of a <regular identifier> is equivalent
            to an <identifier body> in which every letter that is a lower-
            case letter is replaced by the equivalent upper-case letter
            or letters. This treatment includes determination of equiva-
            lence, representation in the Information and Definition Schemas,
            representation in the diagnostics area, and similar uses.

         86  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 5.2 <token> and <separator>


         11)The <identifier body> of a <regular identifier> (with every
            letter that is a lower-case letter replaced by the equivalent
            upper-case letter or letters), treated as the repetition of
            a <character string literal> that specifies a <character set
            specification> of SQL_TEXT, shall not be equal, according to
            the comparison rules in Subclause 8.2, "<comparison predicate>",
            to any <reserved word> (with every letter that is a lower-case
            letter replaced by the equivalent upper-case letter or letters),
            treated as the repetition of a <character string literal> that
            specifies a <character set specification> of SQL_TEXT.

            Note: It is the intention that no <key word> specified in this
            International Standard or revisions thereto shall end with an
            <underscore>.

         12)Two <regular identifier>s are equivalent if their <identifier
            body>s, considered as the repetition of a <character string
            literal> that specifies a <character set specification> of
            SQL_TEXT, compare equally according to the comparison rules
            in Subclause 8.2, "<comparison predicate>".

         13)A <regular identifier> and a <delimited identifier> are equiva-
            lent if the <identifier body> of the <regular identifier> (with
            every letter that is a lower-case letter replaced by the equiva-
            lent upper-case letter or letters) and the <delimited identifier
            body> of the <delimited identifier> (with all occurrences of
            <quote> replaced by <quote symbol> and all occurrences of <dou-
            blequote symbol> replaced by <double quote>), considered as
            the repetition of a <character string literal> that specifies a
            <character set specification> of SQL_TEXT and an implementation-
            defined collation that is sensitive to case, compare equally
            according to the comparison rules in Subclause 8.2, "<comparison
            predicate>".

         14)Two <delimited identifier>s are equivalent if their <delimited
            identifier body>s (with all occurrences of <quote> replaced
            by <quote symbol> and all occurrences of <doublequote symbol>
            replaced by <doublequote>), considered as the repetition of a
            <character string literal> that specifies a <character set spec-
            ification> of SQL_TEXT and an implementation-defined collation
            that is sensitive to case, compare equally according to the
            comparison rules in Subclause 8.2, "<comparison predicate>".

         15)For the purposes of identifying <key word>s, any <simple Latin
            lower case letter> contained in a candidate <key word> shall
            be effectively treated as the corresponding <simple Latin upper
            case letter>.

         Access Rules

            None.



                                                       Lexical elements   87

 





          X3H2-92-154/DBL CBR-002
         5.2 <token> and <separator>


         General Rules

            None.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) No <identifier body> shall end in an <underscore>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) No <regular identifier> or <delimited identifier body> shall
              contain more than 18 <character representation>s.

            b) An <identifier body> shall contain no <simple Latin lower
              case letter>.




































         88  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                               5.3 <literal>


         5.3  <literal>

         Function

         Specify a non-null value.

         Format

         <literal> ::=
                <signed numeric literal>
              | <general literal>

         <unsigned literal> ::=
                <unsigned numeric literal>
              | <general literal>

         <general literal> ::=
                <character string literal>
              | <national character string literal>
              | <bit string literal>
              | <hex string literal>
              | <datetime literal>
              | <interval literal>

         <character string literal> ::=
              [ <introducer><character set specification> ]
              <quote> [ <character representation>... ] <quote>
                [ { <separator>... <quote> [ <character representation>... ] <quote> }... ]


         <introducer> ::= <underscore>

         <character representation> ::=
                <nonquote character>
              | <quote symbol>

         <nonquote character> ::= !! See the Syntax Rules.

         <quote symbol> ::= <quote><quote>

         <national character string literal> ::=
              N <quote> [ <character representation>... ] <quote>
                [ { <separator>... <quote> [ <character representation>... ] <quote> }... ]


         <bit string literal> ::=
              B <quote> [ <bit>... ] <quote>
                [ { <separator>... <quote> [ <bit>... ] <quote> }... ]

         <hex string literal> ::=
              X <quote> [ <hexit>... ] <quote>
                [ { <separator>... <quote> [ <hexit>... ] <quote> }... ]


                                                       Lexical elements   89

 





          X3H2-92-154/DBL CBR-002
         5.3 <literal>


         <bit> ::= 0 | 1

         <hexit> ::= <digit> | A | B | C | D | E | F | a | b | c | d | e | f


         <signed numeric literal> ::=
              [ <sign> ] <unsigned numeric literal>

         <unsigned numeric literal> ::=
                <exact numeric literal>
              | <approximate numeric literal>

         <exact numeric literal> ::=
                <unsigned integer> [ <period> [ <unsigned integer> ] ]
              | <period> <unsigned integer>

         <sign> ::= <plus sign> | <minus sign>

         <approximate numeric literal> ::= <mantissa> E <exponent>

         <mantissa> ::= <exact numeric literal>

         <exponent> ::= <signed integer>

         <signed integer> ::= [ <sign> ] <unsigned integer>

         <unsigned integer> ::= <digit>...

         <datetime literal> ::=
                <date literal>
              | <time literal>
              | <timestamp literal>

         <date literal> ::=
              DATE <date string>

         <time literal> ::=
              TIME <time string>

         <timestamp literal> ::=
              TIMESTAMP <timestamp string>

         <date string> ::=
              <quote> <date value> <quote>

         <time string> ::=
              <quote> <time value> [ <time zone interval> ] <quote>

         <timestamp string> ::=
              <quote> <date value> <space> <time value> [ <time zone interval> ] <quote>


         <time zone interval> ::=

         90  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                               5.3 <literal>


              <sign> <hours value> <colon> <minutes value>

         <date value> ::=
              <years value> <minus sign> <months value> <minus sign> <days value>


         <time value> ::=
              <hours value> <colon> <minutes value> <colon> <seconds value>


         <interval literal> ::=
              INTERVAL [ <sign> ] <interval string> <interval qualifier>

         <interval string> ::=
              <quote> { <year-month literal> | <day-time literal> } <quote>


         <year-month literal> ::=
                <years value>
              | [ <years value> <minus sign> ] <months value>

         <day-time literal> ::=
                <day-time interval>
              | <time interval>

         <day-time interval> ::=
              <days value>
                [ <space> <hours value> [ <colon> <minutes value> [ <colon> <seconds value> ] ] ]


         <time interval> ::=
                <hours value> [ <colon> <minutes value> [ <colon> <seconds value> ] ]

              | <minutes value> [ <colon> <seconds value> ]
              | <seconds value>

         <years value> ::= <datetime value>

         <months value> ::= <datetime value>

         <days value> ::= <datetime value>

         <hours value> ::= <datetime value>

         <minutes value> ::= <datetime value>

         <seconds value> ::=
                <seconds integer value> [ <period> [ <seconds fraction> ] ]


         <seconds integer value> ::= <unsigned integer>

         <seconds fraction> ::= <unsigned integer>

                                                       Lexical elements   91

 





          X3H2-92-154/DBL CBR-002
         5.3 <literal>


         <datetime value> ::= <unsigned integer>


         Syntax Rules

         1) In a <character string literal> or <national character string
            literal>, the sequence:

              <quote> <character representation>... <quote>
              <separator>... <quote> <character representation>... <quote>

            is equivalent to the sequence

              <quote> <character representation>... <character representa-
              tion>... <quote>

            Note: The <character representation>s in the equivalent se-
            quence are in the same sequence and relative sequence as in the
            original <character string literal>.

         2) In a <bit string literal>, the sequence

              <quote> <bit>... <quote> <separator>... <quote> <bit>...
              <quote>

            is equivalent to the sequence

              <quote> <bit>... <bit>... <quote>

            Note: The <bit>s in the equivalent sequence are in the same
            sequence and relative sequence as in the original <bit string
            literal>.

         3) In a <hex string literal>, the sequence

              <quote> <hexit>... <quote> <separator>... <quote> <hexit>...
              <quote>

            is equivalent to the sequence

              <quote> <hexit>... <hexit>... <quote>

            Note: The <hexit>s in the equivalent sequence are in the same
            sequence and relative sequence as in the original <hex string
            literal>.

         4) In a <character string literal>, <national character string
            literal>, <bit string literal>, or <hex string literal>, a <sep-
            arator> shall contain a <newline>.

         5) A <nonquote character> is one of:

            a) Any <SQL language character> other than a <quote>;

            b) Any character other than a <quote> in the character reper-
              toire identified by the <module character set specification>;
              or

            c) Any character other than a <quote> in the character reper-
              toire identified by the <character set specification> or
              implied by "N".

         92  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                               5.3 <literal>


         6) If a <character set specification> is not specified in a <char-
            acter string literal>, then the set of characters contained
            in the <character string literal> shall be wholly contained
            in either <SQL language character> or the character repertoire
            indicated by:

            Case:

            a) If the <character string literal> is contained in a <module>,
              then the <module character set specification>,

            b) If the <character string literal> is contained in a <schema
              definition> that is not contained in a <module>, then the
              <schema character set specification>,

            c) If the <character string literal> is contained in a <prepara-
              ble statement> that is prepared in the current SQL-session
              by an <execute immediate statement> or a <prepare statement>
              or in a <direct SQL statement> that is invoked directly, then
              the default character set name for the SQL-session.

         7) If a <character set specification> is specified in a <character
            string literal>, then

            a) There shall be no <separator> between the <introducer> and
              the <character set specification>.

            b) The set of characters contained in the <character string lit-
              eral> shall be wholly contained in the character repertoire
              indicated by the <character set specification>.

         8) A <national character string literal> is equivalent to a
            <character string literal> with the "N" replaced by "<intro-
            ducer><character set specification>", where "<character set
            specification>" is an implementation-defined <character set
            name>.

         9) The data type of a <character string literal> is fixed-length
            character string. The length of a <character string literal>
            is the number of <character representation>s that it contains.
            Each <quote symbol> contained in <character string literal>
            represents a single <quote> in both the value and the length of
            the <character string literal>. The two <quote>s contained in a
            <quote symbol> shall not be separated by any <separator>.

            Note: <character string literal>s are allowed to be zero-length
            strings (i.e., to contain no characters) even though it is
            not permitted to declare a <data type> that is CHARACTER with
            <length> zero.

         10)The data type of a <bit string literal> is fixed-length bit
            string. The length of a <bit string literal> is the number of
            bits that it contains.

                                                       Lexical elements   93

 





          X3H2-92-154/DBL CBR-002
         5.3 <literal>


         11)The data type of a <hex string literal> is fixed-length bit
            string. Each <hexit> appearing in the literal is equivalent
            to a quartet of bits: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C,
            D, E, and F are interpreted as 0000, 0001, 0010, 0011, 0100,
            0101, 0110, 0111, 1000, 1001, 1010, 1011, 1100, 1101, 1110,
            and 1111, respectively. The <hexit>s a, b, c, d, e, and f have
            respectively the same values as the <hexit>s A, B, C, D, E, and
            F.

         12)An <exact numeric literal> without a <period> has an implied
            <period> following the last <digit>.

         13)The data type of an <exact numeric literal> is exact numeric.
            The precision of an <exact numeric literal> is the number of
            <digit>s that it contains. The scale of an <exact numeric lit-
            eral> is the number of <digit>s to the right of the <period>.

         14)The data type of an <approximate numeric literal> is approximate
            numeric. The precision of an <approximate numeric literal> is
            the precision of its <mantissa>.

         15)The data type of a <date literal> is DATE.

         16)The data type of a <time literal> that does not specify <time
            zone interval> is TIME(P), where P is the number of digits in
            <seconds fraction>, if specified, and 0 otherwise. The data
            type of a <time literal> that specifies <time zone interval>
            is TIME(P) WITH TIME ZONE, where P is the number of digits in
            <seconds fraction>, if specified, and 0 otherwise.

         17)The data type of a <timestamp literal> that does not specify
            <time zone interval> is TIMESTAMP(P), where P is the number of
            digits in <seconds fraction>, if specified, and 0 otherwise.
            The data type of a <timestamp literal> that specifies <time zone
            interval> is TIMESTAMP(P) WITH TIME ZONE, where P is the number
            of digits in <seconds fraction>, if specified, and 0 otherwise.

         18)If <time zone interval> is not specified, then the effective
            <time zone interval> of the datetime data type is the current
            default time zone displacement for the SQL-session.

         19)Let datetime component be either <years value>, <months value>,
            <days value>, <hours value>, <minutes value>, or <seconds
            value>.

         20)Let N be the number of <datetime field>s in the precision of the
            <interval literal>, as specified by <interval qualifier>.

            The <interval literal> being defined shall contain N datetime
            components.

            The data type of <interval literal> specified with an <interval
            qualifier> is INTERVAL with the <interval qualifier>.

         94  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                               5.3 <literal>


         21)Within a <datetime literal>, the <years value> shall contain
            four digits. The <seconds integer value> and other datetime
            components, with the exception of <seconds fraction>, shall each
            contain two digits.

         22)Within the definition of a <datetime literal>, the <datetime
            value>s are constrained by the natural rules for dates and times
            according to the Gregorian calendar.

         23)Within the definition of an <interval literal>, the <datetime
            value>s are constrained by the natural rules for intervals ac-
            cording to the Gregorian calendar.

         24)Within the definition of a <year-month literal>, the <inter-
            val qualifier> shall not specify DAY, HOUR, MINUTE, or SECOND.
            Within the definition of a <day-time literal>, the <interval
            qualifier> shall not specify YEAR or MONTH.

         25)Within the definition of a <datetime literal>, the value of the
            <time zone interval> shall be in the range -12:59 to +13:00.

         Access Rules

            None.

         General Rules

         1) The value of a <character string literal> is the sequence of
            <character representation>s that it contains.

         2) The value of a <bit string literal> or a <hex string literal> is
            the sequence of bits that it contains.

         3) The numeric value of an <exact numeric literal> is determined
            by the normal mathematical interpretation of positional decimal
            notation.

         4) The numeric value of an <approximate numeric literal> is approx-
            imately the product of the exact numeric value represented by
            the <mantissa> with the number obtained by raising the number
            10 to the power of the exact numeric value represented by the
            <exponent>.

         5) The <sign> in a <signed numeric literal> or an <interval lit-
            eral> is a monadic arithmetic operator. The monadic arithmetic
            operators + and - specify monadic plus and monadic minus, re-
            spectively. If neither monadic plus nor monadic minus are spec-
            ified in a <signed numeric literal> or an <interval literal>,
            or if monadic plus is specified, then the literal is positive.
            If monadic minus is specified in a <signed numeric literal> or
            <interval literal>, then the literal is negative.



                                                       Lexical elements   95

 





          X3H2-92-154/DBL CBR-002
         5.3 <literal>


         6) Let V be the integer value of the <unsigned integer> contained
            in <seconds fraction> and let N be the number of digits in the
            <seconds fraction> respectively. The resultant value of the
            <seconds fraction> is effectively determined as follows:

            Case:

            a) If <seconds fraction> is specified within the definition of a
              <datetime literal>, then the effective value of the <seconds
              fraction> is V*10 -N seconds.

            b) If <seconds fraction> is specified within the definition of
              an <interval literal>, then let M be the <interval fractional
              seconds precision> specified in the <interval qualifier>.

              Case:

              i) If N < M, then let V1 be V *10M-N; the effective value of

                 the <seconds fraction> is V1*10 -M seconds.

             ii) If N > M, then let V2 be the integer part of the quotient
                 of V /10N-M; the effective value of the <seconds fraction>

                 is V2*10 -M seconds.

            iii) Otherwise, the effective value of the <seconds fraction> is
                 V*10 -M seconds.

         7) The i-th datetime component in a <datetime literal> or <interval
            literal> assigns the value of the datetime component to the
            i-th <datetime field> in the <datetime literal> or <interval
            literal>.

         8) If <time zone interval> is specified, then the time and times-
            tamp values in <time literal> and <timestamp literal> represent
            a datetime in the specified time zone. Otherwise, the time and
            timestamp values represent a datetime in the current default
            time zone of the SQL-session. The value of the <time literal>
            or the <timestamp literal> is effectively the <time value> or
            the <date value> and <time value> together minus the <time zone
            interval> value, followed by the <time zone interval>.

            Note: <time literal>s and <timestamp literal>s are specified in
            a time zone chosen by the SQL-agent (the default is the cur-
            rent default time zone of the SQL-session). However, they are
            effectively converted to UTC while maintaining the <time zone
            interval> information that permits knowing the original time
            zone value for the time or timestamp value.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) An <unsigned integer> that is a <seconds fraction> shall not
              contain more than 6 <digit>s.

         96  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                               5.3 <literal>


            b) A <general literal> shall not be a <bit string literal> or a
              <hex string literal>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <general literal> shall not be a <national character string
              literal>.

            b) A <general literal> shall not be a <datetime literal> or
              <interval literal>.

            c) A <character string literal> shall contain at least one
              <character representation>.

            d) Conforming Entry SQL language shall contain exactly one rep-
              etition of <character representation> (that is, it shall
              contain exactly one sequence of "<quote> <character represen-
              tation>... <quote>").

            e) A <character string literal> shall not specify a <character
              set specification>.
































                                                       Lexical elements   97

 





          X3H2-92-154/DBL CBR-002
         5.4 Names and identifiers


         5.4  Names and identifiers

         Function

         Specify names.

         Format

         <identifier> ::=
              [ <introducer><character set specification> ] <actual identifier>


         <actual identifier> ::=
                <regular identifier>
              | <delimited identifier>

         <SQL language identifier> ::=
              <SQL language identifier start>
                [ { <underscore> | <SQL language identifier part> }... ]

         <SQL language identifier start> ::= <simple Latin letter>

         <SQL language identifier part> ::=
                <simple Latin letter>
              | <digit>

         <authorization identifier> ::= <identifier>

         <table name> ::=
                <qualified name>
              | <qualified local table name>

         <qualified local table name> ::=
              MODULE <period> <local table name>

         <local table name> ::= <qualified identifier>

         <domain name> ::= <qualified name>

         <schema name> ::=
              [ <catalog name> <period> ] <unqualified schema name>

         <unqualified schema name> ::= <identifier>

         <catalog name> ::= <identifier>

         <qualified name> ::=
              [ <schema name> <period> ] <qualified identifier>

         <qualified identifier> ::= <identifier>

         <column name> ::= <identifier>


         98  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   5.4 Names and identifiers


         <correlation name> ::= <identifier>

         <module name> ::= <identifier>

         <cursor name> ::= <identifier>

         <procedure name> ::= <identifier>

         <SQL statement name> ::=
                <statement name>
              | <extended statement name>

         <statement name> ::= <identifier>

         <extended statement name> ::=
              [ <scope option> ] <simple value specification>

         <dynamic cursor name> ::=
                <cursor name>
              | <extended cursor name>

         <extended cursor name> ::=
              [ <scope option> ] <simple value specification>

         <descriptor name> ::=
              [ <scope option> ] <simple value specification>

         <scope option> ::=
                GLOBAL
              | LOCAL

         <parameter name> ::= <colon> <identifier>

         <constraint name> ::= <qualified name>

         <collation name> ::= <qualified name>

         <character set name> ::= [ <schema name> <period> ] <SQL language identifier>


         <translation name> ::= <qualified name>

         <form-of-use conversion name> ::= <qualified name>

         <connection name> ::= <simple value specification>

         <SQL-server name> ::= <simple value specification>

         <user name> ::= <simple value specification>





                                                       Lexical elements   99

 





          X3H2-92-154/DBL CBR-002
         5.4 Names and identifiers


         Syntax Rules

         1) If a <character set specification> is not specified in an <iden-
            tifier>, then the set of characters contained in the <identi-
            fier> shall be wholly contained in either <SQL language charac-
            ter> or the character repertoire identified by:

            Case:

            a) If the <identifier> is contained in a <module>, then the
              <module character set specification>,

            b) If the <identifier> is contained in a <schema definition>
              that is not contained in a <module>, then the <schema charac-
              ter set specification>,

            c) If the <identifier> is contained in a <preparable statement>
              that is prepared in the current SQL-session by an <execute
              immediate statement> or a <prepare statement> or in a <direct
              SQL statement> that is invoked directly, then the default
              character set name for the SQL-session.

         2) If a <character set specification> is specified in an <identi-
            fier>, then:

            a) There shall be no <separator> between the <introducer> and
              the <character set specification>.

            b) The set of characters contained in the <identifier body>
              or <delimited identifier body> shall be wholly contained
              in the character repertoire indicated by the <character set
              specification>.

         3) The sum of the number of <SQL language identifier start>s and
            the number of <SQL language identifier part>s in an <SQL lan-
            guage identifier> shall not be greater than 128.

         4) An <SQL language identifier> is equivalent to an <SQL language
            identifier> in which every letter that is a lower-case letter
            is replaced by the equivalent upper-case letter or letters. This
            treatment includes determination of equivalence, representation
            in the Information and Definition Schemas, representation in the
            diagnostics area, and similar uses.

         5) An <SQL language identifier> (with every letter that is a lower-
            case letter replaced by the equivalent upper-case letter),
            treated as the repetition of a <character string literal> that
            specifies a <character set specification> of SQL_TEXT, shall not
            be equal, according to the comparison rules in Subclause 8.2,
            "<comparison predicate>", to any <reserved word> (with every
            letter that is a lower-case letter replaced by the equivalent
            upper-case letter), treated as the repetition of a <character
            string literal> that specifies a <character set specification>
            of SQL_TEXT.

         100  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   5.4 Names and identifiers


            Note: It is the intention that no <key word> specified in this
            International standard or revisions thereto shall end with an
            <underscore>.

         6) If <table name> is not a <qualified local table name>, then the
            table identified by <table name> shall not be a declared local
            temporary table.

         7) No <unqualified schema name> shall specify DEFINITION_SCHEMA.

         8) If a <qualified name> does not contain a <schema name>, then

            Case:

            a) If the <qualified name> is contained in a <schema defini-
              tion>, then the <schema name> that is specified or implicit
              in the <schema definition> is implicit.

            b) If the <qualified name> is contained in a <preparable state-
              ment> that is prepared in the current SQL-session by an <ex-
              ecute immediate statement> or a <prepare statement> or in
              a <direct SQL statement> that is invoked directly, then the
              default <unqualified schema name> for the SQL-session is
              implicit.

            c) Otherwise, the <schema name> that is specified or implicit
              for the <module> is implicit.

         9) If a <schema name> does not contain a <catalog name>, then

            Case:

            a) If the <unqualified schema name> is contained in a <mod-
              ule authorization clause>, then an implementation-defined
              <catalog name> is implicit.

            b) If the <unqualified schema name> is contained in a <schema
              definition> other than in a <schema name clause>, then the
              <catalog name> that is specified or implicit in the <schema
              name clause> is implicit.

            c) If the <unqualified schema name> is contained in a <prepara-
              ble statement> that is prepared in the current SQL-session
              by an <execute immediate statement> or a <prepare statement>
              or in a <direct SQL statement> that is invoked directly, then
              the default catalog name for the SQL-session is implicit.

            d) If the <unqualified schema name> is contained in a <schema
              name clause>, then

              Case:

              i) If the <schema name clause> is contained in a <module>,
                 then the explicit or implicit <catalog name> contained in
                 the <module authorization clause> is implicit.

                                                      Lexical elements   101

 





          X3H2-92-154/DBL CBR-002
         5.4 Names and identifiers


             ii) Otherwise, an implementation-defined <catalog name> is
                 implicit.

            e) Otherwise, the explicit or implicit <catalog name> contained
              in the <module authorization clause> is implicit.

         10)Two <qualified name>s are equal if and only if they have the
            same <qualified identifier> and the same <schema name>, regard-
            less of whether the <schema name>s are implicit or explicit.

         11)Two <schema name>s are equal if and only if they have the same
            <unqualified schema name> and the same <catalog name>, regard-
            less of whether the <catalog name>s are implicit or explicit.


         12)An <identifier> that is a <correlation name> is associated with
            a table within a particular scope. The scope of a <correlation
            name> is either a <select statement: single row>, <subquery>, or
            <query specification> (see Subclause 6.3, "<table reference>").
            Scopes may be nested. In different scopes, the same <correlation
            name> may be associated with different tables or with the same
            table.

         13)The <simple value specification> of <extended statement name> or
            <extended cursor name> shall not be a <literal>.

         14)The data type of the <simple value specification> of <ex-
            tended statement name> shall be character string with an
            implementation-defined character set and shall have an octet
            length of 128 octets or less.

         15)The data type of the <simple value specification> of <extended
            cursor name> shall be character string with an implementation-
            defined character set and shall have an octet length of 128
            octets or less.

         16)The data type of the <simple value specification> of <descriptor
            name> shall be character string with an implementation-defined
            character set and shall have an octet length of 128 octets or
            less.

         17)In a <descriptor name>, <extended statement name>, or <extended
            cursor name>, if a <scope option> is not specified, then a
            <scope option> of LOCAL is implicit.

         18)No <authorization identifier> shall specify "PUBLIC".

         19)Those <identifier>s that are valid <authorization identifier>s
            are implementation-defined.

         20)Those <identifier>s that are valid <catalog name>s are implementation-
            defined.


         102  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   5.4 Names and identifiers


         21)If a <character set name> does not specify a <schema name>, then
            INFORMATION_SCHEMA is implicit.

         22)If a <collation name> does not specify a <schema name>, then
            INFORMATION_SCHEMA is implicit.

         23)If a <translation name> does not specify a <schema name>, then
            INFORMATION_SCHEMA is implicit.

         24)The <data type> of <SQL-server name>, <connection name>, and
            <user name> shall be character string with an implementation-
            defined character set and shall have an octet length of 128
            octets or less.

         25)If a <form-of-use conversion name> does not specify a <schema
            name>, then INFORMATION_SCHEMA is implicit; otherwise, INFORMATION_
            SCHEMA shall be specified.

         Access Rules

            None.

         General Rules

         1) A <table name> identifies a table.

         2) Within its scope, a <correlation name> identifies a table.

         3) A <local table name> identifies a declared local temporary ta-
            ble.

         4) A <column name> identifies a column.

         5) A <domain name> identifies a domain.

         6) An <authorization identifier> represents an authorization iden-
            tifier and identifies a set of privileges.

         7) A <module name> identifies a <module>.

         8) A <cursor name> identifies a cursor.

         9) A <procedure name> identifies a <procedure>.

         10)A <parameter name> identifies a parameter.

         11)A <constraint name> identifies a table constraint, a domain
            constraint, or an assertion.

         12)A <statement name> identifies a statement prepared by the execu-
            tion of a <prepare statement>. The scope of a <statement name>
            is the <module> in which it appears and the current SQL-session.


                                                      Lexical elements   103

 





          X3H2-92-154/DBL CBR-002
         5.4 Names and identifiers


         13)The value of an <extended statement name> identifies a statement
            prepared by the execution of a <prepare statement>. If a <scope
            option> of GLOBAL is specified, then the scope of the <extended
            statement name> is the current SQL-session. If a <scope option>
            of LOCAL is specified or implicit, then the scope of the state-
            ment name is further restricted to the <module> in which the
            <extended statement name> appears.

         14)A <dynamic cursor name> identifies a cursor in an <SQL dynamic
            statement>.

         15)The value of an <extended cursor name> identifies a cursor cre-
            ated by the execution of an <allocate cursor statement>. If a
            <scope option> of GLOBAL is specified, then the scope of the
            <extended cursor name> is the current SQL-session. If a <scope
            option> of LOCAL is specified of implicit, then the scope of the
            cursor name is further restricted to the <module> in which the
            <extended cursor name> appears.

         16)A <descriptor name> identifies an SQL descriptor area created
            by the execution of an <allocate descriptor statement>. If a
            <scope option> of GLOBAL is specified, then the scope of the
            <descriptor name> is the current SQL-session. If a <scope op-
            tion> of LOCAL is specified or implicit, then the scope of the
            <descriptor name> is further restricted to the <module> in which
            the <descriptor name> appears.

         17)A <catalog name> identifies a catalog.

         18)A <schema name> identifies a schema.

         19)A <collation name> identifies a collating sequence.

         20)A <character set name> identifies a character set.

         21)A <translation name> identifies a character translation.

         22)A <form-of-use conversion name> identifies a form-of-use con-
            version. All <form-of-use conversion name>s are implementation-
            defined.

         23)A <connection name> identifies an SQL-connection.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) Conforming Intermediate SQL language shall not contain any
              <extended statement name> or <extended cursor name>.

            b) Conforming Intermediate SQL language shall not contain any
              explicit <catalog name>, <connection name>, <collation name>,
              <translation name>, <form-of-use conversion name>, or <quali-
              fied local table name>.

         104  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   5.4 Names and identifiers


         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <domain
              name>, <SQL statement name>, <dynamic cursor name>, <con-
              straint name>, <descriptor name>, or <character set name>.

            b) An <identifier> shall not specify a <character set specifica-
              tion>.













































                                                      Lexical elements   105

 





          X3H2-92-154/DBL CBR-002

























































         106  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002






         6  Scalar expressions



         6.1  <data type>

         Function

         Specify a data type.

         Format

         <data type> ::=
                <character string type> [ CHARACTER SET <character set specification> ]

              | <national character string type>
              | <bit string type>
              | <numeric type>
              | <datetime type>
              | <interval type>

         <character string type> ::=
                CHARACTER [ <left paren> <length> <right paren> ]
              | CHAR [ <left paren> <length> <right paren> ]
              | CHARACTER VARYING <left paren> <length> <right paren>
              | CHAR VARYING <left paren> <length> <right paren>
              | VARCHAR <left paren> <length> <right paren>

         <national character string type> ::=
                NATIONAL CHARACTER [ <left paren> <length> <right paren> ]
              | NATIONAL CHAR [ <left paren> <length> <right paren> ]
              | NCHAR [ <left paren> <length> <right paren> ]
              | NATIONAL CHARACTER VARYING <left paren> <length> <right paren>

              | NATIONAL CHAR VARYING <left paren> <length> <right paren>
              | NCHAR VARYING <left paren> <length> <right paren>

         <bit string type> ::=
                BIT [ <left paren> <length> <right paren> ]
              | BIT VARYING <left paren> <length> <right paren>

         <numeric type> ::=
                <exact numeric type>
              | <approximate numeric type>

         <exact numeric type> ::=
                NUMERIC [ <left paren> <precision> [ <comma> <scale> ] <right paren> ]

              | DECIMAL [ <left paren> <precision> [ <comma> <scale> ] <right paren> ]


                                                    Scalar expressions   107

 





          X3H2-92-154/DBL CBR-002
         6.1 <data type>


              | DEC [ <left paren> <precision> [ <comma> <scale> ] <right paren> ]

              | INTEGER
              | INT
              | SMALLINT

         <approximate numeric type> ::=
                FLOAT [ <left paren> <precision> <right paren> ]
              | REAL
              | DOUBLE PRECISION

         <length> ::= <unsigned integer>

         <precision> ::= <unsigned integer>

         <scale> ::= <unsigned integer>

         <datetime type> ::=
                DATE
              | TIME [ <left paren> <time precision> <right paren> ]
              [ WITH TIME ZONE ]
              | TIMESTAMP [ <left paren> <timestamp precision> <right paren> ]
              [ WITH TIME ZONE ]

         <time precision> ::= <time fractional seconds precision>

         <timestamp precision> ::= <time fractional seconds precision>

         <time fractional seconds precision> ::= <unsigned integer>

         <interval type> ::= INTERVAL <interval qualifier>


         Syntax Rules

         1) CHAR is equivalent to CHARACTER. DEC is equivalent to DECIMAL.
            INT is equivalent to INTEGER. VARCHAR is equivalent to CHARACTER
            VARYING. NCHAR is equivalent to NATIONAL CHARACTER.

         2) "NATIONAL CHARACTER" is equivalent to the corresponding <char-
            acter string type> with a specification of "CHARACTER SET CSN",
            where "CSN" is an implementation-defined <character set name>.

         3) The value of a <length> or a <precision> shall be greater than
            0.

         4) If <length> is omitted, then a <length> of 1 is implicit.

         5) If a <scale> is omitted, then a <scale> of 0 is implicit.

         6) If a <precision> is omitted, then an implementation-defined
            <precision> is implicit.

         7) CHARACTER specifies the data type character string.

         108  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             6.1 <data type>


         8) Characters in a character string are numbered beginning with 1.

         9) Case:

            a) If VARYING is not specified in <character string type>, then
              the length in characters of the character string is fixed and
              is the value of <length>.

            b) If VARYING is specified in <character string type>, then the
              length in characters of the character string is variable,
              with a minimum length of 0 and a maximum length of the value
              of <length>.

            The maximum value of <length> is implementation-defined.
            <length> shall not be greater than this maximum value.

         10)If <character string type> is not contained in a <domain def-
            inition> or a <column definition> and CHARACTER SET is not
            specified, then an implementation-defined <character set speci-
            fication> is implicit.

            Note: Subclause 11.21, "<domain definition>", and Subclause 11.4,
            "<column definition>", specify the result when <character string
            type> is contained in a <domain definition> or <column defini-
            tion>, respectively.

         11)The character set named SQL_TEXT is an implementation-defined
            character set whose character repertoire is SQL_TEXT.

            Note: The character repertoire SQL_TEXT is defined in Subclause 4.2,
            "Character strings".

         12)BIT specifies the data type bit string.

         13)Bits in a bit string are numbered beginning with 1.

         14)Case:

            a) If VARYING is not specified in <bit string type>, then the
              length in bits of the bit string is fixed and is the value of
              <length>.

            b) If VARYING is specified in <bit string type>, then the length
              in bits of the string is variable, with a minimum length of 0
              and a maximum length of the value of <length>.

            The maximum value of <length> is implementation-defined.
            <length> shall not be greater than this maximum value.

         15)The <scale> of an <exact numeric type> shall not be greater than
            the <precision> of the <exact numeric type>.

         16)For the <exact numeric type>s DECIMAL and NUMERIC:

            a) The maximum value of <precision> is implementation-defined.
              <precision> shall not be greater than this value.

                                                    Scalar expressions   109

 





          X3H2-92-154/DBL CBR-002
         6.1 <data type>


            b) The maximum value of <scale> is implementation-defined.
              <scale> shall not be greater than this maximum value.

         17)NUMERIC specifies the data type exact numeric, with the decimal
            precision and scale specified by the <precision> and <scale>.

         18)DECIMAL specifies the data type exact numeric, with the decimal
            scale specified by the <scale> and the implementation-defined
            decimal precision equal to or greater than the value of the
            specified <precision>.

         19)INTEGER specifies the data type exact numeric, with binary or
            decimal precision and scale of 0. The choice of binary versus
            decimal precision is implementation-defined, but shall be the
            same as SMALLINT.

         20)SMALLINT specifies the data type exact numeric, with scale of
            0 and binary or decimal precision. The choice of binary versus
            decimal precision is implementation-defined, but shall be the
            same as INTEGER. The precision of SMALLINT shall be less than or
            equal to the precision of INTEGER.

         21)FLOAT specifies the data type approximate numeric, with binary
            precision equal to or greater than the value of the specified
            <precision>. The maximum value of <precision> is implementation-
            defined. <precision> shall not be greater than this value.

         22)REAL specifies the data type approximate numeric, with implementation-
            defined precision.

         23)DOUBLE PRECISION specifies the data type approximate numeric,
            with implementation-defined precision that is greater than the
            implementation-defined precision of REAL.

         24)For the <approximate numeric type>s FLOAT, REAL, and DOUBLE
            PRECISION, the maximum and minimum values of the exponent are
            implementation-defined.

         25)If <time precision> is not specified, then 0 is implicit. If
            <timestamp precision> is not specified, then 6 is implicit.

         26)The maximum value of <time precision> and the maximum value of
            <timestamp precision> shall be the same implementation-defined
            value that is not less than 6. The values of <time precision>
            and <timestamp precision> shall not be greater than that maximum
            value.

         27)The length of a DATE is 10 positions. The length of a TIME is 8
            positions plus the <time fractional seconds precision>, plus 1
            position if the <time fractional seconds precision> is greater
            than 0. The length of a TIME WITH TIME ZONE is 14 positions
            plus the <time fractional seconds precision> plus 1 position if
            the <time fractional seconds precision> is greater than 0. The
            length of a TIMESTAMP is 19 positions plus the <time fractional

         110  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             6.1 <data type>


            seconds precision>, plus 1 position if the <time fractional sec-
            onds precision> is greater than 0. The length of a TIMESTAMP
            WITH TIME ZONE is 25 positions plus the <time fractional sec-
            onds precision> plus 1 position if the <time fractional seconds
            precision> is greater than 0.

         28)If an <interval qualifier> in an <interval type> includes no
            fields other than YEAR and MONTH, then the <interval type> is a
            year-month interval. If an <interval qualifier> in an <interval
            type> includes any fields other than YEAR or MONTH, then the
            <interval type> is a day-time interval.

         29)The i-th value of an interval data type corresponds to the i-th
            <datetime field>.

         30)Within the non-null values of a <datetime type>, the value of
            the time zone interval shall be in the range -12:59 to +13:00.

            Note: The range for time zone intervals is larger than many
            readers might expect because it is governed by political deci-
            sions in governmental bodies rather than by any natural law.

         Access Rules

            None.

         General Rules

         1) If any specification or operation attempts to cause an item of
            a character type to contain a character that is not a member
            of the character repertoire associated with the character item,
            then an exception condition is raised: data exception-character
            not in repertoire.

         2) For a <datetime type>,

            Case:

            a) If DATE is specified, then the data type contains the <date-
              time field>s years, months, and days.

            b) If TIME is specified, then the data type contains the <date-
              time field>s hours, minutes, and seconds.

            c) If TIMESTAMP is specified, then the data type contains the
              <datetime field>s years, months, days, hours, minutes, and
              seconds.

         3) For a <datetime type>, a <time fractional seconds precision>
            that is an explicit or implicit <time precision> or <timestamp
            precision> defines the number of decimal digits following the
            decimal point in the SECOND <datetime field>.


                                                    Scalar expressions   111

 





          X3H2-92-154/DBL CBR-002
         6.1 <data type>


         4) Table 10, "Valid values for fields in datetime items", specifies
            the constraints on the values of the <datetime field>s in a
            datetime data type.

         _________Table_10-Valid_values_for_fields_in_datetime_items________

         _Keyword____________Valid_values_of_datetime_fields________________

        | YEAR             | 0001 to 9999                                  |
        |                  |                                               |
        | MONTH            | 01 to 12                                      |
        |                  |                                               |
        | DAY              | Within the range 1 to 31, but further con-    |
                             strained by the value of MONTH and YEAR
                             fields, according to the rules for well-
                             formed dates in the Gregorian calendar.

        | HOUR             | 00 to 23                                      |
        |                  |                                               |
        | MINUTE           | 00 to 59                                      |
        |                  |                                               |
        | SECOND           | 00 to 61.9(N) where "9(N)" indicates the num- |
                             ber of digits specified by <time fractional
                             seconds precision>.

        | TIMEZONE_HOUR    | 00 to 13                                      |
        |                  |                                               |
        |_TIMEZONE_MINUTE__|_00_to_59______________________________________|
        |                  |                                               |
            Note: Datetime data types will allow dates in the Gregorian
            format to be stored in the date range 0001-01-01 CE through
            9999-12-31 CE. The range for SECOND allows for as many as two
            "leap seconds". Interval arithmetic that involves leap seconds
            or discontinuities in calendars will produce implementation-
            defined results.

         5) If WITH TIME ZONE is not specified, then the time zone dis-
            placement of the datetime data type is effectively the current
            default time zone displacement of the SQL-session.

         6) If any specification or operation attempts to cause an item
            of type datetime to take a value where the <datetime field>
            YEAR has the value greater than 9999 or less than 1, then an
            exception condition is raised: data exception-invalid datetime
            format.

         7) The values of the <datetime field>s within an interval data type
            are constrained as follows:

            a) The value corresponding to the first <datetime field> is
              an integer with at most N digits, where N is the <interval
              leading field precision>.

            b) Table 11, "Valid values for fields in INTERVAL items", spec-
              ifies the constraints for the other <datetime field>s in the
              interval data type.

         112  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                             6.1 <data type>


         _________Table_11-Valid_values_for_fields_in_INTERVAL_items________

         _Keyword______Valid_values_of_INTERVAL_fields______________________

        | MONTH      | 0 to 11                                             |
        |            |                                                     |
        | HOUR       | 0 to 23                                             |
        |            |                                                     |
        | MINUTE     | 0 to 59                                             |
        |            |                                                     |
        | SECOND     | 0 to 59.9(N) where "9(N)" indicates the number of   |
                       digits specified by <interval fractional seconds
         ______________precision>_in_the_<interval_qualifier>.______________

        |8) An item o| type interval can contain positive or negative inter|
        |   vals.    |                                                     |
        |            |                                                     |
         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) A <datetime type> shall not specify a <time precision> or
              <timestamp precision>.

            b) A <data type> shall not be a <bit string type>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <character string type> shall not specify VARYING or
              VARCHAR.

            b) A <data type> shall not be a <datetime type> or an <interval
              type>.

            c) A <data type> shall not be a <national character string type>
              nor specify CHARACTER SET.

















                                                    Scalar expressions   113

 





          X3H2-92-154/DBL CBR-002
         6.2 <value specification> and <target specification>


         6.2  <value specification> and <target specification>

         Function

         Specify one or more values, parameters, or variables.

         Format

         <value specification> ::=
                <literal>
              | <general value specification>

         <unsigned value specification> ::=
                <unsigned literal>
              | <general value specification>

         <general value specification> ::=
                <parameter specification>
              | <dynamic parameter specification>
              | <variable specification>
              | USER
              | CURRENT_USER
              | SESSION_USER
              | SYSTEM_USER
              | VALUE

         <simple value specification> ::=
                <parameter name>
              | <embedded variable name>
              | <literal>

         <target specification> ::=
                <parameter specification>
              | <variable specification>

         <simple target specification> ::=
                <parameter name>
              | <embedded variable name>

         <parameter specification> ::=
              <parameter name> [ <indicator parameter> ]

         <indicator parameter> ::=
              [ INDICATOR ] <parameter name>

         <dynamic parameter specification> ::= <question mark>

         <variable specification> ::=
              <embedded variable name> [ <indicator variable> ]

         <indicator variable> ::=
              [ INDICATOR ] <embedded variable name>


         114  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                        6.2 <value specification> and <target specification>



         Syntax Rules

         1) The data type of an <indicator parameter> or <indicator vari-
            able> shall be exact numeric with a scale of 0.

         2) Each <parameter name> shall be contained in a <module>. Each
            <embedded variable name> shall be contained in an <embedded
            SQL statement>. Each <dynamic parameter specification> shall
            be contained in a <preparable statement> that is dynamically
            prepared in the current SQL-session through the execution of a
            <prepare statement> or an <execute immediate statement>.

         3) If USER is specified, then CURRENT_USER is implicit.

         4) The data type of CURRENT_USER, SESSION_USER, and SYSTEM_USER is
            character string. Whether the character string is fixed-length
            or variable-length, and its length if it is fixed-length or
            maximum length if it is variable-length, are implementation-
            defined. The character set of the character string is SQL_TEXT.

         5) The <value specification> or <unsigned value specification>
            VALUE shall be contained in a <domain constraint>. The data
            type of an instance of VALUE is the <data type> of the <domain
            definition> containing that <domain constraint>.

         6) If the data type of the <value specification> or <unsigned value
            specification> is character string, then the <value specifi-
            cation> or <unsigned value specification> has the Coercible
            coercibility attribute, and the collating sequence is deter-
            mined by Subclause 4.2.3, "Rules determining collating sequence
            usage".

         Access Rules

            None.

         General Rules

         1) A <value specification> or <unsigned value specification> speci-
            fies a value that is not selected from a table.

         2) A <parameter specification> identifies a parameter or a parame-
            ter and an indicator parameter in a <module>.

         3) A <dynamic parameter specification> identifies a parameter used
            by a dynamically prepared statement.

         4) A <variable specification> identifies a host variable or a host
            variable and an indicator variable.

         5) A <target specification> specifies a parameter or variable that
            can be assigned a value.

                                                    Scalar expressions   115

 





          X3H2-92-154/DBL CBR-002
         6.2 <value specification> and <target specification>


         6) If a <parameter specification> contains an <indicator parame-
            ter> and the value of the indicator parameter is negative, then
            the value specified by the <parameter specification> is null.
            Otherwise, the value specified by a <parameter specification> is
            the value of the parameter identified by the <parameter name>.

         7) If a <variable specification> contains an <indicator vari-
            able> and the value of the indicator variable is negative, then
            the value specified by the <variable specification> is null.
            Otherwise, the value specified by a <variable specification> is
            the value of the variable identified by the <embedded variable
            name>.

         8) The value specified by a <literal> is the value represented by
            that <literal>.

         9) The value specified by CURRENT_USER is the value of the current
            <authorization identifier>.

         10)The value specified by SESSION_USER is the value of the SQL-
            session <authorization identifier>.

         11)The value specified by SYSTEM_USER is equal to an implementation-
            defined string that represents the operating system user who
            executed the <module> that contains the SQL-statement whose ex-
            ecution caused the SYSTEM_USER <general value specification> to
            be evaluated.

         12)A <simple value specification> specifies a <value specifica-
            tion> or <unsigned value specification> that is not null and
            does not have an associated <indicator parameter> or <indicator
            variable>.

         13)A <simple target specification> specifies a parameter or vari-
            able that can be assigned a value that is not null.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) In Intermediate SQL, the specific data type of <indicator
              parameter>s and <indicator variable>s shall be the same
              implementation-defined data type.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <general value specification> shall not be a <dynamic pa-
              rameter specification>.

            b) A <general value specification> shall not specify VALUE.



         116  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                        6.2 <value specification> and <target specification>


            c) A <general value specification> shall not specify CURRENT_
              USER, SYSTEM_USER, or SESSION_USER.

              Note: Although CURRENT_USER and USER are semantically the
              same, in Entry SQL, CURRENT_USER must be specified as USER.

















































                                                    Scalar expressions   117

 





          X3H2-92-154/DBL CBR-002
         6.3 <table reference>


         6.3  <table reference>

         Function

         Reference a table.

         Format

         <table reference> ::=
                <table name> [ [ AS ] <correlation name>
                    [ <left paren> <derived column list> <right paren> ] ]
              | <derived table> [ AS ] <correlation name>
                    [ <left paren> <derived column list> <right paren> ]
              | <joined table>

         <derived table> ::= <table subquery>

         <derived column list> ::= <column name list>

         <column name list> ::=
              <column name> [ { <comma> <column name> }... ]


         Syntax Rules

         1) A <correlation name> immediately contained in a <table refer-
            ence> TR is exposed by TR. A <table name> immediately contained
            in a <table reference> TR is exposed by TR if and only if TR
            does not specify a <correlation name>.

         2) Case:

            a) If a <table reference> TR is contained in a <from clause> FC
              with no intervening <derived table>, then the scope clause
              SC of TR is the <select statement: single row> or innermost
              <query specification> that contains FC. The scope clause of
              the exposed <correlation name> or exposed <table name> of TR
              is the <select list>, <where clause>, <group by clause>, and
              <having clause> of SC, together with the <join condition> of
              all <joined table>s contained in SC that contains TR.

            b) Otherwise, the scope clause SC of TR is the outermost <joined
              table> that contains TR with no intervening <derived table>.
              The scope of the exposed <correlation name> or exposed <table
              name> of TR is the <join condition> of SC and of all <joined
              table>s contained in SC that contain TR.

         3) A <table name> that is exposed by a <table reference> TR shall
            not be the same as any other <table name> that is exposed by a
            <table reference> with the same scope clause as TR.




         118  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       6.3 <table reference>


         4) A <correlation name> that is exposed by a <table reference> TR
            shall not be the same as any other <correlation name> that is
            exposed by a <table reference> with the same scope clause as TR
            and shall not be the same as the <qualified identifier> of any
            <table name> that is exposed by a <table reference> with the
            same scope clause as TR.

         5) A <table name> immediately contained in a <table reference> TR
            has a scope clause and scope defined by that <table reference>
            if and only if the <table name> is exposed by TR.

         6) The same <column name> shall not be specified more than once in
            a <derived column list>.

         7) If a <derived column list> is specified in a <table reference>,
            then the number of <column name>s in the <derived column list>
            shall be the same as the degree of the table specified by the
            <derived table> or the <table name> of that <table reference>,
            and the name of the i-th column of that <derived table> or the
            effective name of the i-th column of that <table name> is the
            i-th <column name> in that <derived column list>.

         8) A <derived table> is an updatable derived table if and only if
            the <query expression> simply contained in the <subquery> of the
            <table subquery> of the <derived table> is updatable.

         Access Rules

         1) Let T be the table identified by the <table name> immediately
            contained in <table reference>. If the <table reference> is
            contained in any of:

            a) a <query expression> simply contained in a <cursor speci-
              fication>, a <view definition>, a <direct select statement:
              multiple rows>, or an <insert statement>; or

            b) a <table expression> or <select list> immediately contained
              in a <select statement: single row>; or

            c) a <search condition> immediately contained in a <delete
              statement: searched> or an <update statement: searched>; or

            d) a <value expression> immediately contained in an <update
              source>,

            then the applicable privileges shall include SELECT for T.

         General Rules

         1) The <correlation name> or exposed <table name> contained in a
            <table reference> defines that <correlation name> or <table
            name> to be an identifier of the table identified by the <table
            name> or <derived table> of that <table reference>.

                                                    Scalar expressions   119

 





          X3H2-92-154/DBL CBR-002
         6.3 <table reference>


         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) A <table reference> shall not be a <derived table>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <table reference> shall not be a <joined table>.

            b) The optional <key word> AS shall not be specified.

            c) <derived column list> shall not be specified.








































         120  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                      6.4 <column reference>


         6.4  <column reference>

         Function

         Reference a column.

         Format

         <column reference> ::= [ <qualifier> <period> ] <column name>

         <qualifier> ::=
                <table name>
              | <correlation name>


         Syntax Rules

         1) Let CR be the <column reference>, let CN be the <column name>
            contained in CR, and let C be the column identified by CN.

         2) If CR contains a <qualifier> Q, then CR shall appear within the
            scope of one or more <table name>s or <correlation name>s that
            are equal to Q. If there is more than one such <table name> or
            <correlation name>, then the one with the most local scope is
            specified. Let T be the table associated with Q.

            a) T shall include a column whose <column name> is CN.

            b) If T is a <table reference> in a <joined table> J, then CN
              shall not be a common column name in J.

              Note: Common column name is defined in Subclause 7.5, "<joined
              table>".

         3) If CR does not contain a <qualifier>, then CR shall be contained
            within the scope of one or more <table name>s or <correlation
            name>s whose associated tables include a column whose <column
            name> is CN. Let the phrase possible qualifiers denote those
            <table name>s and <correlation name>s.

            a) Case:

              i) If the most local scope contains exactly one possible
                 qualifier, then the qualifier Q equivalent to that unique
                 <table name> or <correlation name> is implicit.

             ii) If there is more than one possible qualifier with most
                 local scope, then:

                 1) Each possible qualifier shall be a <table name> or a
                   <correlation name> of a <table reference> that is di-
                   rectly contained in a <joined table> J.


                                                    Scalar expressions   121

 





          X3H2-92-154/DBL CBR-002
         6.4 <column reference>


                 2) CN shall be a common column name in J.

                   Note: Common column name is defined in Subclause 7.5,
                   "<joined table>".

                 3) The implicit qualifier Q is implementation-dependent.
                   The scope of Q is that which Q would have had if J had
                   been replaced by the <table reference>:

                      ( J ) AS Q

            b) Let T be the table associated with Q.

         4) The data type of CR is the data type of column C of T. CN shall
            uniquely identify a column of T.

         5) If the data type of CR is character string, then CR has the
            Implicit coercibility attribute and its collating sequence is
            the default collating sequence for column C of T.

         6) If the data type of CR is TIME or TIMESTAMP, then the implicit
            time zone of the data is the current default time zone for the
            SQL-session.

         7) If the data type of CR is TIME WITH TIME ZONE or TIMESTAMP WITH
            TIME ZONE, then the time zone of the data is the time zone rep-
            resented in the value of CR.

         8) If CR is contained in a <table expression> TE and the scope
            clause of the <table reference> immediately containing the <ta-
            ble name> or <correlation name> Q also contains TE, then CR is
            an outer reference to the table associated with Q.

         9) Let CR be the <column reference> and let C be the column identi-
            fied by CR. C is an underlying column of CR. If C is a <derived
            column>, then every underlying column of C is an underlying
            column of CR.

            Note: The underlying columns of a <derived column> are defined
            in Subclause 7.9, "<query specification>".

         Access Rules

         1) The applicable privileges shall include SELECT for T if CR is
            contained in any of:

            a) a <search condition> immediately contained in a <delete
              statement: searched> or an <update statement: searched>; or

            b) a <value expression> immediately contained in an <update
              source>.



         122  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                      6.4 <column reference>


         General Rules

         1) The <column reference> Q.CN references column C in a given row
            of T.

         2) If the data type of CR is TIME, TIMESTAMP, TIME WITH TIME ZONE
            or TIMESTAMP WITH TIME ZONE, then let TZ be an INTERVAL HOUR
            TO MINUTE containing the value of the time zone displacement
            associated with CR. The value of CR, normalized to UTC, is ef-
            fectively computed as

              CR + TZ.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL;

              None.

         2) The following restrictions apply for Entry SQL;

              None.
































                                                    Scalar expressions   123

 





          X3H2-92-154/DBL CBR-002
         6.5 <set function specification>


         6.5  <set function specification>

         Function

         Specify a value derived by the application of a function to an
         argument.

         Format

         <set function specification> ::=
                COUNT <left paren> <asterisk> <right paren>
              | <general set function>

         <general set function> ::=
                <set function type>
                    <left paren> [ <set quantifier> ] <value expression> <right paren>


         <set function type> ::=
              AVG | MAX | MIN | SUM | COUNT

         <set quantifier> ::= DISTINCT | ALL


         Syntax Rules

         1) If <set quantifier> is not specified, then ALL is implicit.

         2) The argument of COUNT(*) and the argument source of a <general
            set function> is a table or a group of a grouped table as spec-
            ified in Subclause 7.8, "<having clause>", and Subclause 7.9,
            "<query specification>".

            Note: argument source is defined in Subclause 7.8, "<having
            clause>".

         3) Let T be the argument or argument source of a <set function
            specification>.

         4) The <value expression> simply contained in <set function spec-
            ification> shall not contain a <set function specification> or
            a <subquery>. If the <value expression> contains a <column ref-
            erence> that is an outer reference, then that outer reference
            shall be the only <column reference> contained in the <value
            expression>.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         5) If a <set function specification> contains a <column reference>
            that is an outer reference, then the <set function specifica-
            tion> shall be contained in either:

            a) a <select list>, or

         124  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                            6.5 <set function specification>


            b) a <subquery> of a <having clause>, in which case the scope
              of the explicit or implicit <qualifier> of the <column refer-
              ence> shall be a <table reference> that is directly contained
              in the <table expression> that directly contains the <having
              clause>.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         6) Let DT be the data type of the <value expression>.

         7) If COUNT is specified, then the data type of the result is exact
            numeric with implementation-defined precision and scale of 0.

         8) If MAX or MIN is specified, then the data type of the result is
            DT.

         9) If SUM or AVG is specified, then:

            a) DT shall not be character string, bit string, or datetime.

            b) If SUM is specified and DT is exact numeric with scale
              S, then the data type of the result is exact numeric with
              implementation-defined precision and scale S.

            c) If AVG is specified and DT is exact numeric, then the data
              type of the result is exact numeric with implementation-
              defined precision not less than the precision of DT and
              implementation-defined scale not less than the scale of DT.

            d) If DT is approximate numeric, then the data type of the
              result is approximate numeric with implementation-defined
              precision not less than the precision of DT.

            e) If DT is interval, then the data type of the result is inter-
              val with the same precision as DT.

         10)If the data type of the result is character string, then the
            collating sequence and the coercibility attribute are determined
            as in Subclause 4.2.3, "Rules determining collating sequence
            usage".

         Access Rules

            None.

         General Rules

         1) Case:

            a) If COUNT(*) is specified, then the result is the cardinality
              of T.


                                                    Scalar expressions   125

 





          X3H2-92-154/DBL CBR-002
         6.5 <set function specification>


            b) Otherwise, let TX be the single-column table that is the
              result of applying the <value expression> to each row of T
              and eliminating null values. If one or more null values are
              eliminated, then a completion condition is raised: warning-
              null value eliminated in set function.

         2) If DISTINCT is specified, then let TXA be the result of elimi-
            nating redundant duplicate values from TX. Otherwise, let TXA be
            TX.

            Case:

            a) If the <general set function> COUNT is specified, then the
              result is the cardinality of TXA.

            b) If AVG, MAX, MIN, or SUM is specified, then

              Case:

              i) If TXA is empty, then the result is the null value.

             ii) If AVG is specified, then the result is the average of the
                 values in TXA.

            iii) If MAX or MIN is specified, then the result is respec-
                 tively the maximum or minimum value in TXA. These results
                 are determined using the comparison rules specified in
                 Subclause 8.2, "<comparison predicate>".

             iv) If SUM is specified, then the result is the sum of the
                 values in TXA. If the sum is not within the range of the
                 data type of the result, then an exception condition is
                 raised: data exception-numeric value out of range.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) If a <general set function> specifies DISTINCT, then the
              <value expression> shall be a <column reference>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) If a <general set function> specifies or implies ALL, then
              COUNT shall not be specified.

            b) If a <general set function> specifies or implies ALL, then
              the <value expression> shall include a <column reference>
              that references a column of T.

            c) If the <value expression> contains a <column reference> that
              is an outer reference, then the <value expression> shall be a
              <column reference>.

         126  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                            6.5 <set function specification>


            d) No <column reference> contained in a <set function specifica-
              tion> shall reference a column derived from a <value expres-
              sion> that generally contains a <set function specification>.



















































                                                    Scalar expressions   127

 





          X3H2-92-154/DBL CBR-002
         6.6 <numeric value function>


         6.6  <numeric value function>

         Function

         Specify a function yielding a value of type numeric.

         Format

         <numeric value function> ::=
                <position expression>
              | <extract expression>
              | <length expression>

         <position expression> ::=
              POSITION <left paren> <character value expression>
                  IN <character value expression> <right paren>

         <length expression> ::=
                <char length expression>
              | <octet length expression>
              | <bit length expression>

         <char length expression> ::=
              { CHAR_LENGTH | CHARACTER_LENGTH }
                  <left paren> <string value expression> <right paren>

         <octet length expression> ::=
              OCTET_LENGTH <left paren> <string value expression> <right paren>


         <bit length expression> ::=
              BIT_LENGTH <left paren> <string value expression> <right paren>


         <extract expression> ::=
              EXTRACT <left paren> <extract field>
                  FROM <extract source> <right paren>

         <extract field> ::=
                <datetime field>
              | <time zone field>

         <time zone field> ::=
                TIMEZONE_HOUR
              | TIMEZONE_MINUTE

         <extract source> ::=
                <datetime value expression>
              | <interval value expression>





         128  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                6.6 <numeric value function>


         Syntax Rules

         1) If <position expression> is specified, then the character reper-
            toires of the two <character value expression>s shall be the
            same.

         2) If <position expression> is specified, then the data type of the
            result is exact numeric with implementation-defined precision
            and scale 0.

         3) If <extract expression> is specified, then

            Case:

            a) If <extract field> is a <datetime field>, then it shall iden-
              tify a <datetime field> of the <interval value expression> or
              <datetime value expression> immediately contained in <extract
              source>.

            b) If <extract field> is a <time zone field>, then the data
              type of the <extract source> shall be TIME WITH TIME ZONE or
              TIMESTAMP WITH TIME ZONE.

         4) If <extract expression> is specified, then

            Case:

            a) If <datetime field> does not specify SECOND, then the data
              type of the result is exact numeric with implementation-
              defined precision and scale 0.

            b) Otherwise, the data type of the result is exact numeric
              with implementation-defined precision and scale. The
              implementation-defined scale shall not be less than the spec-
              ified or implied <time fractional seconds precision> or <in-
              terval fractional seconds precision>, as appropriate, of the
              SECOND <datetime field> of the <extract source>.

         5) If a <length expression> is specified, then the data type of the
            result is exact numeric with implementation-defined precision
            and scale 0.

         Access Rules

            None.

         General Rules

         1) If <position expression> is specified and neither <character
            value expression> is the null value, then

            Case:

            a) If the first <character value expression> has a length of 0,
              then the result is 1.

                                                    Scalar expressions   129

 





          X3H2-92-154/DBL CBR-002
         6.6 <numeric value function>


            b) If the value of the first <character value expression> is
              equal to an identical-length substring of contiguous char-
              acters from the value of the second <character value ex-
              pression>, then the result is 1 greater than the number of
              characters within the value of the second <character value
              expression> preceding the start of the first such substring.

            c) Otherwise, the result is 0.

         2) If <position expression> is specified and either <character
            value expression> is the null value, then the result is the null
            value.

         3) If <extract expression> is specified, then

            Case:

            a) If the value of the <interval value expression> or the <date-
              time value expression> is not the null value, then

              Case:

              i) If <extract field> is a <datetime field>, then the re-
                 sult is the value of the datetime field identified by that
                 <datetime field> and has the same sign as the <extract
                 source>.

                 Note: If the value of the identified <datetime field> is
                 zero or if <extract source> is not an <interval value ex-
                 pression>, then the sign is irrelevant.

             ii) Otherwise, let TZ be the interval value of the implicit
                 or explicit time zone associated with the <datetime value
                 expression>. If <extract field> is TIMEZONE_HOUR, then the
                 result is calculated as

                   EXTRACT (HOUR FROM TZ)

                 Otherwise, the result is calculated as

                   EXTRACT (MINUTE FROM TZ)

            b) Otherwise, the result is the null value.

         4) If a <char length expression> is specified, then

            Case:

            a) Let S be the <string value expression>. If the value of S is
              not the null value, then

              Case:

              i) If the data type of S is a character data type, then the
                 result is the number of characters in the value of S.

             ii) Otherwise, the result is OCTET_LENGTH(S).

            b) Otherwise, the result is the null value.

         130  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                6.6 <numeric value function>


         5) If an <octet length expression> is specified, then

            Case:

            a) Let S be the <string value expression>. If the value of S is
              not the null value, then the result is the smallest integer
              not less than the quotient of the division (BIT_LENGTH(S)/8).

            b) Otherwise, the result is the null value.

         6) If a <bit length expression> is specified, then

            Case:

            a) Let S be the <string value expression>. If the value of S is
              not the null value, then the result is the number of bits in
              the value of S.

            b) Otherwise, the result is the null value.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) A <numeric value function> shall not be a <position expres-
              sion>.

            b) A <numeric value function> shall not contain a <length ex-
              pression> that is a <bit length expression>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <numeric value function> shall not be a <length expres-
              sion>.

            b) A <numeric value function> shall not be an <extract expres-
              sion>.
















                                                    Scalar expressions   131

 





          X3H2-92-154/DBL CBR-002
         6.7 <string value function>


         6.7  <string value function>

         Function

         Specify a function yielding a value of type character string or bit
         string.

         Format

         <string value function> ::=
                <character value function>
              | <bit value function>

         <character value function> ::=
                <character substring function>
              | <fold>
              | <form-of-use conversion>
              | <character translation>
              | <trim function>

         <character substring function> ::=
              SUBSTRING <left paren> <character value expression> FROM <start position>

                          [ FOR <string length> ] <right paren>

         <fold> ::= { UPPER | LOWER } <left paren> <character value expression> <right paren>


         <form-of-use conversion> ::=
              CONVERT <left paren> <character value expression>
                  USING <form-of-use conversion name> <right paren>

         <character translation> ::=
              TRANSLATE <left paren> <character value expression>
                  USING <translation name> <right paren>

         <trim function> ::=
              TRIM <left paren> <trim operands> <right paren>

         <trim operands> ::=
              [ [ <trim specification> ] [ <trim character> ] FROM ] <trim source>


         <trim source> ::= <character value expression>

         <trim specification> ::=
                LEADING
              | TRAILING
              | BOTH

         <trim character> ::= <character value expression>

         <bit value function> ::=

         132  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 6.7 <string value function>


              <bit substring function>

         <bit substring function> ::=
              SUBSTRING <left paren> <bit value expression> FROM <start position>

                  [ FOR <string length> ] <right paren>

         <start position> ::= <numeric value expression>

         <string length> ::= <numeric value expression>


         Syntax Rules

         1) The data type of a <start position> and <string length> shall be
            exact numeric with scale 0.

         2) If <character substring function> is specified, then:

            a) The data type of the <character substring function> is
              variable-length character string with maximum length equal to
              the fixed length or maximum variable length of the <character
              value expression>. The character repertoire and form-of-use
              of the <character substring function> are the same as the
              character repertoire and form-of-use of the <character value
              expression>.

            b) The collating sequence and the coercibility attribute
              are determined as specified for monadic operators in
              Subclause 4.2.3, "Rules determining collating sequence us-
              age", where the first operand of SUBSTRING plays the role of
              the monadic operand.

         3) If <fold> is specified, then:

            a) The data type of the result of <fold> is the data type of the
              <character value expression>.

            b) The collating sequence and the coercibility attribute
              are determined as specified for monadic operators in
              Subclause 4.2.3, "Rules determining collating sequence us-
              age", where the operand of the <fold> is the monadic operand.

         4) If <form-of-use conversion> is specified, then:

            a) A <form-of-use conversion name> shall identify a form-of-use
              conversion.

            b) The data type of the result is variable-length character
              string with implementation-defined maximum length. The
              character set of the result is the same as the character
              repertoire of the <character value expression> and form-
              of-use determined by the form-of-use conversion identified

                                                    Scalar expressions   133

 





          X3H2-92-154/DBL CBR-002
         6.7 <string value function>


              by the <form-of-use conversion name>. Let CR be that char-
              acter repertoire. The result has the Implicit coercibility
              attribute and its collating sequence is X, where X is the
              default collating sequence of CR.

         5) If <character translation> is specified, then

            a) A <translation name> shall identify a character translation.

            b) The data type of the <character translation> is variable-
              length character string with implementation-defined maximum
              length and character repertoire equal to the character reper-
              toire of the target character set of the translation. Let CR
              be that character repertoire. The result has the Implicit co-
              ercibility attribute and its collating sequence is X, where X
              is the default collating sequence of CR.

         6) If <trim function> is specified, then

            a) If FROM is specified, then either <trim specification> or
              <trim character> or both shall be specified.

            b) If <trim specification> is not specified, then BOTH is im-
              plicit.

            c) If <trim character> is not specified, then ' ' is implicit.

            d) If

                 TRIM ( SRC )

              is specified, then

                 TRIM ( BOTH ' ' FROM SRC )

              is implicit.

            e) The data type of the <trim function> is variable-length char-
              acter string with maximum length equal to the fixed length or
              maximum variable length of the <trim source>.

            f) If a <trim character> is specified, then <trim character> and
              <trim source> shall be comparable.

            g) The character repertoire and form-of-use of the <trim func-
              tion> are the same as those of the <trim source>.

            h) The collating sequence and the coercibility attribute
              are determined as specified for monadic operators in
              Subclause 4.2.3, "Rules determining collating sequence us-
              age", where the <trim source> of TRIM plays the role of the
              monadic operand.

         7) If <bit substring function> is specified, then the data type of
            the <bit substring function> is variable-length bit string with
            maximum length equal to the fixed length or maximum variable
            length of the <bit value expression>.

         134  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 6.7 <string value function>


         Access Rules

         1) The applicable privileges shall include USAGE for every <trans-
            lation name> contained in the <string value expression>.

         General Rules

         1) If <character substring function> is specified, then:

            a) Let C be the value of the <character value expression>, let
              LC be the length of C, and let S be the value of the <start
              position>.

            b) If <string length> is specified, then let L be the value of
              <string length> and let E be S+L. Otherwise, let E be the
              larger of LC + 1 and S.

            c) If either C, S, or L is the null value, then the result of
              the <character substring function> is the null value.

            d) If E is less than S, then an exception condition is raised:
              data exception-substring error.

            e) Case:

              i) If S is greater than LC or if E is less than 1, then the
                 result of the <character substring function> is a zero-
                 length string.

             ii) Otherwise,

                 1) Let S1 be the larger of S and 1. Let E1 be the smaller
                   of E and LC+1. Let L1 be E1-S1.

                 2) The result of the <character substring function> is
                   a character string containing the L1 characters of C
                   starting at character number S1 in the same order that
                   the characters appear in C.

         2) If <fold> is specified, then:

            a) Let S be the value of the <character value expression>.

            b) If S is the null value, then the result of the <fold> is the
              null value.

            c) Case:

              i) If UPPER is specified, then the result of the <fold> is a
                 copy of S in which every <simple Latin lower case letter>
                 that has a corresponding <simple Latin upper case let-
                 ter> in the character repertoire of S is replaced by that
                 <simple Latin upper case letter>.

                                                    Scalar expressions   135

 





          X3H2-92-154/DBL CBR-002
         6.7 <string value function>


             ii) If LOWER is specified, then the result of the <fold> is a
                 copy of S in which every <simple Latin upper case letter>
                 that has a corresponding <simple Latin lower case let-
                 ter> in the character repertoire of S is replaced by that
                 <simple Latin lower case letter>.

         3) If a <character translation> is specified, then

            Case:

            a) If the value of <character value expression> is the null
              value, then the result of the <character translation> is the
              null value.

            b) Otherwise, the value of the <character translation> is the
              value of the <character value expression> after translation
              to the character repertoire of the target character set of
              the translation.

         4) If a <form-of-use conversion> is specified, then

            Case:

            a) If the value of <character value expression> is the null
              value, then the result of the <form-of-use conversion> is the
              null value.

            b) Otherwise, the value of the <form-of-use conversion> is the
              value of the <character value expression> after the applica-
              tion of the form-of-use conversion specified by <form-of-use
              conversion name>.

         5) If <trim function> is specified, then:

            a) Let S be the value of the <trim source>.

            b) If <trim character> is specified, then let SC be the value of
              <trim character>; otherwise, let SC be <space>.

            c) If either S or SC is the null value, then the result of the
              <trim function> is the null value.

            d) If the length in characters of SC is not 1, then an exception
              condition is raised: data exception-trim error.

            e) Case:

              i) If BOTH is specified or if no <trim specification> is spec-
                 ified, then the result of the <trim function> is the value
                 of S with any leading or trailing characters equal to SC
                 removed.

             ii) If TRAILING is specified, then the result of the <trim
                 function> is the value of S with any trailing characters
                 equal to SC removed.

         136  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 6.7 <string value function>


            iii) If LEADING is specified, then the result of the <trim func-
                 tion> is the value of S with any leading characters equal
                 to SC removed.

         6) If <bit substring function> is specified, then:

            a) Let B be the value of the <bit value expression>, let LB be
              the length in bits of B, and let S be the value of the <start
              position>.

            b) If <string length> is specified, then let L be the value of
              <string length> and let E be S+L. Otherwise, let E be the
              larger of LB + 1 and S.

            c) If either B, S, or L is the null value, then the result of
              the <bit substring function> is the null value.

            d) If E is less than S, then an exception condition is raised:
              data exception-substring error.

            e) Case:

              i) If S is greater than LB or if E is less than 1, then the
                 result of the <bit substring function> is a zero-length
                 string.

             ii) Otherwise,

                 1) Let S1 be the larger of S and 1. Let E1 be the smaller
                   of E and LB+1. Let L1 be E1-S1.

                 2) The result of the <bit substring function> is a bit
                   string containing L1 bits of B starting at bit number S1
                   in the same order that the bits appear in B.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) A <character value function> shall not be a <fold>.

            b) Conforming Intermediate SQL language shall contain no <char-
              acter translation>.

            c) Conforming Intermediate SQL language shall contain no <form-
              of-use conversion>.

            d) Conforming Intermediate SQL language shall contain no <bit
              value function>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <character value function> shall not be a <character sub-
              string function>.

                                                    Scalar expressions   137

 





          X3H2-92-154/DBL CBR-002
         6.7 <string value function>


            b) A <character value function> shall not be a <trim function>.





















































         138  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                               6.8 <datetime value function>


         6.8  <datetime value function>

         Function

         Specify a function yielding a value of type datetime.

         Format

         <datetime value function> ::=
                <current date value function>
              | <current time value function>
              | <current timestamp value function>

         <current date value function> ::= CURRENT_DATE

         <current time value function> ::=
                CURRENT_TIME [ <left paren> <time precision> <right paren> ]


         <current timestamp value function> ::=
                CURRENT_TIMESTAMP [ <left paren> <timestamp precision> <right paren> ]



         Syntax Rules

         1) The data type of a <current date value function> is DATE. The
            data type of a <current time value function> is TIME WITH TIME
            ZONE. The data type of a <current timestamp value function> is
            TIMESTAMP WITH TIME ZONE.

            Note: See the Syntax Rules of Subclause 6.1, "<data type>", for
            rules governing <time precision> and <timestamp precision>.

         Access Rules

            None.

         General Rules

         1) The <datetime value function>s CURRENT_DATE, CURRENT_TIME, and
            CURRENT_TIMESTAMP respectively return the current date, current
            time, and current timestamp; the time and timestamp values are
            returned with time zone displacement equal to the current time
            zone displacement of the SQL-session.

         2) If specified, <time precision> and <timestamp precision> respec-
            tively determine the precision of the time or timestamp value
            returned.

         3) If an SQL-statement generally contains more than one reference
            to one or more <datetime value function>s, then all such ref-
            erences are effectively evaluated simultaneously. The time of

                                                    Scalar expressions   139

 





          X3H2-92-154/DBL CBR-002
         6.8 <datetime value function>


            evaluation of the <datetime value function> during the execution
            of the SQL-statement is implementation-dependent.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) Conforming Intermediate SQL language shall contain no <time
              precision> or <timestamp precision>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <datetime
              value function>.







































         140  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       6.9 <case expression>


         6.9  <case expression>

         Function

         Specify a conditional value.

         Format

         <case expression> ::=
                <case abbreviation>
              | <case specification>

         <case abbreviation> ::=
                NULLIF <left paren> <value expression> <comma>
                      <value expression> <right paren>
              | COALESCE <left paren> <value expression>
                      { <comma> <value expression> }... <right paren>

         <case specification> ::=
                <simple case>
              | <searched case>

         <simple case> ::=
              CASE <case operand>
                <simple when clause>...
                [ <else clause> ]
              END

         <searched case> ::=
              CASE
                <searched when clause>...
                [ <else clause> ]
              END

         <simple when clause> ::= WHEN <when operand> THEN <result>

         <searched when clause> ::= WHEN <search condition> THEN <result>

         <else clause> ::= ELSE <result>

         <case operand> ::= <value expression>

         <when operand> ::= <value expression>

         <result> ::= <result expression> | NULL

         <result expression> ::= <value expression>







                                                    Scalar expressions   141

 





          X3H2-92-154/DBL CBR-002
         6.9 <case expression>


         Syntax Rules

         1) NULLIF (V1, V2) is equivalent to the following <case specifica-
            tion>:

              CASE WHEN V1=V2 THEN NULL ELSE V1 END

         2) COALESCE (V1, V2) is equivalent to the following <case specifi-
            cation>:

              CASE WHEN V1 IS NOT NULL THEN V1 ELSE V2 END

         3) COALESCE (V1, V2, . . . ,n ), for n >= 3, is equivalent to the
            following <case specification>:

              CASE WHEN V1 IS NOT NULL THEN V1 ELSE COALESCE (V2, . . . ,n )
              END

         4) If a <case specification> specifies a <simple case>, then let CO
            be the <case operand>:

            a) The data type of each <when operand> WO shall be comparable
              with the data type of the <case operand>.

            b) The <case specification> is equivalent to a <searched case>
              in which each <searched when clause> specifies a <search
              condition> of the form "CO=WO".

         5) At least one <result> in a <case specification> shall specify a
            <result expression>.

         6) If an <else clause> is not specified, then ELSE NULL is im-
            plicit.

         7) The data type of a <case specification> is determined by ap-
            plying Subclause 9.3, "Set operation result data types", to the
            data types of all <result expression>s in the <case specifica-
            tion>.

         Access Rules

            None.

         General Rules

         1) Case:

            a) If a <result> specifies NULL, then its value is the null
              value.

            b) If a <result> specifies a <value expression>, then its value
              is the value of that <value expression>.


         142  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       6.9 <case expression>


         2) Case:

            a) If the <search condition> of some <searched when clause> in
              a <case specification> is true, then the value of the <case
              specification> is the value of the <result> of the first
              (leftmost) <searched when clause> whose <search condition> is
              true, cast as the data type of the <case specification>.

            b) If no <search condition> in a <case specification> is true,
              then the value of the <case expression> is the value of the
              <result> of the explicit or implicit <else clause>, cast as
              the data type of the <case specification>.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <case
              expression>.






























                                                    Scalar expressions   143

 





          X3H2-92-154/DBL CBR-002
         6.10 <cast specification>


         6.10  <cast specification>

         Function

         Specify a data conversion.

         Format

         <cast specification> ::=
              CAST <left paren> <cast operand> AS <cast target> <right paren>


         <cast operand> ::=
                <value expression>
              | NULL

         <cast target> ::=
                <domain name>
              | <data type>


         Syntax Rules

         1) Case:

            a) If a <domain name> is specified, then let TD be the <data
              type> of the specified domain.

            b) If a <data type> is specified, then let TD be the specified
              <data type>.

         2) The data type of the result of the <cast specification> is TD.

         3) If the <cast operand> is a <value expression>, then let SD be
            the underlying data type of the <value expression>.

         4) If the <cast operand> is a <value expression>, then the valid
            combinations of TD and SD in a <cast specification> are given by
            the following table. Y indicates that the combination is syntac-
            tically valid without restriction; M indicates that the combi-
            nation is valid subject to other syntax rules in this Subclause
            being satisfied; and N indicates that the combination is not
            valid:

              <data type>
              SD of                 <data type> of TD
              <value
              expression>   EN   AN   VC   FC   VB   FB   D    T    TS   YM   DT

                  EN        Y    Y    Y    Y    N    N    N    N    N    M    M
                  AN        Y    Y    Y    Y    N    N    N    N    N    N    N
                  C         Y    Y    M    M    Y    Y    Y    Y    Y    Y    Y
                  B         N    N    Y    Y    Y    Y    N    N    N    N    N
                  D         N    N    Y    Y    N    N    Y    N    Y    N    N

         144  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   6.10 <cast specification>


                  T         N    N    Y    Y    N    N    N    Y    Y    N    N
                  TS        N    N    Y    Y    N    N    Y    Y    Y    N    N
                  YM        M    N    Y    Y    N    N    N    N    N    Y    N
                  DT        M    N    Y    Y    N    N    N    N    N    N    Y

              Where:

                  EN = Exact Numeric
                  AN = Approximate Numeric
                  C  = Character (Fixed- or Variable-length)
                  FC = Fixed-length Character
                  VC = Variable-length Character
                  B  = Bit String (Fixed- or Variable-length)
                  FB = Fixed-length Bit String
                  VB = Variable-length Bit String
                  D  = Date
                  T  = Time
                  TS = Timestamp
                  YM = Year-Month Interval
                  DT = Day-Time Interval

         5) If TD is an interval and SD is exact numeric, then TD shall
            contain only a single <datetime field>.

         6) If TD is exact numeric and SD is an interval, then SD shall
            contain only a single <datetime field>.

         7) If SD is character string and TD is fixed-length or variable-
            length character string, then the character repertoires of SD
            and TD shall be the same.

         8) If TD is a fixed-length or variable-length character string,
            then the collating sequence of the result of the <cast speci-
            fication> is the default collating sequence for the character
            repertoire of TD and the result of the <cast specification> has
            the Coercible coercibility attribute.

         Access Rules

         1) If <domain name> is specified, then the applicable privileges
            shall include USAGE.

         General Rules

         1) If the <cast operand> is a <value expression>, then let SV be
            its value.

         2) Case:

            a) If the <cast operand> specifies NULL or if SV is the null
              value, then the result of the <cast specification> is the
              null value.


                                                    Scalar expressions   145

 





          X3H2-92-154/DBL CBR-002
         6.10 <cast specification>


            b) Otherwise, let TV be the result of the <cast specifica-
              tion> as specified in the remaining General Rules of this
              Subclause.

         3) If TD is exact numeric, then

            Case:

            a) If SD is exact numeric or approximate numeric, then

              Case:

              i) If there is a representation of SV in the data type TD
                 that does not lose any leading significant digits after
                 rounding or truncating if necessary, then TV is that rep-
                 resentation. The choice of whether to round or truncate is
                 implementation-defined.

             ii) Otherwise, an exception condition is raised: data exception-
                 numeric value out of range.

            b) If SD is character string, then SV is replaced by SV with any
              leading or trailing <space>s removed.

              Case:

              i) If SV does not comprise a <signed numeric literal> as
                 defined by the rules for <literal> in Subclause 5.3,
                 "<literal>", then an exception condition is raised: data
                 exception-invalid character value for cast.

             ii) Otherwise, let LT be that <signed numeric literal>. The
                 <cast specification> is equivalent to

                   CAST ( LT AS TD )

            c) If SD is an interval data type, then

              Case:

              i) If there is a representation of SV in the data type TD that
                 does not lose any leading significant digits, then TV is
                 that representation.

             ii) Otherwise, an exception condition is raised: data exception-
                 numeric value out of range.

         4) If TD is approximate numeric, then

            Case:

            a) If SD is exact numeric or approximate numeric, then

              Case:

              i) If there is a representation of SV in the data type TD
                 that does not lose any leading significant digits after

         146  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   6.10 <cast specification>


                 rounding or truncating if necessary, then TV is that rep-
                 resentation. The choice of whether to round or truncate is
                 implementation-defined.

             ii) Otherwise, an exception condition is raised: data exception-
                 numeric value out of range.

            b) If SD is character string, then SV is replaced by SV with any
              leading or trailing <space>s removed.

              Case:

              i) If SV does not comprise a <signed numeric literal> as
                 defined by the rules for <literal> in Subclause 5.3,
                 "<literal>", then an exception condition is raised: data
                 exception-invalid character value for cast.

             ii) Otherwise, let LT be that <signed numeric literal>. The
                 <cast specification> is equivalent to

                   CAST ( LT AS TD )

         5) If TD is fixed-length character string, then let LTD be the
            length in characters of TD.

            Case:

            a) If SD is exact numeric, then let YP be the shortest character
              string that conforms to the definition of <exact numeric
              literal> in Subclause 5.3, "<literal>", whose scale is the
              same as the scale of SD and whose interpreted value is the
              absolute value of SV.

              If SV is less than 0, then let Y be the result of

                 '-' | YP

              Otherwise, let Y be YP.

              Case:

              i) If Y contains any <SQL language character> that is not
                 in the repertoire of TD, then an exception condition is
                 raised: data exception-invalid character value for cast.

             ii) If the length in characters LY of Y is equal to LTD, then
                 TV is Y.

            iii) If the length in characters LY of Y is less than LTD, then
                 TV is Y extended on the right by LTD-LY <space>s.

             iv) Otherwise, an exception condition is raised: data exception-
                 string data, right truncation.

            b) If SD is approximate numeric, then:

              i) Let YP be a character string as follows:

                                                    Scalar expressions   147

 





          X3H2-92-154/DBL CBR-002
         6.10 <cast specification>


                 Case:

                 1) If SV equals 0, then YP is '0E0'.

                 2) Otherwise, YP is the shortest character string that con-
                   forms to the definition of <approximate numeric literal>
                   in Subclause 5.3, "<literal>", whose interpreted value
                   is equal to the absolute value of SV and whose <man-
                   tissa> consists of a single <digit> that is not '0',
                   followed by a <period> and an <unsigned integer>.

             ii) If SV is less than 0, then let Y be the result of

                   '-' | YP

                 otherwise, let Y be YP.

            iii) Case:

                 1) If Y contains any <SQL language character> that is not
                   in the repertoire of TD, then an exception condition is
                   raised: data exception-invalid character value for cast.

                 2) If the length in characters LY of Y is equal to LTD,
                   then TV is Y.

                 3) If the length in characters LY of Y is less than LTD,
                   then TV is Y extended on the right by LTD-LY <space>s.

                 4) Otherwise, an exception condition is raised: data
                   exception-string data, right truncation.

            c) If SD is fixed-length character string or variable-length
              character string, then

              Case:

              i) If the length in characters of SV is equal to LTD, then TV
                 is SV.

             ii) If the length in characters of SV is larger than LTD, then
                 TV is the first LTD characters of SV. If any of the re-
                 maining characters of SV are non-<space> characters, then a
                 completion condition is raised: warning-string data, right
                 truncation.

            iii) If the length in characters M of SV is smaller than LTD,
                 then TV is SV extended on the right by LTD-M <space>s.

            d) If SD is fixed-length bit string or variable-length bit
              string, then let LSV be the value of BIT_LENGTH(SV) and let
              B be the BIT_LENGTH of the character with the smallest BIT_
              LENGTH in the form-of-use of TD. Let PAD be the value of the
              remainder of the division LSV/B. Let NC be a character whose
              bits all have the value 0.

         148  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   6.10 <cast specification>


              If PAD is not 0, then append (B - PAD) 0-valued bits to
              the least significant end of SV; a completion condition is
              raised: warning-implicit zero-bit padding.

              Let SVC be the possibly padded value of SV expressed as a
              character string without regard to valid character encodings
              and let LTDS be a character string of LTD characters of value
              NC characters in the form-of-use of TD.

              TV is the result of

                 SUBSTRING (SVC | LTDS FROM 1 FOR LTD)

              Case:

              i) If the length of TV is less than the length of SVC, then a
                 completion condition is raised: warning-string data, right
                 truncation.

             ii) If the length of TV is greater than the length of SVC, then
                 a completion condition is raised: warning-implicit zero-bit
                 padding.

            e) If SD is a datetime data type or an interval data type, then
              let Y be the shortest character string that conforms to the
              definition of <literal> in Subclause 5.3, "<literal>", and
              such that the interpreted value of Y is SV and the inter-
              preted precision of Y is the precision of SD.

              Case:

              i) If Y contains any <SQL language character> that is not
                 in the repertoire of TD, then an exception condition is
                 raised: data exception-invalid character value for cast.

             ii) If the length in characters LY of Y is equal to LTD, then
                 TV is Y.

            iii) If the length in characters LY of Y is less than LTD, then
                 TV is Y extended on the right by LTD-LY <space>s.

             iv) Otherwise, an exception condition is raised: data exception-
                 string data, right truncation.

         6) If TD is variable-length character string, then let MLTD be the
            maximum length in characters of TD.

            Case:

            a) If SD is exact numeric, then let YP be the shortest character
              string that conforms to the definition of <exact numeric
              literal> in Subclause 5.3, "<literal>", whose scale is the
              same as the scale of SD and whose interpreted value is the
              absolute value of SV.

              If SV is less than 0, then let Y be the result of

                 '-' | YP

                                                    Scalar expressions   149

 





          X3H2-92-154/DBL CBR-002
         6.10 <cast specification>


              Otherwise, let Y be YP.

              Case:

              i) If Y contains any <SQL language character> that is not
                 in the repertoire of TD, then an exception condition is
                 raised: data exception-invalid character value for cast.

             ii) If the length in characters LY of Y is less than or equal
                 to MLTD, then TV is Y.

            iii) Otherwise, an exception condition is raised: data exception-
                 string data, right truncation.

            b) If SD is approximate numeric, then

              i) Let YP be a character string as follows:

                 Case:

                 1) If SV equals 0, then YP is '0E0'.

                 2) Otherwise, YP is the shortest character string that con-
                   forms to the definition of <approximate numeric literal>
                   in Subclause 5.3, "<literal>", whose interpreted value
                   is equal to the absolute value of SV and whose <man-
                   tissa> consists of a single <digit> that is not '0',
                   followed by a <period> and an <unsigned integer>.

             ii) If SV is less than 0, then let Y be the result of

                   '-' | YP

                 otherwise, let Y be YP.

            iii) Case:

                 1) If Y contains any <SQL language character> that is not
                   in the repertoire of TD, then an exception condition is
                   raised: data exception-invalid character value for cast.

                 2) If the length in characters LY of Y is less than or
                   equal to MLTD, then TV is Y.

                 3) Otherwise, an exception condition is raised: data
                   exception-string data, right truncation.

            c) If SD is fixed-length character string or variable-length
              character string, then

              Case:

              i) If the length in characters of SV is less than or equal to
                 MLTD, then TV is SV.

         150  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   6.10 <cast specification>


             ii) If the length in characters of SV is larger than MLTD,
                 then TV is the first MLTD characters of SV. If any of the
                 remaining characters of SV are non-<space> characters,
                 then a completion condition is raised: warning-string data,
                 right truncation.

            d) If SD is fixed-length bit string or variable-length bit
              string, then let LSV be the value of BIT_LENGTH(SV) and let
              B be the BIT_LENGTH of the character with the smallest BIT_
              LENGTH in the form-of-use of TD. Let PAD be the value of the
              remainder of the division LSV/B.

              If PAD is not 0, then append (B - PAD) 0-valued bits to
              the least significant end of SV; a completion condition is
              raised: warning-implicit zero-bit padding.

              Let SVC be the possible padded value of SV expressed as a
              character string without regard to valid character encodings.

              Case:

              i) If CHARACTER_LENGTH (SVC) is not greater than MLTD, then TV
                 is SVC.

             ii) Otherwise, TV is the result of

                   SUBSTRING (SVC FROM 1 FOR MLTD)

                 If the length of TV is less than the length of SVC, then a
                 completion condition is raised: warning-string data, right
                 truncation.

            e) If SD is a datetime data type or an interval data type then
              let Y be the shortest character string that conforms to the
              definition of <literal> in Subclause 5.3, "<literal>", and
              such that the interpreted value of Y is SV and the inter-
              preted precision of Y is the precision of SD.

              Case:

              i) If Y contains any <SQL language character> that is not
                 in the repertoire of TD, then an exception condition is
                 raised: data exception-invalid character value for cast.

             ii) If the length in characters LY of Y is less than or equal
                 to MLTD, then TV is Y.

            iii) Otherwise, an exception condition is raised: data exception-
                 string data, right truncation.

         7) If TD is fixed-length bit string, then let LTD be the length in
            bits of TD. Let BLSV be the result of BIT_LENGTH(SV).

            Case:

            a) If BLSV is equal to LTD, then TV is SV expressed as a bit
              string with a length in bits of BLSV.

                                                    Scalar expressions   151

 





          X3H2-92-154/DBL CBR-002
         6.10 <cast specification>


            b) If BLSV is larger than LTD, then TV is the first LTD bits of
              SV expressed as a bit string with a length in bits of LTD,
              and a completion condition is raised: warning-string data,
              right truncation.

            c) If BLSV is smaller than LTD, then TV is SV expressed as a bit
              string extended on the right with LTD-BLSV bits whose values
              are all 0 and a completion condition is raised: warning-
              implicit zero-bit padding.

         8) If TD is variable-length bit string, then let MLTD be the
            maximum length in bits of TD. Let BLSV be the result of BIT_
            LENGTH(SV).

            Case:

            a) If BLSV is less than or equal to MLTD, then TV is SV ex-
              pressed as a bit string with a length in bits of BLSV.

            b) If BLSV is larger than MLTD, then TV is the first MLTD bits
              of SV expressed as a bit string with a length in bits of MLTD
              and a completion condition is raised: warning-string data,
              right truncation.

         9) If TD is the datetime data type DATE, then

            Case:

            a) If SD is character string, then SV is replaced by

                 TRIM ( BOTH ' ' FROM SV )

              Case:

              i) If the rules for <literal> in Subclause 5.3, "<literal>",
                 can be applied to SV to determine a valid value of the data
                 type TD, then let TV be that value.

             ii) Otherwise, an exception condition is raised: data exception-
                 invalid character value for cast.

            b) If SD is a date, then TV is SV.

            c) If SD is a timestamp, then TV is the year, month, and day
              <datetime field>s of SV adjusted to the implicit or explicit
              time zone displacement of SV.

         10)If TD is the datetime data type TIME, then

            Case:

            a) If SD is character string, then SV is replaced by

                 TRIM ( BOTH ' ' FROM SV )

              Case:

         152  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   6.10 <cast specification>


              i) If the rules for <literal> in Subclause 5.3, "<literal>",
                 can be applied to SV to determine a valid value of the data
                 type TD, then let TV be that value.

             ii) Otherwise, an exception condition is raised: data exception-
                 invalid character value for cast.

            b) If SD is a time, then TV is SV. If TD is specified WITH TIME
              ZONE, then TV also includes the implicit or explicit time
              zone displacement of SV; otherwise, TV is adjusted to the
              current time zone displacement of the SQL-session.

            c) If SD is a timestamp, then TV is the hour, minute, and second
              <datetime field>s of SV. If TD is specified WITH TIME ZONE,
              then TV also includes the implicit or explicit time zone
              displacement of SV; otherwise, TV is adjusted to the current
              time zone displacement of the SQL-session.

         11)If TD is the datetime data type TIMESTAMP, then

            Case:

            a) If SD is character string, then SV is replaced by

                 TRIM ( BOTH ' ' FROM SV )

              Case:

              i) If the rules for <literal> in Subclause 5.3, "<literal>",
                 can be applied to SV to determine a valid value of the data
                 type TD, then let TV be that value.

             ii) Otherwise, an exception condition is raised: data exception-
                 invalid character value for cast.

            b) If SD is a date, then the <datetime field>s hour, minute,
              and second of TV are set to 0 and the <datetime field>s year,
              month, and day of TV are set to their respective values in
              SV. If TD is specified WITH TIME ZONE, then the time zone
              fields of TV are set to the current time zone displacement of
              the SQL-session.

            c) If SD is a time, then the <datetime field>s year, month, and
              day of TV are set to their respective values in an execution
              of CURRENT_DATE and the <datetime field>s hour, minute, and
              second of TV are set to their respective values in SV. If TD
              is specified WITH TIME ZONE, then the time zone fields of TV
              are set to the explicit or implicit time zone interval of SV.

            d) If SD is a timestamp, then TV is SV.

         12)If TD is interval, then

            Case:

            a) If SD is exact numeric, then

                                                    Scalar expressions   153

 





          X3H2-92-154/DBL CBR-002
         6.10 <cast specification>


              Case:

              i) If the representation of SV in the data type TD would re-
                 sult in the loss of leading significant digits, then an
                 exception condition is raised: data exception-interval
                 field overflow.

             ii) Otherwise, TV is that representation.

            b) If SD is character string, then SV is replaced by

                 TRIM ( BOTH ' ' FROM SV )

              Case:

              i) If the rules for <literal> in Subclause 5.3, "<literal>",
                 can be applied to SV to determine a valid value of the data
                 type TD, then let TV be that value.

             ii) Otherwise, an exception condition is raised: data exception-
                 invalid character value for cast.

            c) If SD is interval and TD and SD have the same interval preci-
              sion, then TV is SV.

            d) If SD is interval and TD and SD have different interval pre-
              cisions, then let Q be the least significant <datetime field>
              of TD.

              i) Let Y be the result of converting SV to a scalar in units Q
                 according to the natural rules for intervals as defined in
                 the Gregorian calendar.

             ii) Normalize Y to conform to the datetime qualifier "P TO Q"
                 of TD. If this would result in loss of precision of the
                 leading datetime field of Y, then an exception condition is
                 raised: data exception-interval field overflow.

            iii) TV is the value of Y.

         13)If the <cast specification> contains a <domain name> and that
            <domain name> refers to a domain that contains a <domain con-
            straint> and if TV does not satisfy the <check constraint> of
            the <domain constraint>, then an exception condition is raised:
            integrity constraint violation.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <cast
              specification>.

         154  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                     6.11 <value expression>


         6.11  <value expression>

         Function

         Specify a value.

         Format

         <value expression> ::=
                <numeric value expression>
              | <string value expression>
              | <datetime value expression>
              | <interval value expression>

         <value expression primary> ::=
                <unsigned value specification>
              | <column reference>
              | <set function specification>
              | <scalar subquery>
              | <case expression>
              | <left paren> <value expression> <right paren>
              | <cast specification>


         Syntax Rules

         1) The data type of a <value expression> is the data type of the
            <numeric value expression>, <string value expression>, <datetime
            value expression>, or <interval value expression>, respectively.

         2) If the data type of a <value expression primary> is character
            string, then the collating sequence and coercibility attribute
            of the <value expression primary> are the collating sequence and
            coercibility attribute of the <unsigned value specification>,
            <column reference>, <set function specification>, <scalar sub-
            query>, <case expression>, <value expression>, or <cast specifi-
            cation> immediately contained in the <value expression primary>.

         3) Let C be some column. Let VE be the <value expression>. C is an
            underlying column of VE if and only if C is identified by some
            <column reference> contained in VE.

         Access Rules

            None.









                                                    Scalar expressions   155

 





          X3H2-92-154/DBL CBR-002
         6.11 <value expression>


         General Rules

         1) When a <value expression> V is evaluated for a row of a table,
            each reference to a column of that table by a <column reference>
            directly contained in V is a reference to the value of that
            column in that row.

         2) If a <value expression primary> is a <scalar subquery> and the
            result of the <subquery> is empty, then the result of the <value
            expression primary> is the null value.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <value expression> shall not be a <datetime value expres-
              sion>.

            b) A <value expression> shall not be an <interval value expres-
              sion>.

            c) A <value expression primary> shall not be a <case expres-
              sion>.

            d) A <value expression primary> shall not be a <cast specifica-
              tion>.

            e) A <value expression primary> shall not be a <scalar subquery>
              except when the <value expression primary> is simply con-
              tained in a <value expression> that is simply contained in
              the second <row value constructor> of a <comparison predi-
              cate>.

















         156  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                             6.12 <numeric value expression>


         6.12  <numeric value expression>

         Function

         Specify a numeric value.

         Format

         <numeric value expression> ::=
                <term>
              | <numeric value expression> <plus sign> <term>
              | <numeric value expression> <minus sign> <term>

         <term> ::=
                <factor>
              | <term> <asterisk> <factor>
              | <term> <solidus> <factor>

         <factor> ::=
              [ <sign> ] <numeric primary>

         <numeric primary> ::=
                <value expression primary>
              | <numeric value function>


         Syntax Rules

         1) If the data type of both operands of a dyadic arithmetic opera-
            tor is exact numeric, then the data type of the result is exact
            numeric, with precision and scale determined as follows:

            a) Let S1 and S2 be the scale of the first and second operands
              respectively.

            b) The precision of the result of addition and subtraction is
              implementation-defined, and the scale is the maximum of S1
              and S2.

            c) The precision of the result of multiplication is implementation-
              defined, and the scale is S1 + S2.

            d) The precision and scale of the result of division is
              implementation-defined.

         2) If the data type of either operand of a dyadic arithmetic op-
            erator is approximate numeric, then the data type of the re-
            sult is approximate numeric. The precision of the result is
            implementation-defined.

         3) The data type of a <factor> is that of the immediately contained
            <numeric primary>.


                                                    Scalar expressions   157

 





          X3H2-92-154/DBL CBR-002
         6.12 <numeric value expression>


         4) The data type of a <numeric primary> shall be numeric.

         Access Rules

            None.

         General Rules

         1) If the value of any <numeric primary> simply contained in a
            <numeric value expression> is the null value, then the result of
            the <numeric value expression> is the null value.

         2) If the <numeric value expression> contains only a <numeric pri-
            mary>, then the value of the <numeric value expression> is the
            value of the specified <numeric primary>.

         3) The monadic arithmetic operators <plus sign> and <minus sign>
            (+ and -, respectively) specify monadic plus and monadic minus,
            respectively. Monadic plus does not change its operand. Monadic
            minus reverses the sign of its operand.

         4) The dyadic arithmetic operators <plus sign>, <minus sign>, <as-
            terisk>, and <solidus> (+, -, *, and /, respectively) specify
            addition, subtraction, multiplication, and division, respec-
            tively. If the value of a divisor is zero, then an exception
            condition is raised: data exception-division by zero.

         5) If the type of the result of an arithmetic operation is exact
            numeric, then

            Case:

            a) If the operator is not division and the mathematical result
              of the operation is not exactly representable with the pre-
              cision and scale of the result type, then an exception con-
              dition is raised: data exception-numeric value out of range.

            b) If the operator is division and the approximate mathemati-
              cal result of the operation represented with the precision
              and scale of the result type loses one or more leading sig-
              nificant digits after rounding or truncating if necessary,
              then an exception condition is raised: data exception-numeric
              value out of range. The choice of whether to round or trun-
              cate is implementation-defined.

         6) If the type of the result of an arithmetic operation is approx-
            imate numeric and the exponent of the approximate mathematical
            result of the operation is not within the implementation-defined
            exponent range for the result type, then an exception condition
            is raised: data exception-numeric value out of range.




         158  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                             6.12 <numeric value expression>


         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

              None.












































                                                    Scalar expressions   159

 





          X3H2-92-154/DBL CBR-002
         6.13 <string value expression>


         6.13  <string value expression>

         Function

         Specify a character string value or a bit string value.

         Format

         <string value expression> ::=
                <character value expression>
              | <bit value expression>

         <character value expression> ::=
                <concatenation>
              | <character factor>

         <concatenation> ::=
              <character value expression> <concatenation operator>
              <character factor>

         <character factor> ::=
              <character primary> [ <collate clause> ]

         <character primary> ::=
                <value expression primary>
              | <string value function>

         <bit value expression> ::=
                <bit concatenation>
              | <bit factor>

         <bit concatenation> ::=
              <bit value expression> <concatenation operator> <bit factor>

         <bit factor> ::= <bit primary>

         <bit primary> ::=
                <value expression primary>
              | <string value function>


         Syntax Rules

         1) The data type of a <character primary> shall be character
            string.

         2) Character strings of different character repertoires shall
            not be mixed in a <character value expression>. The character
            repertoire of a <character value expression> is the character
            repertoire of its components.




         160  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                              6.13 <string value expression>


         3) Case:

            a) If <concatenation> is specified, then:

              Let D1 be the data type of the <character value expression>
              and let D2 be the data type of the <character factor>. Let
              M be the length in characters of D1 plus the length in char-
              acters of D2. Let VL be the implementation-defined maximum
              length of a variable-length character string and let FL be
              the implementation-defined maximum length of a fixed-length
              character string.

              Case:

              i) If the data type of the <character value expression> or
                 <character factor> is variable-length character string,
                 then the data type of the <concatenation> is variable-
                 length character string with maximum length equal to the
                 lesser of M and VL.

             ii) If the data type of the <character value expression> and
                 <character factor> is fixed-length character string, then M
                 shall not be greater than FL and the data type of the <con-
                 catenation> is fixed-length character string with length M.

            b) Otherwise, the data type of the <character value expression>
              is the data type of the <character factor>.

         4) Case:

            a) If <character factor> is specified, then

              Case:

              i) If <collate clause> is specified, then the <character value
                 expression> has the collating sequence given in <collate
                 clause>, and has the Explicit coercibility attribute.

             ii) Otherwise, if <value expression primary> or <string value
                 function> are specified, then the collating sequence and
                 coercibility attribute of the <character factor> are spec-
                 ified in Subclause 6.2, "<value specification> and <target
                 specification>", and Subclause 6.7, "<string value func-
                 tion>", respectively.

            b) If <concatenation> is specified, then the collating sequence
              and the coercibility attribute are determined as specified
              for dyadic operators in Subclause 4.2.3, "Rules determining
              collating sequence usage".

         5) The data type of a <bit primary> shall be bit string.



                                                    Scalar expressions   161

 





          X3H2-92-154/DBL CBR-002
         6.13 <string value expression>


         6) Case:

            a) If <bit concatenation> is specified, then let D1 be the
              data type of the <bit value expression>, let D2 be the data
              type of the <bit factor>, let M be the length in bits of D1
              plus the length in bits of D2, let VL be the implementation-
              defined maximum length of a variable-length bit string, and
              let FL be the implementation-defined maximum length of a
              fixed-length bit string.

              Case:

              i) If the data type of the <bit value expression> or <bit
                 factor> is variable-length bit string, then the data type
                 of the <bit concatenation> is variable-length bit string
                 with maximum length equal to the lesser of M and VL.

             ii) If the data type of the <bit value expression> and <bit
                 factor> is fixed-length bit string, then M shall not be
                 greater than FL and the data type of the <bit concatena-
                 tion> is fixed-length bit string with length M.

            b) Otherwise, the data type of a <bit value expression> is the
              data type of the <bit factor>.

         Access Rules

            None.

         General Rules

         1) If the value of any <character primary> simply contained in a
            <character value expression> is the null value, then the result
            of the <character value expression> is the null value.

         2) If <concatenation> is specified, then let S1 and S2 be the re-
            sult of the <character value expression> and <character factor>,
            respectively.

            Case:

            a) If either S1 or S2 is the null value, then the result of the
              <concatenation> is the null value.

            b) Otherwise, let S be the string consisting of S1 followed by
              S2 and let M be the length of S.

              Case:

              i) If the data type of either S1 or S2 is variable-length
                 character string, then

                 Case:

                 1) If M is less than or equal to VL, then the result of the
                   <concatenation> is S with length M.

         162  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                              6.13 <string value expression>


                 2) If M is greater than VL and the right-most M-VL charac-
                   ters of S are all the <space> character, then the result
                   of the <concatenation> is the first VL characters of S
                   with length VL.

                 3) Otherwise, an exception condition is raised: data
                   exception-string data, right truncation.

             ii) If the data types of both S1 and S2 are fixed-length char-
                 acter string, then the result of the <concatenation> is
                 S.

         3) If the value of any <bit primary> simply contained in a <bit
            value expression> is the null value, then the result of the <bit
            value expression> is the null value.

         4) If <bit concatenation> is specified, then let S1 and S2 be the
            result of the <bit value expression> and <bit factor>, respec-
            tively.

            Case:

            a) If either S1 or S2 is the null value, then the result of the
              <bit concatenation> is the null value.

            b) Otherwise, let S be the string consisting of S1 followed by
              S2 and let M be the length in bits of S.

              Case:

              i) If the data type of either S1 or S2 is variable-length bit
                 string, then

                 Case:

                 1) If M is less than or equal to VL, then the result of the
                   <bit concatenation> is S with length M.

                 2) If M is greater than VL and the right-most M-VL bits of
                   S are all 0-valued, then the result of the <bit concate-
                   nation> is the first VL bits of S with length VL.

                 3) Otherwise, an exception condition is raised: data
                   exception-string data, right truncation.

             ii) If the data types of both S1 and S2 are fixed-length bit
                 string, then the result of the <bit concatenation> is S.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) Conforming Intermediate SQL language shall not contain any
              <collate clause>.

            b) Conforming Intermediate SQL language shall contain no <bit
              value expression>.

                                                    Scalar expressions   163

 





          X3H2-92-154/DBL CBR-002
         6.13 <string value expression>


         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <character value expression> shall not be a <concatena-
              tion>.

















































         164  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                            6.14 <datetime value expression>


         6.14  <datetime value expression>

         Function

         Specify a datetime value.

         Format

         <datetime value expression> ::=
                <datetime term>
              | <interval value expression> <plus sign> <datetime term>
              | <datetime value expression> <plus sign> <interval term>
              | <datetime value expression> <minus sign> <interval term>

         <datetime term> ::=
                <datetime factor>

         <datetime factor> ::=
                <datetime primary> [ <time zone> ]

         <datetime primary> ::=
                <value expression primary>
              | <datetime value function>

         <time zone> ::=
              AT <time zone specifier>

         <time zone specifier> ::=
                LOCAL
              | TIME ZONE <interval value expression>


         Syntax Rules

         1) The data type of a <datetime primary> shall be datetime.

         2) Case:

            a) If the <datetime value expression> is a <datetime term>, then
              the precision of the result of the <datetime value expres-
              sion> is the precision of the <datetime value function> or
              <value expression primary> that it simply contains.

            b) Otherwise, the precision of the result of the <datetime value
              expression> is the precision of the <datetime value expres-
              sion> or <datetime term> that it simply contains.

         3) If an <interval value expression> or <interval term> is spec-
            ified, then the <interval value expression> or <interval term>
            shall only contain <datetime field>s that are contained within
            the <datetime value expression> or <datetime term>.



                                                    Scalar expressions   165

 





          X3H2-92-154/DBL CBR-002
         6.14 <datetime value expression>


         4) The data type of the <interval value expression> immediately
            contained in a <time zone specifier> shall be INTERVAL HOUR TO
            MINUTE.

         5) Case:

            a) If the data type of the <datetime primary> is DATE, then
              <time zone> shall not be specified.

            b) If the data type of the <datetime primary> is TIME or
              TIMESTAMP and <time zone> is not specified, then "AT LOCAL"
              is implicit.

         Access Rules

            None.

         General Rules

         1) If the result of any <datetime primary>, <interval value expres-
            sion>, <datetime value expression>, or <interval term> simply
            contained in a <datetime value expression> is the null value,
            then the result of the <datetime value expression> is the null
            value.

         2) If <time zone> is specified or implied and the <interval value
            expression> immediately contained in <time zone specifier> is
            the null value, then the result of the <datetime value expres-
            sion> is the null value.

         3) If a <datetime value expression> immediately contains the opera-
            tor + or -, then the result is effectively evaluated as follows:

            a) Case:

              i) If <datetime value expression> immediately contains the
                 operator + and the <interval value expression> or <interval
                 term> is not negative, or if <datetime value expression>
                 immediately contains the operator - and the <interval term>
                 is negative, then successive <datetime field>s of the <in-
                 terval value expression> or <interval term> are added to
                 the corresponding fields of the <datetime value expression>
                 or <datetime term>.

             ii) Otherwise, successive <datetime field>s of the <interval
                 value expression> or <interval term> are subtracted from
                 the corresponding fields of the <datetime value expression>
                 or <datetime term>.

            b) Arithmetic is performed so as to maintain the integrity of
              the datetime data type that is the result of the <datetime
              value expression>. This may involve carry from or to the
              immediately next more significant <datetime field>. If the
              data type of the <datetime value expression> is TIME, then

         166  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                            6.14 <datetime value expression>


              arithmetic on the HOUR <datetime field> is undertaken modulo
              24. If the <interval value expression> or <interval term> is
              a year-month interval, then the DAY field of the result is
              the same as the DAY field of the <datetime term> or <datetime
              value expression>.

            c) If, after the preceding step, any <datetime field> of the
              result is outside the permissible range of values for the
              field or the result is invalid based on the natural rules for
              dates and times, then an exception condition is raised: data
              exception-datetime field overflow.

              Note: For the permissible range of values for <datetime
              field>s, see Table 10, "Valid values for fields in datetime
              items".

         4) If <time zone> is specified or implied, then:

            a) If LOCAL is specified, then let TZ be the current default
              time zone displacement of the SQL-session. Otherwise, let
              TZ be the value of the <simple value specification> simply
              contained in the <time zone>.

            b) If the value of the <interval value expression> immediately
              contained in <time zone specifier> is less than INTERVAL
              -'12:59' or greater than INTERVAL +'13:00', then an excep-
              tion condition is raised: data exception-invalid time zone
              displacement value.

            c) Let DV be the value of the <datetime primary> directly con-
              tained in the <datetime value expression> expressed as a
              datetime normalized to UTC.

            d) The value of the <datetime value expression> is calculated
              as:

                 DV - TZ

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <datetime
              value expression>.





                                                    Scalar expressions   167

 





          X3H2-92-154/DBL CBR-002
         6.15 <interval value expression>


         6.15  <interval value expression>

         Function

         Specify an interval value.

         Format

         <interval value expression> ::=
                <interval term>
              | <interval value expression 1> <plus sign> <interval term 1>

              | <interval value expression 1> <minus sign> <interval term 1>

              | <left paren> <datetime value expression> <minus sign>
                    <datetime term> <right paren> <interval qualifier>

         <interval term> ::=
                <interval factor>
              | <interval term 2> <asterisk> <factor>
              | <interval term 2> <solidus> <factor>
              | <term> <asterisk> <interval factor>

         <interval factor> ::=
              [ <sign> ] <interval primary>

         <interval primary> ::=
                <value expression primary> [ <interval qualifier> ]

         <interval value expression 1> ::= <interval value expression>

         <interval term 1> ::= <interval term>

         <interval term 2> ::= <interval term>


         Syntax Rules

         1) The data type of an <interval value expression> is interval. The
            data type of an <interval primary> shall be interval.

         2) An <interval primary> shall specify <interval qualifier> only if
            the <interval primary> specifies a <dynamic parameter specifica-
            tion>.

         3) Case:

            a) If the <interval value expression> simply contains an <in-
              terval qualifier>, then the result contains the <datetime
              field>s specified in the <interval qualifier>.

            b) If the <interval value expression> is an <interval term>,
              then the result of an <interval value expression> contains
              the same <datetime field>s as the <interval primary>.

         168  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                            6.15 <interval value expression>


            c) If <interval term 1> is specified, then the result contains
              all the <datetime field>s that are contained within either
              <interval value expression 1> or <interval term 1>.

         4) Case:

            a) If <interval term 1> is a year-month interval, then <interval
              value expression 1> shall be a year-month interval.

            b) If <interval term 1> is a day-time interval, then <interval
              value expression 1> shall be a day-time interval.

         5) If <datetime value expression> is specified, then <datetime
            value expression> and <datetime term> shall be comparable.

         Access Rules

            None.

         General Rules

         1) If an <interval term> specifies "<term> * <interval factor>",
            then let T and F be respectively the value of the <term> and
            the value of the <interval factor>. The result of the <interval
            term> is the result of F * T.

         2) If the result of any <interval primary>, <datetime value ex-
            pression>, <datetime term>, or <factor> simply contained in an
            <interval value expression> is the null value, then the result
            of the <interval value expression> is the null value.

         3) If the <sign> of an <interval factor> is <minus sign>, then the
            value of the <interval factor> is the negative of the value of
            the <interval primary>.

         4) If <interval term 2> is specified, then:

            a) Let X be the value of <interval term 2> and let Y be the
              value of <factor>.

            b) Let P and Q be respectively the most significant and least
              significant <datetime field>s of <interval term 2>.

            c) Let E be an exact numeric result of the operation

                 CAST (CAST (X AS INTERVAL Q) AS E1)

              where E1 is an exact numeric data type of sufficient scale
              and precision so as to not lose significant digits.

            d) Let OP be the operator * or / specified in the <interval
              value expression>.


                                                    Scalar expressions   169

 





          X3H2-92-154/DBL CBR-002
         6.15 <interval value expression>


            e) Let I, the result of the <interval value expression> ex-
              pressed in terms of the <datetime field> Q, be the result
              of

                 CAST ((E OP Y) AS INTERVAL Q).

            f) The result of the <interval value expression> is

                 CAST (I AS INTERVAL W)

              where W is an <interval qualifier> identifying the <datetime
              field>s P TO Q, but with <interval leading field precision>
              such that significant digits are not lost.

         5) If <interval term 1> is specified, then let P and Q be respec-
            tively the most significant and least significant <datetime
            field>s in <interval term 1> and <interval value expression 1>,
            let X be the value of <interval value expression 1>, and let Y
            be the value of <interval term 1>.

            a) Let A be an exact numeric result of the operation

                 CAST (CAST (X AS INTERVAL Q) AS E1)

              where E1 is an exact numeric data type of sufficient scale
              and precision so as to not lose significant digits.

            b) Let B be an exact numeric result of the operation

                 CAST (CAST (Y AS INTERVAL Q) AS E2)

              where E2 is an exact numeric data type of sufficient scale
              and precision so as to not lose significant digits.

            c) Let OP be the operator + or - specified in the <interval
              value expression>.

            d) Let I, the result of the <interval value expression> ex-
              pressed in terms of the <datetime field> Q, be the result
              of:

                 CAST ((A OP B) AS INTERVAL Q)

            e) The result of the <interval value expression> is

                 CAST (I AS INTERVAL W)

              where W is an <interval qualifier> identifying the <datetime
              field>s P TO Q, but with <interval leading field precision>
              such that significant digits are not lost.

         6) If <datetime value expression> is specified, then let Y be the
            least significant <datetime field> specified by <interval qual-
            ifier>. Let A be the value represented by <datetime value ex-
            pression> and let B be the value represented by <datetime term>.

         170  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                            6.15 <interval value expression>


            Evaluation of <interval value expression> proceeds as follows:

            a) A and B are converted to integer scalars A2 and B2 respec-
              tively in units Y as displacements from some implementation-
              dependent start datetime.

            b) The result is determined by effectively computing A2-B2 and
              then converting the difference to an interval using an <in-
              terval qualifier> whose <end field> is Y and whose <start
              field> is sufficiently significant to avoid loss of signif-
              icant digits. That interval is then converted to an inter-
              val using the specified <interval qualifier>, rounding or
              truncating if necessary. The choice of whether to round or
              truncate is implementation-defined.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <interval
              value expression>.




























                                                    Scalar expressions   171

 





          X3H2-92-154/DBL CBR-002

























































         172  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002






         7  Query expressions



         7.1  <row value constructor>

         Function

         Specify an ordered set of values to be constructed into a row or
         partial row.

         Format

         <row value constructor> ::=
                <row value constructor element>
              | <left paren> <row value constructor list> <right paren>
              | <row subquery>

         <row value constructor list> ::=
              <row value constructor element>
                  [ { <comma> <row value constructor element> }... ]

         <row value constructor element> ::=
                <value expression>
              | <null specification>
              | <default specification>

         <null specification> ::=
              NULL

         <default specification> ::=
              DEFAULT


         Syntax Rules

         1) If a <row value constructor> simply contains a <null speci-
            fication> or a <default specification>, then the <row value
            constructor> shall be simply contained in a <query expression>
            that is simply contained in an <insert statement>.

         2) A <row value constructor element> immediately contained in a
            <row value constructor> shall not be a <value expression> of the
            form "<left paren> <value expression> <right paren>".

            Note: This Rule removes a syntactic ambiguity. A <row value
            constructor> of this form is permitted, but is parsed in the
            form "<left paren> <row value constructor list> <right paren>".



                                                     Query expressions   173

 





          X3H2-92-154/DBL CBR-002
         7.1 <row value constructor>


         3) A <row value constructor> that immediately contains a <row value
            constructor element> X is equivalent to a <row value construc-
            tor> of the form

              ( X )

         4) The data types of the column or columns of a <row value con-
            structor> are the data types of the <row value constructor ele-
            ment> or <row value constructor element>s or the columns of the
            <row subquery> simply contained in the <row value constructor>.

         5) If a <row value constructor> is derived from a <row subquery>,
            then the degree of the <row value constructor> is the degree
            of the table resulting from the <row subquery>; otherwise, the
            degree of the <row value constructor> is the number of <row
            value constructor element>s that occur in its specification.

         Access Rules

            None.

         General Rules

         1) The value of a <null specification> is the null value.

         2) The value of a <default specification> is the default value in-
            dicated in the column descriptor for the corresponding column in
            the explicit or implicit <insert column list> simply contained
            in the <insert statement>.

         3) Case:

            a) If a <row value constructor list> is specified, then the re-
              sult of the <row value constructor> is a row of columns whose
              i-th column has an implementation-dependent name different
              from the <column name> of all other columns contained in the
              SQL-statement and whose value is the value of the i-th <row
              value constructor element> in the <row value constructor
              list>.

            b) If the <row value constructor> is a <row subquery>, then:

              i) Let R be the result of the <row subquery> and let D be the
                 degree of R.

             ii) If the cardinality of R is 0, then the result of the <row
                 value constructor> is D null values.

            iii) If the cardinality of R is 1, then the result of the <row
                 value constructor> is R.




         174  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                 7.1 <row value constructor>


         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) A <row value constructor> that is not simply contained in
              a <table value constructor> shall not contain more than one
              <row value constructor element>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <row value constructor element> shall not specify DEFAULT.










































                                                     Query expressions   175

 





          X3H2-92-154/DBL CBR-002
         7.2 <table value constructor>


         7.2  <table value constructor>

         Function

         Specify a set of <row value constructor>s to be constructed into a
         table.

         Format

         <table value constructor> ::=
              VALUES <table value constructor list>

         <table value constructor list> ::=
              <row value constructor> [ { <comma> <row value constructor> }... ]



         Syntax Rules

         1) All <row value constructor>s shall be of the same degree.

         Access Rules

            None.

         General Rules

         1) Let Ti be a table whose j-th column has the same data type as
            the j-th <value expression> in the i-th <row value construc-
            tor> and let Ti contain one row whose j-th column has the same
            value as the j-th <value expression> in the i-th <row value
            constructor>.

         2) The result of the <table value constructor> is the same as the
            result of

              T1 [ UNION ALL T2 [ . . . UNION ALL n ] . . . ]

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) A <table value constructor> shall contain exactly one <row
              value constructor> that shall be of the form "(<row value
              constructor list>)".

            b) A <table value constructor> shall be the <query expression>
              of an <insert statement>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

              None.

         176  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                      7.3 <table expression>


         7.3  <table expression>

         Function

         Specify a table or a grouped table.

         Format

         <table expression> ::=
              <from clause>
              [ <where clause> ]
              [ <group by clause> ]
              [ <having clause> ]


         Syntax Rules

         1) The result of a <table expression> is a derived table in which
            the descriptor of the i-th column is the same as the descriptor
            of the i-th column of the table specified by the <from clause>.

         2) Let C be some column. Let TE be the <table expression>. C is an
            underlying column of TE if and only if C is an underlying column
            of some <column reference> contained in TE.

         Access Rules

            None.

         General Rules

         1) If all optional clauses are omitted, then the result of the <ta-
            ble expression> is the same as the result of the <from clause>.
            Otherwise, each specified clause is applied to the result of
            the previously specified clause and the result of the <table ex-
            pression> is the result of the application of the last specified
            clause.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) If the table identified in the <from clause> is a grouped
              view, then the <table expression> shall not contain a <where
              clause>, <group by clause>, or <having clause>.




                                                     Query expressions   177

 





          X3H2-92-154/DBL CBR-002
         7.4 <from clause>


         7.4  <from clause>

         Function

         Specify a table derived from one or more named tables.

         Format

         <from clause> ::= FROM <table reference> [ { <comma> <table reference> }... ]



         Syntax Rules

         1) Case:

            a) If the <from clause> contains a single <table reference> with
              no intervening <derived table> or <joined table>, then the
              descriptor of the result of the <from clause> is the same
              as the descriptor of the table identified by that <table
              reference>.

            b) If the <from clause> contains more than one <table reference>
              with no intervening <derived table> or <joined table>, then
              the descriptors of the columns of the result of the <from
              clause> are the descriptors of the columns of the tables
              identified by the <table reference>s, in the order in which
              the <table reference>s appear in the <from clause> and in the
              order in which the columns are defined within each table.

         Access Rules

            None.

         General Rules

         1) Case:

            a) If the <from clause> contains a single <table reference> with
              no intervening <derived table> or <joined table>, then the
              result of the <from clause> is the table identified by that
              <table reference>.

            b) If the <from clause> contains more than one <table reference>
              with no intervening <derived table> or <joined table>, then
              the result of the <from clause> is the extended Cartesian
              product of the tables identified by those <table reference>s.

              The extended Cartesian product, CP, is the multiset of all
              rows R such that R is the concatenation of a row from each
              of the identified tables in the order in which they are iden-
              tified. The cardinality of CP is the product of the cardi-
              nalities of the identified tables. The ordinal position of a

         178  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                           7.4 <from clause>


              column in CP is N+S, where N is the ordinal position of that
              column in the identified table T from which it is derived and
              S is the sum of the degrees of the tables identified before T
              in the <from clause>.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) If the table identified by <table name> is a grouped view,
              then the <from clause> shall contain exactly one <table ref-
              erence>.





































                                                     Query expressions   179

 





          X3H2-92-154/DBL CBR-002
         7.5 <joined table>


         7.5  <joined table>

         Function

         Specify a table derived from a Cartesian product, inner or outer
         join, or union join.

         Format

         <joined table> ::=
                <cross join>
              | <qualified join>
              | <left paren> <joined table> <right paren>

         <cross join> ::=
              <table reference> CROSS JOIN <table reference>

         <qualified join> ::=
              <table reference> [ NATURAL ] [ <join type> ] JOIN
                <table reference> [ <join specification> ]

         <join specification> ::=
                <join condition>
              | <named columns join>

         <join condition> ::= ON <search condition>

         <named columns join> ::=
              USING <left paren> <join column list> <right paren>

         <join type> ::=
                INNER
              | <outer join type> [ OUTER ]
              | UNION

         <outer join type> ::=
                LEFT
              | RIGHT
              | FULL

         <join column list> ::= <column name list>


         Syntax Rules

         1) Let TR1 and TR2 be the first and second <table reference>s of
            the <joined table>, respectively. Let T1 and T2 be the tables
            identified by TR1 and TR2, respectively. Let TA and TB be the
            correlation names of TR1 and TR2, respectively. Let CP be:

              SELECT * FROM TR1, TR2

         2) If a <qualified join> is specified, then

         180  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                          7.5 <joined table>


            Case:

            a) If NATURAL is specified, then a <join specification> shall
              not be specified.

            b) If UNION is specified, then neither NATURAL nor a <join spec-
              ification> shall be specified.

            c) Otherwise, a <join specification> shall be specified.

         3) If a <qualified join> is specified and a <join type> is not
            specified, then INNER is implicit.

         4) If a <qualified join> containing a <join condition> is speci-
            fied, then;

            a) Each <column reference> directly contained in the <search
              condition> shall unambiguously reference a column of T1 or T2
              or be an outer reference.

            b) If a <value expression> directly contained in the <search
              condition> is a <set function specification>, then the
              <joined table> shall be contained in a <having clause> or
              <select list> and the <set function specification> shall
              contain a <column reference> that is an outer reference.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         5) If neither NATURAL is specified nor a <join specification> sim-
            ply containing a <named columns join> is specified, then the
            descriptors of the columns of the result of the <joined table>
            are the same as the descriptors of the columns of CP.

         6) If NATURAL is specified or if a <join specification> simply
            containing a <named columns join> is specified, then:

            a) If NATURAL is specified, then let common column name be a
              <column name> that is the <column name> of exactly one column
              of T1 and the <column name> of exactly one column of T2. T1
              shall not have any duplicate common column names and T2 shall
              not have any duplicate common column names. Let corresponding
              join columns refer to all columns of T1 and T2 that have
              common column names, if any.

            b) If a <named columns join> is specified, then every <column
              name> in the <join column list> shall be the <column name>
              of exactly one column of T1 and the <column name> of exactly
              one column of T2. Let common column name be the name of such
              a column. Let corresponding join columns refer to the columns
              of T1 and T2 identified in the <join column list>.

            c) Let 1 and C2 be a pair of corresponding join columns con-
              tained in T1 and T2, respectively. C1 and C2 shall be compa-
              rable.

                                                     Query expressions   181

 





          X3H2-92-154/DBL CBR-002
         7.5 <joined table>


            d) Let SLCC be a <select list> of <derived column>s of the form

                 COALESCE ( TA.C, TB.C ) AS C

              for every column C that is a corresponding join column, taken
              in order of their ordinal positions in T1.

            e) Let SL1 be a <select list> of those <column name>s of T1
              that are not corresponding join columns, taken in order of
              their ordinal positions in T1, and let SLT2 be a <select
              list> of those <column name>s of T2 that are not correspond-
              ing join columns, taken in order of their ordinal positions
              in T2.

            f) The descriptors of the columns of the result of the <joined
              table> are the same as the descriptors of the columns of the
              result of

                 SELECT SLCC, SLT1, SLT2 FROM TR1, TR2

         7) For every column CR of the result of the <joined table> that
            is not a corresponding join column and that corresponds to a
            column C1 of T1, CR is possibly nullable if any of the following
            conditions are true:

            a) RIGHT, FULL, or UNION is specified, or

            b) INNER, LEFT, or CROSS JOIN is specified or implicit and 1 is
              possibly nullable.

         8) For every column CR of the result of the <joined table> that
            is not a corresponding join column and that corresponds to a
            column C2 of T2, CR is possibly nullable if any of the following
            conditions are true:

            a) LEFT, FULL, or UNION is specified, or

            b) INNER, RIGHT, or CROSS JOIN is specified or implicit and C
              is possibly nullable.

         9) For every column CR of the result of the <joined table> that
            is a corresponding join column and that corresponds to a column
            C1 of T1 and C2 of T2, CR is possibly nullable if any of the
            following conditions are true:

            a) RIGHT, FULL, or UNION is specified and 1 is possibly nul-
              lable, or

            b) LEFT, FULL, or UNION is specified and 2 is possibly nul-
              lable.

         10)The <joined table> is a read-only table.


         182  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                          7.5 <joined table>


         Access Rules

            None.

         General Rules

         1) Case:

            a) If <join type> is UNION, then let T be the empty set.

            b) If a <cross join> is specified, then let T be the multiset of
              rows of CP.

            c) If a <join condition> is specified, then let T be the multi-
              set of rows of CP for which the specified <search condition>
              is true.

            d) If NATURAL is specified or <named columns join> is specified,
              then

              Case:

              i) If there are corresponding join columns, then let T be the
                 multiset of rows of CP for which the corresponding join
                 columns have equal values.

             ii) Otherwise, let T be the multiset of rows of CP.

         2) Let P1 be the multiset of rows of T1 for which there exists in T
            some row that is the concatenation of some row R1 of T1 and some
            row R2 of T2. Let P2 be the multiset of rows of T2 for which
            there exists in T some row that is the concatenation of some row
            R1 of T1 and some row R2 of T2.

         3) Let U1 be those rows of T1 that are not in P1 and let U2 be
            those rows of T2 that are not in P2.

         4) Let D1 and D2 be the degree of T1 and T2, respectively. Let
            X1 be U1 extended on the right with D2 columns containing the
            null value. Let X2 be U2 extended on the left with D1 columns
            containing the null value.

         5) Let XN1 and XN2 be effective distinct names for X1 and X2, re-
            spectively. Let TN be an effective name for T.

            Case:

            a) If INNER or <cross join> is specified, then let S be the
              multiset of rows of T.

            b) If LEFT is specified, then let S be the multiset of rows
              resulting from:

                 SELECT * FROM TN
                 UNION ALL
                 SELECT * FROM XN1

                                                     Query expressions   183

 





          X3H2-92-154/DBL CBR-002
         7.5 <joined table>


            c) If RIGHT is specified, then let S be the multiset of rows
              resulting from:

                 SELECT * FROM TN
                 UNION ALL
                 SELECT * FROM XN2

            d) If FULL is specified, then let S be the multiset of rows
              resulting from:

                 SELECT * FROM TN
                 UNION ALL
                 SELECT * FROM XN1
                 UNION ALL
                 SELECT * FROM XN2

            e) If UNION is specified, then let S be the multiset of rows
              resulting from:

                 SELECT * FROM XN1
                 UNION ALL
                 SELECT * FROM XN2

         6) Let SN be an effective name of S.

            Case:

            a) If NATURAL is specified or a <named columns join> is speci-
              fied, then the result of the <joined table> is the multiset
              of rows resulting from:

                 SELECT SLCC, SLT1, SLT2 FROM SN

            b) Otherwise, the result of the <joined table> is S.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) Conforming Intermediate SQL language shall contain no <cross
              join>.

            b) Conforming Intermediate SQL language shall not specify UNION
              JOIN.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) Conforming Entry SQL language shall not contain any <joined
              table>.




         184  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                          7.6 <where clause>


         7.6  <where clause>

         Function

         Specify a table derived by the application of a <search condition>
         to the result of the preceding <from clause>.

         Format

         <where clause> ::= WHERE <search condition>


         Syntax Rules

         1) Let T be the result of the preceding <from clause>. Each <column
            reference> directly contained in the <search condition> shall
            unambiguously reference a column of T or be an outer reference.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         2) If a <value expression> directly contained in the <search condi-
            tion> is a <set function specification>, then the <where clause>
            shall be contained in a <having clause> or <select list> and the
            <column reference> in the <set function specification> shall be
            an outer reference.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         3) No <column reference> contained in a <subquery> in the <search
            condition> that references a column of T shall be specified in a
            <set function specification>.

         Access Rules

            None.

         General Rules

         1) The <search condition> is applied to each row of T. The result
            of the <where clause> is a table of those rows of T for which
            the result of the <search condition> is true.

         2) Each <subquery> in the <search condition> is effectively exe-
            cuted for each row of T and the results used in the application
            of the <search condition> to the given row of T. If any executed
            <subquery> contains an outer reference to a column of T, then
            the reference is to the value of that column in the given row of
            T.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

                                                     Query expressions   185

 





          X3H2-92-154/DBL CBR-002
         7.6 <where clause>


         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <value expression> directly contained in the <search condi-
              tion> shall not include a reference to a column that gener-
              ally contains a <set function specification>.










































         186  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                       7.7 <group by clause>


         7.7  <group by clause>

         Function

         Specify a grouped table derived by the application of the <group by
         clause> to the result of the previously specified clause.

         Format

         <group by clause> ::=
              GROUP BY <grouping column reference list>

         <grouping column reference list> ::=
              <grouping column reference> [ { <comma> <grouping column reference> }... ]


         <grouping column reference> ::=
              <column reference> [ <collate clause> ]


         Syntax Rules

         1) If no <where clause> is specified, then let T be the result of
            the preceding <from clause>; otherwise, let T be the result of
            the preceding <where clause>.

         2) Each <column reference> in the <group by clause> shall unambigu-
            ously reference a column of T. A column referenced in a <group
            by clause> is a grouping column.

         3) For every grouping column, if <collate clause> is specified,
            then the data type of the <column reference> shall be character
            string. The column descriptor of the corresponding column in the
            result has the collating sequence specified in <collate clause>
            and the coercibility attribute Explicit.

         Access Rules

            None.

         General Rules

         1) The result of the <group by clause> is a partitioning of T into
            a set of groups. The set is the minimum number of groups such
            that, for each grouping column of each group of more than one
            row, no two values of that grouping column are distinct.

         2) Every row of a given group contains equal values of a given
            grouping column. When a <search condition> or <value expression>
            is applied to a group, a reference to a grouping column is a
            reference to that value.

            Note: See the General Rules of Subclause 8.2, "<comparison pred-
            icate>".

                                                     Query expressions   187

 





          X3H2-92-154/DBL CBR-002
         7.7 <group by clause>


         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) Conforming Intermediate SQL language shall not contain any
              <collate clause>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

              None.











































         188  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                         7.8 <having clause>


         7.8  <having clause>

         Function

         Specify a grouped table derived by the elimination of groups from
         the result of the previously specified clause that do not meet the
         <search condition>.

         Format

         <having clause> ::= HAVING <search condition>


         Syntax Rules

         1) If neither a <where clause> nor a <group by clause> is speci-
            fied, then let T be the result of the preceding <from clause>;
            if a <where clause> is specified, but a <group by clause> is
            not specified, then let T be the result of the preceding <where
            clause>; otherwise, let T be the result of the preceding <group
            by clause>. Each <column reference> directly contained in the
            <search condition> shall unambiguously reference a grouping
            column of T or be an outer reference.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         2) Each <column reference> contained in a <subquery> in the <search
            condition> that references a column of T shall reference a
            grouping column of T or shall be specified within a <set func-
            tion specification>.

         3) The <having clause> is possibly non-deterministic if it contains
            a reference to a column C of T that has a data type of character
            string and:

            a) C is specified within a <set function specification> that
              specifies MIN or MAX, or

            b) C is a grouping column of T.

         Access Rules

            None.

         General Rules

         1) Let T be the result of the preceding <from clause>, <where
            clause>, or <group by clause>. If that clause is not a <group
            by clause>, then T consists of a single group and does not have
            a grouping column.

         2) The <search condition> is applied to each group of T. The result
            of the <having clause> is a grouped table of those groups of T
            for which the result of the <search condition> is true.

                                                     Query expressions   189

 





          X3H2-92-154/DBL CBR-002
         7.8 <having clause>


         3) When the <search condition> is applied to a given group of T,
            that group is the argument or argument source of each <set func-
            tion specification> directly contained in the <search condition>
            unless the <column reference> in the <set function specifica-
            tion> is an outer reference.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         4) Each <subquery> in the <search condition> is effectively exe-
            cuted for each group of T and the result used in the application
            of the <search condition> to the given group of T. If any exe-
            cuted <subquery> contains an outer reference to a column of T,
            then the reference is to the values of that column in the given
            group of T.

            Note: Outer reference is defined in Subclause 6.4, "<column
            reference>".

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

              None.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

              None.

























         190  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   7.9 <query specification>


         7.9  <query specification>

         Function

         Specify a table derived from the result of a <table expression>.

         Format

         <query specification> ::=
              SELECT [ <set quantifier> ] <select list> <table expression>

         <select list> ::=
                <asterisk>
              | <select sublist> [ { <comma> <select sublist> }... ]

         <select sublist> ::=
                <derived column>
              | <qualifier> <period> <asterisk>

         <derived column> ::= <value expression> [ <as clause> ]

         <as clause> ::= [ AS ] <column name>


         Syntax Rules

         1) Let T be the result of the <table expression>.

         2) The degree of the table specified by a <query specification> is
            equal to the cardinality of the <select list>.

         3) Case:

            a) If the <select list> "*" is simply contained in a <subquery>
              that is immediately contained in an <exists predicate>, then
              the <select list> is equivalent to a <value expression> that
              is an arbitrary <literal>.

            b) Otherwise, the <select list> "*" is equivalent to a <value
              expression> sequence in which each <value expression> is a
              <column reference> that references a column of T and each
              column of T is referenced exactly once. The columns are ref-
              erenced in the ascending sequence of their ordinal position
              within T.

         4) The <select sublist> "<qualifier>.*" for some <qualifier> Q is
            equivalent to a <value expression> sequence in which each <value
            expression> is a <column reference> CR that references a column
            of T that is not a common column of a <joined table>. Each col-
            umn of T that is not a common column of a <joined table> shall
            be referenced exactly once. The columns shall be referenced in
            the ascending sequence of their ordinal positions within T.

            Note: common column of a <joined table> is defined in Subclause 7.5,
            "<joined table>".

                                                     Query expressions   191

 





          X3H2-92-154/DBL CBR-002
         7.9 <query specification>


         5) Let C be some column. Let QS be the <query specification>. Let
            DCi, for i ranging from 1 to the number of <derived column>s
            inclusively, be the i-th <derived column> simply contained in
            the <select list> of QS. For all i, C is an underlying column
            of DCi, and of any <column reference> that identifies DCi, if
            and only if C is an underlying column of the <value expression>
            of DCi, or C is an underlying column of the <table expression>
            immediately contained in QS.

         6) Each <column reference> directly contained in each <value ex-
            pression> and each <column reference> contained in a <set
            function specification> directly contained in each <value ex-
            pression> shall unambiguously reference a column of T.

         7) If T is a grouped table, then each <column reference> in each
            <value expression> that references a column of T shall refer-
            ence a grouping column or be specified within a <set function
            specification>. If T is not a grouped table and any <value ex-
            pression> contains a <set function specification> that contains
            a reference to a column of T or any <value expression> directly
            contains a <set function specification> that does not contain an
            outer reference, then every <column reference> in every <value
            expression> that references a column of T shall be specified
            within a <set function specification>.

         8) Each column of the table that is the result of a <query spec-
            ification> has a column descriptor that includes a data type
            descriptor that is the same as the data type descriptor of the
            <value expression> from which the column was derived.

         9) Case:

            a) If the i-th <derived column> in the <select list> specifies
              an <as clause> that contains a <column name> C, then the
              <column name> of the i-th column of the result is C.

            b) If the i-th <derived column> in the <select list> does not
              specify an <as clause> and the <value expression> of that
              <derived column> is a single <column reference>, then the
              <column name> of the i-th column of the result is C.

            c) Otherwise, the <column name> of the i-th column of the <query
              specification> is implementation-dependent and different
              from the <column name> of any column, other than itself, of
              a table referenced by any <table reference> contained in the
              SQL-statement.

         10)A column of the table that is the result of a <query specifica-
            tion> is possibly nullable if and only if it contains a <column
            reference> for a column C that is possibly nullable, an <indica-
            tor parameter>, an <indicator variable>, a <subquery>, CAST NULL
            AS X (X represents a <data type> or a <domain name>), SYSTEM_
            USER, or a <set function specification> that does not contain
            COUNT.

         192  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   7.9 <query specification>


         11)Let TREF be the <table reference>s that are simply contained
            in the <from clause> of the <table expression>. The simply un-
            derlying tables of the <query specification> are the tables
            identified by the <table name>s and <derived table>s contained
            in TREF without an intervening <derived table>.

         12)A <query specification> QS is updatable if and only if the fol-
            lowing conditions hold:

            a) QS does not specify DISTINCT.

            b) Every <value expression> contained in the <select list> imme-
              diately contained in QS consists of a <column reference>, and
              no <column reference> appears more than once.

            c) The <from clause> immediately contained in the <table ex-
              pression> immediately contained in QS specifies exactly one
              <table reference> and that <table reference> refers either to
              a base table or to an updatable derived table.

              Note: updatable derived table is defined in Subclause 6.3,
              "<table reference>".

            d) If the <table expression> immediately contained in QS imme-
              diately contains a <where clause> WC, then no leaf generally
              underlying table of QS shall be a generally underlying table
              of any <query expression> contained in WC.

            e) The <table expression> immediately contained in QS does not
              include a <group by clause> or a <having clause>.

         13)A <query specification> is possibly non-deterministic if any of
            the following conditions are true:

            a) The <set quantifier> DISTINCT is specified and one of the
              columns of T has a data type of character string; or

            b) The <query specification> directly contains a <having clause>
              that is possibly non-deterministic; or

            c) The <select list> contains a reference to a column C of T
              that has a data type of character string and either

              i) C is specified with a <set function specification> that
                 specifies MIN or MAX, or

             ii) C is a grouping column of T.

         Access Rules

            None.



                                                     Query expressions   193

 





          X3H2-92-154/DBL CBR-002
         7.9 <query specification>


         General Rules

         1) Case:

            a) If T is not a grouped table, then

              Case:

              i) If the <select list> contains a <set function specifica-
                 tion> that contains a reference to a column of T or di-
                 rectly contains a <set function specification> that does
                 not contain an outer reference, then T is the argument or
                 argument source of each such <set function specification>
                 and the result of the <query specification> is a table con-
                 sisting of 1 row. The i-th value of the row is the value
                 specified by the i-th <value expression>.

             ii) If the <select list> does not include a <set function spec-
                 ification> that contains a reference to T, then each <value
                 expression> is applied to each row of T yielding a table of
                 M rows, where M is the cardinality of T. The i-th column of
                 the table contains the values derived by the evaluation of
                 the i-th <value expression>.

                 Case:

                 1) If the <set quantifier> DISTINCT is not specified, then
                   the table is the result of the <query specification>.

                 2) If the <set quantifier> DISTINCT is specified, then the
                   result of the <query specification> is the table derived
                   from that table by the elimination of any redundant
                   duplicate rows.

            b) If T is a grouped table, then

              Case:

              i) If T has 0 groups, then the result of the <query specifica-
                 tion> is an empty table.

             ii) If T has one or more groups, then each <value expression>
                 is applied to each group of T yielding a table of M rows,
                 where M is the number of groups in T. The i-th column of
                 the table contains the values derived by the evaluation of
                 the i-th <value expression>. When a <value expression> is
                 applied to a given group of T, that group is the argument
                 or argument source of each <set function specification> in
                 the <value expression>.

                 Case:

                 1) If the <set quantifier> DISTINCT is not specified, then
                   the table is the result of the <query specification>.

         194  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                   7.9 <query specification>


                 2) If the <set quantifier> DISTINCT is specified, then the
                   result of the <query specification> is the table derived
                   from T by the elimination of any redundant duplicate
                   rows.

         Leveling Rules

         1) The following restrictions apply for Intermediate SQL:

            a) The <set quantifier> DISTINCT shall not be specified more
              than once in a <query specification>, excluding any <sub-
              query> of that <query specification>.

         2) The following restrictions apply for Entry SQL in addition to
            any Intermediate SQL restrictions:

            a) A <query specification> is not updatable if the <where
              clause> of the <table expression> contains a <subquery>.

            b) A <select sublist> shall be a <derived column>.

            c) If the <table expression> of the <query specification> is a
              grouped view, then the <select list> shall not contain a <set
              function specification>.






























                                                     Query expressions   195

 





          X3H2-92-154/DBL CBR-002
         7.10 <query expression>


         7.10  <query expression>

         Function

         Specify a table.

         Format

         <query expression> ::=
                <non-join query expression>
              | <joined table>

         <non-join query expression> ::=
                <non-join query term>
              | <query expression> UNION  [ ALL ] [ <corresponding spec> ] <query term>

              | <query expression> EXCEPT [ ALL ] [ <corresponding spec> ] <query term>


         <query term> ::=
                <non-join query term>
              | <joined table>

         <non-join query term> ::=
                <non-join query primary>
              | <query term> INTERSECT [ ALL ] [ <corresponding spec> ] <query primary>


         <query primary> ::=
                <non-join query primary>
              | <joined table>

         <non-join query primary> ::=
                <simple table>
              | <left paren> <non-join query expression> <right paren>

         <simple table> ::=
                <query specification>
              | <table value constructor>
              | <explicit table>

         <explicit table> ::= TABLE <table name>

         <corresponding spec> ::=
              CORRESPONDING [ BY <left paren> <corresponding column list> <right paren> ]


         <corresponding column list> ::= <column name list>






         196  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                     7.10 <query expression>


         Syntax Rules

         1) Let T be the table specified by the <query expression>.

         2) The <explicit table>

              TABLE <table name>

            is equivalent to the <query expression>

              ( SELECT * FROM <table name> )

         3) Let set operator be UNION [ALL], EXCEPT [ALL], or INTERSECT
            [ALL].

         4) T is an updatable table and the <query expression> is updatable
            if and only if it simply contains a <query expression> QE or a
            <query specification> QS and:

            a) the <query expression> contains QE or QS without an inter-
              vening <non-join query expression> that specified UNION or
              EXCEPT;

            b) the <query expression> contains QE or QS without an interven-
              ing <non-join query term> that specifies INTERSECT; and

            c) QE or QS is updatable.

         5) Case:

            a) If a <simple table> is a <query specification>, then the
              column descriptor of the i-th column of the <simple table> is
              the same as the column descriptor of the i-th column of the
              <query specification>.

            b) If a <simple table> is an <explicit table>, then the column
              descriptor of the i-th column of the <simple table> is the
              same as the column descriptor of the i-th column of the table
              identified by the <table name> contained in the <explicit
              table>.

            c) Otherwise, the column descriptor of the i-th column of the
              <simple table> is same as the column descriptor of the i-
              th column of the <table value constructor>, except that the
              <column name> is implementation-dependent and different from
              the <column name> of any column, other than itself, of a
              table referenced by any <table reference> contained in the
              SQL-statement.

         6) Case:

            a) If a <non-join query primary> is a <simple table>, then the
              column descriptor of the i-th column of the <non-join query
              primary> is the same as the column descriptor of the i-th
              column of the <simple table>.

                                                     Query expressions   197

 





          X3H2-92-154/DBL CBR-002
         7.10 <query expression>


            b) Otherwise, the column descriptor of the i-th column of the
              <non-join query primary> is the same as the column descriptor
              of the i-th column of the <non-join query expression>.

         7) Case:

            a) If a <query primary> is a <non-join query primary>, then the
              column descriptor of the i-th column of the <query primary>
              is the same as the column descriptor of the i-th column of
              the <non-join query primary>.

            b) Otherwise, the column descriptor of the i-th column of the
              <query primary> is the same as the column descriptor of the
              i-th column of the <joined table>.

         8) If a set operator is specified in a <non-join query term> or a
            <non-join query expression>, then let T1, T2, and TR be respec-
            tively the first operand, the second operand, and the result of
            the <non-join query term> or <non-join query expression>. Let
            TN1 and TN2 be the effective names for T1 and T2, respectively.

         9) If a set operator is specified in a <non-join query term> or a
            <non-join query expression>, then let OP be the set operator.

            Case:

            a) If CORRESPONDING is specified, then:

              i) Within the columns of T1, the same <column name> shall not
                 be specified more than once and within the columns of T2,
                 the same <column name> shall not be specified more than
                 once.

             ii) At least one column of T1 shall have a <column name> that
                 is the <column name> of some column of T2.

            iii) Case:

                 1) If <corresponding column list> is not specified, then
                   let SL be a <select list> of those <column name>s that
                   are <column name>s of both T1 and T2 in the order that
                   those <column name>s appear in T1.

                 2) If <corresponding column list> is specified, then let
                   SL be a <select list> of those <column name>s explic-
                   itly appearing in the <corresponding column list> in
                   the order that these <column name>s appear in the <cor-
                   responding column list>. Every <column name> in the
                   <corresponding column list> shall be a <column name> of
                   both T1 and T2.

             iv) The <non-join query term> or <non-join query expression> is
                 equivalent to:

                   ( SELECT SL FROM TN1 ) OP ( SELECT SL FROM TN2 )

         198  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                     7.10 <query expression>


            b) If CORRESPONDING is not specified, then T1 and T2 shall be of
              the same degree.

         10)Case:

            a) If the <non-join query term> is a <non-join query primary>,
              then the column descriptor of the i-th column of the <non-
              join query term> is same as the column descriptor of the i-th
              column of the <non-join query primary>.

            b) Otherwise,

              i) Case:

                 1) Let C be the <column name> of the i-th column of T1. If
                   the <column name> of the i-th column of T2 is C, then
                   the <column name> of the i-th column of TR is C.

                 2) Otherwise, the <column name> of the i-th column of TR is
                   implementation-dependent and different from the <column
                   name> of any column, other than itself, of any table
                   referenced by any <table reference> contained in the
                   SQL-statement.

             ii) The data type of the i-th column of TR is determined by
                 applying Subclause 9.3, "Set operation result data types",
                 to the data types of the i-th column of T1 and the i-th
                 column of T2. If the i-th column of both T1 and T2 are
                 known not nullable, then the i-th column of TR is known
                 not nullable; otherwise, the i-th column of T is possibly
                 nullable.

         11)Case:

            a) If a <query term> is a <non-join query term>, then the column
              descriptor of the i-th column of the <query term> is the same
              as the column descriptor of the i-th column of the <non-join
              query term>.

            b) Otherwise, the column descriptor of the i-th column of the
              <query term> is the same as the column descriptor of the i-th
              column of the <joined table>.

         12)Case:

            a) If a <non-join query expression> is a <non-join query term>,
              then the column descriptor of the i-th column of the <non-
              join query expression> is the same as the column descriptor
              of the i-th column of the <non-join query term>.





                                                     Query expressions   199

 





          X3H2-92-154/DBL CBR-002
         7.10 <query expression>


            b) Otherwise,

              i) Case:

                 1) Let C be the <column name> of the i-th column of T1. If
                   the <column name> of the i-th column of T2 is C, then
                   the <column name> of the i-th column of TR is C.

                 2) Otherwise, the <column name> of the i-th column of TR is
                   implementation-dependent and different from the <column
                   name> of any column, other than itself, of any table
                   referenced by any <table reference> contained in the
                   SQL-statement.

             ii) The data type of the i-th column of TR is determined by
                 applying Subclause 9.3, "Set operation result data types",
                 to the data types of the i-th column of T1 and the i-th
                 column of T2. If the i-th column of both T1 and T2 are
                 known not nullable, then the i-th column of TR is known
                 not nullable; otherwise, the i-th column of T is possibly
                 nullable.

         13)Case:

            a) If a <query expression> is a <non-join query expression>,
              then the column descriptor of the i-th column of the <query
              expression> is the same as the column descriptor of the i-th
              column of the <non-join query expression>.

            b) Otherwise, the column descriptor of the i-th column of the
              <query expression> is the same as the column descriptor of
              the i-th column of the <joined table>.

         14)The simply underlying tables of a <query expression> are the
            tables identified by those <table name>s, <query expression>s,
            and <derived table>s contained in the <query expression> without
            an intervening <derived table>, an intervening <query specifica-
            tion>, or an intervening <join condition>.

         15)A <query expression> is possibly non-deterministic if

            a) it contains a set operator UNION and ALL is not specified, or
              if it contains EXCEPT or INTERSECT; and

            b) the first or second operand contains a column that has a data
              type of character string.

         16)The underlying columns of each column of QE and of QE itself are
            defined as follows:

            a) A column of a <table value constructor> has no underlying
              columns.

            b) The underlying columns of every i-th column of a <simple
              table> ST are the underlying columns of the i-th column of
              the table immediately contained in ST.

         200  Database Language SQL

 





                                                    X3H2-92-154/DBL CBR-002
                                                     7.10 <query expression>


            c) If no set operator is specified, then the underlying columns
              of every i-th column of QE are the underlying columns of the
              i-th column of the <simple table> simply contained in QE.

            d) If a set operator is specified, then the underlying columns
              of every i-th column of QE are the underlying columns of the
              i-th column of T1 and those of the i-th column of T2.

            e) Let C be some column. C is an underlying column of QE if and
              only if C is an underlying column of some column of QE.

         Access Rules

            None.

         General Rules

         1) Case:

            a) If no set operator is specified, then T is the result of the
              specified <simple table> or <joined table>.

            b) If a set operator is specified, then the result of applying
              the set operator is a table containing the following rows:

              i) Let R be a row that is a duplicate of some row in T1 or of
                 some row in T2 or both. Let m be the number of duplicates
                 of R in T1 and let n be the number of duplicates of R in
                 T2, where m 


## 参考资料
> - [http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)
> - []()
