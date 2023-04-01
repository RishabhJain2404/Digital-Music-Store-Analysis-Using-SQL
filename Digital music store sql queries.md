heading

```sql
SELECT billing_country,COUNT(billing_country) AS invoices_count
FROM invoice
GROUP BY billing_country
ORDER BY invoices_count DESC;
```

Result Set:

![result 1](https://user-images.githubusercontent.com/127675963/229294366-a3d2cdae-0403-4720-9ebe-28c85ac8429c.png)

analysis

---
```sql
SELECT billing_city, SUM(total) AS invoice_totals
FROM invoice
GROUP BY billing_city
ORDER BY invoice_totals DESC
LIMIT 1;
```

Result Set:

![result 2](https://user-images.githubusercontent.com/127675963/229294443-94ea2553-9e33-43e0-bc07-bf2b91b4a35c.png)

analysis

---
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

<img title="" src="file:///D:/Sql%20project-%20Digital%20music%20store%20analysis/result%203.png" alt="" width="833">

analysis

---

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

![](D:\Sql%20project-%20Digital%20music%20store%20analysis\result%207.png)

analysis

---

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

<img title="" src="file:///D:/Sql%20project-%20Digital%20music%20store%20analysis/result%204.png" alt="" width="585">

analysis

---
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

<img src="file:///D:/Sql%20project-%20Digital%20music%20store%20analysis/result%205.png" title="" alt="" width="425">

analysis

---
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

<img title="" src="file:///D:/Sql%20project-%20Digital%20music%20store%20analysis/result%206.png" alt="" width="658">

analysis

---
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

<img title="" src="file:///D:/Sql%20project-%20Digital%20music%20store%20analysis/result%208.png" alt="" width="727">

analysis

---
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

<img src="file:///D:/Sql%20project-%20Digital%20music%20store%20analysis/result%209.png" title="" alt="" width="673">

---
