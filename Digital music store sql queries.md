Analysis Of Digital Music Store Data Using Sql
---

1. Let's find out the countries which have the most Invoices of our digital music store.
```sql
SELECT billing_country,COUNT(billing_country) AS invoices_count
FROM invoice
GROUP BY billing_country
ORDER BY invoices_count DESC;
```

Result Set:

![result 1](https://user-images.githubusercontent.com/127675963/229294366-a3d2cdae-0403-4720-9ebe-28c85ac8429c.png)

USA has the most invoices among all the countries. The result set only shows the first ten rows of the query result. 

---

2. Our digial music store would like to throw a promotional Music Festival in the city we made the most money. 
The best city is the city that has the highest sum of invoice totals.
Let's find out the city that has the best customers.
```sql
SELECT billing_city, SUM(total) AS invoice_totals
FROM invoice
GROUP BY billing_city
ORDER BY invoice_totals DESC
LIMIT 1;
```

Result Set:

![result 2](https://user-images.githubusercontent.com/127675963/229294443-94ea2553-9e33-43e0-bc07-bf2b91b4a35c.png)

Prague is the city that has the best customers with the highest sum of invoice totals.

---

3. Let's find out who is the best customer of our digital music store.
The customer who has spent the most money will be declared the best customer. 
 
```sql
SELECT cust.customer_id, cust.first_name, cust.last_name, SUM(inv.total) AS total_spending
FROM customer cust
INNER JOIN invoice inv
ON cust.customer_id = inv.customer_id
GROUP BY cust.customer_id, cust.first_name, cust.last_name
ORDER BY total_spending DESC
LIMIT 1;
```

Result Set:

![result 3](https://user-images.githubusercontent.com/127675963/229294778-7fa1d71d-2208-442e-991d-49cca1d40ee1.png)

Frantisek Wichterlova is the best customer who has spent the most money on our digital music store.

---

4. We also want to find out the best customers from each country who have spent the most money on our digital music store. 
Let's find out the best customers for each country.

```sql
WITH RECURSIVE
    country_customers AS (
		SELECT cust.customer_id, cust.first_name, cust.last_name,
		cust.country, SUM(inv.total) AS total_spending
		FROM customer cust
		INNER JOIN invoice inv
		ON cust.customer_id = inv.customer_id
		GROUP BY cust.customer_id, cust.first_name,
		cust.last_name, cust.country

	)
	,country_best_customers AS (
		SELECT customer_id, first_name, last_name,
		country, total_spending,
		RANK() OVER (PARTITION BY country ORDER BY total_spending DESC)
		AS rank_num 
		FROM country_customers
	)
SELECT customer_id, first_name, last_name,
country, total_spending
FROM country_best_customers
WHERE rank_num = 1
ORDER BY total_spending DESC;
```

Result Set:

![result 7](https://user-images.githubusercontent.com/127675963/229294931-e1fc401b-f307-46b3-a765-4491dae3ce70.png)

The best customer belongs to Czech Republic followed by Ireland, India and so on. The result set displays only the first ten rows of the query result.

---

5. We want to find out the most popular music Genre for each country. 
We determine the most popular genre as the genre with the highest amount 
of purchases. Let's find out.

```sql
WITH RECURSIVE
    sales_per_country AS (
        SELECT customer.country, COUNT(*) AS purchases_per_genre,
        genre.name AS genre_name, genre.genre_id
        FROM invoice_line
        JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
        JOIN customer ON customer.customer_id = invoice.customer_id
        JOIN track ON track.track_id = invoice_line.track_id
        JOIN genre ON genre.genre_id = track.genre_id
        GROUP BY customer.country, genre.name, genre.genre_id
    )
    ,max_genre_per_country AS (
        SELECT MAX(purchases_per_genre) AS max_genre_number, country
        FROM sales_per_country
        GROUP BY country
    )

SELECT sales_per_country.* 
FROM sales_per_country
JOIN max_genre_per_country 
ON sales_per_country.country = max_genre_per_country.country
WHERE sales_per_country.purchases_per_genre = max_genre_per_country.max_genre_number
ORDER BY purchases_per_genre DESC;
```

Result Set:

![result 4](https://user-images.githubusercontent.com/127675963/229295058-318d8584-4e87-4da2-bb43-fa2cf0781f36.png)

It seems that the Rock music genre is the most popular genre for most of the countries. The result set displays only the first ten rows of the query result.

---

6. Now that we know that our customers love rock music, we can decide which musicians to
invite to play at the concert.
Let's invite the top 10 artists who have written the most rock music in our dataset.
```sql
SELECT artist.artist_id, artist.name, COUNT(track.track_id) AS total_songs 
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id, artist.name
ORDER BY total_songs DESC
LIMIT 10;
```

Result Set:

![result 5](https://user-images.githubusercontent.com/127675963/229295152-3e54f4df-e775-4988-bc1b-ba20bec88232.png)

These are the top 10 artists who have written the most rock music in our dataset.

---

7. We want to find which artist has earned the most according to the InvoiceLines.
Then we will use this artist to find which customer has spent the most on this artist.
```sql
WITH best_artist AS(
    SELECT artist.artist_id, artist.name AS artist_name,
    SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY artist.artist_id, artist_name
    ORDER BY total_sales DESC
    LIMIT 1
)

SELECT best_artist.artist_name,
SUM(invoice_line.unit_price * invoice_line.quantity) AS amount_spent,
customer.customer_id, customer.first_name, customer.last_name
FROM invoice
JOIN customer ON customer.customer_id = invoice.customer_id
JOIN invoice_line ON invoice_line.invoice_id = invoice.invoice_id
JOIN track ON track.track_id = invoice_line.track_id
JOIN album ON album.album_id = track.album_id
JOIN best_artist ON best_artist.artist_id = album.artist_id
GROUP BY best_artist.artist_name, customer.customer_id,
customer.first_name, customer.last_name
ORDER BY amount_spent DESC;
```

Result Set:

![result 6](https://user-images.githubusercontent.com/127675963/229295243-1e496bd4-cdb7-4ff8-a5a0-abe9f46258e9.png)

Queen is the artist that has earned the most and Hugh O'Reilly is the customer that has spent the most on this artist.
The result set displays only the first eight rows of the query result.

---

8. We want to find all the track names that have a song length longer than the average
song length.
```sql
SELECT name AS track_name, milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds) AS avg_track_length
    FROM track
)
ORDER BY milliseconds DESC;
```

Result Set:

![result 8](https://user-images.githubusercontent.com/127675963/229295345-c6c52ea6-5e39-49a8-92ba-386ad324f758.png)

These are the track names that have a song length longer than the average song length. The result set displays only the first eight rows of the query result.

---

9. We want to find out the customer that has spent the most on music
for each country along with the money they spent. 
For countries where the top amount spent is shared, provide all customers who
spent this amount.
```sql
WITH RECURSIVE 
    customter_with_country_cte AS (
        SELECT customer.customer_id, first_name, last_name, billing_country,
        SUM(total) AS total_spending
        FROM invoice
        JOIN customer ON customer.customer_id = invoice.customer_id
        GROUP BY customer.customer_id, first_name, last_name, billing_country
    )
    ,country_max_spending_cte AS (
        SELECT billing_country, MAX(total_spending) AS max_spending
        FROM customter_with_country_cte
        GROUP BY billing_country
    )

SELECT cwc.billing_country, cwc.customer_id, cwc.first_name,
cwc.last_name, cwc.total_spending
FROM customter_with_country_cte cwc
JOIN country_max_spending_cte cms
ON cwc.billing_country = cms.billing_country
WHERE cwc.total_spending = cms.max_spending
ORDER BY cwc.billing_country;
```

Result Set:

![result 9](https://user-images.githubusercontent.com/127675963/229295415-ee68efb7-6bf8-4662-96a4-f78095ea9f50.png)

These are the customers that have spent the most on music for each country along with their respective total spendings. The result set displays only the first eight rows of the query result.

---
