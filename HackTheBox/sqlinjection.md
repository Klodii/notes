## How to detect SQL injection vulnerabilities

 You can detect SQL injection manually using a systematic set of tests against
 every entry point in the application. To do this, you would typically submit:

-    The single quote character `'` and look for errors or other anomalies.
-    Some SQL-specific syntax that evaluates to the base (original) value of the
     entry point, and to a different value, and look for systematic differences
     in the application responses.
-    Boolean conditions such as `OR 1=1` and `OR 1=2`, and look for differences
     in the application's responses.
-    Payloads designed to trigger time delays when executed within a SQL query,
     and look for differences in the time taken to respond.
-    OAST payloads designed to trigger an out-of-band network interaction when
     executed within a SQL query, and monitor any resulting interactions.


## SQL injection UNION attacks

With this attack we can return values from different tables other that the one
that is currently queried.

When an application is vulnerable to SQL injection, and the results of the query
are returned within the application's responses, you can use the `UNION` keyword
to retrieve data from other tables within the database. This is commonly known
as a SQL injection UNION attack.

The `UNION` keyword enables you to execute one or more additional `SELECT` queries
and append the results to the original query.
For example:

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```

This SQL query returns a single result set with two columns, containing values
from columns `a` and `b` in `table1` and columns `c` and `d` in `table2`.


For a UNION query to work, two key requirements must be met:

- The individual queries must return the same number of columns.
- The data types in each column must be compatible between the individual
  queries.

To carry out a SQL injection UNION attack, make sure that your attack meets
these two requirements. This normally involves finding out:

- How many columns are being returned from the original query.
- Which columns returned from the original query are of a suitable data type to
  hold the results from the injected query.


### Determining the number of columns required

When you perform a SQL injection UNION attack, there are two effective methods
to determine how many columns are being returned from the original query.

#### First method
One method involves injecting a series of `ORDER BY` clauses and incrementing
the specified column index until an error occurs.
For example, if the injection point is a quoted string within the `WHERE` clause
of the original query, you would submit:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

This series of payloads modifies the original query to order the results by
different columns in the result set. The column in an `ORDER BY` clause can be
specified by its index, so you don't need to know the names of any columns. When
the specified column index exceeds the number of actual columns in the result
set, the database returns an error, such as:
```
The ORDER BY position number 3 is out of range of the number of items in the
select list.
```

The application might actually return the database error in its HTTP response,
but it may also issue a generic error response. In other cases, it may simply
return no results at all. Either way, as long as you can detect some difference
in the response, you can infer how many columns are being returned from the
query.

#### Second Method

The second method involves submitting a series of `UNION SELECT` payloads
specifying a different number of null values:

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

If the number of nulls does not match the number of columns, the database
returns an error, such as:
```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an
equal number of expressions in their target lists.
```

We use `NULL` as the values returned from the injected `SELECT` query because the
data types in each column must be compatible between the original and the
injected queries. `NULL` is convertible to every common data type, so it maximizes
the chance that the payload will succeed when the column count is correct.

As with the `ORDER BY` technique, the application might actually return the
database error in its HTTP response, but may return a generic error or simply
return no results. When the number of nulls matches the number of columns, the
database returns an additional row in the result set, containing null values in
each column. The effect on the HTTP response depends on the application's code.
If you are lucky, you will see some additional content within the response, such
as an extra row on an HTML table. Otherwise, the null values might trigger a
different error, such as a `NullPointerException`. In the worst case, the response
might look the same as a response caused by an incorrect number of nulls. This
would make this method ineffective.

#### Database-specific syntax

On Oracle, every `SELECT` query must use the `FROM` keyword and specify a valid
table. There is a built-in table on Oracle called `dual` which can be used for
this purpose. So the injected queries on Oracle would need to look like:

```
' UNION SELECT NULL FROM DUAL--
```

The payloads described use the double-dash comment sequence `--` to comment out
the remainder of the original query following the injection point. On MySQL, the
double-dash sequence must be followed by a space. Alternatively, the hash
character `#` can be used to identify a comment.

### Determin the column data type

A SQL injection UNION attack enables you to retrieve the results from an
injected query. The interesting data that you want to retrieve is normally in
string form. This means you need to find one or more columns in the original
query results whose data type is, or is compatible with, string data.

After you determine the number of required columns, you can probe each column to
test whether it can hold string data. You can submit a series of `UNION SELECT`
payloads that place a string value into each column in turn. For example, if the
query returns four columns, you would submit:

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

If the column data type is not compatible with string data, the injected query
will cause a database error, such as:
```
Conversion failed when converting the varchar value 'a' to data type int.
```

If an error does not occur, and the application's response contains some
additional content including the injected string value, then the relevant column
is suitable for retrieving string data.


## Warning

Take care when injecting the condition `OR 1=1` into a SQL query. Even if it
appears to be harmless in the context you're injecting into, it's common for
applications to use data from a single request in multiple different queries. If
your condition reaches an `UPDATE` or `DELETE` statement, for example, it can
result in an accidental loss of data.

## Notes

If the injection is beeing done via query paramters, we have to use the `+`
instead of the ` ` (space), like this

```
GET /filter?category=Accessories'+UNION+SELECT+NULL,NULL,NULL--
```
