# SPL-to-KQL Cheatsheet: Why this?
The idea is to simply save some quick notes that will make it easier for Splunk users to leverage KQL (Kusto), especially giving projects requiring both technologies (Splunk and Azure/Sentinel) or any other hybrid environments. The way the data (stream) is _manipulated_ is of course different but still quite easy for long-time Splunkers, the idea here is to get a head start before diving into formal KQL documentation.

Feel free to add/suggest entries or to fork the content into another repo/project.

If you are looking for _code translators_ or something similar, consider this project (never used though): https://uncoder.io

## How to get started?
For me the easiest was to get access to [Azure's Data Explorer](https://dataexplorer.azure.com) and start playing from there as it provides multiple datasets for interactiing and even allowing charts/dataviz rendering.

You can also start from [MS Tutorials](https://docs.microsoft.com/en-us/azure/data-explorer/write-queries) on how to write KQL queries.

### KQL Doc Reference

[Kusto Query Language (KQL) reference doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)

Also consider this nice cheatsheet doc from Markus Bakker: https://github.com/marcusbakker/KQL/blob/master/kql_cheat_sheet_v01.pdf

# Cheatsheet
SPL Quick Reference doc can be found [here](https://docs.splunk.com/Documentation/Splunk/8.1.0/SearchReference/ListOfSearchCommands).

A few notes:
* In SPL we usually refer to _fields_ instead of _columns_. In KQL docs there are many references similar to SQL lang.
* In SPL, every _command_ starts with a pipe char (|). In KQL, each filter prefixed by the pipe character is an instance of an [operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/queries), with some parameters.
* Aforementioned pipe char (SPL's command prefix) is suppressed from the table below for simplicity, except for multi-line examples.

| SPL | KQL | Remarks |  Ref/Doc |
| --- | --- | --- | --- |
|`table <field(s)>` | `project <field(s)>` | This [operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/queries) can also evaluate/calculate new values (more below). In gerenal, fields = columns and multiple parameters usually are separated by comma (,). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/projectoperator)
|`fields - <field(s)>` | `project-away <field(s)>` | Also consider [`project-keep`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/project-keep-operator). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/projectawayoperator)
|`rename source_addr AS src_ip` | `project-rename source_addr = src_ip` | I haven't figured out how to use wildcards (multiple column renaming). Also check [this](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/rename-column#rename-columns). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/projectrenameoperator)
|`search OS="*win*"`| `where OS contains "win"` | Also consider [`search`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/searchoperator). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/whereoperator)
|`where OS="Windows 10"`| `where OS=="Windows 10"` | Case sensitive 
|`search OS="windows 10"`| `where OS=~"windows 10"` | Case insensitive 
|`search OS IN ("windows", "linux")`| `where OS in~ ("windows", "linux")` | Case insensitive full-match (implied OR operation)
|`where match(OS, "<regex>")`| `where OS matches regex "<regex>"` | Complies with re2 https://github.com/google/re2/wiki/Syntax
|`eval mshake = milk."+".fruit`| `extend mshake = strcat(milk + "+" fruit)` | Many more string operators [here](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/datatypes-string-operators) | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/extendoperator)
|`eval sum = num1 + num2`| `extend sum = num1 + num2` | Also consider understanding [`let`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/letstatement) statement
|`base search`<br>`\| top 5 State`|`StormEvents`<br>`\| summarize c=count() by State`<br>`\| top-hitters 5 of State by c` | A combination of `summarize`, `sort` and `take`is also possible here | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/tophittersoperator)
