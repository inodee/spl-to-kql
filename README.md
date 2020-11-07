# KQL for Splunkers: Why this?
The idea is to share some quick notes making it easier for Splunk users to leverage KQL (Kusto), especially giving projects requiring both technologies (Splunk and Azure/Sentinel) or any other hybrid environments. The way the data (stream) is _manipulated_ is of course different, but still quite easy for long-time Splunkers, the idea here is to get a head start before diving into formal KQL documentation.

Please note I have **zero** experience in KQL, so feel free to point any mistakes, add/suggest entries or to fork the content into another repo/project.

If you are looking for _code translators_ or something similar, consider this project (never used though): https://uncoder.io

## How to get started?
For me the easiest was to get access to [Azure's Data Explorer](https://dataexplorer.azure.com) and start playing from there as it provides multiple datasets for interactiing and even allowing charts/dataviz rendering.

You can also start from [MS Tutorials](https://docs.microsoft.com/en-us/azure/data-explorer/write-queries) on how to write KQL queries.

### KQL Doc Reference

[Kusto Query Language (KQL) reference doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)

Also consider this nice cheatsheet doc from Markus Bakker: https://github.com/marcusbakker/KQL/blob/master/kql_cheat_sheet_v01.pdf

# SPL-to-KQL Cheatsheet
SPL Quick Reference doc can be found [here](https://docs.splunk.com/Documentation/Splunk/8.1.0/SearchReference/ListOfSearchCommands).

Notes:
* In SPL we usually refer to _fields_ instead of _columns_. In KQL docs there are many references similar to SQL lang.
* In SPL, every _command_ starts with a pipe (|). Likewise, in KQL, each filter prefixed by the pipe is an instance of an [operator](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/queries).
* Aforementioned pipe char (SPL's command prefix) is suppressed from the table below for simplicity, except for multi-line examples.

| SPL | KQL | Remarks |  Ref/Doc |
| --- | --- | --- | --- |
|<pre>head <n></pre> | <pre>take <n></pre> | `limit` is a synonym. Consider sorting for consitency (SPL's head/tail). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/takeoperator)
|<pre>table <field(s)></pre> | <pre>project <field(s)></pre> | Multiple columns are separated by comma (,). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/projectoperator)
|<pre>fields - <field(s)></pre> | <pre>project-away <field(s)></pre> | Also consider [`project-keep`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/project-keep-operator). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/projectawayoperator)
|<pre>rename source_addr AS src_ip</pre> | <pre>project-rename source_addr = src_ip</pre> | I haven't figured out how to use wildcards. Also check [this](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/rename-column#rename-columns). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/projectrenameoperator)
|<pre>search OS="*win*"</pre>| <pre>where OS contains "win"</pre> | Also consider [`search`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/searchoperator). | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/whereoperator)
|<pre>where OS="Windows 10"</pre>| <pre>where OS=="Windows 10"</pre> | Case sensitive 
|<pre>search OS="windows 10"</pre>| <pre>where OS=~"windows 10"</pre> | Case insensitive 
|<pre>search OS IN ("windows", "linux")</pre>| <pre>where OS in~ ("windows", "linux")</pre> | Case insensitive full-match (implied OR operation)
|<pre>where match(OS, "<regex>")</pre>| <pre>where OS matches regex "<regex>"</pre> | Complies with re2 https://github.com/google/re2/wiki/Syntax
|<pre>eval shake = milk."+".fruit</pre>| <pre>extend shake = strcat(milk, "+", fruit)</pre> | Many more string operators [here](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/datatypes-string-operators) | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/extendoperator)
|<pre>eval sum = num1 + num2</pre>| <pre>extend sum = num1 + num2</pre> | Also consider understanding [`let`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/letstatement) statement
|<pre>base search for StormEvents<br>\| stats count AS record_count</pre>| <pre>StormEvents<br>\| summarize record_count=count()</pre>| Also consider [`count`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/countoperator) operator. Similar use for distinct counting with [`dcount`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/dcount-aggfunction)| [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/count-aggfunction)
|<pre>base search for StormEvents<br>\| stats count(eval(len(State)>10)) AS long_count</pre>| <pre>StormEvents<br>\| summarize long_count=countif(strlen(State)>10)</pre>| Also consider [`count`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/countoperator) operator| [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/countif-aggfunction)
|<pre>base search for StormEvents<br>\| stats dc(eval(match(state, "^I"))) AS dcount</pre>| <pre>StormEvents<br>\| summarize dcount=dcountif(State, State startwith "I")</pre>| Also consider [`count`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/countoperator) operator| [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/dcountif-aggfunction)
|<pre>base search for StormEvents<br>\| stats c by State, EventType<br>\| sort 5 -num(c)</pre>| <pre>StormEvents<br>\| summarize c=count() by State, EventType<br>\| top 5 by c</pre>| KQL's [`top`](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/topoperator) behaves differently (_EventType_ is kept in the output) rather than SPL's transformation [`top`](https://docs.splunk.com/Documentation/Splunk/6.5.0/SearchReference/Top) (see below)| [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/topoperator)
|<pre>base search for StormEvents<br>\| top 5 State</pre>| <pre>StormEvents<br>\| summarize c=count() by State<br>\| top-hitters 5 of State by c</pre>| A combination of `summarize`, `sort` and `take` is also possible here | [Doc](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/tophittersoperator)

