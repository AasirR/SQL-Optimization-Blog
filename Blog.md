# Enhancing SQL Skills with Query Optimization Techniques 🚀

## Introduction
SQL is a powerful and versatile language for working with relational databases, but as datasets grow larger, queries often become more complex and slower to execute. In this post, we will explore how to optimize a slow-running SQL query using various techniques, including analyzing the execution plan, using CTEs (Common Table Expressions), window functions, and query refactoring.

By understanding the process and applying these techniques, you will gain the skills needed to tackle performance bottlenecks in your SQL queries.

---

## Challenge: Optimizing a Slow-Running SQL Query

In this post, we’ll optimize a slow-running SQL query that retrieves customer and product sales information from an e-commerce dataset. We will walk through each step of the optimization process, improving query performance while maintaining accuracy.

### 1. Identify a Query

We'll use the following query, which calculates customer order summaries and product category sales over the past year:

### Original Query

```sql
WITH CustomerOrderSummary AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        COUNT(DISTINCT o.order_id) AS total_orders,
        SUM(oi.quantity * p.price) AS total_spent
    FROM 
        customers c
        LEFT JOIN orders o ON c.customer_id = o.customer_id
        LEFT JOIN order_items oi ON o.order_id = oi.order_id
        LEFT JOIN products p ON oi.product_id = p.product_id
    WHERE 
        o.order_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
    GROUP BY 
        c.customer_id, c.customer_name
),
ProductCategorySales AS (
    SELECT 
        pc.category_name,
        SUM(oi.quantity) AS total_quantity_sold,
        SUM(oi.quantity * p.price) AS total_sales
    FROM 
        product_categories pc
        JOIN products p ON pc.category_id = p.category_id
        JOIN order_items oi ON p.product_id = oi.product_id
        JOIN orders o ON oi.order_id = o.order_id
    WHERE 
        o.order_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
    GROUP BY 
        pc.category_name
)
SELECT 
    cos.customer_name,
    cos.total_orders,
    cos.total_spent,
    pcs.category_name,
    pcs.total_quantity_sold,
    pcs.total_sales,
    (cos.total_spent / NULLIF(pcs.total_sales, 0)) * 100 AS percent_of_category_sales
FROM 
    CustomerOrderSummary cos
    CROSS JOIN ProductCategorySales pcs
WHERE 
    cos.total_spent > (SELECT AVG(total_spent) FROM CustomerOrderSummary)
ORDER BY 
    cos.total_spent DESC, pcs.total_sales DESC;
