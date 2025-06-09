# Database manager

Functions to help with database management. To establish connection, pass a `\PDO` object to the constructor or `null` if you are using [DB Pool](https://github.com/Simbiat/db-pool) library.

## checkTable

```php
\Simbiat\Database\Manage::checkTable(string|array $table, string|array $schema = '');
```

Check if table (or tables) exist and get a number (`integer`) of tables that are found. You can also provide a schema (or a list of those), in case you have tables with the same name in multiple schemas. Note that if both `$table` and `$schema` are arrays then all possible combinations found will be counted, so be careful with that.

## getColumnType

```php
\Simbiat\Database\Manage::getColumnType(string $table, string $column, string $schema = '');
```

Get a column's type. You need to provide at least table and column names. Schema is optional but recommended, since if you have multiple schemas with the same table and column name in them, the first hit will be returned if schema is not provided.

## getColumnDescription

```php
\Simbiat\Database\Manage::getColumnDescription(string $table, string $column, string $schema = '');
```

Get column description (comment). You need to provide at least table and column names. Schema is optional but recommended, since if you have multiple schemas with the same table and column name in them, the first hit will be returned if schema is not provided.

## isNullable

```php
\Simbiat\Database\Manage::isNullable(string $table, string $column, string $schema = '');
```

Checks if a column is nullable and returns respective `boolean`. You need to provide at least table and column names. Schema is optional but recommended, since if you have multiple schemas with the same table and column name in them, the first hit will be used if schema is not provided.

## checkFK

```php
\Simbiat\Database\Manage::checkFK(string $table, string $fk, string $schema = '');
```

Checks if a foreign key with a provided name exists in a table and returns respective `boolean`. Schema is optional but recommended, since if you have multiple schemas with the same table and column name in them, the first hit will be used if schema is not provided.

## checkColumn

```php
\Simbiat\Database\Manage::checkColumn(string $table, string $column, string $schema = '');
```

Checks if a column with a provided name exists in a table and returns respective `boolean`. Schema is optional but recommended, since if you have multiple schemas with the same table and column name in them, the first hit will be used if schema is not provided.

## showOrderedTables

```php
\Simbiat\Database\Manage::showOrderedTables(string $schema = '', bool $bySize = false);
```

Function to get a list of all tables for a schema (or all schemas, if `$schema` is empty) in order, where first you have tables without dependencies (no foreign keys), and then tables that are dependent on tables that have come before. This is useful if you want to dump backups in a specific order so that you can then restore the data without disabling foreign keys. Set `$bySize` to `true` to also sort by size from smallest to largest.  
Can't guarantee work on anything besides MySQL/MariaDB. Does not work with cyclic dependencies.

## checkCyclicForeignKeys

```php
\Simbiat\Database\Manage::checkCyclicForeignKeys(?array $tables = null)
```

This function allows you to check for cyclic foreign keys when 2 (or more) tables depend on each other. This is considered bad practice even with nullable columns, but you may easily miss them as your database grows, especially if you have chains of three or more tables.  
This will not return the specific FKs you need to deal with, but rather just a list of tables referencing tables that refer to the initial ones.  You will need to analyze the references yourself to "untangle" them properly.  
You can pass a prepared list of tables with a format of

```php
['schema' => 'schema_name', 'table' => 'table_name']
```

to limit tables that need to be checked.  
Can't guarantee work on anything besides MySQL/MariaDB.

## selectAllDependencies

```php
\Simbiat\Database\Manage::selectAllDependencies(string $schema, string $table);
```

Function to recursively get all dependencies (foreign keys) of a table. This does not mean just all tables referenced by the table's foreign keys, but also tables that those tables depend on.  
Can't guarantee work on anything besides MySQL/MariaDB.

## showCreateTable

```php
\Simbiat\Database\Manage::showCreateTable(string $schema, string $table, bool $noIncrement = true, bool $ifNotExist = false, bool $addUse = false);
```

Function to restore `ROW_FORMAT` value to table definition.  
MySQL/MariaDB may now have `ROW_FORMAT` in `SHOW CREATE TABLE` output or have a value, which is different from the current one. This function amends that.  
A few options are supported:
- `$noIncrement` - Remove the `AUTO_INCREMENT=X` table option. Column attribute will still be present.
- `$ifNotExist` - Add the `IF NOT EXISTS` clause to the resulting definition.
- `$addUse`  - Add the `USE` statement before the `CREATE` statement.

## hasFKViolated

```php
\Simbiat\Database\Manage::(?string $schema = null, ?string $table = null, bool $nullableOnly = false);
```

Function to check if your table (or schema or whole database server) has any violations of `FOREIGN KEY` constraints. While `schema` and `table` are optional, it's recommended to pass them in case there are large tables in the database. Also keep in mind that due to nature of `information_schema` first run may take a while. You can additionally pass `$nullableOnly` as `true` to get only constraints that have `DELETE_RULE` set to `SET NULL`. This is mainly useful for `fixFKViolations` function.

The function will return an array like this:
```php
array(2) {
    #Name of the constraint
    ["test__multiple_constraint"]=>
    array(9) {
        #Table where the constraint is set (with its schema)
        ["child_table"]=>
        string(32) "`simbiatr_simbiat`.`test__child`"
        #Table the constraint is dependent on
        ["parent_table"]=>
        string(33) "`simbiatr_simbiat`.`test__parent`"
        #Value of `DELETE_RULE`
        ["on_delete"]=>
        string(7) "CASCADE"
        #Columns that are used in the constraint with their respective linkage
        ["columns"]=>
        array(2) {
            [0]=>
            array(2) {
                ["child"]=>
                string(12) "parentid_alt"
                ["parent"]=>
                string(2) "id"
            }
            [1]=>
            array(2) {
                ["child"]=>
                string(4) "type"
                ["parent"]=>
                string(4) "type"
            }
        }
        #Generated query you can run to get the values of the violations
        ["select"]=>
        string(333) "SELECT `child`.`parentid_alt`, `child`.`type` FROM `simbiatr_simbiat`.`test__child` AS `child` LEFT JOIN `simbiatr_simbiat`.`test__parent` AS `parent` ON `child`.`parentid_alt` <=> `parent`.`id` AND `child`.`type` <=> `parent`.`type` WHERE (`child`.`parentid_alt` IS NOT NULL OR `child`.`type` IS NOT NULL) AND `parent`.`id` IS NULL;"
        #Generated query that can be run to fix the violations by setting them to `NULL`
        ["update"]=>
        string(446) "UPDATE `simbiatr_simbiat`.`test__child` SET `parentid_alt`=NULL, `type`=NULL WHERE (`parentid_alt`, `type`) IN (SELECT `child`.`parentid_alt`, `child`.`type` FROM `simbiatr_simbiat`.`test__child` AS `child` LEFT JOIN `simbiatr_simbiat`.`test__parent` AS `parent` ON `child`.`parentid_alt` <=> `parent`.`id` AND `child`.`type` <=> `parent`.`type` WHERE (`child`.`parentid_alt` IS NOT NULL OR `child`.`type` IS NOT NULL) AND `parent`.`id` IS NULL);"
        #Generated query that can be run to fix the violations by deleting them
        ["delete"]=>
        string(414) "DELETE FROM `simbiatr_simbiat`.`test__child` WHERE (`parentid_alt`, `type`) IN (SELECT `child`.`parentid_alt`, `child`.`type` FROM `simbiatr_simbiat`.`test__child` AS `child` LEFT JOIN `simbiatr_simbiat`.`test__parent` AS `parent` ON `child`.`parentid_alt` <=> `parent`.`id` AND `child`.`type` <=> `parent`.`type` WHERE (`child`.`parentid_alt` IS NOT NULL OR `child`.`type` IS NOT NULL) AND `parent`.`id` IS NULL);"
        #Number of violations found
        ["count"]=>
        int(1)
    }
    ["test__single_constraint"]=>
    array(9) {
        ["child_table"]=>
        string(32) "`simbiatr_simbiat`.`test__child`"
        ["parent_table"]=>
        string(33) "`simbiatr_simbiat`.`test__parent`"
        ["on_delete"]=>
        string(8) "SET NULL"
        ["columns"]=>
        array(1) {
            [0]=>
            array(2) {
                ["child"]=>
                string(8) "parentid"
                ["parent"]=>
                string(2) "id"
            }
        }
        ["select"]=>
        string(236) "SELECT `child`.`parentid` FROM `simbiatr_simbiat`.`test__child` AS `child` LEFT JOIN `simbiatr_simbiat`.`test__parent` AS `parent` ON `child`.`parentid` <=> `parent`.`id` WHERE (`child`.`parentid` IS NOT NULL) AND `parent`.`id` IS NULL;"
        ["update"]=>
        string(320) "UPDATE `simbiatr_simbiat`.`test__child` SET `parentid`=NULL WHERE (`parentid`) IN (SELECT `child`.`parentid` FROM `simbiatr_simbiat`.`test__child` AS `child` LEFT JOIN `simbiatr_simbiat`.`test__parent` AS `parent` ON `child`.`parentid` <=> `parent`.`id` WHERE (`child`.`parentid` IS NOT NULL) AND `parent`.`id` IS NULL);"
        ["delete"]=>
        string(305) "DELETE FROM `simbiatr_simbiat`.`test__child` WHERE (`parentid`) IN (SELECT `child`.`parentid` FROM `simbiatr_simbiat`.`test__child` AS `child` LEFT JOIN `simbiatr_simbiat`.`test__parent` AS `parent` ON `child`.`parentid` <=> `parent`.`id` WHERE (`child`.`parentid` IS NOT NULL) AND `parent`.`id` IS NULL);"
        ["count"]=>
        int(1)
    }
}
```

## fixFKViolations

```php
\Simbiat\Database\Manage::fixFKViolations(?string $schema = null, ?string $table = null, bool $nullableOnly = true, bool $forceDelete = false);
```

Fix constraints' violations found by `hasFKViolated`. While `schema` and `table` are optional, it's recommended to pass them in case there are large tables in the database. Also keep in mind that due to nature of `information_schema` first run may take a while.  
Also has two other options governing how violations will be fixed:
- `nullableOnly` - Whether to get only nullable constraints. If set to `false` entries that are not nullable will be **REMOVED** (`DELETE` will be used, so use with caution). If set to `true` (default), only nullable constraints will be picked up violations will be updated by settings the values to `NULL`.
- `forceDelete` - Whether to use `DELETE` even for nullable constraints. Use with caution.

Returns array similar to `hasFKViolated`, but with extra key `fixed` which represents number of rows that were updated/deleted as part of the fix.

## rebuildIndexQuery

```php
\Simbiat\Database\Manage::rebuildIndexQuery(string $schema, string $table, string $index, bool $run = false);
```

Generates and optionally runs (if `$run` is `true`) a query to rebuild (DROP and then ADD) an index. You need to pass the schema name, the table name and the index name for this to work.