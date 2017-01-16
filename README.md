## ili2db

ili2db is a suite of DB tool for managing interlis import/export to relational databases.

## 1. ili2pg


Ili2pg is an interlis 2 loader for postgis DB, it translates the intelis 2 data model definitions to a postgis schema. There are several feature for import and export of data from the DB.

### 1.1 USAGE

java -jar ili2pg.jar [Options] [file.xtf|file.ili]
  
### 1.2 Options
-import               import data to the db
-update               update dataset on the db
-replace              replace dataset on the db
-delete               delete data from the db
-export               do an export.
-schemaimport         do an schema import.
-dbhost  host         The host name of the server. Defaults to localhost.
-dbport  port         The port number the server is listening on. Defaults to 5432.
-dbdatabase database  The database name.
-dbusr  username      User name to access database.
-dbpwd  password      Password of user used to access database.
-validConfig file     Config file for validation.
-disableValidation    Disable validation of data.
-deleteData           on schema/data import, delete existing data from existing tables.
-defaultSrsAuth  auth Default SRS authority EPSG
-defaultSrsCode  code Default SRS code 21781
-modeldir  path       Path(s) of directories containing ili-files.
-models modelname     Name(s) of ili-models to generate an db schema for.
-dataset name         Name of dataset.
-baskets BID          Basket-Id(s) of ili-baskets to export.
-topics topicname     Name(s) of ili-topics to export.
-createscript filename  Generate a sql script that creates the db schema.
-dropscript filename  Generate a sql script that drops the generated db schema.
-noSmartMapping       disable all smart mappings
-smart1Inheritance     enable smart1 mapping of class/structure inheritance
-smart2Inheritance     enable smart2 mapping of class/structure inheritance
-coalesceCatalogueRef enable smart mapping of CHBase:CatalogueReference
-coalesceMultiSurface enable smart mapping of CHBase:MultiSurface
-expandMultilingual   enable smart mapping of CHBase:MultilingualText
-createGeomIdx        create a spatial index on geometry columns.
-createEnumColAsItfCode create enum type column with value according to ITF (instead of XTF).
-createEnumTxtCol     create an additional column with the text of the enumeration value.
-createEnumTabs       generate tables with enum definitions.
-createSingleEnumTab  generate all enum definitions in a single table.
-beautifyEnumDispName replace underscore with space in dispName of enum table entries
-createStdCols        generate T_User, T_CreateDate, T_LastChange columns.
-t_id_Name name       change name of t_id column (T_Id)
-idSeqMin minValue    sets the minimum value of the id sequence generator.
-idSeqMax maxValue    sets the maximum value of the id sequence generator.
-createTypeDiscriminator  generate always a type discriminaor colum.
-structWithGenericRef  generate one generic reference to parent in struct tables.
-disableNameOptimization disable use of unqualified class name as table name.
-nameByTopic          use topic+class name as table name.
-maxNameLength length max length of sql names (60)
-sqlEnableNull        create no NOT NULL constraints in db schema.
-strokeArcs           stroke ARCS on import.
-skipPolygonBuilding  keep linetables; don't build polygons on import.
-skipPolygonBuildingErrors  report build polygon errors as info.
-keepAreaRef          keep arreaRef as additional column on import.
-importTid            read TID into additional column T_Ili_Tid
-createBasketCol      generate T_basket column.
-createFk             generate foreign key constraints.
-createFkIdx          create an index on foreign key columns.
-createUnique         create UNIQUE db constraints.
-ILIGML20             use eCH-0118-2.0 as transferformat
-dbschema  schema     The name of the schema in the database. Defaults to not set.
-oneGeomPerTable      If more than one geometry per table, create secondary table.
-log filename         log message to given file.
-gui                  start GUI.
-trace                enable trace messages.
-help                 Display this help text.
-version              Display the version of ili2pg




## Credits

_Developed by Eisenhut Informatik AG, CH-3401 Burgdorf 
 See http://www.interlis.ch for information about INTERLIS
 Parts of this program have been generated by ANTLR; see http://www.antlr.org
 This product includes software developed by the
 Apache Software Foundation (http://www.apache.org/)._