# vscpl2drv-table

<img src="https://vscp.org/images/logo.png" width="100">

    Available for: Linux, Windows
    Driver Linux: vscpl2drv-table.so
    Driver Windows: vscpl2drv-table.dll

A driver with functionality that comes from the pre 14.0.0 version of the VSCP daemon and contains measurement database handling.

VSCP tables was a built in feature of the VSCP daemon that now is available in this driver and that let you collect data (measurements) from events in a SQL database over time in an automatic way. The only thing you need to do is to add an entry for the collection of data and a definition of your collection to the configuration.

You can build two types of tables

*  **Dynamic table** which works like any other database and save data in an ever growing database.

*  **Static table** which is a table with a static number of record. A table of this type can for example collect temperatures from temperature sensor over the last 24 hours and start all over with this collectiuon when tyhe 24 hour period has elapsed.

*  **Max table** which is a table that grows up to a specified **size** of elements and then clear all elements and start to fill the table again.

The collected data can be reached over


*  The tcp/ip interface.
*  The websocket interface.
*  MQTT

The interfaces allow you to do statistical analysis of the data but the main use for the collected data is to display it in a user interface element such as a diagram or a table. The way VSCP handles measurements one display solution will work for all types of measurements. For a JavaScript application data over specific ranges or hole (static) tables can be fetched in an easy way over the tcp/ip or the websocket API's. Examples are provided. For higher level languages such as C/C++/C#/Java/Python etc the VSCP helper library is a great tool to fetch datasets.

Tables are by default stored in

*  **/var/lib/vscp/tables** on Linux type systems.
*  **/programdata/vscp/tables** on Windows systems.
*  **In memory** on both Linux type and Windows systems.

The in memory type is very fast but is of course not persistent over time.

## Configuration

### Linux

#### VSCP daemon driver config

The VSCP daemon configuration is (normally) located at */etc/vscp/vscpd.conf*. To use the vscpl2drv-table driver there must be an entry in the

```

```

section on the following format

```xml
<!-- Level II table -->
<driver enable="true"
    name="table"
    driver-path="/var/lib/vscp/drivers/level2/vscpl2drv-table.so"
    config-path="/var/lib/vscp/vscp-table-drv.json"
    guid="FF:FF:FF:FF:FF:FF:FF:FC:88:99:AA:BB:CC:DD:EE:FF"
</driver>
```

##### enable
Set enable to "true" if the driver should be loaded.

##### name
This is the name of the driver. Used when referring to it in different interfaces.

##### driver-path
This is the path to the driver. If you install from a Debian package this will be */usr/bin/vscpl2drv-table.so* and if you build and install the driver yourself it will be */usr/local/bin/vscpl2drv-table.so* or a custom location if you configured that.

#### config-path
The path to the JSON configuration file for the driver

##### guid
All level II drivers must have a unique GUID. There is many ways to obtain this GUID, Read more [here](https://grodansparadis.gitbooks.io/the-vscp-specification/vscp_globally_unique_identifiers.html).

#### vscpl2drv-table driver config

On start up the configuration is read from the path set in the driver configuration of the VSCP daemon, usually */etc/vscp/conf-file-name* and values are set from this location. If the **write** parameter is set to "true" the above location is a bad choice as the VSCP daemon will not be able to write to it. A better location is */var/lib/vscp/drivername/configure.xml* or some other writable location in this cased.

You can define and configure tables in the driver configuration file and manipulate and save configurations with HLO (high level object) events.

The configuration file have the following format

```xml
<?xml version = "1.0" encoding = "UTF-8" ?>
    <!-- Version 0.0.1    2019-11-05   -->
    <config debug="true|false"
            write="true|false"
            guid="FF:FF:FF:FF:FF:FF:FF:FC:88:99:AA:BB:CC:DD:EE:FF"
            zone="1"
            subzone="2"  >
        <table ..... />
        <table ..... />
        .....
        <table ..... />
    </config>

```

##### debug
Set debug to "true" to get debug information written to syslog. This can be a valuable help if things does nor behave as expected.

##### write
If write is true a configuration file will be looked for in */var/lib/vscp/drivername/configure.xml*. If this file is found it wil be read and any value present in it will replace a value read from the main configuration file.

##### guid
All level II drivers must have a unique GUID. There is many ways to obtain this GUID, Read more [here](https://grodansparadis.gitbooks.io/the-vscp-specification/vscp_globally_unique_identifiers.html).

##### zone
Enter the zone that should be used for events from the driver. Any number is good if you do not intend to check this field in events. Default is zero.

##### subzone
Enter the subzone that should be used for events from the driver. Any number is good if you do not intend to check this field in events. Default is zero.

#### The table definition

```xml
<table enable="true|false"
    name="testtable1"
    bmemory="false|true"
    type="dynamic|static|max"
    size="0"

    owner="admin"
    permission="0x777"

    vscp-class="10"
    vscp-type="6"
    vscp-guid="00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00"
    vscp-sensor-index="0"
    vscp-unit="1"
    vscp-zone="0"
    vscp-subzone="0"

    xname="x-label name"
    yname="y-label name"
    title="Title for diagram"
    note="Note for diagram"

    sql-create="SQL expression to create table"
    sql-insert="SQL expression to insert value into table"
    sql-delete="SQL expression to delete table"

    description="Description of database" />
```

This is the definition of a table. It defines the information for the table and what VSCP measuremeny event that should be collected.

##### enable

If set to true (enable="true") the table will be read in. If set to false (enable="false" the table will not be read in.

##### name

This is the name of the table. It must be unique in the system and this is also the name the database file will get.

##### bmemory

If bMemory is set to true (bMemory="true") the database will be created in memory. This s a very fast database but the data will be lost when the VSCP daemon is restarted. Care must also be taken on how much you let this database grow in size and therefore it may we wise to make this database static.

If set to "false" (bMemory="false") the database will be created on disk. This means it will be possible to work with by other tools as well and that it is persistent over time.

##### type

This is the type for the table. Two types are available. The first is **dynamic** (type="dynamic") which is a table that grows over time with not limits. The second type is **static** (type="static") which is a table with a fixed size. A fixed size table is perfect if you want to collect data for the last 24 hours, the last week or similar while the normal table is more like a traditional database.

##### size

Size only have meaning for a table with type="static" and then set the max size for the table. Every time data is added to the table the size will be checked and the oldest records will be removed until the size is equal to the value set here.

##### owner

Is the user that owns this table.

##### permission

Is the rights for users to read/write records in this table.

##### vscp-class

This is the VSCP class data is collected from.

##### vscp-type

This is the type of the VSCP class data is collected from.

##### vscp-guid

This is the GUID for the sender of the collected event. For a level I node set to the interface GUID with the node nickname in the LSB.

##### vscp-sensor-index

This is the sensor index for the sensor data is collected from. Default to zero. Can be max 7 for Level I events and have a value of max 255 for Level II events.

##### vscp-unit

This is the unit data is collected for. Defaults to the default unit zero. Not used if the event dos not have a zone specified. Can be max 3 for Level I events and have a value of max 255 for Level II events.

##### vscp-zone

This is the zone the sensor data is collected from should be part of. Defaults to 255 (all zones). Not used if the event dos not have a zone specified.

##### vscp-subzone

This is the subzone the sensor data is collected from should be part of. Defaults to 255 (all subzones).

##### xname

This is the x label name for a UI diagram or the column name for a UI table. Used for click and play diagrams and tables.

##### yname

This is the Y label name for a UI diagram or the column name for a UI table. Used for click and play diagrams and tables.

##### title

This is the title for a UI diagram or a UI table. Used for click and play diagrams and tables.

##### note

This is a note for a UI diagram or a UI table. Used for click and play diagrams and tables.

##### sql-create (optional)

This is the SQL expression used to create the database for the table if it does not exist. For a in memory table this means every time the VSCP daemon is started. If this attribute is left blank it will be set to the following SQL expression

```sql
CREATE TABLE 'vscptable' (
    `idx` INTEGER NOT NULL PRIMARY KEY UNIQUE,
    `datetime` TEXT,
    `value` REAL DEFAULT 0 );
```

But you can use your own SQL expression to create tables with other fields. The following restrictions is the only ones


*  One table must be named **vscptable** but you are free to create any number of tables with other names in the database.

*  The vscptable must have the field **datetime** defined which is of type **TEXT** and the field **value** defined which is of type **REAL**.

Before the expression is executed by the SLQ engine [VSCP decision matrix escapes](./decision_matrix.md#variable_substitution_for_parameters_escapes) are replaced with real values.

##### sql-insert (optional)

This is the SQL expression used to insert data into the table. If this attribute is left blank it will be set to the following SQL expression

```sql
INSERT INTO 'vscptable' (datetime,value) VALUES ('%%s','%%f');
```

You can set your own SQL expression. But **datetime** must be followed by a %%s somewhere and in the same way the **value** need to be followed by %%f somewhere. The order does not matter. %%s will be replaced by %s (string insert) by the [VSCP decision matrix escapes](./decision_matrix.md#variable_substitution_for_parameters_escapes) handling and %%f will be replaced by %lf (floating point (double) value insert) by the [VSCP decision matrix escapes](./decision_matrix.md#variable_substitution_for_parameters_escapes) handling.

Before the expression is executed by the SQL engine [VSCP decision matrix escapes](./decision_matrix.md#variable_substitution_for_parameters_escapes) are replaced with real values. So any number of other values and content of variables can be inserted in the table when a new data point is inserted.

##### sql-delete (optional)

To be defined

#####  description (optional)

This is a description of the table.

##  Example 1

```xml
<table enable="true"
    bmemory="false"
    type="static"
    name="test1"
    owner="admin"
    permission="0x777"
    labelx="Time"
    labely="Temperature in degrees Celsius"
    title="Temperature outside"
    note="Diagram showing temperature outside at Donald Ducks house at the North pole fetched from a Kelvin NTC10K module."
    size="0"
    sqlcreate="CREATE TABLE 'vscptable' ( `idx` INTEGER NOT NULL PRIMARY KEY UNIQUE, `datetime` TEXT, `value` REAL DEFAULT 0, `event` TEXT, 'timestamp  INTEGER DEFAULT 0' );"
    sqlinsert="INSERT INTO 'vscptable' (datetime,value,event,timestamp) VALUES ('%%s','%%f','%event','%event.timestamp');"
    sqldelete="DELETE FROM 'vscptable' WHERE idx IN (SELECT idx FROM temp ORDER BY idx ASC LIMIT 1);"
    vscpclass="10"
    vscptype="6"
    vscpsensorindex="0"
    vscpunit="1"
    vscpzone="255"
    vscpsubzone="255"
    description="Temperature data collections from the North pole s:t Clause house, Kelvin NTC10K, Sensor 3"
/>
```

## High level functionality

### Remote variable

Send a HLO (High Level Object) event to the driver with the following payload to read a remote variable

```xml
<vscp-cmd  op = "readvar"
           name = "xvar"
           full = "false"
}
```

response will be

```xml
<vscp-resp op="readvar"
            name="xvar"
            result="true"
            type="15"
            value="BASE64(HH:MM:SS)"
            ---- more attributes for full="true"
}
```

where

**result** is 'true' for success and 'false' if not.

**type** is **15** indicating that this is a remote variable with a value that should be interpreted as a time.

**value** in this example is the BASE64 encoded value of the time string HH:MM:SS.

You can read the following remote variable values from the vscpl2drv-table driver

| Variable | Type | Description |
| -------- | :----: | ----------------- |
| count | 15 (TIME) | Number of defined tables. |

### Commands


| Command | Description |
| ------- | ----------- |
| Create | Create a new table |
| Delete | Delete a table |
| Log | Log table data |
| list | list tables |
| save | Save configuration to disk (if writable) |
| load | Load configuration from disk |

With table commands you can write and readetc table data. You can also create/delete/manage tables-

The VSCP daemon can handle three types of tables.


*  Dynamic tables which is ever growing tables that will grow until all disk space is used or deleted or cleared by a users. Just as any database.

*  Static tables which consist of a limited set of data which is filled in up to a specified size and when that size is reached the number of items in the table will be kept at this level by removing the oldest records.

*  Max tables which is tables that just like static tables have a set size but which collect data up to the set size and then delete all content and start to collect data again.

Both are intended as a way for time-stamp/measurement pairs to be logged in an easy way and then being displayed in diagram form in a user UI.

### create table

```xml
<vscp-cmd  op = "create-table"
    enable="true"
    bmemory="false"
    type="static"
    name="test1"
    owner="admin"
    permission="0x777"
    labelx="Time"
    labely="Temperature in degrees Celsius"
    title="Temperature outside"
    note="Diagram showing temperature outside at Donald Ducks house at the North pole fetched from a Kelvin NTC10K module."
    size="0"
    sqlcreate="CREATE TABLE 'vscptable' ( `idx` INTEGER NOT NULL PRIMARY KEY UNIQUE, `datetime` TEXT, `value` REAL DEFAULT 0, `event` TEXT, 'timestamp  INTEGER DEFAULT 0' );"
    sqlinsert="INSERT INTO 'vscptable' (datetime,value,event,timestamp) VALUES ('%%s','%%f','%event','%event.timestamp');"
    sqldelete="DELETE FROM 'vscptable' WHERE idx IN (SELECT idx FROM temp ORDER BY idx ASC LIMIT 1);"
    vscpclass="10"
    vscptype="6"
    vscpsensorindex="0"
    vscpunit="1"
    vscpzone="255"
    vscpsubzone="255"
    description="Temperature data collections from the North pole s:t Clause house, Kelvin NTC10K, Sensor 3"
/>
```

Create a new table with name 'table-name' using the supplied parameters described in config above.

 Some parameters which could have *invalid characters* in them can be encoded in BASE64 and if so should be preceded with "BASE64:".

 | Parameter   | Description  |
 | ---------   | -----------  |
 | tblname     | Name to be used for the table. Must not already be used for a table.                                                            |
 | bEnable     | "true" or "false" to enable or disable the loading of the table.                                                                |
 | bInMemory   | "true" or "false" to create the table in memory or on disk.                                                                     |
 | type        | "dynamic", "static" or "max" for the type of table to construct,                                                                |
 | size        | Size for the table when it is of type "static" or "max". Ignored for other types.                                               |
 | owner       | The user that should own the table. The user should exist.                                                                      |
 | rights      | Rights for usage of the table.                                                                                                  |
 | title       | Title of the table. If "BASE64:" is preceding this value it is decoded from base64 before use.                                  |
 | xname       | X axis label. If "BASE64:" is preceding this value it is decoded from base64 before use.                                        |
 | yname       | y axis label. If "BASE64:" is preceding this value it is decoded from base64 before use.                                        |
 | note        | Note used for table diagram. If "BASE64:" is preceding this value it is decoded from base64 before use.                         |
 | sqlcreate   | SQL statement to create table. If "BASE64:" is preceding this value it is decoded from base64 before use.                       |
 | sqlinsert   | SQL statement used to insert a value into the table. If "BASE64:" is preceding this value it is decoded from base64 before use. |
 | sqldelete   | SQL statement used to delete values from table. If "BASE64:" is preceding this value it is decoded from base64 before use.      |
 | description | Description of tables use. If "BASE64:" is preceding this value it is decoded from base64 before use.                           |
 | vscpclass   | VSCP class for data in table.                                                                                                   |
 | vscptype    | VSCP type for data in the table.                                                                                                |
 | sensorindex | Sensor index for data in the table (optional).                                                                                  |
 | unit        | Unit used for data in the table (optional).                                                                                     |
 | zone        | Zone use for data in the table (optional).                                                                                      |
 | subzone     | Subzone used for data in the table (optional).                                                                                  |

### delete table

```xml
<vscp-cmd  op = "delete-table"
           name="name of table"
/>

Delete an existing table and optional remove it's associating database file if the last argument is 'true'. Default is to leave the database file. If this is done and a new table with the same name later is created this file will be reused. This can cause troubles if the two tables have different columns defined.

**'del'** can be used instead of **'delete'**

### list tables

```xml
<vscp-cmd  op = "list-all-tables"
           format="comma|xml|json"
/>

List all available tables.

**format** selects the format for the value. *comma* is a comma separated list, *xml* is XML format and *json* is JSON format.

The response is

```xml
<vscp-resp op="list-tables"
            name="name-of-table"
            result="true"
            type="1"
            value="BASE64(str)"
/>
```

#### comma separated response format

```
    'count' rows<CR><LF>
    table-name1;table-type;description<CR><LF>
    table-name2;table-type;description<CR><LF>
    ...
    table-namen;table-type;description<CR><LF>
    +OK - Success.<CR><LF>
```

#### XML response format
```xml
<list count="n">
<table name="table-name",
        type="dynamic | static | max"
        description="BASE64 encoded description"
        >
</list>
```

#### comma separated response format
```json
{
    "count"  : n,
    "tables" : [
        {
            "name" : "table name 0",
            "type" : "dynamic | static | max",
            "description" : "BASE64 encoded description"
        },
        {
            "name" : "table name 1",
            "type" : "dynamic | static | max",
            "description" : "BASE64 encoded description"
        },
        {
            "name" : "table name n-1",
            "type" : "dynamic | static | max",
            "description" : "BASE64 encoded description"
        }
    ]
}
```

Where type is either 'dynamic', 'static' or 'max'.

Description will be BASE64 encoded (preceded with "BASE64:").

Response can be multievent if many tables are defined.

### list table

```xml
<vscp-resp op="list-table"
            name="name-of-table"
            result="true"
            format="comma|xml|json"
            value="BASE64(str)"
/>
```

List full information for a specific table with name 'table-name'. The standard output is

```json
    name=table-name<CR><LF>
    enable=true|false<CR><LF>
    type="dynamic"|"static"|"max"//<CR><LF>
    bmemory=true|false<CR><LF>
    size=Size of table or zero<CR><LF>
    owner=name of owner<CR><LF>
    permission=user permissions<CR><LF>
    description=Free text description of the table<CR><LF>
    xname=diagram x-axis text<CR><LF>
    yname=diagram y-axis text<CR><LF>
    title=diagram title<CR><LF>
    note=diagram note<CR><LF>
    vscp-class=n<CR><LF>
    vscp-type=n<CR><LF>
    vscp-sensor-index=n<CR><LF>
    vscp-unit=n<CR><LF>
    vscp-zone=n<CR><LF>
    vscp-subzone=n<CR><LF>
    sql-create=Table create SQL expression<CR><LF>
    sql-insert="Table insert SQL expression<CR><LF>
    sql-delete="Table delete SQL expression<CR><LF>
    +OK - Success.<CR><LF>
```

If the optional argument **'xml'** is given the output will be a BASE64 encoded XML content with the following format

```xml
<table
    name = "test_table1"
    benable = "true"
    binmemory = "false"
    type = "dynamic"
    size = "0"
    owner = "admin"
    rights = "0x777"
    title = "This is a title"
    xname = "This the x-lable"
    yname = "This is the y-label"
    note = "This is a note"
    sqlcreate = "CREATE TABLE 'vscptable' ( `idx` INTEGER NOT NULL PRIMARY KEY UNIQUE, `datetime` TEXT, `value` REAL DEFAULT 0 );"
    sqlinsert = "INSERT INTO 'vscptable' (datetime,value) VALUES ('%%s','%%f');"
    sqldelete = ""
    description = "This is the description"
    vscpclass = "10"
    vscptype = "6"
    sensorindex = "0"
    unit = "1"
    zone = "0"
    subzone = "0"
/>
```

which will be outputted as


	  PHRhYmxlcz4gICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIA0KPHRhYmxlIA0KICAgIG5hbWUgPSAidGVzdF90YWJsZTEiDQogICAgYmVuYWJsZSA9ICJ0cnVlIg0KICAgIGJpbm1lbW9yeSA9ICJmYWxzZSINCiAgICB0eXBlID0gImR5bmFtaWMiDQogICAgc2l6ZSA9ICIwIg0KICAgIG93bmVyID0gImFkbWluIg0KICAgIHJpZ2h0cyA9ICIweDc3NyINCiAgICB0aXRsZSA9ICJUaGlzIGlzIGEgdGl0bGUiDQogICAgeG5hbWUgPSAiVGhpcyB0aGUgeC1sYWJsZSINCiAgICB5bmFtZSA9ICJUaGlzIGlzIHRoZSB5LWxhYmVsIg0KICAgIG5vdGUgPSAiVGhpcyBpcyBhIG5vdGUiDQogICAgc3FsY3JlYXRlID0gIkNSRUFURSBUQUJMRSAndnNjcHRhYmxlJyAoIGBpZHhgIElOVEVHRVIgTk9UIE5VTEwgUFJJTUFSWSBLRVkgVU5JUVVFLCBgZGF0ZXRpbWVgIFRFWFQsIGB2YWx1ZWAgUkVBTCBERUZBVUxUIDAgKTsiDQogICAgc3FsaW5zZXJ0ID0gIklOU0VSVCBJTlRPICd2c2NwdGFibGUnIChkYXRldGltZSx2YWx1ZSkgVkFMVUVTICgnJSVzJywnJSVmJyk7Ig0KICAgIHNxbGRlbGV0ZSA9ICIiDQogICAgZGVzY3JpcHRpb24gPSAiVGhpcyBpcyB0aGUgZGVzY3JpcHRpb24iDQogICAgdnNjcGNsYXNzID0gIjEwIg0KICAgIHZzY3B0eXBlID0gIjYiDQogICAgc2Vuc29yaW5kZXggPSAiMCINCiAgICB1bml0ID0gIjEiDQogICAgem9uZSA9ICIwIg0KICAgIHN1YnpvbmUgPSAiMCINCi8+CQ0KPC90YWJsZXM+DQo=<CR><LF>
	  +OK - Success.<CR><LF>


### get

```xml
<vscp-cmd op="get"
            name="name-of-table"
            from="start date"
            to="end date"
            bfull="true|false"
            format="comma|xml|json"
/>
```

```xml
<vscp-resp op="get"
            name="name-of-table"
            result="true"
            count="n"
<row idx="0"
        dt="YY-MM-DDTHH:MM:SSsss"
        val=""
        fileldx1="data"
        fileldx2="data"
        ....
        />
<row idx="1"
        dt="YY-MM-DDTHH:MM:SSsss"
        val=""
        fileldx1="data"
        fileldx2="data"
        ....
        />
...
<row idx="n-1"
        dt="YY-MM-DDTHH:MM:SSsss"
        val=""
        fileldx1="data"
        fileldx2="data"
        ....
        />
/>
```

Get table data from a named table over an optional interval **to - from** which each should be on the form (ISO 8601) YY-MM-DDTHH:MM:SS.

If "full" is specified all fields of the table is listed. **datetime** is always listed first followed by **value** and after that the rest of the fields are listed.


### log 'table-name' value [datetime]

```xml
<vscp-cmd op="log"
            name="name-of-table"
            datetime="Date time for log value"
            value="Floating point value"
/>
```

Response is

```xml
<vscp-resp op="log"
            name="name-of-table"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

log data to a named table.

Table-name and value must be given. Table-name must be a valid and existing table. The value should be a floating point value. A period ('.') is always used as a decimal separator regardless of language settings. The datettime value is on standard ISO date/time format (YYYY-MM-DDTHH:MM:SS.sssZ) and should always be given as UTC time. The "sss" is millisecond part.

If datetime is not given the current UTC time will be used.

### logsql

```xml
<vscp-cmd op="logsql"
            name="name-of-table"
            sql="sqlexpression"
/>
```

Response is

```xml
<vscp-resp op="logsql"
            name="name-of-table"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Log data to a named table using custom SQL expression.

Regardless to say this is a **very dangerous** command and you need the highest privileges to use it as a user which normally means admin privileges.

### records

```xml
<vscp-cmd op="count"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="count"
            name="name-of-table"
            count="number of records"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the number of records in the database over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### firstdate

```xml
<vscp-cmd op="firstdate"
            name="name-of-table"
            from="start date"
            to="end date"6
/>
```

Response is

```xml
<vscp-resp op="firstdate"
            name="name-of-table"
            firstdate="first date"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the first date in a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### lastdate

```xml
<vscp-cmd op="lastdate"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="lastdate"
            name="name-of-table"
            lastdate="last date"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the last date in a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### sum

```xml
<vscp-cmd op="sum"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="sum"
            name="name-of-table"
            sum="sum of values"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the sum of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### min

```xml
<vscp-cmd op="min"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="min"
            name="name-of-table"
            min="min value in range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the minimum value of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### max

```xml
<vscp-cmd op="max"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="max"
            name="name-of-table"
            max="max value in range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the max of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### average

```xml
<vscp-cmd op="average"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="average"
            name="name-of-table"
            average="average value for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the average of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### median

```xml
<vscp-cmd op="median"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="median"
            name="name-of-table"
            median="median value for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the median / third quartile of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.


### stddev

```xml
<vscp-cmd op="stddev"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="stddev"
            name="name-of-table"
            stddev="stddev value for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the standard deviation of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### variance

```xml
<vscp-cmd op="variance"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="variance"
            name="name-of-table"
            variance="variance for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the variance of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### mode

```xml
<vscp-cmd op="mode"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="mode"
            name="name-of-table"
            mode="mode value for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the mode of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### lowerq

```xml
<vscp-cmd op="lowerq"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="lowerq"
            name="name-of-table"
            lowerq="lowerq value for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the first quartile / lower quartile of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### upperq

```xml
<vscp-cmd op="upperq"
            name="name-of-table"
            from="start date"
            to="end date"
/>
```

Response is

```xml
<vscp-resp op="upperq"
            name="name-of-table"
            upperq="upperq value for range"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Statistical function that return the third quartile / upper quartile of all values over a date range *from to* or over the full table if no range is given. The date time ranges should be given in ISO format YYYYMMDDTHH:MM:SS as usual.

### clear

```xml
<vscp-cmd op="clear"
            name="name-of-table"
/>
```

Response is

```xml
<vscp-resp op="clear"
            name="name-of-table"
            result="true|false"
            erro-code="error code if any"
            error="erro if any"
```

Clear records in table, possibly to-from a date time range.




-----

Write table data to a named table. You can access the written table data through the web socket, TCP/IP or the REST interface. Previous version wrote data to a text file on disk, now a full SQL table is used instead.

##### Parameters

Two formats are currently supported selected by the first part of the action parameter string.

**Row type 0**

```javascript
    0;tablename;datetime;value[;[BASE64:]sql-expression]
```


*  **0** - Select the first type.
*  **tablename** - Name of the table (system unique)
*  **datetime** - Date/time on ISO format (YY-MM-DDTHH:MM:SS)
*  **value** - A floating point value for the data point.
*  *Optional* **[BASE64:]sql-expression** - Optional SQL expression. If left out the sqlinsert expression set in the table configuration is used. If set here a SQL expression updating the 'vscptable' is expected. The table always have this name but the database can have other user defined tables which can be updated here. If the SQL expression is preceded by **BASE64:** a conversion from base64 is done before the SQL expression is used. After the decoding from base64 is done the VSCP decision matrix escapes are resolved which so the SQL expression can have VSCP decision matrix escapes in it even if it is encoded in base64.

Before the configured SQL insert expression for the table is used the VSCP decision matrix escapes are resolved. So

   0;test1;%isoboth;%measurement.float

will be translated to

   0;test1;2017-02-17T23:47:19;-19.2

and then if the table stored sqlinsert expression is

```sql
   INSERT INTO 'vscptable' (date,value,event) VALUES ('%%s','%%f','%event')%;
```

will have the escapes handled. That is

*  %%s will be %s
*  %%f will be %f
*  %event will be translated to the string form of the event.
*  %; will be translated to ;  **Important** if you have multi line SQL expressions.


**Row type 1**

```javascript
    1;tablename;[BASE64:]sql-expression]
```


*  0
*  **tablename** Name of table (system unique)
*  sql expression. Se description of SQL expression above.


The requirements for the table is that it should be named **vscptable**, it should have a text field named **time** and a numeric (integer) field named **value**. Apart from this the database can contain any number of additional fields or tables.

You can read more about tables [here](vscp-tables).

----

### Clear Table {#VSCP_DAEMON_ACTION_CODE_CLEAR_TABLE}

`VSCP_DAEMON_ACTION_CODE_CLEAR_TABLE    0x81/129`

Clear data for a named table. An optional SQL expression can be given but normally the table configured delete SQL expression is used. Typical use is when a table should collect data over a period of time and then restart that collection when the time have passes. Note that the MAX table type can be used for this to, at least if the the period of the measurements are constant.

##### Parameters

```javascript
    tablename[;[BASE64:]sql-delete-expression]
```


*  tablename - Name for an existing table.
*  sql-delete-expression - This optional expression can be used to delete the table data. Normally the configured delete expression is used.

You can read more about tables [here](./vscp-tables.md).

----

## Install the driver on Linux
tbd

## Install the driver on Windows
tbd

## How to build the driver on Linux
To build this driver you to clone the driver source

```
git clone --recurse-submodules -j8 https://github.com/grodansparadis/vscpl2drv-table.git
cd vscpl2drv-table
./configure
make
make install
```

Default install folder is */usr/local/lib*

You need build-essentials and git installed on your system

>sudo apt update && sudo apt -y upgrade
>sudo apt install build-essential git

## How to build the driver on Windows
tbd



{% include "./bottom_copyright.md" %}
