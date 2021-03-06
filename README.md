# sql

# (1a.) Actor
#---------------------------#
use sakila;

SELECT * FROM sakila.actor;

SELECT first_name, last_name FROM sakila.actor;


# (1b.) Actor Name
#-----------------------------#
SELECT ucase(CONCAT(first_name,' ', last_name)) as 'Actor Name' from sakila.actor; -- same result without uppercase command


# (2a.) Find Joe
#-----------------------------#
SELECT actor_id, first_name, last_name FROM sakila.actor
WHERE first_name = 'Joe';


# (2b.) Last name contains "GEN"
#-----------------------------#
SELECT actor_id, first_name, last_name FROM sakila.actor
WHERE last_name LIKE '%GEN%'; 
-- ORDER BY actor_id ASC; -- same
-- GROUP BY actor_id ASC; -- same


# (2c.) last name contains "LI"
#-----------------------------#
SELECT actor_id, first_name, last_name FROM sakila.actor
WHERE last_name LIKE '%LI%' 
GROUP BY last_name, first_name ASC;

/*SELECT actor_id, first_name, last_name FROM sakila.actor
WHERE last_name LIKE '%LI%' 
ORDER BY last_name, first_name ASC;  */ -- same


# (2d.) Locate country info using IN command
#-----------------------------#
SELECT * FROM sakila.country;

SELECT country_id, country FROM sakila.country
WHERE country IN ('Afghanistan', 'Bangladesh', 'China');


# (3a.) Add middle name column in between first and last
#-----------------------------#
ALTER TABLE sakila.actor
ADD middle_name VARCHAR(30) AFTER first_name;

SELECT * FROM sakila.actor; -- middle name column successfully added

# (3b.) Middle name datatype change to BLOB
#-----------------------------#
ALTER TABLE sakila.actor
MODIFY middle_name BLOB; 

-- Successful: 18:08:03	ALTER table sakila.actor MODIFY middle_name BLOB	200 row(s) affected Records: 200  Duplicates: 0  Warnings: 0	0.312 sec


# (3c.) delete middle_name column
#-----------------------------#
ALTER TABLE sakila.actor
DROP middle_name; -- ALTER table sakila.actor DROP middle_name

SELECT * FROM sakila.actor; -- result grid check


# (4a.) list number of actor names
#-----------------------------#

SELECT * FROM sakila.actor
GROUP BY last_name;

SELECT COUNT(*) as 'Last Name Count', last_name, first_name FROM sakila.actor
GROUP BY last_name;

# (4b.) list number of actor names with 2 or more
#-----------------------------#

SELECT COUNT(*) as 'Last Name Count', last_name, first_name FROM sakila.actor
GROUP BY last_name
HAVING COUNT(last_name) > 1;


# (4c.) Find Harpo/Groucho williams fix
#-----------------------------#

SELECT * FROM sakila.actor
WHERE last_name LIKE 'williams'; -- find groucho/harpo williams

SELECT * from sakila.actor
WHERE actor_id = 172;

UPDATE sakila.actor
SET first_name = 'HARPO'
WHERE actor_id = 172; -- success: 20:42:49	UPDATE sakila.actor SET first_name = 'HARPO' WHERE actor_id = 172	1 row(s) affected Rows matched: 1  Changed: 1  Warnings: 0	0.047 sec


# (4d.) Actor name fix again
#-----------------------------#
UPDATE sakila.actor
SET first_name = 'GROUCHO'
WHERE actor_id = 172; -- success: 20:55:00	UPDATE sakila.actor SET first_name = 'GROUCHO' WHERE actor_id = 172	1 row(s) affected Rows matched: 1  Changed: 1  Warnings: 0	0.031 sec


SELECT * FROM sakila.actor
WHERE actor_id = 172; -- check



# (5a.) address table info
#-----------------------------#
SHOW CREATE TABLE sakila.address;

-- result grid: 'address', 'CREATE TABLE `address` (\n  `address_id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,\n  `address` varchar(50) NOT NULL,\n  `address2` varchar(50) DEFAULT NULL,\n  `district` varchar(20) NOT NULL,\n  `city_id` smallint(5) unsigned NOT NULL,\n  `postal_code` varchar(10) DEFAULT NULL,\n  `phone` varchar(20) NOT NULL,\n  `location` geometry NOT NULL,\n  `last_update` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,\n  PRIMARY KEY (`address_id`),\n  KEY `idx_fk_city_id` (`city_id`),\n  SPATIAL KEY `idx_location` (`location`),\n  CONSTRAINT `fk_address_city` FOREIGN KEY (`city_id`) REFERENCES `city` (`city_id`) ON UPDATE CASCADE\n) ENGINE=InnoDB AUTO_INCREMENT=606 DEFAULT CHARSET=utf8'


# (6a.) Join tables 'staff' and 'address'
#-----------------------------#
SELECT * FROM sakila.staff;
SELECT * FROM sakila.address;

SELECT s.first_name, s.last_name, a.address
FROM sakila.staff s
LEFT JOIN sakila.address a 
ON s.address_id = a.address_id; 



# (6b.) staff, payment join
#-----------------------------#

SELECT * FROM sakila.staff; -- staff_id, store_id, address_id
#SELECT * FROM sakila.address; -- address_id, city_id
SELECT * FROM sakila.payment;

SELECT  s.staff_id, s.first_name, s.last_name, sum(p.amount) as 'august total amount rung'
FROM sakila.staff s
LEFT JOIN sakila.payment p 
ON s.store_id = p.staff_id
WHERE payment_date >= ('2005-08-01') AND payment_date<=('2005-08-31')
GROUP BY s.staff_id;

# (6c.) Inner Join film and film_actor
#-----------------------------#

SELECT * FROM sakila.film; -- film_id
#SELECT * FROM sakila.actor; -- actor_id
SELECT * FROM sakila.film_actor; -- film_id, actor_id


SELECT f.film_id, f.title, count(a.film_id) as 'number of actors'
FROM sakila.film f
INNER JOIN sakila.film_actor a
ON f.film_id = a.film_id
group by film_id;


/*SELECT f.title, concat(a.first_name,' ' , a.last_name) as Actor
FROM sakila.film f
INNER JOIN sakila.actor a
ON f.film_id = a.actor_id; -- Extra practice*/



# (6d.) How Many Hunchback Impossible film in inventory
#-----------------------------#
SELECT * FROM sakila.inventory
where film_id = 439; -- research
#having (film_id);
SELECT * FROM sakila.film
;#where title = 'HUNCHBACK IMPOSSIBLE'; -- research

SELECT film_id, count(film_id) FROM sakila.inventory
where film_id = 439; -- research
#having (film_id);
SELECT * FROM sakila.film
where title = 'HUNCHBACK IMPOSSIBLE'; -- research


SELECT f.film_id, f.title, COUNT(i.film_id) as 'count'
FROM sakila.inventory i
INNER JOIN sakila.film f
ON f.film_id = i.film_id
WHERE f.film_id ='439'
GROUP BY f.film_id
HAVING (f.film_id); -- 6 copies


# (6e.) Join payment and customer
#-----------------------------#
SELECT * FROM sakila.payment; -- payment_id, customer_id, staff_id
SELECT * FROM sakila.customer; -- customer_id, store_id, address_id


SELECT p.customer_id, c.first_name, c.last_name, SUM(p.amount) as 'Total Amount Paid'
FROM sakila.payment p
INNER JOIN sakila.customer c
ON c.customer_id = p.customer_id
GROUP BY customer_id ASC;

# (7a.) Music of Queen and Kris K.
#-----------------------------#

SELECT * FROM sakila.film;
SELECT * FROM sakila.language;

select f.title, l.name from sakila.film f
Inner join sakila.language l
on l.language_id = f.language_id
where l.name = 'English'
group by f.title
having f.title like 'Q%' or f.title like 'K%'; -- also considered subquery

select title, language_id 
from sakila.film 
where title 
like 'Q%' or title like 'K%' 
having language_id 
IN (select language_id from sakila.language where name IN (select name from sakila.language)); -- Subquery



# (7b.) Alone Trip
#-----------------------------#

select * from sakila.actor where actor_id = 17;
select * from sakila.film where title = 'Alone Trip';
select * from sakila.inventory where film_id = 17;
#GROUP BY a.actor_id

SELECT f.title, concat(a.first_name,' ' , a.last_name) as Actor, count(f.film_id) as Count
FROM sakila.film f
INNER JOIN sakila.actor a
ON f.film_id = a.actor_id
WHERE f.film_id ='17';


#HAVING count(f.film_id) ;


# (7c.) Email marketing
#-----------------------------#
SELECT * FROM sakila.country where country = 'Canada'; -- country_id
SELECT first_name, last_name, email, address_id FROM sakila.customer; -- customer_id, 
SELECT * FROM sakila.address; -- address_id, city_id
SELECT * FROM sakila.city where country_id = 20; -- city_id, country_id


SELECT c.first_name, c.last_name, c.email, a.address
    FROM sakila.customer c
    left JOIN sakila.address a
    ON c.customer_id = a.address_id;
   # where ci.country_id= '20';
    #where co.country = 'Canada'
    
    SELECT c.first_name, c.last_name, c.email, a.address
    FROM sakila.customer c
    left JOIN sakila.address a
    ON c.customer_id = a.address_id
    having count(a.city_id =20);
    
    SELECT *
    FROM sakila.address a
    LEFT JOIN sakila.city c
    ON c.city_id = a.city_id;


   SELECT *
    FROM sakila.address a
    left JOIN sakila.city ci
    ON ci.country_id = a.address_id;
    #where  = ''
    #group by city asc;

# (7d.) Lagging Sales, Rated G Family movies
#-----------------------------#

SELECT f.title, f.rating
FROM sakila.inventory i
INNER JOIN sakila.film f
ON f.film_id = i.film_id
WHERE rating ='G'
GROUP BY f.film_id
HAVING (f.film_id);



# (7e.) Frequent rents desc
#-----------------------------#

SELECT title, count(rental_duration)
FROM sakila.inventory i
left JOIN sakila.film f
ON f.film_id = i.film_id
GROUP BY f.film_id
ORDER BY (rental_duration) DESC;


# (7f.) Business
#-----------------------------#
SELECT * FROM sakila.payment;

SELECT store_id, sum(amount) as Totals
FROM sakila.inventory i
left JOIN sakila.payment p
ON i.store_id = p.payment_id
group by store_id;

# (7g.) Top Five Genres
#-----------------------------#
SELECT * FROM sakila.category; -- category_id
SELECT * FROM sakila.film_category; -- film_id, category_id
SELECT * FROM sakila.inventory; -- inventory_id, film_id, store_id
SELECT * FROM sakila.payment; -- payment_id, customer_id, staff_id, rental_id
SELECT * FROM sakila.rental; -- rental_id, customer_id, staff_id

SELECT * FROM sakila.category c
LEFT JOIN sakila.film_category f
ON f.category_id = c.category_id
group by name;

SELECT * FROM sakila.payment p
LEFT JOIN sakila.rental r
ON p.rental_id = r.rental_id;

SELECT name as Genre, count(name) as Count FROM sakila.category c
LEFT JOIN sakila.film_category f
ON f.category_id = c.category_id
group by name ASC limit 5;


# (8a.) View
#-----------------------------#
CREATE VIEW TOP_FIVE_SUMMARY AS
 SELECT name as Genre, count(name) as Count FROM sakila.category c
LEFT JOIN sakila.film_category f
ON f.category_id = c.category_id
group by name ASC limit 5;

# (8b.) create view
#-----------------------------#
SELECT * FROM TOP_FIVE_SUMMARY;

# (8c.) drop view
#-----------------------------#

drop view TOP_FIVE_SUMMARY;
 
