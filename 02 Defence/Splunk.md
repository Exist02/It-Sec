# Search Operators
## Relational Operators

|Operator|Example|Explanation|
|---|---|---|
|Equals `=`|`UserName=Mark`|Search for all events in which the field name `UserName` is equal to `Mark`|
|Not Equal To `!=`|`UserName!=Mark`|Search for all events in which the field name `UserName` is not equal to `Mark`|
|Less Than `<`|`Age<10`|The field `Age` has a value of less than `10`|
|Less Than or Equal To `<=`|`Age<=10`|The field `Age` has a value of less than or equal to `10`|
|Greater Than `>`|`Outbound_Traffic>50`|The `Outbound_Traffic` field value is greater than `50`|
|Greater Than or Equal To `>=`|`Outbound_Traffic>=50`|The `Outbound_Traffic` field value is greater than or equal to `50`|
## Logical Operators 
|Operator|Example|Explanation|
|---|---|---|
|`NOT`|`NOT UserName=*`|Returns events where `UserName` field does not exist. Don't confuse it with `!=` operator|
|`AND`|`UserName=David AND IPAddress=10.10.10.10`|Returns all events in which the `UserName` field is equal to `David` and the `IPAddress` field is equal to`10.10.10.10`|
|`OR`|`UserName=David OR UserName=John`|Returns all events in which the `UserName` field is equal to `David` or `John`|
|`IN`|`UserName IN(David, John)`|A more convenient alternative to the `OR` keyword, especially for long lists.|

## Wildcards and CIDR Search

|Symbol|Example|Explanation|
|---|---|---|
|`*`|`status=*fail*`|This will return all events that have `status` field set to `failed`, `failure`, `appfail`, etc.|
|`*`|`DestinationIp=172.*`|This will return all events that contain values like `DestinationIp=172.90.0.0.1` or `DestinationIp=172.18.5.22`|
|`N/A`|`DestinationIp=172.18.0.0/16`|This will return all events where `DestinationIp` field is within the `172.18.0.0/16` subnet|
## Order of Evaluation 

In Splunk, quotation marks `""` are used to define exact phrases or strings. You can wrap text in quotes, and Splunk will treat it as a single value. Quotes can also be used to escape search operators. For example:

- `index=windowslogs failed login`: Search for events with **failed** and **login** keywords, in any order
- `index=windowslogs "failed login"`: Search for the exact phrase "**failed login**", word order matters
- `index=windowslogs "TO BE OR NOT TO BE"`: Search for the exact phrase containing **NOT** and **OR**

Bsp: 

|Your Search|How Splunk Evaluates It|
|---|---|
|`index=windowslogs alice AND bob OR charlie`  <br>  <br>(Implicit search, no parentheses)|`index=windowslogs alice AND (bob OR charlie)`  <br>  <br>(Mistake! Splunk evaluated OR before AND)|
|`index=windowslogs (alice AND bob) OR charlie`  <br>  <br>(Explicit search with parentheses)|`index=windowslogs (alice AND bob) OR charlie`  <br>  <br>(Correct! The results match your requirements)|
# Filtering Results 

## Fields 

The [fields (opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/fields)command is used to include or exclude specific fields from your search results. To exclude a field, use a minus sign `-` before the field name. The plus sign `+` can be used to include a field explicitly, but it isn’t required. By default, `fields` includes any fields listed after the command. Let’s try it out in our Splunk instance by highlighting the following fields. This makes it easy to see how useful the `fields` command can be when working with logs that contain hundreds of available fields.

```
index=windowslogs | fields host User SourceIp
```

## Dedup

The [dedup(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/dedup) command removes duplicate values from your search results. For example, if our logs contain seven distinct IP addresses in the `SourceIp` field, the results will return seven events, one for each unique IP. The command is useful for subsearches and for cleaning identical events (e.g., Microsoft 365 often sends 50 events for a single activity).

```
index=windowslogs
| fields EventID User Image Hostname SourceIp
| dedup SourceIp
```

## Rename 

The [rename(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/rename) command allows you to change the name of a field in your search results. This can help improve the readability of your search results, especially if the original fields are too long or not suitable for showing them in the screenshots in formal SOC reports.

```c
index = windowslogs
| fields EventID User Image Hostname SourceIp
| rename User as Employee
```

The command is also useful to flatten JSON or XML subfields. For example, for a JSON log entry like `{"request": {"path": "/admin", "ip": "10.0.0.2"}}`, Splunk will create two fields: `request.path` and `request.ip`. If you don't want to type the prefix every time, consider removing it like in the example below:

```c
index=jsondata
| rename request.* as * // request.path -> path; request.ip -> ip
```

## Regex

The [regex(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/regex) command allows you to filter search results using regular expressions, which match specific text patterns in field values. This is useful when you need to find events that follow a specific format rather than an exact keyword. Splunk regular expressions are PCRE (Perl Compatible Regular Expressions) and use the PCRE C library.

```
index = windowslogs | regex Image = "\.exe$"
```

The query above applies a regular expression to the `Image` field, returning only events where the field value ends with `.exe`. The `$` symbol specifies that the match must occur at the end of the string. That was the simplest example, but regex is irreplaceable for complex searches, especially on custom or poorly-parsed data sources.

# Structuring Results

## Table 

The [table(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/table) command allows you to select only the fields you are interested in viewing and displays them in a clean, readable format. This is especially useful when building timelines, investigating specific hosts or users, or comparing multiple fields. This query will create a table out of named fields and organize them by timestamp. Use the `table` command to answer the first question.

`index=windowslogs | table _time EventID Hostname SourceName`

https://imgur.com/NvTi8X8

## ## Useful Structuring Commands

Other commands can be used alone or combined with `table` to hone in on the data you're really interested in. Let's look at some examples in the table below.

|Command|Example|Explanation|
|---|---|---|
|`head`|`index=windowslogs \| head 20`|Returns the first (newest) 20 events. Useful to speed up the search if you don't need complete results|
|`tail`|`index=windowslogs \| tail 20`|Returns the last (oldest) 10 events. Useful to speed up the search if you don't need complete results|
|`sort`|`index=windowslogs \| sort User`|This will sort the logs in alphabetical order based on the field `User`|
|`reverse`|`index=windowslogs \| reverse`|This command reverses the order of events (descending order)|

## Timelining With Table

The `table` command can be used to create timelines that help analysts visualize how events unfolded. By organizing key fields, we can reconstruct the sequence of actions that occurred on a system. For example, we can list all actions happening on the `Salena.Adam` host in a chronological order, and then exclude system noise or add additional columns, if required.

```
index = windowslogs Hostname = Salena.Adam
| table _time Hostname EventID Category
| reverse
```

https://imgur.com/hUxkbQz


## Subsearches

Imagine you are reviewing Sysmon process creation events and want to understand their logon context: did a process originate from a remote session (**LogonType 3/10**) or a service (**LogonType 5**)? Sysmon doesn't log the **LogonType** field, so you need to correlate across two data sources: Sysmon ID **1** for the process creation, and Security ID **4624** for the logon context. The **LogonId** field, present in both, is your link between them:

1. You get the **Image,** **User**, **LogonId** from the original Sysmon event (EventID=1)
2. Using **LogonId** field, you find the corresponding Logon event (EventID=4624)
3. You get the **LogonType** and **IpAddress** from the corresponding Logon event

With Splunk [subsearches(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/get-started/search-tutorial/10.2/part-4-searching-the-tutorial-data/use-a-subsearch) and `join` keyword, you can correlate across multiple data sources within one search, and, for our example, build a unified tablecontaining both process and logon details:

https://imgur.com/EYivxc4

```c
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress]
| table _time Image User LogonType IpAddress
```

Let's unwrap the query above step by step:

|   |   |
|---|---|
|The subsearch within `[ ... ]` is executed|- It starts from a simple search (note the **search** command at the beginning)<br>- It renames the field from **TargetLogonId** to **LogonId** to match Sysmon naming<br>- Splunk saves (**LogonId**, **LogonType**, **IpAddress**) tuples in a temporary lookup|
|The main search is executed|- It finds all Sysmon events matching the EventID=1 query<br>- Within each Sysmon event, it looks for the **LogonId** field<br>- If Sysmon **LogonId** matches some of the subsearch results,  <br>    Splunk adds the **LogonType** and **IpAddress** to the main result|
# Transforming Commands

[Transforming commands(opens in new tab)](https://help.splunk.com/en/splunk-cloud-platform/search/search-manual/10.0.2503/create-statistical-tables-and-chart-visualizations/about-transforming-commands-and-searches) allow you to change raw event data into useful summaries, statistics, and visualizations. Instead of viewing every individual log, they help analysts aggregate, count, and analyze patterns across many events. Searches that utilize transforming commands are referred to as transforming searches in Splunk.

## General Transformational Commands

|Command|Example|Explanation|
|---|---|---|
|top|`index=windowslogs \| top User limit=5`|Returns the ten most frequent values of the field specified. A numerical value can be included with `limit=` to reduce or expand the results|
|rare|`index=windowslogs \| rare User limit=5`|Returns the ten least frequent values of the field specified. A numerical value can be included with `limit=` to reduce or expand the results|

**Highlight**

You can use the [highlight(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/highlight) command to visually mark the chosen field values when viewing raw log data. In the example below, we can use the following query to highlight the terms specified. Remember to change the view format from `List` to `Raw` to view the results.

```c
index=windowslogs | highlight User EventID Image "Process accessed"
```


## **Stats**

The [stats(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/stats) command is a powerful tool in Splunk. It allows you to calculate statistics, such as counts, sums, and averages, of fields within your search results. This can be useful when summarizing large volumes of data to identify trends or anomalies. The table below covers some standard `stats` functions.

|Command Function|Example|Explanation|
|---|---|---|
|Average|`stats avg(ProcessCount)`|Calculates the average value of the chosen field|
|Max|`stats max(Price)`|Returns the maximum value of the chosen field|
|Min|`stats min(UserAge)`|Returns the minimum value of the chosen field|
|Sum|`stats sum(Cost)`|Returns the sum of the chosen field values|
|Count|`stats count by SourceIp`|Returns the number of occurrences of the chosen field|

You can try the `stats` command with the example below, which returns the total number of occurrences for each `EventID` and displays them in ascending order.

```c
index=windowslogs | stats count by EventID | sort EventID
```

## Chart 

The [chart(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/chart) command returns your search results in a table, which you can then use to create helpful visualizations. This command utilizes many of the same functions as `stats`. Let's give it a shot to visualize the `count` of events containing the `User` field with this query.

```
index=windowslogs | chart count by User
```

## Timechart

The [timechart(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/timechart) command is used to visualize how data changes over time. It is beneficial for spotting trends, peaks, and anomalies in your log data. In the example below, we use `timechart` to track process activity over time. The following query removes any NULL `Image` field values and creates a time-based area chart showing the top five most frequently occurring process images within 30-minute intervals.

```c
index=windowslogs Image!="" | timechart span=30m count by Image limit=5
```

## Data Enrichment and Field Manipulation

### **IP Location**

You can use the [iplocation(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/iplocation) command to enrich your search results with geographic information about IP addresses. It uses Splunk's built-in geolocation tables to add fields such as `City`, `Region`, and `Country`. Try it out with the query below.

```c
index=windowslogs | iplocation SourceIp | stats count by Country
```

### **Lookup**

Similarly, [lookup(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/lookup) is used to enrich events using external data sources. It matches a field in your search to a corresponding field in a `CSV` file or lookup table. In this example, a `CSV` was created that associates the `Hostname` field with an employee role signified by `UserRole`.

```c
index=windowslogs
| lookup user_roles Hostname OUTPUT UserRole
| stats count by Hostname UserRole
```

### **Eval**

The [eval(opens in new tab)](https://help.splunk.com/en/splunk-enterprise/spl-search-reference/9.0/search-commands/eval) command is one of the most versatile tools in Splunk. It allows you to create new fields, modify existing ones, and perform calculations directly within your searches. It can be used to make data more readable and prepare fields for use in visualizations. In the example below, we created a new field called `LogonTypeDesc` to give a more descriptive name to numeric `LogonType` values.

```c
index=windowslogs
| eval LogonTypeDesc = case(LogonType == 3, "Network Logon", LogonType == 5, "Service")
| stats count by LogonType LogonTypeDesc
```

The query assigns:

- Network Logon when `LogonType` is 3
- Service when `LogonType` is 5

