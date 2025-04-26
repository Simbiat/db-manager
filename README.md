# Database manager

Functions to help with database management.

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