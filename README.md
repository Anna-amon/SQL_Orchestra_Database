# SQL_Orchestra_Database
Querying the Orchestras database containing details of concerts, members and orchestras. 

We can start by looking at the ER diagram of the database, to understand the database structure, its primary and foreign keys and how to query the database:

![image](https://github.com/Anna-amon/SQL_Orchestra_Database/assets/144830015/61e1a104-9b7f-48cb-a8de-5c9f0746f5b1)

1) Select Orchestras with a City of Origin Where a Concert Was Held in 2013

One way of doing this is by use of a sub-query:

```sql
SELECT name
FROM orchestras
WHERE city_origin IN
              (SELECT city 
               FROM concerts
               WHERE year = 2013);
```

Alternatively, you can use a Join:

```sql
SELECT o.name, o.city_origin, c.year
FROM orchestras AS o
JOIN concerts AS c ON c.orchestra_id = o.id
WHERE c.year = 2013 AND c.city = o.city_origin;
```

2) Select all orchestras that have never performed a concert:

We may use LEFT JOIN: 

```sql
SELECT o.id, o.name
FROM orchestras AS o
LEFT JOIN concerts AS c ON o.id = c.orchestra_id
WHERE c.orchestra_id IS NULL;
```

However the best way will be a sub-query:

```sql
SELECT id, name
FROM orchestras 
WHERE id NOT IN 
                (SELECT orchestra_id
                 FROM concerts);
```

3) Select Members that Belong to High-Rated Orchestras

Firstly, we can use a sub-query;

```sql
SELECT id
FROM members
WHERE orchestra_id IN 
             (SELECT id 
              FROM orchestras 
              WHERE rating > 8.0);
```

Or, we can use the JOIN function;

The first JOIN fetches all ratings in the top 20%.

```sql
SELECT m.id, o.rating
FROM members AS m
JOIN orchestras AS o ON m.orchestra_id = o.id
WHERE o.rating > 8.0
```

The second join orders the ratings in descending order and then limits the query to the top 10, so this way, we can retrieve the top ten members belonging to the highest rated orchestras. The above query is perhaps better, as it will retrieve all members from the top 20%, rather than limiting the number of members, as some members may be further down than 10 in the list, yet still belong to the same orchestra. 

```sql
SELECT m.id, o.rating
FROM members AS m
JOIN orchestras AS o ON m.orchestra_id = o.id
ORDER BY o.rating DESC
LIMIT 10;
```

4) Add a new member to the 'Vienna Philharmonic' orchestra.

```sql
INSERT INTO members (name, position, experience, orchestra_id, wage)
VALUES ('Tom Jones',
                 'Lead Violin', 
                 5, 
                (SELECT id FROM orchestras WHERE name = 'Vienna Philharmonic'), 
                 500);
```

5) Update the rating of all concerts in 'Amsterdam' to reflect a 0.1 increase.

```sql
UPDATE concerts
SET rating = rating+0.1
WHERE city = ‘Amsterdam’;
```



6) Insert a new concert that took place in 'New York' with a rating of 4.7.

```sql
INSERT INTO concerts (city, country, year, rating, orchestra_id)
VALUES ('New York', 'USA', 2010, 4.7, 1);
```

7) Increase the wage of all members in the 'London Symphony Orchestra' by 10%.

```sql
UPDATE members
SET wage = wage * 1.1
WHERE orchestra_id IN 
    (SELECT id 
     FROM orchestras 
     WHERE name = 'London Symphony Orchestra');
```

8) List the names of members and their positions, orchestras they belong to, and member numbers, sorted by member name.

```sql
SELECT m.member_id, m.name AS member_name, m.position, o.name AS orchestra_name
FROM members m
JOIN orchestras o ON m.orchestra_id = o.id
ORDER BY m.name;
```

9) List the names of orchestra members and their respective wages who earn higher than the average wage of the members in their orchestra.

```sql
SELECT m.name, m.wage
FROM members m
WHERE m.wage >
      (SELECT AVG(m2.wage)
       FROM members m2
       WHERE m2.orchestra_id = m.orchestra_id);
```

10) List the names of orchestras along with their average concert ratings, ordered by average rating in descending order.

```sql
SELECT o.name, AVG(c.rating) AS average_rating, 
FROM orchestras o
JOIN concerts c
ON o.id = c.orchestra_id
GROUP BY o.name
ORDER BY average_rating DESC;
```

11) Write a query that finds the names of orchestras with the highest average member wage among orchestras where the average wage is greater than a certain threshold.

```sql
SELECT o.name, AVG(m.wage)
FROM orchestras o
JOIN members m
ON o.id = m.orchestra_id
HAVING AVG(m.wage) > 50
GROUP BY o.name
ORDER BY average_wage DESC;
LIMIT 1;
```                  

12) List all concerts with their corresponding orchestra names where the rating of the concert is higher than the average rating of all concerts held in the same city.

```sql
SELECT c.id, o.name
FROM concerts c
JOIN orchestras o ON c.orchestra_id = o.id
WHERE c.rating >
       (SELECT AVG(c2.rating)
        FROM concerts c2
        WHERE c2.city = c.city);
```
