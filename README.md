# JetBrains BI Homework Instructions


This assignment is designed to give you some familiarity with JetBrains BI daily tasks. Also, it is designed to give you awareness of our database structure. Both two tasks are real business cases, which were solved by our team. The database structure was reduced, nevertheless, table and column names are identical. Some sensitive facts were changed, data were randomly generated in order to protect our internal information.
Feel free to use any software, programming language, database and approach you prefer. Save your answers with all solution steps (source code and files) to your GitHub storage and send us a link. We will commit back our detailed feedback.
The following aspects we will evaluate:
* the correctness and fullness of answers
* the way how homework was solved
* code readability
* used tools and technology


## Task 1: Sale analysis – revenue decline in ROW region
### Business description
JetBrains company sales software products all over the world. We have two big sales teams:

1) US – sale representatives responsible for the huge American market with top IT companies.
2) ROW – the rest of the world, Wien based team responsible for sales worldwide except for the US.

Both formally independent teams cooperate very closely and intensively. The vast majority of US and ROW sales campaigns are planned and launched together. However, every market has different local factors that need to be considered, so there are also small local sales campaigns.

Revenue YTD growth is the most important metric for both sales teams. The metric is downloaded from BI Summary report every day, written with chalk on blackboard and discussed on weekly meetings. Both teams achieved similar results 2018 vs 2017. Unfortunately, ROW 2019 YTD sales team performance is getting significantly worse than American. American and ROW sales regional managers have discussed their 2019 strategies several times, but they haven’t the reason of ROW decline.

### Task
ROW regional manager asks you to analyze the difference, find out the source of the issue and recommend action steps.
### Technical information
All relevant data stored on SQL Server, database bi, schema sales. Source code of summary report:
```
SELECT cou.region,
  SUM(CASE WHEN YEAR(ord.exec_date) = 2018 THEN ordItm.amount_total / exRate.rate ELSE 0.00 END) as SalesUsd2018,
  SUM(CASE WHEN YEAR(ord.exec_date) = 2019 THEN ordItm.amount_total / exRate.rate ELSE 0.00 END) as SalesUsd2019
FROM bi.sales.Orders ord
JOIN bi.sales.OrderItems ordItm ON ord.id = ordItm.order_id
JOIN bi.sales.Customer cust ON ord.customer = cust.id
JOIN bi.sales.Country cou ON cust.country_id = cou.id
JOIN bi.sales.ExchangeRate exRate ON ord.exec_date = exRate.date AND ord.currency = exRate.currency
JOIN bi.sales.Product prod ON ordItm.product_id = prod.product_id
WHERE ord.is_paid = 1 -- Only paid orders, exclude pre-orders
  AND YEAR(ord.exec_date) IN (2018, 2019) -- Year 2018, 2019
  AND MONTH(ord.exec_date) BETWEEN 1 AND 6 -- H1
GROUP BY cou.region
ORDER BY 1
;
```

## Task 2: Financial analysis – the difference between NetSuite ERP and payment gateway
### Business description
JetBrains company provides different payment methods to customers. The most comfortable way is online card payment via payment gateway Adyen. The great advantage of online payment is speed, software license key could be activated and used immediately after the payment.

According to accounting principles and rules, we must properly and fully book all transactions. In our case, it means to process several CSV settlement files every month. Right now, settlement processing is fully automated, but previous CSV files were processed manually by an external tool. Unfortunately, there were several unknown and not logged errors during settlement processing. As a result, there are differences between our accounting software NetSuite and settlement reports, which must be identified and fixed asap.

Accounting department provided overall comparison by batch number and merchant account. 

|Merchant Account|Batch Number|NetSuite|Payment gateway|Difference|
| ------------- | ------ | ------ | ------ | ------ |
|JetBrainsAmericasUSD||12,004,331|0|12,004,331|
|JetBrainsAmericasUSD|24|0|2,237,653|-2,237,653|
|JetBrainsAmericasUSD|25|0|4,696,815|-4,696,815|
|JetBrainsAmericasUSD|26|0|5,069,863|-5,069,863|
|JetBrainsAmericasUSD|27|5,015,432|5,015,432|0|
|JetBrainsAmericasUSD|28|5,249,033|5,249,033|0|
|JetBrainsAmericasUSD|29|5,071,506|5,071,506|0|
|JetBrainsAmericasUSD|30|5,231,113|5,231,113|0|
|JetBrainsAmericasUSD|31|312,908|312,908|0|
|JetBrainsAmericasUSD|32|-1,018|-1,018|0|
|JetBrainsAmericasUSD|33|-1,185|-1,185|0|
|JetBrainsEUR|24|12,626,283|12,718,742|-92,459|
|JetBrainsEUR|25|27,161,210|27,153,731|7,479|
|JetBrainsEUR|26|30,153,291|30,153,291|0|
|JetBrainsEUR|27|28,694,588|28,694,588|0|
|JetBrainsEUR|28|29,681,899|29,681,899|0|
|JetBrainsEUR|29|29,006,430|29,006,430|0|
|JetBrainsEUR|30|30,115,279|30,115,279|0|
|JetBrainsEUR|31|1,863,397|1,863,397|0|
|JetBrainsEUR|32|-10,853|-10,853|0|
|JetBrainsEUR|33|-5,672|-5,672|0|
|JetBrainsUSD|24|2,924,913|2,924,913|0|
|JetBrainsUSD|25|6,438,787|6,438,787|0|
|JetBrainsUSD|26|7,008,511|7,008,511|0|
|JetBrainsUSD|27|6,864,467|6,864,467|0|
|JetBrainsUSD|28|7,110,794|7,110,794|0|
|JetBrainsUSD|29|0|6,934,852|-6,934,852|
|JetBrainsUSD|30|7,014,594|7,014,594|0|
|JetBrainsUSD|31|449,185|449,185|0|
|JetBrainsUSD|32|-4,858|-4,858|0|
|JetBrainsUSD|33|-2,215|-2,215|0|

Column *NetSuite* – a sum of all Payments and Customer Deposits foreign amounts on accounts:
* 315700 JBCZ: Receivables against ADYEN-EUR
* 315710 JBCZ: Receivables against ADYEN-USD
* 315720 JBCZ: Receivables against ADYEN-GBP
* 315800 JBA: Receivables against ADYEN-USD
* 548201 Other operating costs

Column *Payment gateway* – a sum of Adyen settlement overview, equals to the content of CSV settlement files.
### Task
Analyze and provide a list of all differences between NetSuite and payment gateway. Do not forget to compare all accounting relevant columns.

Bonus task: prepare SQL code to fix NetSuite data.
### Technical information
CSV settlement files are stored in settlement folder.

Schema netsuite contains all relevant NetSuite data.
