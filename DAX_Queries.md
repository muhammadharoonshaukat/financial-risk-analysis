# 📊 DAX Queries for Financial Analysis

## Create Database
CREATE DATABASE financial_data; 

## Create Table to Import CVS File

CREATE TABLE financial_data (
    age INT,
    occupation TEXT,
    risk_tolerance TEXT,
    investment_goals TEXT,
    income_level NUMERIC,
    account_balance NUMERIC,
    deposits NUMERIC,
    withdrawals NUMERIC,
    transfers NUMERIC,
    international_transfers NUMERIC,
    investments NUMERIC,
    loan_amount NUMERIC,
    loan_purpose TEXT,
    employment_status TEXT,
    loan_term_months INT,
    interest_rate NUMERIC,
    loan_status TEXT
);

## See the Structure of Table

select * from financial_data

-- Import directly or Use codes 
COPY financial_data 
FROM 'path\finance_data.csv'
WITH (FORMAT csv, HEADER true);


## 1. Data Preparation (Feature Engineering)

### Net Cash Flow variable
-------------------------

SELECT *,
       deposits - withdrawals AS net_cash_flow
FROM financial_data;


### Total Transction Acitivity
------------------------------

SELECT *,
       deposits + withdrawals + transfers + international_transfers AS total_activity
FROM financial_data;

### Loan Repayment with Interest
--------------------------------

SELECT *,
       loan_amount * interest_rate AS total_repayment
FROM financial_data;

## 2. Fraud Detection Model (Threshold)
---------------------------------------

SELECT *,
CASE
    WHEN international_transfers > 50000 THEN 'Suspicious'
    WHEN withdrawals > deposits * 1.5 THEN 'Suspicious'
    WHEN withdrawals > account_balance THEN 'Suspicious'
    WHEN risk_tolerance = 'High' AND international_transfers > 20000 THEN 'Suspicious'
    ELSE 'Normal'
END AS fraud_flag
FROM financial_data;


## 3. KPIs (Metrics)
----------------------

SELECT 
    SUM(deposits) AS total_deposits,
    SUM(withdrawals) AS total_withdrawals,
    SUM(deposits - withdrawals) AS net_cash_flow,
    SUM(loan_amount) AS total_loan_amount,
    SUM(investments) AS total_investments
FROM financial_data;


### Fraud KPIs
--------------

SELECT 
    COUNT(*) AS total_transactions,
    
    COUNT(*) FILTER (
        WHERE 
            international_transfers > 50000 OR
            withdrawals > deposits * 1.5 OR
            withdrawals > account_balance OR
            (risk_tolerance = 'High' AND international_transfers > 20000)
    ) AS suspicious_count,

    COUNT(*) FILTER (
        WHERE 
            international_transfers > 50000 OR
            withdrawals > deposits * 1.5 OR
            withdrawals > account_balance OR
            (risk_tolerance = 'High' AND international_transfers > 20000)
    ) * 100.0 / COUNT(*) AS fraud_rate_percent
FROM financial_data;


### Fraud Loss Estimate
------------------------

SELECT 
    SUM(withdrawals) FILTER (
        WHERE 
            international_transfers > 50000 OR
            withdrawals > deposits * 1.5 OR
            withdrawals > account_balance OR
            (risk_tolerance = 'High' AND international_transfers > 20000)
    ) AS fraud_loss
FROM financial_data;


### 4. Loan Analysis
-----------------------

### Loan Approval Rate By Loan Purposes
----------------------------------------

SELECT 
    loan_purpose,
    COUNT(*) AS total_loans,
    COUNT(*) FILTER (WHERE loan_status = 'approved') AS approved_loans,
    COUNT(*) FILTER (WHERE loan_status = 'approved') * 100.0 / COUNT(*) AS approval_rate_percent
FROM financial_data
GROUP BY loan_purpose;

### Loan Approval By Occupation
--------------------------------

SELECT 
    occupation,
    COUNT(*) AS total_loans,
    COUNT(*) FILTER (WHERE loan_status = 'approved') * 100.0 / COUNT(*) AS approval_rate_percent
FROM financial_data
GROUP BY occupation
ORDER BY approval_rate_percent DESC;

## 5. Fraud Analysis
---------------------

### Fraud rate by Occupation
------------------------------

SELECT 
    occupation,
    COUNT(*) AS total,
    COUNT(*) FILTER (
        WHERE 
            international_transfers > 50000 OR
            withdrawals > deposits * 1.5 OR
            withdrawals > account_balance OR
            (risk_tolerance = 'High' AND international_transfers > 20000)
    ) * 100.0 / COUNT(*) AS fraud_rate_percent
FROM financial_data
GROUP BY occupation
ORDER BY fraud_rate_percent DESC;


## Fraud Rate By Employment Status 
-----------------------------------

SELECT 
    employment_status,
    COUNT(*) AS total,
    COUNT(*) FILTER (
        WHERE 
            international_transfers > 50000 OR
            withdrawals > deposits * 1.5 OR
            withdrawals > account_balance OR
            (risk_tolerance = 'High' AND international_transfers > 20000)
    ) * 100.0 / COUNT(*) AS fraud_rate_percent
FROM financial_data
GROUP BY employment_status;

## 6. Risk Analysis
---------------------

### Risk Distribution
----------------------

SELECT 
    risk_tolerance,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER() AS percentage
FROM financial_data
GROUP BY risk_tolerance;

### Risk vs Income
------------------

SELECT 
    risk_tolerance,
    AVG(income_level) AS avg_income,
    AVG(account_balance) AS avg_balance
FROM financial_data
GROUP BY risk_tolerance;


## 7. Behavioral Analysis
-----------------------------

### Income vs Withdrawal
-------------------------

SELECT 
    income_level,
    AVG(withdrawals) AS avg_withdrawals,
    AVG(international_transfers) AS avg_international_transfers
FROM financial_data
GROUP BY income_level
ORDER BY income_level;

### Transction Behavior
-----------------------

SELECT 
    occupation,
    AVG(deposits) AS avg_deposits,
    AVG(withdrawals) AS avg_withdrawals,
    AVG(transfers) AS avg_transfers
FROM financial_data
GROUP BY occupation;


### 8. Loan Repayment Trend
------------------------

SELECT 
    loan_term_months,
    AVG(loan_amount * (1 + interest_rate/100)) AS avg_repayment
FROM financial_data
GROUP BY loan_term_months
ORDER BY loan_term_months;

## 9. Correlation Style Analysis
------------------------------------

SELECT 
    income_level,
    loan_amount
FROM financial_data;


## 10 Fraud Exposure Ratio
-----------------------------

SELECT 
    SUM(withdrawals) FILTER (WHERE 
        international_transfers > 50000 OR
        withdrawals > deposits * 1.5
    ) * 100.0 / SUM(withdrawals) AS fraud_exposure_percent
FROM financial_data;



