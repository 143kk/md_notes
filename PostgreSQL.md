
# Creating a database

```shell
$ su postgres
$ createdb mydb
```

> Database names must have an alphabetic first character and are limited to 63 bytes in length.

To create that database that have the same name as current user name, simply type:

```shell
$ createdb
```

# Remove a database

```shell
$ dropdb mydb
```

# Access a database

```shell
$ psql mydb
```

> If you do not supply the database name then it will default to your user account name.

> In `psql`, you will be greeted with the following message:
>
> ```
> psql (14.1)
> Type "help" for help.
> mydb=>
> ```
>
> The last line could also be:
>
> ```
> mydb=#
> ```
>
> That would mean you are a database superuser.

Internal commands begin with the backslash character `\`.

To get out of `psql`, type:

```
mydb=> \q
```

# SQL language

## Ch.4 SQL Syntax

### 4.1 Lexical Structure

The PostgreSQL scanner/parser divides lexical elements into five fundamental categories: integers, non-integer numbers, strings, identifiers, and key words.

#### 4.1.1 Identifiers and key words

**Keywords**: Tokens such as `SELECT`, `UPDATE`, or `VALUES` are examples of key words.

**Identifiers**: Names of tables, columns, or other database objects are identifiers.

Identifiers and key words must begin with a letter. Subsequent characters in an identifier or key word can be letters, underscores, digits (0-9), or dollar signs ($).

The quoted identifier, or delimited identifier is formed by enclosing an arbitrary sequence of characters in double-quotes ("). 

Key words and **unquoted** identifiers are case **insensitive**. In PostgreSQL, unquoted identifiers are converted to lower case.tu

A convention often used is to write key words in upper case and identifiers in lower case.

#### 4.1.2 Constants

There are three kinds of implicitly-typed constants in PostgreSQL: strings, bit strings, and numbers.

**String constants**: bounded by single quotes ('). To include a single-quote character within a string constant, write **two adjacent single quotes**, e.g., `'Dianne''s horse'`.

**Dollar-Quoted String Constants**: consists of a dollar sign ($), an optional “tag” of zero or more characters, another dollar sign, the string content, a dollar sign, the same tag that began this dollar quote, and a dollar sign.

A constant of an arbitrary type can be entered using any one of the following notations:

```
type 'string'
'string'::type
CAST ( 'string' AS type )
```

The result is a constant of the indicated type.

#### 4.1.5 Comments

Two dashes (“--”) introduce comments. Whatever follows them is ignored up to the end of the line.

```sql
-- This is a standard SQL comment
```

Alternatively, C-style block comments can be used:

```sql
/* multiline comment
* with nesting: /* nested block comment */
*/
```

As shown above, **nested** block comments are allowed in SQL.

## Ch.5 Data definition

### 5.1 Table Basics

Create a table:

```cpp
CREATE TABLE weather (
	city varchar(80),
	temp_lo int, -- low temperature
	temp_hi int,
	prcp real,
	date date
);
```

Remove a table:

```sql
DROP TABLE tablename;
DROP TABLE IF EXISTS tablename;
```

### 5.2 Default Value

```sql
CREATE TABLE products (
	product_no integer,
    name text,
    price numeric DEFAULT 9.99
);
```

The default value can be an expression, which will be evaluated whenever the default value is inserted(NOT when the table is created).

If no default value is declared explicitly, the default value is the null value.

### 5.3 Generated Columns

A generated column is a special column that is always computed from other columns.

There are two kinds of generated columns: **stored** and **virtual**. A stored generated column is computed when it is written (inserted or updated) and occupies storage as if it were a
normal column. A virtual generated column occupies no storage and is computed when it is read.

PostgreSQL currently implements only **stored** generated columns.

```sql
CREATE TABLE people(
    ...,
    height_cm numeric,
    height_in numeric GENERATED ALWAYS AS (height_cm / 2.54) STORED
);
```

### 5.4 Constraints

#### 5.4.1 Check constraints

A check constraint is satisfied if the check expression evaluates to true or the **null** value.

You should be careful that check constraints will **NOT** prevent null values in the constrained columns because most expressions will evaluate to the null value if any operand is null.

A check constraint consists of the key word `CHECK` followed by an expression in parentheses.

```sql
CREATE TABLE products (
	product_no integer,
    name text,
    price numeric CHECK (price > 0)
);
```

You can give the constraint a separate name. If not, the system chooses a name for it.

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric CONSTRAINT positive_price CHECK (price > 0)
);
```

A check constraint is not necessarily attached to a particular column, instead, it can be a table constraint.

```sql
CREATE TABLE products (
	product_no integer,
    name text,
    price numeric CHECK (price > 0),
    discounted_prie numeric CHECK (discounted > 0),
    CHECK (price > discounted_price)
);
```

```sql
CREATE TABLE products (
	product_no integer,
    name text,
    price numeric CHECK (price > 0),
    discounted_prie numeric CHECK (discounted > 0),
    CONSTRAINT valid_discount CHECK (price > discounted_price)
);
```

#### 5.4.2 Not-Null constraints

```sql
CREATE TABLE products (
	product_no integer NOT NULL,
    name text NOT NULL,
    price numeric NOT NULL CHECK (price > 0)
);
```

#### 5.4.3 Unique constraints

Unique constraints ensure that the data contained in a column, or a group of columns, is unique among all the rows in the table.

**It's allowed to contained null values in a column among multiple rows even in the presence of a unique constraint.**

```sql
CREATE TABLE products (
    product_no integer UNIQUE,
    name text,
    price numeric
);
```

When written as a table constraint:

```sql
CREATE TABLE products (
	product_no integer,
    name text,
    price numeric,
    UNIQUE (product_no)
);
```

Define a unique constraint for a group of columns:

```sql
CREATE TABLE example (
	a integer,
    b integer,
    c integer,
    UNIQUE (a, c)
);
```

This specifies that the combination of values in the indicated columns is unique across the whole table, though any one of the columns need not be (and ordinarily isn't) unique.

Assign a name for a unique constraint:

```sql
CREATE TABLE products (
	product_no integer CONSTRAINT must_be_different UNIQUE,
    ...
);
```

#### 5.4.4 Primary keys

A primary key constraint indicates the value of the column be both unique and not null.

```sql
CREATE TABLE products (
	prodduct_no integer PRIMARY KEY,
    name text,
    price numeric
);
```

Primary keys can span more than one column.

```sql
CREATE TABLE example (
	a integer,
    b integer,
    c integer,
    PRIMARY KEY (a, c)
);
```

#### 5.4.5 Foreign keys

A foreign key constraint specifies that the values in a column (or a group of columns) must match the values appearing in some row of another table.

A foreign key must reference columns that either are a primary key or form a unique constraint.

```sql
CREATE TABLE products (
    product_no integer PRIMARY KEY,
    ...
);

CREATE TABLE orders (
	order_id integer PRIMARY KEY,
    product_no integer REFERENCES products (product_no),
    ...
);
```

Now it is **impossible** to create orders with **non-NULL** product_no entries that do not appear in the products table.

We can shorten the above command to:

```sql
CREATE TABLE orders (
	order integer PRIMARY KEY,
    product_no integer REFERENCES products,
    ...
);
```

because the primary key of the referenced table is used as the referenced column.

A foreign key can also constrain and reference a group of columns.

```sql
CREATE TABLE t1 (
	a integer PRIMARY KEY,
    b integer,
    c integer,
    FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
);
```

Below is an example of implementing many-to-many relationships: You have tables about products and orders and you want to allow one order to contain possibly many products.

```SQL
CREATE TABLE products (
	product_no integer PRIMARY KEY,
    name text,
    price numeric
);

CREATE TABLE orders (
	order_id integer PRIMARY KEY,
    shipping_address text,
    ...
);

CREATE TABLE order_items (
	product_no integer REFERENCES products,
    order_id integer REFERENCES orders,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

Deletion policies:

- `RESTRICT`: prevents the deletion of a referenced row.
- `NO ACTION`: the default behavior if you do not specify anything. Similar to `RESTRICT`, the essential difference between these two is s that `NO ACTION` allows the check to be deferred until later in the transaction, whereas `RESTRICT` does not.
- `CASCADE`: when a referenced row is deleted, row(s) referencing it should be automatically deleted as well.
- `SET NULL`: cause the referencing column(s) in the the referencing row(s) to be set to nulls.
- `SET DEFAULT`: cause the referencing column(s) in the the referencing row(s) to be set to their default values.

Below is an example using `RESTRICT` and `CASCADE` options: when someone wants to remove a product that is still referenced by an order (via order_items), we disallow it. If someone removes an order, the order items are removed as well:

```sql
CREATE TABLE products (
	product_no integer PRIMARY KEY,
    ...
);

CREATE TABLE orders (
	order_id integer PRIMARY KEY,
    shipping_address text,
    ...
);

CREATE TABLE order_items (
	product_no integer REFERENCES products ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```

Analogous to `ON DELETE` there is also `ON UPDATE` which is invoked when a referenced column is changed (updated). The possible actions are the same. In this case, `CASCADE` means that the updated values of the referenced column(s) should be copied into the referencing row(s).

### 5.5 System columns

| Column Name | Explanation                                                  |
| ----------- | ------------------------------------------------------------ |
| tableoid    | The OID of the table containing this row                     |
| xmin        | The identity (transaction ID) of the inserting transaction for this row version. (A row version is an individual state of a row; each update of a row creates a new row version for the same logical row.) |
| cmin        | The command identifier (starting at zero) within the inserting transaction. |
| xmax        | The identity (transaction ID) of the deleting transaction, or zero for an undeleted row version. It is possible for this column to be nonzero in a visible row version. That usually indicates that the deleting transaction hasn't committed yet, or that an attempted deletion was rolled back. |
| cmax        | The command identifier within the deleting transaction, or zero. |
| ctid        | The physical location of the row version within its table.   |

### 5.6 Modifying tables

You can perform the following actions using the `ALTER TABLE	` command:

- Add/Remove columns
- Add/Remove constraints
- Change default values
- Change column data types
- Rename columns
- Rename tables

#### 5.6.1 Adding a column

```sql
ALTER TABLE products ADD COLUMN description text;
```

The new column is initially filled with whatever default value is given (null if you don't specify a `DEFAULT` clause).

All the options that can be applied to a column description in `CREATE TABLE` can be used in `ALTER TABLE`.

For example, define constraints on the column at the same time:

```sql
ALTER TABLE products ADD COLUMN description text CEHCK (description <> '');
```

#### 5.6.2 Removing a column

```sql
ALTER TABLE products DROP COLUMN description;
```

If the column is referenced by a foreign key constraint of another table, the command above is failed to execute. You can authorize dropping everything that depends on the column by adding `CASCADE`:

```sql
ALTER TABLE products DROP COLUMN description CASCADE;
```

#### 5.6.3 Adding a constraint

```sql
ALTER TABLE products ADD CHECK (name <> '');
ALTER TABLE products ADD CONSTRAINT some_name UNIQUE (product_no);
ALTER TABLE products ADD FOREIGN KEY (product_group_id) REFERENCES product_groups;
```

Not-null constraint:

```sql
ALTER TABLE products ALTER COLUMN product_no SET NOT NULL;
```

#### 5.6.4 Removing a constraint

```SQL
ALTER TABLE products DROP CONSTRAINT some_name;
```

If you don't the the name of the constraint, then the psql command `\d tablename` can be helpful here.

To drop a not null constraint use:

```sql
ALTER TABLE products ALTER COLUMN product_no DROP NOT NULL;
```

#### 5.6.5 Changing a column's default value

To set a new default value for a column, use:

```SQL
ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;
```

To remove any default value, use:

```sql
ALTER TABLE products ALTER COLUMN price DROP DEFAULT;
```

This is effectively the same thing as setting the default to null.

#### 5.6.6 Changing a column's data type

```sql
ALTER TABLE products ALTER COLUMN price TYPE numeric(10,2);
```

#### 5.6.7 Renaming a column

```sql
ALTER TABLE products RENAME COLUMN product_no TO product_number;
```

#### 5.6.8 Renaming a table

```sql
ALTER TABLE products RENAME TO items;
```

### 5.7 Privileges

There are different kinds of privileges: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `REFERENCES`, `TRIGGER`, `CREATE`, `CONNECT`, `TEMPORARY`, `EXECUTE`, and `USAGE`.

Assign a table to a new owner:

```sql
ALTER TABLE table_name OWNER TO new_owner;
```

To assign privileges, the `GRANT` command is used. For example, if `joe` is an existing role, and `accounts` is an existing table, the privilege to update the table can be granted with:

```sql
GRANT UPDATE ON accounts TO joe;
```

You can also write `ALL` in place of `UPDATE` in the example above to grant all privileges that are relevant for the object type.

To revoke the previously-granted privilege, use `REVOKE` command:

```sql
REVOKE UPDATE ON accounts FROM joe;
```

The special "role" name `PUBLIC` can be used to grant a privilege to every role on the system.

```sql
REVOKE ALL ON accounts FROM PUBLIC;
```

Ordinarily, only the object's owner (or a superuser) can grant or revoke privileges on an object. However, it is possible to grant a privilege “with grant option”, which gives the recipient the right to grant it in turn to others. If the grant option is subsequently revoked then all who received the privilege from that recipient (directly or through a chain of grants) will lose the privilege.

### 5.8 Row security policies

## 5.9 Schemas

A database contains one or more named schemas, which in turn contain tables. Schemas also contain other kinds of named objects, including data types, functions, and operators.

It's somewhat similar to the concept of namespace in other languages.

### 5.9.1 Creating a Schema

To create a schema named myschema, use:

```sql
CREATE SCHEMA myschema;
```

To create a schema owned by someone else:

```sql
CREATE SCHEMA myschema AUTHORIZATION user_name;
```

To create or access objects in a schema, write a qualified name consisting of the schema name and object name separated by a dot.

For example, create a table in the new schema:

```sql
CREATE TABLE myschema.mytable (
	...
);
```

To drop a schema if it's empty, use:

```sql
DROP SCHEMA myschema;
```

To drop a schema including all contained objects, use:

```sql
DROP SCHEMA myschema CASCADE;
```

### 5.9.2 The Public Schema

Every new database contains a schema named "public".

### 5.9.3 The Schema Search Path

The system use the search path to determine which schemas to look in when faced with unqualified names.

To show the current search path, use:

```sql
SHOW search_path;
```

In the default setup this returns:

```
   search_path   
-----------------
 "$user", public
```

The first element specifies that a schema with the same name as the current user is to be searched. The second element refers to the public schema that we've seen already.

The first schema in the search path that exists is the default location for creating new objects. That is the reason that by default objects are created in the public schema. When objects are referenced in any other context without schema qualification (table modification, data modification, or query commands), the search path is traversed until a matching object is found. Therefore, in the default configuration, any unqualified access again can only refer to the public schema.

To set the search path, use:

```sql
SET search_path TO myschema,public;
```

### 5.9.4 Schemas and Privileges

By default, users cannot access any objects in schemas they do not own. To allow that, the owner of the schema must grant the `USAGE` privilege on the schema.

To allow a user to create objects in someone else's schema, the `CREATE` privilege on the schema needs to be granted. 

Note that by default, everyone has `CREATE` and `USAGE` privileges on the schema `public`. This allows all users that are able to connect to a given database to create objects in its public schema. Some usage patterns call for revoking that privilege:

```sql
REVOKE CREATE ON SCHEMA public FROM PUBLIC;
```

### 5.9.5 The System Catalog Schema

In addition to public and user-created schemas, each database contains a `pg_catalog` schema, which contains the system tables and all the built-in data types, functions, and operators. 

`pg_catalog` is always effectively part of the search path. If it is not named explicitly in the path then it is implicitly searched before searching the path's schemas. This ensures that built-in names will always be findable. 

However, you can explicitly place `pg_catalog` at the end of your search path if you prefer to have user-defined names override built-in names.







## Ch.6 Data Manipulation

### 6.1 Inserting data

For example, consider the following table:

```sql
CREATE TABLE products (
	product_no integer,
    name text,
    price numeric
);
```

Commands to insert a row would be:

```sql
INSERT INTO products VALUES (1, 'Cheese', 9.99);
INSERT INTO products (product_no, name) VALUES (1, 'Cheese');
INSERT INTO products VALUES (1, 'Cheese');
```

The third form is a PostgreSQL extension. It fills the columns from the left with as many values as are given, and the rest will be defaulted.

You can request default value explictly.

```sql
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', DEFAULT);
INSERT INTO products DEFAULT VALUES;
```

You can insert multiple rows in a single command:

```sql
INSERT INTO products (product_no, name, price) VALUES
    (1, 'Cheese', 9.99),
    (2, 'Bread', 1.99),
    (3, 'Milk', 2.99);
```

Insert the result of a query:

```sql
INSERT INTO products (product_no, name, price)
	SELECT product_no, name, price FROM new_products
		WHERE release_date = 'today';
```

When inserting large amounts of data, consider using the `COPY` command.

```sql
COPY weather FROM '/home/user/weather.txt';
```

### 6.2 Updating data

```sql
UPDATE weather SET temp_hi = temp_hi - 2, temp_lo = temp_lo - 2 WHERE date > '1994-11-28';
```

### 6.3 Deleting data

```sql
DELETE FROM weather WHERE city = 'Hayward';
```

Without a qualification, `DELETE` will remove **ALL** rows from the given table. Hence, one should be wary of such statement:

```sql
DELETE FROM tablename;
```

### 6.4 Returning data from modified rows

Sometimes it is useful to obtain data from modified rows while they are being manipulated. The `INSERT`, `UPDATE`, and `DELETE` commands all have an optional `RETURNING` clause that supports this.

The allowed contents of a `RETURNING` clause are the same as a `SELECT` command's output list.

```sql
INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;

UPDATE products SET price = price * 1.10 WHERE price <= 99.99 RETURNING name, price AS new_price;

DELETE FROM products WHERE obsoletion_date = 'today' RETURNING *;
```

## Ch.7 Queries

The general syntax of the `SELECT` command is:

<pre>
[WITH <em>with_queries</em>] SELECT <em>select_list</em> FROM <em>table_expression</em> [<em>sort_specification</em>]
</pre>

Select all columns:

```sql
SELECT * FROM weather;
```

Expressions are available in the select list:

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

A qualified query is made by adding a `WHERE` clause:

```sql
SELECT * FROM weather WHERE city = 'San Francisco' AND prcp > 0.0;
```

Specify the sorted order of the results returned:

```sql
SELECT * FROM weather ORDER BY city;
```

Remove duplicate rows from the result:

```sql
SELECT DISTINCT city FROM weather;
```

Call a function:

```sql
SELECT random();
```

### 7.2 Table expressions

The table expression contains a `FROM` clause that is optionally followed by `WHERE`, `GROUP BY`, and `HAVING` clauses.

#### 7.2.1 The `FROM` clause

<pre>
FROM <em>table_reference</em> [, <em>table_reference</em> [, ...]]
</pre>

A table reference can be a table name (possibly schema-qualified), or a derived table such as a subquery, a `JOIN` construct, or complex combinations of these. 

If more than one table reference is listed in the `FROM` clause, the tables are cross-joined (that is, the Cartesian product of their rows is formed; see below).

##### 7.2.1.1 Joined tables

The general syntax of a joined table is:

<pre>
<em>T1</em> <em>join_type</em> <em>T2</em> [ <em>join_condition</em> ]
</pre>

Joins of all types can be chained together, or nested: either or both `T1` and `T2` can be joined tables. Parentheses can be used around JOIN clauses to control the join order. In the absence of parentheses, JOIN clauses nest left-to-right.

**Cross join**: The joined table contains the Cartesian product of every rows from `T1` and every rows from `T2`.

<pre>
<em>T1</em> CROSS JOIN <em>T2</em>
</pre>

`FROM T1 CROSS JOIN T2` is equivalent to `FROM T1 INNER JOIN T2 ON TRUE` and `FROM T1, T2`.

**Qualified joins**

<pre>
<em>T1</em> { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN <em>T2</em> ON <em>boolean_expression</em>
<em>T1</em> { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN <em>T2</em> USING ( <em>join column list</em> )
<em>T1</em> NATURAL { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN <em>T2</em>
</pre>

`INNER` is the default; `LEFT`, `RIGHT`, and `FULL` imply an outer join.

- **`INNER JOIN`**: For each row of `T1`, the joined table has a row for each row in `T2` that satisfies the join condition.
- **`LEFT OUTER JOIN`**: First, an inner join is performed. Then, for each row in `T1` that does not satisfy the join condition with any row in `T2`, a joined row is added with null values in columns of `T2`. Thus, the joined table always has at least one row for each row in `T1`.
- **`RIGHT OUTER JOIN`**: This is the converse of a left join: the result table will always have a row for each row in `T2`.
- **`FULL OUTER JOIN`**: First, an inner join is performed. Then, for each row in `T1` that does not satisfy the join condition with any row in `T2`, a joined row is added with null values in columns of `T2`. Also, for each row of `T2` that does not satisfy the join condition with any row in `T1`, a joined row with null values
  in the columns of `T1` is added.

The `ON` clause takes a Boolean value expression of the same kind as is used in a `WHERE` clause. A pair of rows from `T1` and `T2` match if the boolean expression evaluates to true.

Joining `T1` and `T2` with `USING (a, b)` produces the join condition `ON T1.a = T2.a AND T1.b = T2.b`. Furthermore, the output of `JOIN USING` suppresses redundant columns.

Finally, `NATURAL` is a shorthand form of `USING`: it forms a `USING` list consisting of all column names that appear in both input tables.

##### 7.2.1.2 Table and column aliases

To create a table alias, use:

<pre>
FROM <em>table_reference</em> [AS] <em>alias</em>
</pre>

A typical application of table aliases is to assign short identifiers to long table names to keep the join clauses readable. For example:

```sql
SELECT * FROM some_very_long_table_name s JOIN
	another_fairly_long_name a ON s.id = a.num;
```

It's necessary to use table aliases when joining a table to itself.

Another form of table aliasing allow you to give temporary names to columns of the table, as well as the table itself:

<pre>
FROM <em>table_reference</em> [AS] <em>alias</em> (<em>column1</em> [, <em>column2</em> [, ...]])
</pre>

##### 7.2.1.3 Subqueries

Subqueries specifying a derived table must be **enclosed in parentheses** and must be **assigned a table alias name** (as in Section 7.2.1.2). For example:

```sql
FROM (SELECT * FROM table1) AS alias_name
```

A subquery can also be a `VALUES` list:

```sql
FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow')) AS names(first, last)
```

##### 7.2.1.4 Table functions

#### 7.2.2 The `WHERE` clause

If the result of the condition is **true**, the row is kept in the output table, otherwise (i.e., if the result is false or null) it is discarded.

```sql
SELECT ... FROM fdt WHERE c1 > 5;
SELECT ... FROM fdt WHERE c1 IN (1, 2, 3);
SELECT ... FROM fdt WHERE c1 IN (SELECT c1 FROM t2);
SELECT ... FROM fdt WHERE c1 IN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10);
SELECT ... FROM fdt WHERE c1 BETWEEN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10) AND 100;
SELECT ... FROM fdt WHERE EXISTS (SELECT c1 FROM t2 WHERE c2 > fdt.c1);
```

#### 7.2.3 The `GROUP BY` and `HAVING` clauses

After passing the `WHERE` filter, the derived input table might be subject to grouping, using the `GROUP BY` clause, and *elimination of group rows using the `HAVING` clause.

In general, if a table is grouped, **columns that are not listed in `GROUP BY` cannot be referenced except in aggregate expressions (One of the exceptions is to group by primary key)**.

```sql
SELECT product_id, p.name, (sum(s.units) * p.price) AS sales
	FROM products p LEFT JOIN sales s USING (product_id)
	GROUP BY product_id, p.name, p.price;
```

In strict SQL, `GROUP BY` can only group by columns of the source table but PostgreSQL extends this to also allow `GROUP BY` to group by columns in the select list, which makes grouping by value expressions available.

Expressions in the `HAVING` clause can refer both to grouped expressions and to ungrouped expressions (which necessarily involve an aggregate function).

```sql
SELECT product_id, p.name, (sum(s.units) * (p.price - p.cost)) AS profit
	FROM products p LEFT JOIN sales s USING (product_id)
	WHERE s.date > CURRENT_DATE - INTERVAL '4 weeks'
	GROUP BY product_id, p.name, p.price, p.cost
	HAVING sum(p.price * s.units) > 5000;
```

### 7.3 Select lists

#### 7.3.1 Select-list items

When working with multiple tables, it can also be useful to ask for all the columns of a particular table:

```sql
SELECT tbl1.*, tbl2.a FROM ...
```

#### 7.3.2 Column labels

The `AS` key word is usually optional, but in some cases where the desired column name matches a PostgreSQL key word, you must write AS or double-quote the column name in order to avoid ambiguity.

```sql
-- The following query does not work:
-- SELECT a from, b + c AS sum FROM ...

-- But Either of these do:
SELECT a AS from, b + c AS sum FROM ...;
SELECT a "from", b + c AS sum FROM ...;
```

#### 7.3.3 `DISTINCT`

After the select list has been processed, the result table can optionally be subject to the elimination of duplicate rows. The `DISTINCT` key word is written directly after `SELECT` to specify this:

<pre>
    SELECT DISTINCT <em>select_list</em> ...
</pre>
Note that the keyword `DISTINCT` is applied to the **entire `select_list`**, rather than a specific column.

Two rows are considered distinct if they differ in at least one column value. Null values are
considered equal in this comparison.

Alternatively, an arbitrary expression can determine what rows are to be considered distinct:

<pre>
    SELECT DISTINCT ON (<em>expression</em> [, <em>expression</em>...]) <em>select_list</em> ...
</pre>

### 7.4 Combining queries

<pre>
<em>query1</em> UNION [ALL] <em>query2</em>
<em>query1</em> INTERSECT [ALL] <em>query2</em>
<em>query1</em> EXCEPT [ALL] <em>query2</em>
</pre>

- `UNION` effectively appends the result of `query2` to the result of `query1`. Furthermore, it eliminates duplicate rows from its result, in the same way as `DISTINCT`, unless `UNION ALL` is used.
- `INTERSECT` returns all rows that are both in the result of `query1` and in the result of `query2`. Duplicate rows are eliminated unless `INTERSECT ALL` is used.
- `EXCEPT` returns all rows that are in the result of `query1` but not in the result of `query2`. (called the difference between two queries.) Again, duplicates are eliminated unless `EXCEPT ALL` is used.

In order to calculate the union, intersection, or difference of two queries, the two queries must be “union compatible”, which means that they return the same number of columns and the corresponding columns have compatible data types.

Without parentheses, `UNION` and `EXCEPT` associate left-to-right, but `INTERSECT` binds more tightly than those two operators. For example,

```sql
query1 UNION query2 EXCEPT query3
-- equivalent to
(query1 UNION query2) EXCEPT query3

query1 UNION query2 INTERSECT query3
-- euqivalent to
query1 UNION (query2 INTERSECT query3)
```

It's necessary to surround an individual query with parentheses if if the query needs to use any of the clauses discussed in following sections, such as `LIMIT`. For example,

```sql
SELECT a FROM b UNION SELECT x FROM y LIMIT 10;
```

is accepted, but it means

```sql
(SELECT a FROM b UNION SELECT x FROM y) LIMIT 10;
```

rather than

```sql
SELECT a FROM b UNION (SELECT x FROM y LIMIT 10);
```

### 7.5 Sorting rows

<pre>
SELECT <em>select_list</em>
	FROM <em>table_expression</em>
	ORDER BY <em>sort_expression1</em> [ASC | DESC] [NULLS { FIRST | LAST }] 
		[, <em>sort_expression2</em> [ASC | DESC] [NULLS { FIRST | LAST }] ...]
</pre>

When more than one expression is specified, the later values are used to sort rows that are equal according to the earlier values.

**`ASC`(Ascending) is the default order.**

The `NULLS FIRST` and `NULLS LAST` options can be used to determine whether nulls appear before or after non-null values in the sort ordering. By default, null values sort as if larger than any non-null value; that is, `NULLS FIRST` is the default for `DESC` order, and `NULLS LAST` otherwise.

Note that the ordering options are considered **independently** for each sort column. For example `ORDER BY x, y DESC` means `ORDER BY x ASC, y DESC`, which is not the same as `ORDER BY x DESC, y DESC`.

A sort_expression can also be the column label or **number of an output column**, as in:

```sql
SELECT a, max(b) FROM table1 GROUP BY a ORDER BY 1;
```

will sort by the first output column.

### 7.6 `LIMIT` and `OFFSET`

<pre>
SELECT <em>select_list</em>
    FROM <em>table_expression</em>
    [ ORDER BY ... ]
    [ LIMIT { <em>number</em> | ALL } ] [ OFFSET <em>number</em> ]
</pre>

If a limit count is given, no more than that many rows will be returned (but possibly fewer, if the query itself yields fewer rows). `LIMIT ALL` is the same as omitting the `LIMIT` clause.

`OFFSET` says to skip that many rows before beginning to return rows. `OFFSET 0` is the same as omitting the OFFSET clause.

If both `OFFSET` and `LIMIT` appear, then `OFFSET` rows are skipped before starting to count the `LIMIT` rows that are returned.

**The rows skipped by an `OFFSET` clause still have to be computed inside the server; therefore a large `OFFSET` might be inefficient.**

### 7.6 `VALUES` lists

<pre>
VALUES ( <em>expression</em> [, ...] ) [, ...]
</pre>

As an example:

```
=> VALUES (1, 'one'), (2, 'two'), (3, 'three');
 column1 | column2 
---------+---------
       1 | one
       3 | three
       2 | two
(3 rows)
```

The parenthesized lists must all have the same number of elements (i.e., the number of columns in the table), and corresponding entries in each list must have compatible data types.

By default, PostgreSQL assigns the names column1, column2, etc. to the columns of a `VALUES `table.

Syntactically, `VALUES` followed by expression lists is treated as equivalent to a `SELECT` command. Therefore, it can appear anywhere a `SELECT` can.

For example, you can use it as part of a `UNION`, or attach a <em>sort_specification</em> (`ORDER BY`, `LIMIT`, and/or `OFFSET`) to it. `VALUES` is most commonly used as the data source in an `INSERT` command, and next most commonly as a subquery.

### 7.8 `WITH` queries

`WITH` provides a way to write auxiliary statements for use in a larger query. These statements, which are often referred to as **Common Table Expressions** or **CTE**s.

Each auxiliary statement in a `WITH` clause can be a `SELECT`, `INSERT`, `UPDATE`, or `DELETE`; and the `WITH` clause itself is attached to a primary statement that can also be a `SELECT`, `INSERT`, `UPDATE`, or `DELETE`.

#### 7.8.1 `SELECT` in `WITH`

The basic value of SELECT in WITH is to break down complicated queries into simpler parts.

An example is:

```sql
WITH regional_sales AS (
	SELECT region, SUM(amount) AS total_sales FROM orders GROUP BY region
), top_regions AS (
	SELECT region FROM regional_sales
	WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region, product, SUM(quantity) AS product_units, SUM(amount) AS product_sales FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

#### 7.8.2 Recursive queries

Using `RECURSIVE`, a WITH query can refer to its own output. 

A very simple example is this query to sum the integers from 1 through 100:

```sql
WITH RECURSIVE t(n) AS (
	VALUES (1)
	UNION ALL
	SELECT n+1 FROM t where n < 100
)
SELECT sum(n) FROM t;
```

The general form of a recursive WITH query is always a non-recursive term, then `UNION` (or `UNION ALL`), then a recursive term, where only the recursive term can contain a reference to the query's own output. 

Such a query is executed as follows:

1. Evaluate the non-recursive term. Include the resulting rows in the result of the recursive query and also place them in a temporary *working table*.
2. Repeat these steps as long as the working table is not empty:
   1. Evaluate the recursive term, substituting the current contents of the working table for the recursive self-reference. Include the resulting rows in the result of the recursive query and also place them in a temporary *intermediate table*.
   2. Replace the contents of the working table with the contents of the intermediate table, then empty the intermediate table.

## Ch.8 Data types

### 8.1 Numeric types

![image-20220208190446244](/home/arv/.config/Typora/typora-user-images/image-20220208190446244.png)

#### 8.1.2 Arbitrary precision numbers

Types `numeric` and `decimal` are equivalent.

Calculations with numeric values yield **exact** results. However, calculations on `numeric` values are very slow compared to the integer types, or to the floating-point types.

Terms used in the following:

- **Precision**: the total count of significant digits in the whole number.
- **Scale**: the count of decimal digits in the fractional part.

To declare a column of type `numeric`, use the syntax:

<pre>
    NUMERIC(<em>precision</em>, <em>scale</em>)
</pre>

Alternatively, 

<pre>
    NUMERIC(<em>precision</em>)
</pre>

selects a scale of 0.

Moreover,

<pre>
    NUMERIC
</pre>

without any precision or scale creates an "unconstrained numeric" column in which numeric values of any length can be stored, up to the implementation limits(in the above table).

The declared precision and scale of a column are maximums, not fixed allocations. (In this sense the numeric type is more akin to `varchar(n)` than to `char(n)`.)

#### 8.1.4 Serial types

The data types `smallserial`, `serial` and `bigserial` are not true types, but merely a notational convenience for creating unique identifier columns (similar to the `AUTO_INCREMENT` property supported
by some other databases). In the current implementation, specifying:

<pre>
    CREATE TABLE <em>tablename</em> (
    	<em>colname</em> SERIAL
    );
</pre>

is equivalent to specifying:

<PRE>
    CREATE SEQUENCE <em>tablename_colname_seq</em> AS integer;
    CREATE TABLE <em>tablename</em> (
    	<em>colname</em> integer NOT NULL DEFAULT nextval('<em>tablename_colname_seq</em>');
    );
    ALTER SEQUENCE <em>tablename_colname_seq</em> OWNED BY <em>tablename.colname</em>;
</pre>

The sequence created for a serial column is automatically dropped when the owning column is dropped.

The type `serial` creates a `integer` column, whereas `bigserial` create a `bigint` column and `small serial` creates a `smallint` column.

### 8.2 Monetary types

![image-20220208225028540](/home/arv/.config/Typora/typora-user-images/image-20220208225028540.png)

The `money` type stores a currency amount with a fixed fractional precision. The fractional precision is determined by the database's `lc_monetary` setting.

Input is accepted in a variety of formats, including integer and floating-point literals, as well as typical currency formatting, such as '$1,000.00'. Output is generally in the latter form but depends on the locale.

### 8.3 Character types

| Name                                 | Description                |
| ------------------------------------ | -------------------------- |
| `character varying(n)`, `varchar(n)` | variable-length with limit |
| `character(n)`, `char(n)`            | fixed-length, blank padded |
| `text`                               | variable unlimited length  |

Both of `character(n)` and `character varying(n)` can store strings up to n **characters** (**NOT bytes**) in length.

If the string to be stored is shorter than the declared length, values of type `character` will be space-padded; values of type `character varying` will simply store the shorter string.

`character` without length specifier is equivalent to `character(1)` and if `character varying` is used without length specifier, the type accepts strings of any size.

In addition, PostgreSQL provides the text type, which stores strings of any length.

> There is **NO** performance difference among these three types, apart from increased storage space when using the blank-padded type, and a few extra CPU cycles to check the length when storing into a length-constrained column. 
>
> While `character(n)` has performance advantages in some other database systems, there is no such advantage in PostgreSQL; in fact `character(n)` is usually the slowest of the three because of its additional storage costs. In most situations `text` or `character varying` should be used instead.

Zero-length strings(`''`) are not equal to `null`.

### 8.4 Binary data types

| Name    | Storage Size                               | Description                   |
| ------- | ------------------------------------------ | ----------------------------- |
| `bytea` | 1 or 4 bytes plus the actual binary string | variable-length binary string |

The `bytea` type supports two formats for input and output: “hex” format and PostgreSQL's historical “escape” format. Both of these are always accepted on input. The output format depends on the configuration parameter `bytea_output`; the default is hex.

#### 8.4.1 `bytea` Hex format

The entire "hex" string is preceded by the sequence `\x` (to distinguish it from the escape format).

Example:

```sql
SELECT '\xDEADBEEF';
```

#### 8.4.2 `bytea` Escape format

The “escape” format is the traditional PostgreSQL format for the `bytea` type. It takes the approach of representing a binary string as a sequence of ASCII characters, while converting those bytes that cannot be represented as an ASCII character into special escape sequences.

This format should probably be avoided for most new applications.

When entering `bytea` values in escape format, octets of certain values must be escaped, while all octet values can be escaped. In general, to escape an octet, convert it into its three-digit octal value and precede it by a backslash.

![image-20220209201705728](/home/arv/.config/Typora/typora-user-images/image-20220209201705728.png)

### 8.5 Date/Time types

<table class="table" summary="Date/Time Types" border="1">
        <colgroup>
          <col>
          <col>
          <col>
          <col>
          <col>
          <col>
        </colgroup>
        <thead>
          <tr>
            <th>Name</th>
            <th>Storage Size</th>
            <th>Description</th>
            <th>Low Value</th>
            <th>High Value</th>
            <th>Resolution</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td><code class="type">timestamp [ (<em class="replaceable"><code>p</code></em>) ] [ without time zone ]</code></td>
            <td>8 bytes</td>
            <td>both date and time (no time zone)</td>
            <td>4713 BC</td>
            <td>294276 AD</td>
            <td>1 microsecond</td>
          </tr>
          <tr>
            <td><code class="type">timestamp [ (<em class="replaceable"><code>p</code></em>) ] with time zone</code></td>
            <td>8 bytes</td>
            <td>both date and time, with time zone</td>
            <td>4713 BC</td>
            <td>294276 AD</td>
            <td>1 microsecond</td>
          </tr>
          <tr>
            <td><code class="type">date</code></td>
            <td>4 bytes</td>
            <td>date (no time of day)</td>
            <td>4713 BC</td>
            <td>5874897 AD</td>
            <td>1 day</td>
          </tr>
          <tr>
            <td><code class="type">time [ (<em class="replaceable"><code>p</code></em>) ] [ without time zone ]</code></td>
            <td>8 bytes</td>
            <td>time of day (no date)</td>
            <td>00:00:00</td>
            <td>24:00:00</td>
            <td>1 microsecond</td>
          </tr>
          <tr>
            <td><code class="type">time [ (<em class="replaceable"><code>p</code></em>) ] with time zone</code></td>
            <td>12 bytes</td>
            <td>time of day (no date), with time zone</td>
            <td>00:00:00+1559</td>
            <td>24:00:00-1559</td>
            <td>1 microsecond</td>
          </tr>
          <tr>
            <td><code class="type">interval [ <em class="replaceable"><code>fields</code></em> ] [ (<em class="replaceable"><code>p</code></em>) ]</code></td>
            <td>16 bytes</td>
            <td>time interval</td>
            <td>-178000000 years</td>
            <td>178000000 years</td>
            <td>1 microsecond</td>
          </tr>
        </tbody>
      </table>
`timestamp` alone is equivalent to `timestamp without time zone`.

`time` alone is equivalent to `time without time zone`.

The value `p` specifies the number of fractional digits retained in the seconds field. The allowed range of `p` is from 0 to 6.

The `interval` type has an additional option, which is to restrict the set of stored fields by writing one of these phrases:

```
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND
```

#### 8.5.1 Date/Time input

Date and time input is accepted in almost any reasonable format.

##### 8.5.1.1 Dates

<table class="table" summary="Date Input" border="1">
            <colgroup>
              <col class="col1">
              <col class="col2">
            </colgroup>
            <thead>
              <tr>
                <th>Example</th>
                <th>Description</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>1999-01-08</td>
                <td>ISO 8601; January 8 in any mode (recommended format)</td>
              </tr>
              <tr>
                <td>January 8, 1999</td>
                <td>unambiguous in any <code class="varname">datestyle</code> input mode</td>
              </tr>
              <tr>
                <td>1/8/1999</td>
                <td>January 8 in <code class="literal">MDY</code> mode; August 1 in <code class="literal">DMY</code> mode</td>
              </tr>
              <tr>
                <td>1/18/1999</td>
                <td>January 18 in <code class="literal">MDY</code> mode; rejected in other modes</td>
              </tr>
              <tr>
                <td>01/02/03</td>
                <td>January 2, 2003 in <code class="literal">MDY</code> mode; February 1, 2003 in <code class="literal">DMY</code> mode; February 3, 2001 in <code class="literal">YMD</code> mode</td>
              </tr>
              <tr>
                <td>1999-Jan-08</td>
                <td>January 8 in any mode</td>
              </tr>
              <tr>
                <td>Jan-08-1999</td>
                <td>January 8 in any mode</td>
              </tr>
              <tr>
                <td>08-Jan-1999</td>
                <td>January 8 in any mode</td>
              </tr>
              <tr>
                <td>99-Jan-08</td>
                <td>January 8 in <code class="literal">YMD</code> mode, else error</td>
              </tr>
              <tr>
                <td>08-Jan-99</td>
                <td>January 8, except error in <code class="literal">YMD</code> mode</td>
              </tr>
              <tr>
                <td>Jan-08-99</td>
                <td>January 8, except error in <code class="literal">YMD</code> mode</td>
              </tr>
              <tr>
                <td>19990108</td>
                <td>ISO 8601; January 8, 1999 in any mode</td>
              </tr>
              <tr>
                <td>990108</td>
                <td>ISO 8601; January 8, 1999 in any mode</td>
              </tr>
              <tr>
                <td>1999.008</td>
                <td>year and day of year</td>
              </tr>
              <tr>
                <td>J2451187</td>
                <td>Julian date</td>
              </tr>
              <tr>
                <td>January 8, 99 BC</td>
                <td>year 99 BC</td>
              </tr>
            </tbody>
          </table>

##### 8.5.1.2 Times

<table class="table" summary="Time Input" border="1">
            <colgroup>
              <col class="col1">
              <col class="col2">
            </colgroup>
            <thead>
              <tr>
                <th>Example</th>
                <th>Description</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="literal">04:05:06.789</code></td>
                <td>ISO 8601</td>
              </tr>
              <tr>
                <td><code class="literal">04:05:06</code></td>
                <td>ISO 8601</td>
              </tr>
              <tr>
                <td><code class="literal">04:05</code></td>
                <td>ISO 8601</td>
              </tr>
              <tr>
                <td><code class="literal">040506</code></td>
                <td>ISO 8601</td>
              </tr>
              <tr>
                <td><code class="literal">04:05 AM</code></td>
                <td>same as 04:05; AM does not affect value</td>
              </tr>
              <tr>
                <td><code class="literal">04:05 PM</code></td>
                <td>same as 16:05; input hour must be &lt;= 12</td>
              </tr>
              <tr>
                <td><code class="literal">04:05:06.789-8</code></td>
                <td>ISO 8601, with time zone as UTC offset</td>
              </tr>
              <tr>
                <td><code class="literal">04:05:06-08:00</code></td>
                <td>ISO 8601, with time zone as UTC offset</td>
              </tr>
              <tr>
                <td><code class="literal">04:05-08:00</code></td>
                <td>ISO 8601, with time zone as UTC offset</td>
              </tr>
              <tr>
                <td><code class="literal">040506-08</code></td>
                <td>ISO 8601, with time zone as UTC offset</td>
              </tr>
              <tr>
                <td><code class="literal">040506+0730</code></td>
                <td>ISO 8601, with fractional-hour time zone as UTC offset</td>
              </tr>
              <tr>
                <td><code class="literal">040506+07:30:00</code></td>
                <td>UTC offset specified to seconds (not allowed in ISO 8601)</td>
              </tr>
              <tr>
                <td><code class="literal">04:05:06 PST</code></td>
                <td>time zone specified by abbreviation</td>
              </tr>
              <tr>
                <td><code class="literal">2003-04-12 04:05:06 America/New_York</code></td>
                <td>time zone specified by full name</td>
              </tr>
            </tbody>
          </table>

----

Time zone input:

<table class="table" summary="Time Zone Input" border="1">
            <colgroup>
              <col>
              <col>
            </colgroup>
            <thead>
              <tr>
                <th>Example</th>
                <th>Description</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="literal">PST</code></td>
                <td>Abbreviation (for Pacific Standard Time)</td>
              </tr>
              <tr>
                <td><code class="literal">America/New_York</code></td>
                <td>Full time zone name</td>
              </tr>
              <tr>
                <td><code class="literal">PST8PDT</code></td>
                <td>POSIX-style time zone specification</td>
              </tr>
              <tr>
                <td><code class="literal">-8:00:00</code></td>
                <td>UTC offset for PST</td>
              </tr>
              <tr>
                <td><code class="literal">-8:00</code></td>
                <td>UTC offset for PST (ISO 8601 extended format)</td>
              </tr>
              <tr>
                <td><code class="literal">-800</code></td>
                <td>UTC offset for PST (ISO 8601 basic format)</td>
              </tr>
              <tr>
                <td><code class="literal">-8</code></td>
                <td>UTC offset for PST (ISO 8601 basic format)</td>
              </tr>
              <tr>
                <td><code class="literal">zulu</code></td>
                <td>Military abbreviation for UTC</td>
              </tr>
              <tr>
                <td><code class="literal">z</code></td>
                <td>Short form of <code class="literal">zulu</code> (also in ISO 8601)</td>
              </tr>
            </tbody>
          </table>

##### 8.5.1.3 Time stamps

Valid input for the time stamp types consists of the concatenation of a date and a time, followed by an optional time zone, followed by an optional `AD` or `BC`.

Example:

```
1999-01-08 04:05:06
1999-01-08 04:05:06 -8:00
```

In addition, the common format:

```
January 8 04:05:06 1999 PST
```

is supported.

----

In a literal that has been determined to be `timestamp without time zone`, PostgreSQL will silently ignore any time zone indication. That is, the resulting value is derived from the date/time fields in the input value, and is **NOT** adjusted for time zone.

For `timestamp with time zone`, the internally stored value is always in UTC (Universal
Coordinated Time, traditionally known as Greenwich Mean Time, GMT). 

An input value that has an explicit time zone specified is converted to UTC using the appropriate offset for that time zone. If no time zone is stated in the input string, then it is assumed to be in the time zone indicated by the system's TimeZone parameter, and is converted to UTC using the offset for the timezone zone.

When a `timestamp with time zone` value is output, it is always converted from UTC to the **current `timezone` zone**, and displayed as local time in that zone.

##### 8.5.1.4 Special values

All of these values need to be enclosed in single quotes when used as constants in SQL commands.

<table class="table" summary="Special Date/Time Inputs" border="1">
            <colgroup>
              <col>
              <col>
              <col>
            </colgroup>
            <thead>
              <tr>
                <th>Input String</th>
                <th>Valid Types</th>
                <th>Description</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="literal">epoch</code></td>
                <td><code class="type">date</code>, <code class="type">timestamp</code></td>
                <td>1970-01-01 00:00:00+00 (Unix system time zero)</td>
              </tr>
              <tr>
                <td><code class="literal">infinity</code></td>
                <td><code class="type">date</code>, <code class="type">timestamp</code></td>
                <td>later than all other time stamps</td>
              </tr>
              <tr>
                <td><code class="literal">-infinity</code></td>
                <td><code class="type">date</code>, <code class="type">timestamp</code></td>
                <td>earlier than all other time stamps</td>
              </tr>
              <tr>
                <td><code class="literal">now</code></td>
                <td><code class="type">date</code>, <code class="type">time</code>, <code class="type">timestamp</code></td>
                <td>current transaction's start time</td>
              </tr>
              <tr>
                <td><code class="literal">today</code></td>
                <td><code class="type">date</code>, <code class="type">timestamp</code></td>
                <td>midnight (<code class="literal">00:00</code>) today</td>
              </tr>
              <tr>
                <td><code class="literal">tomorrow</code></td>
                <td><code class="type">date</code>, <code class="type">timestamp</code></td>
                <td>midnight (<code class="literal">00:00</code>) tomorrow</td>
              </tr>
              <tr>
                <td><code class="literal">yesterday</code></td>
                <td><code class="type">date</code>, <code class="type">timestamp</code></td>
                <td>midnight (<code class="literal">00:00</code>) yesterday</td>
              </tr>
              <tr>
                <td><code class="literal">allballs</code></td>
                <td><code class="type">time</code></td>
                <td>00:00:00.00 UTC</td>
              </tr>
            </tbody>
          </table>

> **While the input strings `now`, `today`, `tomorrow`, and `yesterday` are fine to use in interactive SQL commands, they can have surprising behavior when the command is saved to be executed later.*
>
> For example in prepared statements, views, and function definitions. The string can be converted to a specific time value that continues to be used long after it becomes stale. Use one of the SQL functions instead in such contexts. For example, `CURRENT_DATE + 1` is safer than `'tomorrow'::date`.

### 8.6 Boolean type

| Name      | Storage size | Description            |
| --------- | ------------ | ---------------------- |
| `boolean` | 1 byte       | state of true or false |

### 8.7 Enumerated types

```
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
CREATE TABLE person (
name text,
current_mood mood
);
INSERT INTO person VALUES ('Moe', 'happy');
SELECT * FROM person WHERE current_mood = 'happy';
name | current_mood
------+--------------
Moe | happy
(1 row)
```

An enum value occupies four bytes on disk.

The ordering of the values in an enum type is the order in which the values were listed when the type was created.

Enum labels are **case sensitive**, so 'happy' is not the same as 'HAPPY'. White space in the labels is significant too.

### 8.15 Arrays

#### 8.15.1 Declaration of Array Types

```sql
CREATE TABLE sal_emp (
	name text,
    pay_by_quarter integer[],
    schedule text[][]
);
```

You can specify the array size in the brackets. However, the current implementation ignores any supplied array size limits, i.e., the behavior is the same as for arrays of unspecified length.

Arrays of a particular element type are all considered to be of the same type, regardless of size or number of dimensions.

An alternative syntax to create one-dimensional arrays, which conforms to the SQL standard by using the keyword `ARRAY`:

```sql
pay_by_quarter integer ARRAY[4], # array size specified
pay_by_quarter integer ARRAY # not specified
```

Again, the size restriction is ignored.

#### 8.15.2 Array Value Input

To write an array value as a literal constant(actually, a string before the array input conversion), enclose the element values within curly braces and separate them by commas. 

You can put double quotes around any element value, and must do so if it contains commas or curly braces.

Multidimensional arrays must have matching extents for each dimension.

```sql
INSERT INTO sal_emp
    VALUES ('Bill',
    '{10000, 10000, 10000, 10000}',
    '{{"meeting", "lunch"}, {"training", "presentation"}}');
```

The ARRAY constructor syntax can also be used:

```sql
INSERT INTO sal_emp
    VALUES ('Bill',
    ARRAY[10000, 10000, 10000, 10000],
    ARRAY[['meeting', 'lunch'], ['training', 'presentation']]);
```

#### 8.15.3 Accessing Arrays

By default, the **array index starts from 1**.

Retrieves the names of the employees whose pay changed in the second quarter:

```sql
SELECT name FROM sal_emp WHERE pay_by_quarter[1] <> pay_by_quarter[2];
```

An array slice is denoted by writing `lower-bound:upper-bound` for one or more array dimensions.

```sql
SELECT schedule[1:2][1:1] FROM sal_emp WHERE name = 'Bill';
```

If you omit the `lower-bound` and/or `upper-bound` of a slice specifier, the missing bound is replaced by the lower or upper limit of the array's subscripts. For example,

```sql
SELECT schedule[:][1:1] FROM sal_emp WHERE name = 'Bill';
```

If any dimension is written as a slice, i.e., contains a colon, then all dimensions are treated as slices. Any dimension that has only a single number (no colon) is treated as being from 1 to the number specified. For
example, `[2]` is treated as `[1:2]`.

 #### 8.15.4 Modifying Arrays (Array Concatenation)

Replace an array value completely:

```sql
UPDATE sal_emp SET pay_by_quarter = '{25000, 25000, 27000, 26000}' WHERE name = 'Carol';
UPDATE sal_emp SET pay_by_quarter = ARRAY[25000, 25000, 27000, 26000] WHERE name = 'Carol';
```

An array can also be updated at a single element:

```sql
UPDATE sal_emp SET pay_by_quarter[4] = 15000 WHERE name = 'Bill';
```

or updated in a slice:

```sql
UPDATE sal_emp SET pay_by_quarter[1:2] = '{27000,27000}' WHERE name = 'Carol';
```

A stored array value can be enlarged by assigning to elements not already present. However, currently, enlargement in this fashion is only allowed for one-dimensional arrays, not multidimensional arrays.

----

New array values can also be constructed using the concatenation operator, `||`:

```
SELECT ARRAY[1,2] || ARRAY[3,4];
?column?
-----------
{1,2,3,4}
(1 row)

SELECT ARRAY[5,6] || ARRAY[[1,2],[3,4]];
?column?
---------------------
{{5,6},{1,2},{3,4}}
(1 row)
```

The concatenation operator accepts two N-dimensional arrays, or an N-dimensional and an N+1-dimensional array. In the first case, the two arrays are merged. In the second case, the N-dimensional array is pushed onto the beginning or end of the N+1-dimensional array.

Besides, An array can also be constructed by using the functions `array_prepend`, `array_append`, or `array_cat`.

#### 8.15.5 Searching in Arrays

Search for rows of which an array column contains a specific element:

```sql
SELECT * FROM sal_emp WHERE 10000 = ANY(pay_by_quarter);
```

In addition, you can find rows where the array has all values equal to 10000 with:

```sql
SELECT * from sal_emp WHERE 10000 = ALL(pay_by_quarter);
```

You can also search for specific values in an array using the `array_position` and `array_positions` functions.

# Ch.9 Functions and Operators

## 9.1 Logical Operators

SQL uses a three-valued logic system with `true`, `false`, and `null`, which represents “unknown”.

Logic operators are `AND`, `OR` and `NOT`. All of them yields `null` as the result if any operand are `null`.

## 9.2 Comparison Functions and Operators

`<` `>` `<=` `>=` `=` `<>` `!=`

`<>` is the standard SQL notation for "not equal" and `!=` is an alias of `<>`.

----

Comparison predicates:

- `BETWEEN ... AND ...` , `NOT BETWEEN ... AND ...`

- `BETWEEN SYMMETRIC ... AND ...` , `NOT BETWEEN SYMMETRIC ... AND ... `

- `IS DISTINCT FROM` , `IS NOT DISTINCT FROM`

- `IS NULL`, `IS NOT NULL`

- `IS TRUE`, `IS NOT TRUE`, `IS FALSE`, `IS NOT FALSE`, `IS UNKNOWN`, `IS NOT UNKNOWN`

  

`BETWEEN SYMMETRIC` is like `BETWEEN` except there's no requirements that the left argument is less than or equal to the right argument.

Ordinary comparison operators yield null when either input is null. If you don't expect this behavior, you can use `IS DISTINCT FROM`, which treats null as a comparable value.

Do **NOT** write expression = NULL because NULL is not “equal to” NULL. Instead, you should use `IS NULL` or `IS NOT NULL` to check whether a value is or is not null.

Similarly, `IS TRUE`, `IS FALSE` and `IS UNKNOWN` will always return true or false, never a null value, even when the operand is null.

## 9.7 Pattern Matching

### 9.7.1 `LIKE`, `ILIKE`

<pre>
    <em>string</em> LIKE <em>pattern</em> [ESCAPE <em>escape-character</em>]
	<em>string</em> NOT LIKE <em>pattern</em> [ESCAPE <em>escape-character</em>]
</pre>

The `LIKE` expression returns true if the string matches the supplied pattern.

- If pattern does not contain percent signs or underscores, then the pattern only represents the string itself; in that case LIKE acts like the equals operator. 
- An underscore (**_**) in pattern stands for (matches) any **single** character; 
- a percent sign (**%**) matches any sequence of **zero or more** characters.

To match a literal underscore or percent sign without matching other characters, the respective character in pattern must be preceded by the escape character. The default escape character is the backslash but a different one can be selected by using the `ESCAPE` clause. To match the escape character itself, write two escape characters.

----

The key word `ILIKE` can be used instead of `LIKE` to make the match **case-insensitive** according to the active locale. 

The operator `~~` is equivalent to `LIKE`, and `~~*` corresponds to `ILIKE`. There are also `!~~` and `!~~*` operators that represent `NOT LIKE` and `NOT ILIKE`, respectively.

## 9.18 Conditional Expressions

### 9.18.1 `CASE`

`CASE` clauses can be used whenever an expression is valid.

There are two forms of `CASE` expression. 

<pre>
CASE WHEN <em>condition</em> THEN <em>result</em>
	[WHEN ...]
	[ELSE <em>result</em>]
END
</pre>

For example,

```
SELECT a,
	CASE WHEN a=1 THEN 'one'
		 WHEN a=2 THEN 'two'
         ELSE 'other'
    END
FROM test;

a | case
---+-------
1 | one
2 | two
3 | other
```

<pre>
CASE <em>expression</em>
	WHEN <em>value</em> THEN <em>result</em>
	[WHEN ...]
	[ELSE <em>result</em>]
END
</pre>

For example,

```
SELECT a,
    CASE a 
    WHEN 1 THEN 'one'
    WHEN 2 THEN 'two'
    ELSE 'other'
    END
FROM test;
a | case
---+-------
1 | one
2 | two
3 | other
```

Since the second form of `CASE` clause implicitly use equality test(`=`), you cannot not write something like `CASE a WHEN null...`. Instead, you should use `CASE WHEN a is null...`. 

## 9.21 Aggregate Functions

Aggregate functions compute a single result from a set of input values.

Commonly used aggregate functions:

| Function                             | Description                                                  |
| ------------------------------------ | ------------------------------------------------------------ |
| `count(*)` → `bigint`                | Computes the number of input rows.                           |
| `count("any")`→`bigint`              | Computes the number of input rows in which the input value is not null.<br />Count distinct values: `count(distinct colname)` |
| `avg(see text)` → see text           | Computes the average (arithmetic mean) of all the non-null input values.<br />For `smallint`, `integer`, `bigint` and  `numeric`, the return type is `numeric`; For `real` and `double precision`, the return type is the `double precision`; For `interval`, the return type is `interval`. |
| `max(see text)` → same as input type |                                                              |
| `min(see text)` → same as input type |                                                              |



## 9.23 Subquery Expressions

### 9.23.1 `EXISTS`

<pre>
    EXISTS (<em>subquery</em>)
</pre>

The subquery is evaluated to determine whether it returns any rows. If it returns at least one row, the result of `EXISTS` is “true”; if the subquery returns no rows, the result of `EXISTS` is “false”.

Since the result depends only on whether any rows are returned, and not on the contents of those rows, the output list of the subquery is normally unimportant. A common coding convention is to write all `EXISTS` tests in the form `EXISTS(SELECT 1 WHERE ...)`.

### 9.23.2 `IN`

<PRE>
    <em>expression</em> IN (<em>subquery</em>)
</pre>

The subquery must return exactly one column. The result of `IN` is true if any equal subquery row is found and false otherwise.

Note that if the left-hand expression yields null, or if there are no equal right-hand values and at least one right-hand row yields null, the result of the IN construct will be null, not false. This is in accordance with SQL's normal rules for Boolean combinations of null values.

Another usage:

<pre>
    <em>row_constructor</em> IN (<em>subquery</em>)
</pre>
### 9.23.3 `NOT IN`

Refer to section 9.23.2 `IN`.

### 9.23.4 `ANY`/`SOME`

<pre>
    <em>expression</em> <em>operator</em> ANY (<em>subquery</em>)
    <em>expression</em> <em>operator</em> SOME (<em>subquery</em>)
</pre>

The subquery must return exactly one column. 

The left-hand expression is evaluated and compared to each row of the subquery result using the given operator, which must yield a Boolean result. 

The result of ANY is “true” if any true result is obtained. The result is “false” if no true result is found (including the case where the subquery returns no rows).

`SOME` is a synonym for `ANY`. `IN` is equivalent to `= ANY`.

Note that if there are no successes and at least one right-hand row yields null for the operator's result, the result of the `ANY` construct will be null, not false. This is in accordance with SQL's normal rules for Boolean combinations of null values.

Another possible syntax:

<pre>
    <em>row_constructor</em> <em>operator</em> ANY (<em>subquery</em>)
    <em>row_constructor</em> <em>operator</em> SOME (<em>subquery</em>)
</pre>

### 9.23.5 `ALL`

<pre>
    <em>expression</em> <em>operator</em> ALL (<em>subquery</em>)
</pre>

The restrictions on *expression* and *subquery* is the same as in `ANY`/`SOME`.

The result of `ALL` is “true” if all rows yield true (**including the case where the subquery returns no rows**). The result is “false” if any false result is found. The result is NULL if no comparison with a subquery row returns false, and at least one comparison returns NULL.

Note that `NOT IN` is equivalent to `<> ALL`

Another possible syntax:

<pre>
    <em>row_constructor</em> <em>operator</em> ALL (<em>subquery</em>)
</pre>
## 9.24 Row and Array Comparisons

This section describes several specialized constructs for making multiple comparisons between groups of values. These forms are syntactically related to the subquery forms of the previous section, but do not involve subqueries.

### 9.24.1 `IN`

<pre>
    <em>expression</em> IN (<em>value</em> [, ...])
</pre>

The result is “true” if the left-hand expression's result is equal to any of the right-hand expressions.

Note that if the left-hand expression yields null, or if there are no equal right-hand values and at least one right-hand expression yields null, the result of the `IN` construct will be null, not false. This is in accordance with SQL's normal rules for Boolean combinations of null values.

### 9.24.2 `NOT IN`

The negation of the `IN` operator.





## Joins between tables

Queries that access multiple tables (or multiple instances of the same table) at one time are called **join
queries**.

Return all the weather records together with the location of the associated city:

```sql
SELECT city, temp_lo, temp_hi, prcp, date, location FROM weather JOIN cities ON city = name;
```

Since the columns all had different names, the parser automatically found which table they belong to. 
If there were duplicate column names in the two tables you'd need to qualify the column names to show
which one you meant, as in:

```sql
SELECT weather.city, weather.temp_lo, weather.temp_hi,weather.prcp, weather.date, cities.location FROM weather JOIN cities ON weather.city = cities.name;
```

An alternative form of join queries:

```sql
SELECT * FROM weather, cities WHERE city = name;
```

### Outer joins

The joins we have seen so far are **inner joins**.

If no matching row is found we want some “empty values” to be substituted for the cities table's columns. This kind of query is called an **outer join**.

An example of (left) outer join:

```sql
SELECT * FROM weather LEFT OUTER JOIN cities ON weather.city = cities.name;
```

This query is called a **left outer join** because the table mentioned on the left of the join operator will have each of its rows in the output **at least once**. When outputting a left-table row for which there is no right-table match, empty (null) values are substituted for the right-table columns.

Apart from left outer joins, there are also right outer joins and full outer joins.

### Self joins

We can also join a table against itself. This is called a **self join**.

As an example, suppose we wish to find all the weather records that are in the temperature range of other weather records.

```sql
SELECT w1.city, w1.temp_lo AS low, w1.temp_hi AS high, w2.city, w2.temp_lo AS low, w2.temp_hi AS high FROM weather w1 JOIN weather w2 ON w1.temp_lo < w2.temp_lo AND w1.temp_hi > w2.temp_hi;
```

# Advanced features

## Views

**Views** gives a name to the query that you can refer to like an ordinary table.

```sql
CREATE VIEW myview AS
	SELECT name, temp_lo, temp_hi, prcp, date, location
		FROM weather, cities
		WHERE city = name;
		
SELECT * FROM myview;
```

## Foreign keys

You want to make sure that no one can insert rows in the weather table that do not have a matching entry in the cities table.

```sql
CREATE TABLE cities (
	name varchar(80) primary key,
	location point
);

CREATE TABLE weather (
    city varchar(80) references cities(name),
    temp_lo int,
    temp_hi int,
    prcp real,
    date date
);
```

## Transactions

A transaction bundles multiple steps into a single, all-or-nothing operation. 

A transaction is said to be **atomic**: from the point of view of other transactions, it either happens completely or not at all.

In PostgreSQL, a transaction is set up by surrounding the SQL commands of the transaction with `BEGIN` and `COMMIT` commands.

If we decide not to commit, we can issue the command `ROLLBACK` instead of `COMMIT`, and all our updates so far will be canceled.

Furthermore, you can define a savepoint with `SAVEPOINT` and roll back to it with `ROLLBACK TO`.

```SQL
BEGIN;
UPDATE accounts SET balance = balance - 100.00 WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00 WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00 WHERE name = 'Wally';
COMMIT;
```

## Window functions

Window functions do not cause rows to become grouped into a single output row like non-
window aggregate calls would. Instead, the rows retain their separate identities.(new columns are added)

Window functions are executed after `GROUP BY`, `HAVING` and`WHERE` clauses and non-window aggregate functions.

Here is an example that shows how to compare each employee's salary with the average salary in his or her department:

```sql
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;
```

You can also control the order in which rows are processed by window functions using `ORDER BY` within `OVER`.

```sql
SELECT depname, empno, salary, rank() OVER (PARTITION BY depname ORDER BY salary DESC) FROM empsalary;
```

```
 depname | empno | salary | rank
-----------+-------+--------+------
 develop | 8 | 6000 | 1
 develop | 10 | 5200 | 2
 develop | 11 | 5200 | 2
 develop | 9 | 4500 | 4
 develop | 7 | 4200 | 5
 personnel | 2 | 3900 | 1
 personnel | 5 | 3500 | 2
 sales | 1 | 5000 | 1
 sales | 4 | 4800 | 2
 sales | 3 | 4800 | 2
(10 rows)
```

It is also possible to omit `PARTITION BY`, in which case there is a single partition containing all rows.

There is another important concept associated with window functions: for each row, there is a set of rows within its partition called its **window frame**. Some window functions act only on the rows of the window frame, rather than of the whole partition.

By default, if `ORDER BY` is supplied then the frame consists of all rows from the start of the partition up through the current row, plus any following rows that are equal to the current row according to the `ORDER BY` clause. When `ORDER BY` is omitted, the default frame
consists of all rows in the partition.

```sql
SELECT salary, sum(salary) OVER () FROM empsalary;
```

```
 salary | sum
--------+-------
 5200 | 47100
 5000 | 47100
 3500 | 47100
 4800 | 47100
 3900 | 47100
 4200 | 47100
 4500 | 47100
 4800 | 47100
 6000 | 47100
 5200 | 47100
(10 rows)
```

```SQL
SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary;
```

```
 salary | sum
--------+-------
 3500 | 3500
 3900 | 7400
 4200 | 11600
 4500 | 16100
 4800 | 25700
 4800 | 25700
 5000 | 30700
 5200 | 41100
 5200 | 41100
 6000 | 47100
(10 rows)
```

When a query involves multiple window functions, you can name each windowing behavior in a `WINDOW` clause and then reference it instead of writing each.

```sql
SELECT sum(salary) OVER w, avg(salary) OVER w FROM empsalary WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

## Inheritance

```sql
CREATE TABLE cities (
    name text,
    population real,
    elevation int
);

CREATE TABLE capitals (
    state char(2) UNIQUE NOT NULL
) INHERITS (cities);
```

The following query finds the names of all cities, **including state capitals**, that are located at an elevation over 500 feet:

```sql
SELECT name, elevation FROM cities WHERE elevation > 500;
```

On the other hand, the following query finds all the cities that are **NOT** state capitals and are situated at an elevation over 500 feet:

```sql
SELECT name, elevation FROM ONLY cities WHERE elevation > 500;
```

Here the `ONLY` before `cities` indicates that the query should be run over only the `cities` table, and not tables below `cities` in the inheritance hierarchy.



