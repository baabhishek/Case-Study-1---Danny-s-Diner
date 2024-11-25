# Case-Study-1-Danny-s-Diner
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.
![l-intro-1664219638](https://github.com/user-attachments/assets/01dbec41-68ab-47e3-89c6-52a8dc02e771)
Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

#### sales
#### menu
#### members
## Entity Relationship Diagram: 
<img width="796" alt="Screenshot 2567-11-26 at 1 56 20 AM" src="https://github.com/user-attachments/assets/6448132a-e4b7-4ed6-8503-b549a667b1a6">

[View Database Diagram on dbdiagram.io](https://dbdiagram.io/d/Dannys-Diner-608d07e4b29a09603d12edbd?utm_source=dbdiagram_embed&utm_medium=bottom_open)

### Case Study Questions:
#### What is the total amount each customer spent at the restaurant?
```
SELECT s.customer_id, sum(mn.price) as spent_ammount
FROM sales s 
INNER join menu mn 
on s.product_id=mn.product_id
GROUP by customer_id 
```
#### How many days has each customer visited the restaurant?
```
SELECT customer_id, count(DISTINCT (order_date)) as count_of_visited_restuant
FROM sales
GROUP by customer_id
```

#### What was the first item from the menu purchased by each customer?
```
GO 

with Row_no_cte as 
(
select *,
        ROW_NUMBER() OVER(partition by customer_id  ORDER BY ORDER_DATE) AS ROW_NUM
FROM sales
),

MENU_CTE AS 
(
SELECT RC.*, 
       MN.product_name
FROM Row_no_cte AS RC 
INNER JOIN menu AS MN 
      ON RC.product_id = MN.product_id
)

SELECT customer_id,
       ORDER_DATE AS first_ORDER_DATE,
       product_NAME AS FIRST_ITEM_ORDER
FROM MENU_CTE
WHERE ROW_NUM=1

GO
```
#### What is the most purchased item on the menu and how many times was it purchased by all customers?
```
SELECT top 1 
       s.product_id, 
       count(s.product_id) as  count_of_order_most,
       mn.product_name
FROM sales as s 
INNER join menu as mn 
on s.product_id=mn.product_id
GROUP BY s.product_id, mn.product_name
ORDER BY count_of_order_most DESC
```
#### Which item was the most popular for each customer?
```
With item_rank as
(
Select S.customer_ID ,
       Mn.product_name, 
       Count(S.product_id) as Count,
       Dense_rank()  Over (Partition by S.Customer_ID order by Count(S.product_id) DESC ) as Rank
From Menu mn
join Sales s
On mn.product_id = s.product_id
group by S.customer_id,S.product_id,Mn.product_name
)

Select Customer_id as id ,Product_name as item ,Count 
From item_rank
where rank = 1 
```
#### Which item was purchased first by the customer after they became a member?
```
with joining_cte as 
(select s.customer_id,
       mn.product_name,
      Dense_rank()  OVER(partition by s.Customer_id ORDER by s.ORDER_DATE) as dense_rnk
FROM sales as s
INNER join menu as mn
On mn.product_id = s.product_id
INNER join members as mb 
on mb.customer_id=s.customer_id
WHERE s.ORDER_DATE >= mb.join_date)

SELECT customer_id ,
       product_name
FROM joining_cte
WHERE dense_rnk=1 
```
#### Which item was purchased just before the customer became a member?
```
with before_joining_cte as 
(select s.customer_id,
       mn.product_name,
      Dense_rank()  OVER(partition by s.Customer_id ORDER by s.ORDER_DATE) as dense_rnk
FROM sales as s
INNER join menu as mn
On mn.product_id = s.product_id
INNER join members as mb 
on mb.customer_id=s.customer_id
WHERE s.ORDER_DATE < mb.join_date)

SELECT customer_id ,
       product_name
FROM before_joining_cte
WHERE dense_rnk=1 
```
#### What is the total items and amount spent for each member before they became a member?
```
SELECT 
    s.customer_id,
    COUNT(s.product_id) AS total_items,
    SUM(mn.price) AS total_amount_spent
FROM sales AS s
INNER JOIN menu AS mn
    ON s.product_id = mn.product_id
INNER JOIN members AS mb
    ON s.customer_id = mb.customer_id
WHERE s.order_date < mb.join_date
GROUP BY s.customer_id
```
#### If each $1 spent equates to 10 points and sushi has a 2x points multiplier -  how many points would each customer have?
```
go 

with Point_eaned_cal as 
(select s.customer_id,
       s.product_id,
       mn.product_name,
       sum (mn.price) as cost_,
       CASE 
        WHEN mn.product_name = 'sushi' then (sum (mn.price) *10)*2 
            ELSE sum (mn.price) *10 
                end as Point_eaned
FROM sales s 
INNER JOIN menu mn 
on s.product_id=mn.product_id
GROUP by s.customer_id,
       s.product_id,
       mn.product_name)

SELECT customer_id,
       SUM(Point_eaned) as Reedemption_point
FROM Point_eaned_cal
GROUP by customer_id
```
#### In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```
Select
     s.customer_id
	,Sum(CASE
            When (DATEDIFF(DAY, mb.join_date, s.order_date) between 0 and 7) or 
                 (mn.product_ID = 1) Then mn.price * 20
                        Else mn.price * 10
                           END) As Points
From members as mb
    Inner Join sales as s 
    on s.customer_id = mb.customer_id
    Inner Join menu as mn 
    on mn.product_id = s.product_id
where s.order_date >= mb.join_date 
     and s.order_date <= CAST('2021-01-31' AS DATE)
Group by s.customer_id
```
