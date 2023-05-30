# Music-Store-Data-Analysis
Question Set 1 - Easy
Q1: Who is the senior most employee based on the job title?

select first_name,last_name
from employee
ORDER BY levels desc
limit 1

Q2: Which Country have the maximum invoices?

select billing_country, count(*) as num_invoices
from invoice
group by billing_country
order by num_invoices desc
limit 1

Q3: What are the top 3 values of total invoice?

select total 
from invoice
order by total desc
limit 3

Q4: Which city has the best customers? We would like to throw a 
promotional Music Festival in the city we made the most money. Write
a query that returns one city that has the highest sum of invoice 
totals. Return both the city name & sum of all invoice totals.

select billing_city, sum(total) as total_invoice
from invoice
group by billing_city
order by total_invoice desc
limit 1

Q5: Who is the best customer? The customer who has spend the most 
money will be declared the best customer. Write a query that returns
the person who has spent the most money.

select customer.customer_id,customer.first_name,customer.last_name,
sum(invoice.total) as total_invoice
from customer
join invoice on customer.customer_id = invoice.customer_id
group by customer.customer_id
order by total_invoice desc
limit 1

order by total_invoice desc
limit 1

Question Set 2 - Moderate
Q1: Write a query to return email, first name, last name and 
Genre of all Rock Music listeners. Return your list ordered 
alphabetically by email starting with A.

select distinct email,first_name,last_name 
from customer
join invoice on customer.customer_id = invoice .customer_id
join invoice_line on invoice.invoice_id = invoice_line.invoice_id
where track_id in (
	select track_id from track
	join genre on track.genre_id = genre.genre_id
	where genre.name like 'Rock'
)
order by email

Q2: Let's invite the artists who have written the most rock music 
in our dataset. Write a query that returns the artist name and total
track count of the top 10 rock bands.


SELECT artist.name, artist.artist_id, 
COUNT(track.track_id) AS "no_of tracks"
FROM track
JOIN album ON track.album_id = album.album_id
JOIN artist ON album.artist_id = artist.artist_id
JOIN genre ON track.genre_id = genre.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id, artist.name
ORDER BY "no_of tracks" DESC
LIMIT 10;

Q3: Return all track names that have a song length longer than the
average song length. Return the Name and Milliseconds for each track.
Order by the song length with the longest songs listed first.

select name, milliseconds
from track
where milliseconds > (select avg(milliseconds) from track)
order by milliseconds desc
limit 10

Question Set 3 - Advance
Q1: Find how much amount spent by customer on artists? Write a query 
to return customer name, artist name and total spent.

with best_selling_artist as ( 
	select artist.artist_id as artist_id, artist.name as artist_name,
	sum(invoice_line.unit_price * invoice_line.quantity) as total_sales
	from invoice_line
	join track on invoice_line.track_id = track.track_id
	join album on track.album_id = album.album_id
	join artist on album.artist_id = artist.artist_id
	group by 1
	order by 3 desc
	limit 1
)
select customer.customer_id, customer.first_name, customer.last_name,
best_selling_artist.artist_name,sum(invoice_line.unit_price * invoice_line.quantity) 
as amount_spent
from invoice
join customer on invoice.customer_id = customer.customer_id
join invoice_line on invoice.invoice_id = invoice_line.invoice_id
join track on invoice_line.track_id = track.track_id
join album on track.album_id = album.album_id
join best_selling_artist on album.artist_id = best_selling_artist.artist_id
group by 1,2,3,4
order by 5 desc

Q2. We want to find the most popular music Genre in each country.
We determine the most popular genre as the genre with the higest amount of purchases.
Write a query that returns each countryalong with the top Genre. For countries where the maximum 
number of purchases is shared return all Genres.

WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1

Q3: Write a query that determines the customer that has spent the most on music for each country. 
Write a query that returns the country along with the top customer and how much they spent. 
For countries where the top amount spent is shared, provide all customers who spent this amount.

WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1
