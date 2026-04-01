# Rhymix Database Reference

## Schema Definition (schemas/*.xml)

Location: `modules/{module}/schemas/{table_name}.xml`

```xml
<table name="my_items">
    <column name="item_srl" type="number" size="11"
            notnull="notnull" primary_key="primary_key" />
    <column name="module_srl" type="number" size="11"
            default="0" notnull="notnull" index="idx_module_srl" />
    <column name="title" type="varchar" size="250" />
    <column name="content" type="bigtext" notnull="notnull" />
    <column name="status" type="varchar" size="20" default="PUBLIC" />
    <column name="regdate" type="date" index="idx_regdate" />
    <column name="ipaddress" type="varchar" size="128"
            notnull="notnull" />
    <column name="list_order" type="number" size="11"
            notnull="notnull" index="idx_list_order" />
</table>
```

### Column Types

| Type | Maps To |
|------|---------|
| `number` / `bignumber` | `bigint` |
| `varchar` | `varchar(size)` |
| `char` | `char(size)` |
| `text` | `text` |
| `bigtext` | `longtext` |
| `date` | `char(14)` (stored as YYYYMMDDHHmmss) |
| `float` | `float` |
| `int` / `integer` | `int` / `bigint` |

### Column Attributes

| Attribute | Description |
|-----------|-------------|
| `name` | Column name |
| `type` | Data type |
| `size` | Size |
| `notnull="notnull"` | NOT NULL constraint |
| `primary_key="primary_key"` | Primary key |
| `index="index_name"` | Index (same name = composite index) |
| `unique="unique_name"` | Unique index |
| `default="value"` | Default value |
| `charset="latin1"` | Charset override |
| `generated="STORED\|VIRTUAL"` | Generated column |

## Query Definition (queries/*.xml)

### SELECT

```xml
<query id="getItemList" action="select">
    <tables>
        <table name="my_items" />
        <table name="modules" type="left join">
            <conditions>
                <condition operation="equal"
                    column="my_items.module_srl"
                    default="modules.module_srl" />
            </conditions>
        </table>
    </tables>
    <columns>
        <column name="my_items.*" />
        <column name="modules.mid" />
    </columns>
    <conditions>
        <condition operation="equal" column="module_srl"
                   var="module_srl" />
        <condition operation="like" column="title"
                   var="search_keyword" pipe="and" />
        <group pipe="and">
            <condition operation="equal" column="status"
                       var="status" pipe="or" />
        </group>
    </conditions>
    <navigation>
        <index var="sort_index" default="list_order" order="asc" />
        <list_count var="list_count" default="20" />
        <page_count var="page_count" default="10" />
        <page var="page" default="1" />
    </navigation>
</query>
```

### INSERT

```xml
<query id="insertItem" action="insert">
    <tables>
        <table name="my_items" />
    </tables>
    <columns>
        <column name="item_srl" var="item_srl" />
        <column name="module_srl" var="module_srl" />
        <column name="title" var="title" />
        <column name="content" var="content" />
        <column name="regdate" var="regdate" default="now()" />
    </columns>
</query>
```

### UPDATE

```xml
<query id="updateItem" action="update">
    <tables>
        <table name="my_items" />
    </tables>
    <columns>
        <column name="title" var="title" />
        <column name="content" var="content" />
    </columns>
    <conditions>
        <condition operation="equal" column="item_srl"
                   var="item_srl" required="true" />
    </conditions>
</query>
```

### DELETE

```xml
<query id="deleteItem" action="delete">
    <tables>
        <table name="my_items" />
    </tables>
    <conditions>
        <condition operation="equal" column="item_srl"
                   var="item_srl" required="true" />
    </conditions>
</query>
```

### Full Operation List

| Operation | SQL | Notes |
|-----------|-----|-------|
| `equal` | `column = value` | |
| `notequal` / `not_equal` | `column != value` | |
| `more` / `gte` | `column >= value` | |
| `excess` / `gt` | `column > value` | |
| `less` / `lte` | `column <= value` | |
| `below` / `lt` | `column < value` | |
| `like` | `column LIKE '%value%'` | |
| `like_prefix` / `like_head` | `column LIKE 'value%'` | |
| `like_suffix` / `like_tail` | `column LIKE '%value'` | |
| `notlike` | `column NOT LIKE '%value%'` | |
| `notlike_prefix` | `column NOT LIKE 'value%'` | |
| `notlike_suffix` | `column NOT LIKE '%value'` | |
| `search` | Auto-splits words into multiple LIKE conditions | |
| `regexp` | `column REGEXP value` | |
| `notregexp` / `not_regexp` | `column NOT REGEXP value` | |
| `null` | `column IS NULL` | |
| `notnull` | `column IS NOT NULL` | |
| `in` | `column IN (value)` | |
| `notin` | `column NOT IN (value)` | |

### Condition Attributes

| Attribute | Description |
|-----------|-------------|
| `column` | Column name |
| `var` | Variable name from arguments |
| `default` | Default value when var is empty |
| `required="true"` | Query fails if value is missing |
| `pipe` | `and` or `or` (default: `and`) |
| `if` | Conditional variable name |

## Executing Queries in PHP

```php
// Single result
$output = executeQuery('module_name.query_id', $args);
$data = $output->data;  // stdClass or null

// Array result
$output = executeQueryArray('module_name.query_id', $args);
$data = $output->data;  // array of stdClass
$page_nav = $output->page_navigation;  // Pagination object

// Check success
if (!$output->toBool())
{
    return $output;
}

// Generate unique serial number
$srl = getNextSequence();
```

## Ruleset Definition (ruleset/*.xml)

```xml
<?xml version="1.0" encoding="utf-8"?>
<ruleset version="1.5.0">
    <customrules>
        <rule name="my_rule" type="regex" test="/^[a-z]+$/">
            <message xml:lang="ko">올바르지 않은 값입니다.</message>
            <message xml:lang="en">Invalid value.</message>
        </rule>
    </customrules>
    <fields>
        <field name="title" required="true" length="1:250" />
        <field name="list_count" required="true" rule="number" />
        <field name="email" rule="email" />
        <field name="user_id" rule="userid" />
    </fields>
</ruleset>
```

Built-in rules: `number`, `alpha`, `alpha_number`, `email`, `url`, `userid`, `korean`
