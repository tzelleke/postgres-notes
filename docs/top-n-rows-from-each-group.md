# Top N rows from each group

## Sample data

{{ read_csv('data/top-n-rows-from-each-group--employees.csv') }}

## Problem

For each department, find the N employee(s) with the highest salary.

## Solutions

=== "DISTINCT ON"

    ``` sql
    select distinct on (dep)
        *
    from
        employees
    order by
        dep
      , salary desc
    ```

    ``` text title="Output"
     id | name |    dep     | salary  
    ----+------+------------+--------
     3  | Jane | Accounting |  1500   
     5  | Alex |     IT     |  3000   
     1  | Mike | Management |  5000   
    ```

    !!! note

        With `DISTINCT ON` we are limited to exactly the first row from each group.

        If there are multiple rows with the same salary in a group,
        provide additional `ORDER BY` columns to resolve the ambiguity.

    !!! note

        The `DISTINCT ON` expression(s) must match the **leftmost** `ORDER BY` expression(s).

        `ORDER BY` will usually contain additional expression(s)
        that determine the desired precedence of rows within each group.

    `DISTINCT ON` accepts multiple columns.

    ``` sql
    select distinct on (department, sub_department)
        ...
    ```

    `DISTINCT ON` is a PostgreSQL extension to the SQL standard.

=== "Lateral join"

    ``` sql hl_lines="13"
    select
        e.*
    from
        employees e
      , lateral (
            select
                *
            from
                employees
            where
                dep = e.dep
            order by salary desc
            limit 1  -- or limit N
            ) sq
    where
        e.id = sq.id
    order by
        dep, salary desc
    ```

    ``` text title="Output"
     id | name |    dep     | salary  
    ----+------+------------+--------
     3  | Jane | Accounting |  1500   
     5  | Alex |     IT     |  3000   
     1  | Mike | Management |  5000   
    ```

    !!! note

        If there are multiple rows with the same salary in a group,
        provide additional `ORDER BY` columns to the lateral subquery
        to resolve the ambiguity.

    !!! note

        The `,` in the `FROM` clause is shorthand for `CROSS JOIN`.

=== "Window function"

    ``` sql hl_lines="12"
    select
        *
    from
        (
            select
                *
              , rank() over (partition by dep order by salary desc) rank
            from
                employees
        ) sq
    where
        rank = 1  -- or rank <= N
    ```

    ``` text title="Output"
     id | name |    dep     | salary | rank  
    ----+------+------------+--------+------
     3  | Jane | Accounting |  1500  |  1    
     5  | Alex |     IT     |  3000  |  1    
     6  | Tim  |     IT     |  3000  |  1    
     1  | Mike | Management |  5000  |  1    
    ```

=== "Correlated subquery"

    ``` sql hl_lines="14"
    select
        *
    from
        employees e
    where
            id = (
            select
                id
            from
                employees
            where
                dep = e.dep
            order by salary desc
            limit 1  -- or limit N
            )
        )
    ```

    ``` text title="Output"
     id | name |    dep     | salary  
    ----+------+------------+--------
     1  | Mike | Management |  5000   
     3  | Jane | Accounting |  1500   
     5  | Alex |     IT     |  3000   
    ```

    !!! note

        If there are multiple rows with the same salary in a group,
        provide additional `ORDER BY` columns to the subquery
        to resolve the ambiguity.

## References

- [SO: Select first row in each GROUP BY group?](https://stackoverflow.com/questions/3800551/select-first-row-in-each-group-by-group)
- [PostgreSQL: DISTINCT ON](https://www.postgresql.org/docs/current/sql-select.html#SQL-DISTINCT)
- [PostgreSQL: LATERAL](https://www.postgresql.org/docs/current/sql-select.html#SQL-FROM)
- [PostgreSQL: Window functions](https://www.postgresql.org/docs/current/tutorial-window.html)
- [PostgreSQL: Scalar subqueries](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES)
