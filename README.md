1.	We would like to know the 5 movies with the lowest income and the 5 with the highest. Provide a table with the payment received by movie and the AVG of the payment add a new column classifying the movies in two groups depending on the amount of the payment (one between 0 and 6 and other greater than 6) and divide them into 4 levels (first_quarter, second_quarter, third_quarter, and final_quarter) based on the quartiles (25%, 50%, 75%) of the rental duration for movies across all categories.

SQL QUERY
1.
WITH tab1 AS (SELECT p.amount AS payment_amt, f.title AS title,
				AVG (p.amount) AS AVG_payment_amt,
				NTILE (4) OVER (PARTITION BY f.film_id ORDER BY f.rental_duration) AS rental_duration_Q
				FROM film f
				JOIN inventory i
				ON i.film_id = f.film_id
				JOIN rental r
				ON r.inventory_id = i.inventory_id
				JOIN payment p
				ON p.rental_id = r.rental_id
				GROUP BY 1, 2, f.film_id, f.rental_duration)
SELECT payment_amt, title, AVG_payment_amt, rental_duration_Q,
		CASE WHEN payment_amt BETWEEN 0 AND 6 THEN 'first_group'
		WHEN payment_amt > 6 THEN 'second_group' END AS payment_classification
FROM tab1
ORDER BY payment_amt
LIMIT 5
;

2.
WITH tab1 AS (SELECT p.amount AS payment_amt, f.title AS title,
				AVG (p.amount) AS AVG_payment_amt,
				NTILE (4) OVER (PARTITION BY f.film_id ORDER BY f.rental_duration) AS rental_duration_Q
				FROM film f
				JOIN inventory i
				ON i.film_id = f.film_id
				JOIN rental r
				ON r.inventory_id = i.inventory_id
				JOIN payment p
				ON p.rental_id = r.rental_id
				GROUP BY 1, 2, f.film_id, f.rental_duration)
SELECT payment_amt, title, AVG_payment_amt, rental_duration_Q,
		CASE WHEN payment_amt BETWEEN 0 AND 6 THEN 'first_group'
		WHEN payment_amt > 6 THEN 'second_group' END AS payment_classification
FROM tab1
ORDER BY payment_amt DESC
LIMIT 5
;

2.	We want to know how many movies of each category we have (Animation, Children, Classics, Comedy, Family and Music) and would be helpful if we could know the rental count for each category.

SQL QUERY
SELECT c.name,
		COUNT(f.film_id) AS count_movies,
		COUNT (r.rental_id) AS count_rental
FROM film f
JOIN film_category fc
ON fc.film_id = f.film_id
JOIN category c
ON c.category_id = fc.category_id
JOIN inventory i
ON i.film_id = f.film_id
JOIN rental r
ON r.inventory_id = i.inventory_id
GROUP BY 1
;

3.	We would like to know who were our top 10 paying customers, how many payments they made monthly during 2007, and what was the amount of the monthly payments. Can you write a query to capture the customer’s name, month and year of payment, and total payment amount for each month by these top 10 paying customers?

SQL QUERY
SELECT p.amount,
		c.first_name || ' ' || c.last_name AS customer_name,
		DATE_PART ('year', p.payment_date) AS year,
		DATE_PART ('month', p.payment_date) AS month,
		COUNT (p.payment_id) AS num_payments
FROM customer c
JOIN payment p
ON p.customer_id = c.customer_id
GROUP BY 1, 2, 3, 4
ORDER BY num_payments DESC
LIMIT 10
;

4.	Finally, we want to make an online rental service for our customers for that we need to know which customers still active and generate a temporary password for their account with the first letter of the name, following for the number of letters of their country, the last name and finishing with the last letter of their name. We need a table with the full name, the email, and the new password.

SQL QUERY
WITH tab1 AS (SELECT c.email AS email, c.last_name AS last_name,
			  LENGTH (co.country) AS country,
					c.first_name || ' ' || c.last_name AS full_name,
		   			LEFT (c.first_name, STRPOS(c.first_name, '') - LENGTH(c.first_name)) AS first_letter,
		   			RIGHT (c.first_name, STRPOS(c.first_name, '') - LENGTH(c.first_name)) AS last_letter
			FROM customer c
			JOIN address a
			ON a.address_id = c.address_id
			JOIN city ci
			ON ci.city_id =a.city_id
			JOIN country co
			ON co.country_id = ci.country_id
			 WHERE c.activebool = 'true')
SELECT full_name, email,
		CONCAT (first_letter, country, last_name, last_letter) AS password
FROM tab1
;
