# Helium Analysis Results Document (HARD)

## Introduction

Helium Analysis Results Document (HARD) is a specification for how a processor (e.g. image processor) should return its results to be made accessible through the Helium platform.

The specification is meant to address two conflicting problems. One is to provide a format that does not restrict future advances like new results, new methods, new data types etc. The other is to allow for relatively easy processing on the receiving end in terms of storage and integration.

## Conventions

### Key words

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### We don't use "name"

When going through a formal schema to try and figure out how it works there is nothing more frustrating than unclear naming. Therefore, we have eliminated most cases where the variable name `name` is used.

Instead of lazily just using `name`, we should try state the purpose explicitly in the variable name. A good example of this is a person. A person certainly has a name but before using name think about this. Am I 100% sure that I will never refer to `full_name`, `first_name` or `last_name`? Because if you are not 100% sure you might end up adding attributes creating an ambiguous context where `name` might mean the `full_name` or .

As you can see, the variable name `name` can usually be stated more explicitly like `first_name` or a `last_name`. However, don't overthink it and remember that keep-it-simple-stupid (KISS) should always be considered first.

### We use snake_case

The most widely used are snake_case, camelCase and PascalCase. Some languages use uppercase letters to signal an export and in some languages prefix names with underscore (e.g. `_reserved`) to signal something that should not be used.

In this document and in the schema described within we use snake_case. This is for two simple and very opinionated reasons. Readability and ease-of-use. The snake_case naming convention is unique in a way that it has *built_in_white_spacing* which allows for quicker read-throughs. It also eliminates the need to memorize what is uppercase and what should be lowercase.

Just to be clear, **snake_case** is not considered to be better than any other naming convention and they can all be argued in the same way. The choice of a naming convention just has to be made and for the most part it is a pure matter of taste and familiarity.

## KISS & DRY

The "Keep it simple, stupid" (KISS) principle states that most systems work best if they are kept simple rather than made complicated; therefore simplicity should be a key goal in design and unnecessary complexity should be avoided.

The "Don't repeat yourself" (DRY) principle is stated as "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system".

The above principles are simple and applying them rigorously when designing usually results in a simple, beautiful and maintainable end product.

## Document Structure

### Top Level

The object "analysis object" **MUST** contain __at least one__ of the following top-level members:

* `results`: the set of results returned from processing
* `errors`: an array of error objects

For example, the following analysis contains a single analyzed result:

```json
{
    "results": [{
        "key": "my_variable",
        "type": "text",
        "text_value": "Hello World!"
    }]
}
```

This is a valid "analysis object" containing only errors:

```json
{
    "errors": [{
        "message": "Something went wrong ..."
    }]
}
```

The "analysis object" **MAY** be contained inside an "analysis envelope". The below example illustrates this.

```json
{
    "analysis": {
        "results": [{
            "key": "my_variable",
            "type": "text",
            "text_value": "Hello World!"
        }]
    }
}
```

### Result Objects

"Result objects" appear in a JSON API document to represent processing results.

A "result object" **MUST** contain the following top-level members:

* `key`: the key of a result also referred to as a variable name
* `type`: a reference to the result type of the data contained in the result

In addition, a "result object" **MAY** contain any of these top-level members:

* `metadata`: a metadata object that contains meta-information for the result

**NOTE**: 

#### Identification

Every "result object" **MUST** contain a `key` member. The values of the `key` **MUST** be a string in snake_case as described [here](https://en.wikipedia.org/wiki/Snake_case).

### Specifying Types

A "result object" has a `type` member that **MUST** be one of the following:

* `text`: a single value text result
* `numeric`: a single value numeric result
* `boolean`: a single boolean result
* `table`: a tabular data result with or without headers 
* `chart2d`: a two-dimensional chart with one or more series
* `file`: a result containing a Base64 encoded file

#### Labelling

A "result object" **MAY** have a `label` member that **SHOULD** typically contain a more human-readable alternative to `key`.

```json
{
    "results": [{
        "key": "my_variable",
        "label": "My Variable",
        "type": "text",
        "text_value": "Hello World!"
    }]
}
```

#### Tagging

A tag is a keyword or a term attached to results and are described generally [here](https://en.wikipedia.org/wiki/Tag_(metadata)).

A "result object" **MAY** have a `tags` member that **MUST** be an array of strings in snake_case format (a "tag list").

The following is an example of a result containing a tag:

```json
{
    "results": [{
        "key": "my_variable",
        "type": "text",
        "text_value": "Hello World!",
        "tags": ["my_tag"]
    }]
}
```

### Result Types

Result types define the expected data structure of a "result object" that a processor returns.

#### text

The "text" result type defines is a single value text result.

A "text result object" **MUST** contain the following members at top-level:

* `text_value`: the text value of the result

The following defines a basic text result:

```json
{
    "results": [{
        "key": "my_variable",
        "type": "text",
        "text_value": "Hello World!"
    }]
}
```

#### numeric

The "numeric" result type defines is a single value numerical result. A numerical value can be any valid number specified [here](http://json.org)

A "numeric result object" **MUST** contain the following members at top-level:

* `numeric_value`: the numeric value of the result that **MUST** be either a number.

The following defines a basic numeric result:

```json
{
    "results": [{
        "key": "my_variable",
        "type": "numeric",
        "numeric_value": 3.1415
    }]
}
```

#### bool

The "bool" result type defines a single boolean (true/false) result as defined [here](http://json.org).

A "boolean result object" **MUST** contain the following members at top-level:

* `bool_value`: the boolean value of the result

It **MAY** also include any the following members:

* `message`: a text describing the boolean value in more detail

The following defines a basic boolean result with a message:

```json
{
    "results": [{
        "key": "my_variable",
        "type": "bool",
        "boolean_value": true,
        "message": "All checks are go!"
    }]
}
```

#### file

A "file" defines a single file result.

A "file result object" **MUST** only contain one of the following at top-level:

* `file_path`: a path to the result file that **MUST** be a valid [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)
* `file_data`: an inline inclusion a file's contents that **MUST** be valid according to the [data URI scheme](https://en.wikipedia.org/wiki/Data_URI_scheme)

If the `file_data` member is used you **MUST** also define the following members:

* `file_name`: a string that **MUST** contain a valid URL safe filename

#### table

The "table" result type defines multi-value structure that **MAY** contain text, numbers and booleans.

A "table result object" **MUST** contain the following members at top-level:

* `table_columns`: an array of "table column objects" that **MUST** contain __at least one__ object
* `table_rows`: an array containing arrays of length equal to the length of `table_columns`

The `table_rows` member can contain any number of "rows" but each row **MUST** be an array of equal length to `table_columns`. Additionally a row within `table_rows` **MUST** only contain numbers, text or boolean.

Additionally a "table result object" **MAY** contain the following members:

* `table_transpose`: a boolean value indicating whether this table is transposed
* `table_row_headers`: a boolean value indicating that the first column of each row should be treated like a header value.
* `table_summaries`: an array of summary objects

Below is a basic example of a table with two columns and two rows.

```json
{
    "results": [{
        "type": "table",
        "table_columns": [
            { "key": "column_a", "context": "metadata" },
            { "key": "column_b" }
        ],
        "table_rows": [
            [1, 2],
            [2, 3]
        ],
        "table_summaries": [{
            "key": "avg",
            "label": "Average",
            "column_values": {
                "column_a": 1.5,
                "column_b": 2.5
            }
        }]
    }]
}
```

##### Defining Columns

A "table column object" describes a table column in terms of its key or variable name and its context in regards to other columns within the table object.

A "table column object" **MUST** contain the following members:

* `key`: the key of a column also referred to as the column's variable name and **MUST** be in snake_case format

A table column **MAY** additionally define any of the following members:

* `context`: a string describing how the column relates to other columns in the table if the values contained within the column are not concrete values
* `label`: a string that **SHOULD** typically contain a more human-readable alternative to `key`.

The value of the `context` member **MUST** be one of the following:

* `metadata`: the values in this column are considered important context for other columns but can not stand on their own
* `trivial`: the values in this column are considered unimportant context for other columns and can not stand on their own

##### Summarizing Columns

A "table summary object" is a special object that defines a summary  for a column. A summary object **MUST** include the following members:

* `key`: the key of the summary value that **MUST** be in snake_case format
* `values`: an object where each member defined must match a key in `table_columns`

Values of members in `values` **MUST NOT** be arrays or objects. The set of members is always less than or equal to the length `table_columns`.

* `label`: a string that **SHOULD** typically contain a more human-readable alternative to `key`.

The following shows a table with a summary that contains column averages:

```json
{
    "results": [{
        "type": "table",
        "table_columns": [
            { "key": "column_a", "context": "metadata" },
            { "key": "column_b" }
        ],
        "table_rows": [
            [1, 2],
            [2, 3]
        ],
        "table_summaries": [{
            "key": "avg",
            "label": "Average",
            "column_values": {
                "column_a": 1.5,
                "column_b": 2.5
            }
        }]
    }]
}
```

#### chart2d

A "chart result object" defines a multi-value structure that describes a mathematical 2D chart. 

A "chart result object" **MUST** contain the following members:

* `chart_type`: a reference to the type of the chart object which
* `series`: an array of "series objects"

A chart can also include any of the following members:

* `x_min`: a number defining the minimum value on the x-axis
* `x_max`: a number defining the maximum value on the x-axis
* `y_min`: a number defining the minimum value on the y-axis
* `y_max`: a number defining the maximum value on the y-axis
* `x_scale_type`: a string that defines the scale type of the x-axis. The value **MUST** be either "linear", "logarithmic"
* `y_scale_type`: a string that defines the scale type of the y-axis. The value **MUST** be either "linear", "logarithmic"

The value of the `chart_type` **MUST** be one of the following:

* `line`: a line chart as described [here](https://en.wikipedia.org/wiki/Line_chart)
* `scatter`: a scatter plot as described [here](https://en.wikipedia.org/wiki/Scatter_plot)
* `bar`: a bar chart as described [here](https://en.wikipedia.org/wiki/Bar_chart)
* `histogram`: an histogram chart as described [here](https://en.wikipedia.org/wiki/Histogram)
* `area`: an area chart as described [here](https://en.wikipedia.org/wiki/Area_chart)
* `box`: a box plot as described [here](https://en.wikipedia.org/wiki/Box_plot)

The following shows a simple chart with a single line:

```json
{
    "results": [{
        "key": "my_chart_key",
        "type": "chart2d",
        "chart_type": "line",
        "series": [{
            "key": "my_series_key",
            "data": [[1,2], [2,3], [3,4], [4,5], [5,6]]
        }]
    }]
}
```

##### Defining Series

A "chart series object" within a chart object is a container for X,Y pairs. A series object **MUST** must contain the following members:

* `key`: the key of the series that **MUST** be in snake_case format
* `data`: an array of [x,y] pairs that **MUST** only contain numbers or strings

### Metadata

The "analysis object" and the "result object" **MAY** contain the "metadata object".

A "metadata object" **MAY** contain any members where the names are in snake_case format. A "metadata object" may contain any depth of data and provides a deeper context the both the analysis and the results.

The following illustrates a "metadata object" on the top-level "analysis object":

```json
{
    "metadata": { 
        "foo": 42,
        "bar": {
            "contained_array": [1, 2, 3]
        }
    },
    "results": []
}
```

The following shows a "metadata object" defined on a "text result object":

```json
{
    "results": [{
        "key": "my_variable",
        "type": "text",
        "text_value": "Hello World!",
        "metadata": { 
            "foo": 42,
            "bar": {
                "contained_array": [1, 2, 3]
            }
        }
    }]
}
```

### Errors

The "error object" can be included at top-level or as a part of a "result object".

An "error object" **MUST** contain the following members:

* `message`: a string describing the error that occurred

In addition the "error object" **MAY** also include the following standard members:

* `detail`: a string used to the error in more detail

Additionally the "error object" **MAY** define any non-standard member that provides further details.

The following shows an error on top-level for the analysis:

```json
{
    "errors": [{
        "message": "Something went wrong with the analysis ..."
    }]
}
```

The following shows an error on an individual "result object":

```json
{
    "results": [{
        "key": "my_variable",
        "type": "text",
        "errors": [{
            "message": "Something went wrong when processing this result ..."
        }]
    }]
}
```

## Complementary Hierarchy

Result objects that define the `tags` do no define a clear internal hierarchy on their own. Therefore, an analysis object **MAY** include the `hierarchy` member. This is a flexible object that only describes the hierarchy does not have a strict structure imposed on it. It can be an array or object or a combination thereof with string members or values that **MUST** only refer to tags defined within `results`.

The following example shows a simple direct hierarchy for a set of tagged results:

```json
{
    "results": [{
        "key": "x",
        "type": "numeric",
        "tags": ["foo"],
        "numeric_value": 1.0
    }, {
        "key": "x",
        "type": "numeric",
        "tags": ["bar"],
        "numeric_value": 2.0
    }, {
        "key": "x",
        "type": "numeric",
        "tags": ["baz"],
        "numeric_value": 3.0
    }],
    "hierarchy": {
        "foo": {
            "bar": "baz"
        }
    }
}
```

The following example describes a hierarchy with only one parent:

```json
{
    "results": [{
        "key": "x",
        "type": "numeric",
        "tags": ["foo"],
        "numeric_value": 1.0
    }, {
        "key": "x",
        "type": "numeric",
        "tags": ["bar"],
        "numeric_value": 2.0
    }, {
        "key": "x",
        "type": "numeric",
        "tags": ["baz"],
        "numeric_value": 3.0
    }],
    "hierarchy": {
        "foo": ["bar", "baz"]
    }
}
```

## Examples

This chapter defines more complex examples and special use-cases ...
