/* ALT MOBILITY ASSIGNMENT */

use altmobility;

/* TASK 1 - ORDER AND SALES ANALYSIS */
/* TASK 1.1 - TOTAL ORDERS AND REVENUE */
SELECT 
    COUNT(*) AS Total_Orders,
    SUM(order_amount) AS Total_Revenue
FROM customer_orders; 

/* TASK 1.2 - ORDER BY STATUS */
SELECT 
    order_status,
    COUNT(*) AS num_orders,
    SUM(order_amount) AS total_amount
FROM customer_orders
GROUP BY order_status;

/* TASK 1.3 - MONTHLY REVENUE TREND */
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS order_month,
    SUM(order_amount) AS monthly_revenue
FROM customer_orders
GROUP BY order_month
ORDER BY order_month;


/* TASK 2 - CUSTOMER ANALYSIS */
/* TASK 2.1 - REPEAT CUSTOMERS */
SELECT 
    customer_id,
    COUNT(*) AS order_count
FROM customer_orders
GROUP BY customer_id
HAVING COUNT(*) > 1
ORDER BY order_count DESC;

/* TASK 2.2 - FIRST TIME VS RETURNING CUSTOMER */
WITH customer_order_counts AS (
    SELECT customer_id, COUNT(*) AS order_count
    FROM customer_orders
    GROUP BY customer_id
)
SELECT 
    SUM(CASE WHEN order_count = 1 THEN 1 ELSE 0 END) AS first_time_customers,
    SUM(CASE WHEN order_count > 1 THEN 1 ELSE 0 END) AS returning_customers
FROM customer_order_counts;

/* TASK 2.3 - MONTHLY ACTIVE CUSTOMERS */
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS order_month,
    COUNT(DISTINCT customer_id) AS active_customers
FROM customer_orders
GROUP BY order_month
ORDER BY order_month;

/* TASK 2.4 - CUSTOMER SEGMENTATION BY TOTAL SPEND */
SELECT 
    customer_id,
    SUM(order_amount) AS total_spent,
    CASE 
        WHEN SUM(order_amount) >= 700 THEN 'High-Value'
        WHEN SUM(order_amount) >= 400 THEN 'Medium-Value'
        ELSE 'Low-Value'
    END AS segment
FROM customer_orders
GROUP BY customer_id
ORDER BY total_spent DESC;


/* TASK 3 - PAYMENT STATUS ANALYSIS */
/* TASK 3.1 - PAYMENT SUCCESS VS FAILURE COUNTS */
SELECT 
    payment_status,
    COUNT(*) AS count_payments,
    SUM(payment_amount) AS total_amount
FROM payments
GROUP BY payment_status;

/* TASK 3.2 - PAYMENT METHODS BREAKDOWN */
SELECT 
    payment_method,
    COUNT(*) AS method_usage_count,
    SUM(payment_amount) AS total_amount
FROM payments
WHERE payment_status = 'completed' -- Only successful payments
GROUP BY payment_method
ORDER BY total_amount DESC;

/* TASK 3.3 - FAILED PAYMENT BY MONTHS */
SELECT 
    DATE_FORMAT(payment_date, '%Y-%m') AS payment_month,
    COUNT(*) AS failed_payments
FROM payments
WHERE payment_status = 'failed'
GROUP BY payment_month
ORDER BY payment_month;

/* TASK 3.4 - ORDER WITH FAILED PAYMENTS */
SELECT 
    p.order_id,
    o.customer_id,
    p.payment_amount,
    p.payment_method,
    p.payment_date
FROM payments p
JOIN customer_orders o ON p.order_id = o.order_id
WHERE p.payment_status = 'failed';


/* TASK 4 - ORDER DETAILS REPORT */
/* TASK 4.1 - FULL ORDER PAYMENT JOIN */
SELECT 
    o.order_id,
    o.customer_id,
    o.order_date,
    o.order_amount,
    o.order_status,
    IFNULL(p.payment_id, 'No Payment') AS payment_id,
    IFNULL(p.payment_date, 'No Payment') AS payment_date,
    IFNULL(p.payment_amount, 0) AS payment_amount,
    IFNULL(p.payment_method, 'Not Paid') AS payment_method,
    IFNULL(p.payment_status, 'Not Paid') AS payment_status
FROM customer_orders o
LEFT JOIN payments p ON o.order_id = p.order_id
ORDER BY o.order_date;

/* TASK 4.2 - ORDER BY PAYMENT OUTCOME */
SELECT 
    COUNT(DISTINCT o.order_id) AS total_orders,
    COUNT(DISTINCT CASE WHEN p.payment_status = 'completed' THEN o.order_id END) AS successfully_paid_orders,
    COUNT(DISTINCT CASE WHEN p.payment_status = 'failed' THEN o.order_id END) AS failed_payment_orders,
    COUNT(DISTINCT CASE WHEN p.order_id IS NULL THEN o.order_id END) AS unpaid_orders
FROM customer_orders o
LEFT JOIN payments p ON o.order_id = p.order_id;


/* TASK 5 - CUSTOMER RETENTION ANALYSIS */
-- Step 1: Get each customer's first order month (cohort)
WITH customer_first_order AS (
    SELECT 
        customer_id,
        MIN(DATE(order_date)) AS first_order_date
    FROM customer_orders
    GROUP BY customer_id
),
-- Step 2: Join with all orders to track activity in later months
customer_orders_with_cohort AS (
    SELECT 
        o.customer_id,
        DATE(o.order_date) AS order_date,
        c.first_order_date
    FROM customer_orders o
    JOIN customer_first_order c ON o.customer_id = c.customer_id
),
-- Step 3: Count returning customers by cohort and month difference
deduplicated_orders AS (
    SELECT 
        customer_id,
        DATE_FORMAT(first_order_date, '%Y-%m') AS cohort_month,
        TIMESTAMPDIFF(MONTH, first_order_date, order_date) AS month_number
    FROM customer_orders_with_cohort
    GROUP BY customer_id, cohort_month, month_number
)

SELECT 
    cohort_month,
    month_number,
    COUNT(*) AS retained_customers
FROM deduplicated_orders
GROUP BY cohort_month, month_number
ORDER BY cohort_month, month_number;














