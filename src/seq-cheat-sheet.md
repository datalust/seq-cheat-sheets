# Seq 2021 Cheat Sheet

## Installation

### Windows

Download the latest MSI from [datalust.co/download](https://datalust.co/download). Also available on Chocolatey and `winget`.

### Docker

For Linux or Mac users, use the Docker command:

```
$ docker run --name seq -d --restart unless-stopped -e ACCEPT_EULA=Y -p 5341:80 datalust/seq:latest
```

### Kubernetes

Helm instructions at [docs.datalust.co/docs/using-helm](https://docs.datalust.co/docs/using-helm).

## Browse

The Seq UI is served at [http://localhost:5341](http://localhost:5341) by default.

## Navigation

### Events

Search and query logs. Filter events by selecting signals. Create signals from search expressions to index all matching events, for better search performance.

### Dashboards

Create and organize charts based on queries and signals.

### Data > Ingestion

View the volume of logs sent to Seq in the last 24 hours. Create and manage API keys and input apps.

### Data > Storage

View disk usage, check if retention policies are working as expected, add new retention policies.

### Settings

Enabled authentication. Manage Seq configuration. Install and manage Seq apps. Add users (subscription required).

### User Settings

Create and manage personal API keys, default workspace, and more.

## Search expression, or query?

There are two ways to explore log data in Seq:

- **Search expressions** for filtering and signals
- **Queries** for analyzing and dashboarding

Run search expressions and queries in the same "search box" by pressing Enter, or clicking ![the refresh button](https://github.com/datalust/seq-cheat-sheets/blob/main/src/img/refresh-btn.png).

![A screenshot of the Seq search box, at the top left of the Events page](https://github.com/datalust/seq-cheat-sheets/blob/main/src/img/searchbox.png)

## Text Fragments

Text fragments use "double-quotes", and are a shortcut to searching for logs containing the text between the quotes.

### As free text

```
New user created
```

See note below about no double-quotes

### With logical operators

```
"operation" and not "timed out"
```

Double-quotes required when operators are used

### Escaping

```
"New users \"Lucy\" created"
```

The text fragment `"prod"` is equivalent to the expression:

```
@Message like '%prod%' ci or @Exception like '%prod%' ci
```

Note: If there are **no double-quotes** around an input, Seq will try to determine if it is a valid search expression before treating it as free text.

You can click on the small *text* or *lambda* icon that appears below the filter bar, to inspect your input.

## Keywords

Keywords are **not** case-sensitive.

```
ci
and  or  not
for  in
like
true  false
null
if  then  else
select  as  from  stream  where  group  by  having  order  asc  desc  limit  refresh  time  window
```

## Strings

Unlike text fragments, string literals use **'single-quotes'** are **case-sensitive**.

### Exact match

```
Email = 'seq@example.com'
```

Single-quotes required for strings.

### Case-insensitive

```
Environment = 'PRODUCTION' ci
```

### Like operator

```
Environment like 'Prod%'
```

Single-character wildcard, underscore (`_`), also supported.

### Regular expression

```
Source = /System.(Web|Api)/
```

### Escaping

```
@Exception like '%can''t find host%'
```

Use a single-quote to escape a single-quote, `%%` escapes `%` in `like` expressions.

### String functions

|        |   |
| ----------- | -----------: |
| `=  <>  like`      | Comparators       |
| `Length(text)`   |          |
| `ToLower(text)`   |          |
| `ToUpper(text)`   |          |
| `IndexOf(text, pattern)`   |    `pattern` can be a string or regex|
| `StartsWith(text, pattern)`   |          |
| `EndsWith(text, pattern)`   |          |
| `Contains(text, pattern)`   |          |
| `Substring(text, startIndex, length)`   | Pass `length` as `null` to capture to end-of-string |

## Numbers

Integers, decimals (floats), and hexadecimal are all represented as 128-bit decimal values.

### Equality

```
StatusCode = 200
```

### Comparison

```
ElapsedMilliseconds > 1500
```

### Convert string to object

```
StatusCode = ToNumber('401')
```

## Number functions

|        |   |
| ----------- | -----------: |
| `=  <>  >  >=  <  <=`      | Comparators       |
| `+  -  *  /  ^  %`      | Arithmetic operators       |
| `Round(num, places)`   |          |
| `ToNumber(text)`   |          |

## Null

Like JavaScript, Seq treats null as a value which you can compare using `=`, `<>`, `in`, and other operators. By contrast, the `is null` operator (inherited from SQL) checks for **existence**.

### Check if a property exists

```
OrderId is not null
```

`Has(OrderId)` is also valid

### Check if a property exists, and has the value `null`

```
OrderId = null
```

### Conditional `if`/`then`/`else`

```
if Quantity = 0 then 'None' else 'Some'
```

## Boolean and null functions

|        |   |
| ----------- | -----------: |
| `if {expr} then {x} else {y}`      | If/then/else       |
| `is null`  and `is not null`      | Check for existence       |
| `Coalesce(first, second)`   | If `first` is `null`, return `second`, else return `first` |

## Date and time

Seq uses **ticks** — total number of 100 nanosecond intervals since 1 Jan, 2001 — as its time representation.

Note: ticks are **not** the same as milliseconds since Epoch.

There are 10,000 ticks in 1 millisecond.

### Get events after 2pm, 31 Mar 2020 GMT+10

```
@Timestamp > DateTime('2020-03-21 14:00:00 +10')
```

Most valid date and time string formats are supported.

Seq supports **duration literals** like `1d` or `30m` as a shorthand way of writing a duration in ticks.

### Get events in the last day

```
@Timestamp >= Now() - 1d
```

### Get events before 11am GMT+10

```
TimeOfDay(@Timestamp, 10) < 11h
```

## Date and time functions

|   |   |
| ----------- | -----------: |
| `Now()` | Get current time in ticks |
| `DateTime(text)` | Convert DateTime string to ticks |
| `ToIsoString(ticks)` | Convert ticks to ISO-8601 string |
| `TimeOfDay(ticks, offset)` | Convert ticks to time of day |
| `TimeSpan(timespan)` | Convert timespan string to ticks |
| `TotalMilliseconds(ticks)` | Convert ticks to ms |

## Duration units

|   |   |
| ----------- | -----------: |
| `d  h  m  s  ms  us` | Duration units |

`us` is microseconds

## Collections (arrays and objects)

In Seq, collections include **arrays** and **objects**.

### Object sub-properties

```
User.Email = 'seq@example.com'
```

### Match at any index

```
Products[?].Name = 'Coffee'
```

### Match all wildcard

```
Post.Tags[*] in ['Seq', '2021']
```

### Match all wildcard, chained

```
Order.Shipments[*].Items[*].Tax > 0
```

### `in` operator

```
@Level in ['Error', 'Warning']
```

## Collection functions

|   |   |
| ----------- | -----------: |
| `.  [?]  [*]  in` | Dot-accessor, wildcards, and `in` operator |
| `Keys(obj)` | Return array of keys in an object |
| `Values(obj)` | Return array of values in an object |
| `FromJson(string)` | Convert string to array or object |
| `ToJson(obj)` | Convert any value to a string |
| `ElementAt(col, indexer)` | Element at key or index |


## Debugging or testing expressions

Sometimes, we need to check if expressions are doing as we expect.

Debug an expression by wrapping it in a `select <expr> from stream limit 1`.

### Testing `1 + 1` expression output

```
select 1 + 1 from stream limit 1
```

This outputs 2.

## Queries

Seq uses SQL-like query syntax.

### Query syntax

```
select [<column> [as <label>],]
from stream
    [where <predicate>]
    [group by [<grouping>|time(<d>),]]
    [having <predicate>]
    [order by [time|<label>] [asc|desc]]
    [limit <n>]
    [for refresh]
```

`for refresh` is for bypassing the query cache — only use if data is stale.

### Example query

```
select count(*) as count
from stream
where StatusCode > 399
group by RequestPath
order by count desc
limit 10
```

Find the top 10 `RequestPath`s that have returned a `StatusCode` of 400, or above.

## Time slice queries

Seq can generate time series data using the special `time()` grouping, in conjunction with a **duration literal** (e.g. `1d`, `30m`) that determines the time interval to use.

### Count by time slice

```
select count(*)
from stream
group by time(15m)
```

### Count by time slice, grouped by a property

```
select count(*)
from stream
group by Environment, time(1h)
```

### Using the `interval()` function

```
select count(*)/interval() as RateOfOrders
from stream
where @Properties['Order'] is not null
group by time(5m)
limit 1000
```

Tip: Chart your query results as a line, bar, or pie chart directly in the events screen, using these buttons:

![A row of 4 buttons: 'Rowset', 'Line chart', 'Bar graph', 'Pie chart'](https://github.com/datalust/seq-cheat-sheets/blob/main/src/img/chart-button-row.png)

## Duration functions

|   |   |
| ----------- | -----------: |
| `interval()` | Returns the current time grouping duration. When used in a query with `group by time(x)`, returns `x`. |

## Aggregate functions

Aggregate functions perform a calculation on a set of values, and return a single value.

### `any()` `all()`

```
select any(@Level = 'Error') from stream...
```

### `count()`

```
select count(*) from stream
```

Counts all events.

```
select count(IsAdmin) from stream
```

Counts number of events that contain `IsAdmin = true`.

### `distinct()`

```
select distinct(@Level) from stream
```

List all distinct `@Level` values.

```
select count(distinct(@EventType)) from stream
```

Count all distinct event types.

### `min()` `max()` `sum()` `mean()` `percentile()`

```
select min(Elapsed), max(Elapsed) from stream
```

Get smallest and largest `Elapsed`.

```
select min(Elapsed), max(Elapsed), mean (Elapsed), percentile(Elapsed, 90) from stream group by time(1h)
```

Plot aggregates over time.

### `first()` `last()`

```
select first(ExceptionType) from stream where Application <> 'Admissions'
```

## Seq event anatomy, built-in properties, and properties

Seq uses Serilog's *Compact Log Event Format* (CLEF) as its native JSON event format. If you export any event from Set, it will look something like below:

```json
{
  "@t":"2021-01-26T22:37:02.6297213Z",
  "@mt":"HTTP {RequestMethod} {RequestPath} responded {StatusCode}",
  "@m":"HTTP GET /example?q=123 responded 501",
  "@i":123456,  "@l":"Error",
  "@x":"System.Threading.Tasks.TaskCancelledException\nAt line...",
  "RequestMethod":"GET",
  "RequestPath":"/example?q=123",
  "StatusCode":501,
  "Elapsed":0.75233,
  "RequestId":"0AB12C3DE4FGH:00000001",
  "Application":"MyExampleApp",
  "Site":"Production"
}
```

Here is a table that outlines the relationship between CLEF properties and **built-in properties**.

You can use built-in properties in search of expressions and queries.

Note: All property names are case-sensitive.

| Description | CLEF | Built-in property | Notes |
| --- | --- | --- | --- |
| Timestamp | `@t` | `@Timestamp` | Timestamp in ticks |
| Message | `@m` | `@Message` |  |
| Event Type | `@i` | `@EventType` | Maybe a number, or hexadecimal string |
| Event Id | | `@Id` | e.g. 'event-313db38ac31d...' |
| Level | `@l` | `@Level` | |
| Exception | `@x` | `@Exception` | Usually a stack trace |
| Arrival | | `@Arrived` | Number representing order of arrival |
| Properties | | `@Properties` | All event properties without `@` prefix |
| Message Template | `@mt` | `@MessageTemplate` | |
| Data | | `@Data` | Event as JSON object |
| Document | | `@Document` | Event as JSON string |

More details on CLEF at [Serilog formatting compact page](https://github.com/serilog/serilog-formatting-compact).
