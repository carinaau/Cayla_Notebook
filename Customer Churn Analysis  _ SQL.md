``` sql
use BankData;

-- How much revenue did Maven lose to churned customers?

SELECT

Customer_Status,

COUNT(Customer_ID) AS Customer_Count,

ROUND((SUM(Total_Revenue) * 100) / SUM(SUM(Total_Revenue)) OVER(),1) AS Revenue_Percentage

FROM telecom_customer_churn

GROUP BY Customer_Status;


-- Typical tenure for churners

SELECT

CASE

WHEN Tenure_in_Months <= 6 THEN '6 months'

WHEN Tenure_in_Months <= 12 THEN '1 Year'

WHEN Tenure_in_Months <= 24 THEN '2 Years'

ELSE '> 2 Years'

END AS Tenure,

ROUND(COUNT(Customer_ID) * 100/ SUM(COUNT(Customer_ID)) OVER(),1) AS Churn_Percentage

FROM telecom_customer_churn

WHERE Customer_Status = 'Churned'

GROUP BY

CASE

WHEN Tenure_in_Months <= 6 THEN '6 months'

WHEN Tenure_in_Months <= 12 THEN '1 Year'

WHEN Tenure_in_Months <= 24 THEN '2 Years'

ELSE '> 2 Years'

END

ORDER BY Churn_Percentage DESC;

  
  

-- Which cities have the highest churn rates?

SELECT

City,

COUNT(Customer_ID) AS Churned,

COUNT(CASE WHEN Customer_Status = 'Churned' THEN Customer_ID ELSE NULL END)/ COUNT(Customer_ID) AS Churn_Rate

FROM telecom_customer_churn

GROUP BY City

HAVING COUNT(Customer_ID) > 30

AND COUNT(CASE WHEN Customer_Status = 'Churned' THEN Customer_ID ELSE NULL END) > 0

ORDER BY Churn_Rate DESC

LIMIT 4;

  

  
-- Why did customers leave?

SELECT

Churn_Category,

ROUND(SUM(Total_Revenue),0)AS Churned_Rev,

CEILING((COUNT(Customer_ID) * 100.0) / SUM(COUNT(Customer_ID)) OVER()) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Churn_Category

ORDER BY

Churn_Percentage DESC;



-- why exactly did customers churn?

SELECT

Churn_Reason,

Churn_Category,

ROUND(COUNT(Customer_ID) *100 / SUM(COUNT(Customer_ID)) OVER(), 1) AS churn_percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Churn_Reason,

Churn_Category

ORDER BY churn_percentage DESC

LIMIT 5;



-- What offers did churners have?

SELECT

Offer,

ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS churned

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Offer

ORDER BY

churned DESC;


  

-- What Internet Type did churners have?

SELECT

Internet_Type,

COUNT(Customer_ID) AS Churned,

ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Internet_Type

ORDER BY

Churned DESC;

  

-- What Internet Type did 'Competitor' churners have?

SELECT

Internet_Type,

Churn_Category,

ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

AND Churn_Category = 'Competitor'

GROUP BY

Internet_Type,

Churn_Category

ORDER BY Churn_Percentage DESC;

  
  

-- Did churners have premium tech support?

SELECT

Premium_Tech_Support,

COUNT(Customer_ID) AS Churned,

ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY Premium_Tech_Support

ORDER BY Churned DESC;

  

-- What contract were churners on?

SELECT

Contract,

COUNT(Customer_ID) AS Churned,

ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Contract

ORDER BY

Churned DESC;


-- Are high value customers at risk?

SELECT

CASE

WHEN (num_conditions >= 3) THEN 'High Risk'

WHEN num_conditions = 2 THEN 'Medium Risk'

ELSE 'Low Risk'

END AS risk_level,

COUNT(Customer_ID) AS num_customers,

ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS cust_percentage,

num_conditions

FROM

(

SELECT

Customer_ID,

SUM(CASE WHEN Offer = 'Offer E' OR Offer = 'None' THEN 1 ELSE 0 END)+

SUM(CASE WHEN Contract = 'Month-to-Month' THEN 1 ELSE 0 END) +

SUM(CASE WHEN Premium_Tech_Support = 'No' THEN 1 ELSE 0 END) +

SUM(CASE WHEN Internet_Type = 'Fiber Optic' THEN 1 ELSE 0 END) AS num_conditions

FROM

telecom_customer_churn

WHERE

Monthly_Charge > 70.05

AND Customer_Status = 'Stayed'

AND Number_of_Referrals > 0

AND Tenure_in_Months > 9

GROUP BY

Customer_ID

HAVING

SUM(CASE WHEN Offer = 'Offer E' OR Offer = 'None' THEN 1 ELSE 0 END) +

SUM(CASE WHEN Contract = 'Month-to-Month' THEN 1 ELSE 0 END) +

SUM(CASE WHEN Premium_Tech_Support = 'No' THEN 1 ELSE 0 END) +

SUM(CASE WHEN Internet_Type = 'Fiber Optic' THEN 1 ELSE 0 END) >= 1

) AS subquery

GROUP BY

CASE

WHEN num_conditions >= 3 THEN 'High Risk'

WHEN num_conditions = 2 THEN 'Medium Risk'

ELSE 'Low Risk'

END, num_conditions;

  

  

-- How old were churned customers?

SELECT

CASE

WHEN Age <= 30 THEN '19 - 30 yrs'

WHEN Age <= 40 THEN '31 - 40 yrs'

WHEN Age <= 50 THEN '41 - 50 yrs'

WHEN Age <= 60 THEN '51 - 60 yrs'

ELSE '> 60 yrs'

END AS Age,

ROUND(COUNT(Customer_ID) * 100 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

CASE

WHEN Age <= 30 THEN '19 - 30 yrs'

WHEN Age <= 40 THEN '31 - 40 yrs'

WHEN Age <= 50 THEN '41 - 50 yrs'

WHEN Age <= 60 THEN '51 - 60 yrs'

ELSE '> 60 yrs'

END

ORDER BY

Churn_Percentage DESC;

  

-- What gender were churners?

SELECT

Gender,

ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Gender

ORDER BY

Churn_Percentage DESC;

  

  

-- Were churners married

SELECT

Married,

ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

Married

ORDER BY

Churn_Percentage DESC;

  

  

-- Did churners have dependents

SELECT

CASE

WHEN Number_of_Dependents > 0 THEN 'Has Dependents'

ELSE 'No Dependents'

END AS Dependents,

ROUND(COUNT(Customer_ID) *100 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage

  

FROM

telecom_customer_churn

WHERE

Customer_Status = 'Churned'

GROUP BY

CASE

WHEN Number_of_Dependents > 0 THEN 'Has Dependents'

ELSE 'No Dependents'

END

ORDER BY Churn_Percentage DESC;
```