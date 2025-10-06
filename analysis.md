#  Microfinance SQL Business Insights

A collection of SQL queries and business insights designed for a microfinance management system.  
Each query addresses a real operational or financial challenge — tracking savings, loans, customer activity, and performance metrics to support **reporting, decision-making, and portfolio management**.


## 📑 Table of Contents

1. [Problem 1: All Financial Transactions](#problem-1-retrieve-a-complete-list-of-all-financial-transactions-credits-debits-and-balance-updates-across-customer-accounts-along-with-related-customer-and-account-details)
2. [Problem 2: Customers with Fixed Savings Account](#problem-2-retrieve-and-report-on-customers-who-currently-hold-fixed-savings-account-with-a-non-zero-balance)
3. [Problem 3: Customers with Current Savings Account](#problem-3-retrieve-and-report-on-customers-who-currently-hold-cuurent-savings-account-with-a-non-zero-balance)
4. [Problem 4: Active Savings (Not Yet Matured)](#problem-4-list-all-savings-accounts-that-are-active-and-not-yet-matured)
5. [Problem 5: Total Active Savings by Currency](#problem-5-calculate-the-total-number-of-active-savings-accounts-not-matured-and-the-the-total-amount-in-currency)
6. [Problem 6: Loans Issued and Not Yet Matured](#problem-6-retrieve-all-loans-that-have-been-issued-but-are-not-yet-matured-including-loan-details-outstanding-balances-and-related-customer-information)
7. [Problem 7: Matured Fixed Savings Withdrawals](#problem-7-retrieve-all-fixed-savings-accounts-that-have-matured-and-been-disbursed-including-whether-withdrawals-occurred-before-or-after-maturity)
8. [Problem 8: Matured Fixed Savings and Interest](#problem-8-retrieve-all-matured-fixed-savings-accounts-and-calculate-the-interest-earned-for-each-account-up-to-its-maturity-date)
9. [Problem 9: Customers with Zero Savings Balance](#problem-9-identify-customers-who-have-one-or-more-savings-accounts-but-currently-have-no-balance-in-any-of-them)
10. [Problem 10: Cancelled Fixed Accounts (Early Withdrawals)](#problem-10-identify-customers-who-withdrew-their-savings-from-a-fixed-deposit-account-before-the-maturity-date-effectively-cancelling-the-ongoing-fixed-account)
11. [Problem 11: Pending or Overdue Loans](#problem-11-identify-loans-that-have-reached-or-passed-their-due-date-but-still-have-outstanding-balances-including-unpaid-principal-interest-or-penalties)


## Problem 1: Retrieve a complete list of all financial transactions (credits, debits, and balance updates) across customer accounts, along with related customer and account details.

**Insight:** This query provides a comprehensive transaction ledger for the institution. It is the single most important dataset for auditing, reconciliation, fraud detection, and reporting. It connects every movement of money to a customer and the account type, offering a full picture of operational activity in the microfinance system.

## SQL Query

```sql

SELECT 
    tl.id,
    tl.customer_id,
    c.msisdn1,
    a.account_type,
    tl.reference_id,
    tl.currency_code,
    tl.dr,
    tl.cr,
    tl.bal_before,
    tl.bal_after,
    tl.ref_no,
    tl.desc AS description,
    tl.created_at
FROM transaction_log tl
LEFT JOIN customer c 
    ON tl.customer_id = c.id
LEFT JOIN account a 
    ON tl.account_type = a.id
ORDER BY tl.created_at DESC;

```

## Problem 2: Retrieve and report on customers who currently hold Fixed Savings Account with a non-zero balance. 

**Insight:** This query shows customers with Fixed Savings Account.  During visualization on superset BI, a currency filter has been applied to either show in USD/CDF or both. It provides insight on product performance, liquidity obligations, customer behaviour and portfolio size. This can be used in financial planning and customer relation management.
 
## SQL Query

```sql
SELECT 
    c.id AS customer_id,
    c.msisdn1 AS msisdn,
    sp.name AS product_name,
    sp.description AS account_type,
    s.balance,
    s.currency_code,
    s.date_approved,
    s.maturity_date
FROM 
    customer c
JOIN 
    savings_account s ON s.customer_id = c.id
JOIN 
    savings_product sp ON s.product_id = sp.id
WHERE 
    sp.description LIKE '%Fixed Account%'
    AND s.balance !=0 
ORDER BY 
    s.created_at DESC;

```

## Problem 3: Retrieve and report on customers who currently hold Cuurent Savings Account with a non-zero balance. 

**Insight:** This query highlights active customers maintaining current accounts, which are typically used for day-to-day transactions rather than long-term savings. These accounts are important for monitoring customer transactional behavior, product performance, cash flow, and operational liquidity. 

## SQL Query

```sql

SELECT 
    c.id AS customer_id,
    c.msisdn1 AS msisdn,
    sp.name AS product_name,
    sp.description AS account_type,
    s.balance,
    s.currency_code,
    s.date_approved,
    s.created_at
FROM 
    customer c
JOIN 
    savings_account s ON s.customer_id = c.id
JOIN 
    savings_product sp ON s.product_id = sp.id
WHERE 
    sp.description LIKE '%Current Account%'
    AND s.balance !=0 
ORDER BY 
    s.created_at DESC;

```

## Problem 4: List all savings accounts that are active and not yet matured. 

**Insight:** This is  limited to Fixed Savings as it has a maturity date. This query helps the organization monitor active savings accounts that are within their operation limits.

## SQL QUERY 

```sql

SELECT 
    c.msisdn1 AS msisdn,
    c.id,
    s.balance AS savings,
    s.currency_code AS currency,
    s.date_activated,
    s.maturity_date
FROM 
    savings_account s
JOIN 
    customer c ON s.customer_id = c.id
WHERE 
    s.date_activated IS NOT NULL
    AND s.balance IS NOT NULL
    AND s.balance != 0
    AND (s.maturity_date IS NULL OR s.maturity_date > CURDATE());

```

## Problem 5: Calculate the total number of active savings accounts not matured and the the total amount in currency. 

**Insight:** This query aggregates all active accounts whose savings have not matured giving the organization a snapshot of the deposit base per currency. It helps the organization with financial planning and understanding customer preference in termes of currency. 

## SQL Query

```sql

SELECT
    s.currency_code AS currency,
    SUM(s.balance) AS total_savings,
    COUNT(*) AS number_of_accounts
FROM 
    savings_account s
JOIN 
    customer c ON s.customer_id = c.id
WHERE 
    s.date_activated IS NOT NULL
    AND s.balance IS NOT NULL
    AND s.balance != 0
    AND (s.maturity_date IS NULL OR s.maturity_date > CURDATE())
GROUP BY 
    s.currency_code;

```

## Problem 6: Retrieve all loans that have been issued but are not yet matured, including loan details, outstanding balances, and related customer information.

**Insight:** This query provides visibility into the current active loan portfolio, focusing on loans that are still within their repayment period (not yet matured). For a microfinance institution, understanding the composition and status of these active loans is critical for portfolio health monitoring, revenue forecasting, and risk management.

## SQL Query

```sql

SELECT 
    c.msisdn1 AS msisdn,
    l.customer_id,
    l.currency_code AS currency,
    l.loan_amount,
    l.loan_balance,
    l.amount_paid,
    l.interest_earned,
    l.created_at AS loan_issued_date,
    l.due_date AS loan_due_date
FROM 
    loan_account l
JOIN 
    customer c ON l.customer_id = c.id
WHERE 
    l.loan_amount IS NOT NULL
    AND l.loan_amount != 0
    AND l.loan_balance != 0
    AND l.due_date > CURDATE();

```

## Problem 7: Retrieve all fixed savings accounts that have matured and been disbursed, including whether withdrawals occurred before or after maturity.

**Insight:** This query tracks withdrawals from matured fixed deposit accounts, offering clear visibility into how customers interact with time-bound savings products. By distinguishing between Early Withdrawals and At Maturity disbursements, it provides valuable insights into customer behavior, product performance, and liquidity.

## SQL Query

```sql

SELECT DISTINCT
    c.id AS customer_id,
    c.msisdn1 AS msisdn,
    s.savings_id,
    sp.description,
    s.date_activated,
    s.maturity_date,
    t.created_at AS transaction_date,
    t.currency_code,
    t.dr AS transaction_amount,
    t.desc AS trans_desc,
    CASE
        WHEN t.created_at < s.maturity_date THEN 'Early Withdrawal'
        ELSE 'At Maturity'
    END AS withdrawal_status
FROM savings_account s
JOIN customer c 
    ON s.customer_id = c.id
JOIN savings_product sp 
    ON s.product_id = sp.id
JOIN transaction_log t 
    ON t.reference_id = s.savings_id
WHERE sp.description LIKE '%Fixed Account%'
  AND s.maturity_date <= NOW()          
  AND t.dr > 0                          
  AND t.desc LIKE '%Retrait Compte Bloqué%'
ORDER BY t.created_at DESC;

```

## Problem 8: Retrieve all matured fixed savings accounts and calculate the interest earned for each account up to its maturity date.

**Insight:** This query identifies all fixed savings deposits that have reached their maturity date and computes the interest accrued based on an annual rate of 11%. For a microfinance institution, this analysis is critical for understanding liability obligations, interest expense forecasts, and product profitability.

## SQL Query

```sql

SELECT
    c.id AS customer_id,
    c.msisdn1 AS msisdn,
    s.savings_id,
    sp.name AS product_name,
    sp.description AS account_type,
    s.date_activated,
    s.maturity_date,
    s.currency_code,
    s.balance,
    ROUND(s.balance * 0.11 * DATEDIFF(s.maturity_date, s.date_approved) / 365, 2) AS interest_earned
FROM savings_account s
JOIN customer c ON s.customer_id = c.id
JOIN savings_product sp ON s.product_id = sp.id
WHERE sp.description LIKE '%Fixed Account%'
  AND s.maturity_date <= NOW()
  AND s.balance > 0;

```

## Problem 9: Identify customers who have one or more savings accounts but currently have no balance in any of them.

**Insight:** This query pinpoints customers who are registered for savings products but currently have zero balances across all their accounts. This insight directly relates to credit eligibility, financial risk, and portfolio sustainability, since customers are typically required to maintain savings before accessing loans.

## SQL Query

```sql

SELECT
    c.msisdn1 AS msisdn
FROM
    customer c
JOIN
    savings_account s ON s.customer_id = c.id
GROUP BY
    c.msisdn1
HAVING
    SUM(CASE WHEN s.balance <> 0 THEN 1 ELSE 0 END) = 0
    AND COUNT(*) > 0;

```

## Problem 10: Identify customers who withdrew their savings from a fixed deposit account before the maturity date, effectively cancelling the ongoing fixed account.

**Insight:** This query detects premature withdrawals from fixed savings accounts ie; situations where customers have broken their deposit agreement before maturity. This data helps management track customer behavior, risk monitoring and audit control. 

## SQL Query

```sql

SELECT
    c.id AS customer_id,
    c.msisdn1 AS msisdn,
    s.savings_id,
    sp.name AS product_name,
    sp.description AS account_type,
    s.date_approved,
    s.maturity_date,
    s.balance AS current_balance,
    t.created_at AS withdrawal_date,
    t.currency_code,
    t.dr AS withdrawn_amount,
    t.bal_before,
    t.bal_after,
    t.`desc` AS transaction_desc
FROM transaction_log t
JOIN customer c ON t.customer_id = c.id
JOIN savings_account s ON t.reference_id = s.savings_id
JOIN savings_product sp ON s.product_id = sp.id
WHERE sp.description LIKE '%Fixed Account%'
  AND t.created_at < s.maturity_date    
  AND t.bal_after < t.bal_before         
  AND t.dr > 0                  
  AND s.balance = 0                      
  AND s.maturity_date > NOW()            
  AND t.`desc` LIKE '%Retrait Compte Bloqué%'  
ORDER BY t.created_at DESC;

```

## Problem 11: Identify loans that have reached or passed their due date but still have outstanding balances, including unpaid principal, interest, or penalties.

**Insight:** This query gives the organization a clear view of loans that are pending full repayment i.e. those that are overdue, partially paid, or defaulted. Monitoring these loans is critical for maintaining portfolio quality and managing credit risk.

## SQL Query

```sql

SELECT 
    la.customer_id,
    c.msisdn1 AS msisdn,
    la.loan_id,
    la.currency_code,
    la.loan_amount,
    la.interest_earned,
    la.amount_paid,
    la.loan_balance,
    la.outstanding_principle,
    la.outstanding_penalty_fees,
    la.created_at AS loan_issued_date,
    la.due_date,
    la.defaulted
FROM loan_account la
JOIN customer c ON la.customer_id = c.id
JOIN loan_product lp ON la.loan_product_id = lp.id
WHERE la.due_date < NOW()  
  AND (
       la.loan_balance > 0 
       OR la.outstanding_principle > 0 
       OR la.outstanding_interest > 0 
       OR la.outstanding_penalty_fees > 0
      )
ORDER BY la.due_date DESC;

```

