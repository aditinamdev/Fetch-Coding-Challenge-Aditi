# Fetch-Coding-Challenge-Aditi

## 01. Structured Relational Data Model 

**Review Existing Unstructured Data and Diagram a New Structured Relational Data Model**

After reviewing the unstructured data model, I created a starflake relational schema of the data. It is normalized. The following are the entities under it-
Fact Table: ReceiptItem
Dimension Tables: Receipt, User, Brand

In this normalized starflake schema, the "User" and "Brand" and "Receipt" entities serve as dimension tables, capturing descriptive attributes about users and brands and the "ReceiptItem" table represents the fact table, linking the "Receipt" and "Brand" dimensions using foreign keys. It is connected indirectly with the "User" entity.

Relations: 

One-to-Many Relationships in the schema:

User to Receipt: A user can have multiple receipts, but each receipt belongs to a single user. This is represented by the foreign key (FK) "user_id" in the Receipt table, referencing the primary key (PK) "user_id" in the User table.

Brand to Receipt Item: Each brand can be associated with multiple receipt items, but each receipt item corresponds to a single brand. This is represented by the foreign key (FK) "brand_id" in the Receipt Item table, referencing the primary key (PK) "brand_id" in the Brand table.

Many-to-One Relationships in the schema:

Receipt Item to Receipt: Multiple receipt items can be part of a single receipt, but each receipt item corresponds to a single receipt. This is represented by the foreign key (FK) "receipt_id" in the Receipt Item table, referencing the primary key (PK) "receipt_id" in the Receipt table.

## 02. SQL Queries 

**Write a query that directly answers a predetermined question from a business stakeholder**

**Queries**

***
- What are the top 5 brands by receipts scanned for most recent month?
- How does the ranking of the top 5 brands by receipts scanned for the recent month compare to the ranking for the previous month?
- When considering average spend from receipts with 'rewardsReceiptStatus’ of ‘Accepted’ or ‘Rejected’, which is greater?
- When considering total number of items purchased from receipts with 'rewardsReceiptStatus’ of ‘Accepted’ or ‘Rejected’, which is greater?
- Which brand has the most spend among users who were created within the past 6 months?
- Which brand has the most transactions among users who were created within the past 6 months?
***

_The answers can also be found in the pdf in the Git repo_

I tried to work on PostgreSQL by inserting the json files, and by converting it to csv, but encountered JSON and file errors, the screenshot of which I've uploaded to the repository as "Error Screenshots".

I’ve thus answered all the mentioned queries theoretically based on the previously created normalized structured starflake relational data model in question 01. 
I’ve kept the queries simple and concise, and made it as performant as possible based on the existing requirements which I had, with basic assumptions.
All the used variables & entities are consistent with the normalized structured starflake relational data model created.

1. What are the top 5 brands by receipts scanned for the most recent month?
SELECT b.name AS brand_name, COUNT(*) AS scanned_count
FROM Brand b
JOIN Receipt r ON b.brand_id = r.brand_id
WHERE DATE_TRUNC('month', r.createDate) = DATE_TRUNC('month', CURRENT_DATE)
GROUP BY b.name
ORDER BY scanned_count DESC
LIMIT 5;

2. How does the ranking of the top 5 brands by receipts scanned for the recent month compare to the ranking for the previous month?
WITH recent_month AS (
    SELECT b.name AS brand_name, COUNT(*) AS scanned_count
    FROM Brand b
    JOIN Receipt r ON b.brand_id = r.brand_id
    WHERE DATE_TRUNC('month', r.createDate) = DATE_TRUNC('month', CURRENT_DATE)
    GROUP BY b.name
    ORDER BY scanned_count DESC
    LIMIT 5
), previous_month AS (
    SELECT b.name AS brand_name, COUNT(*) AS scanned_count
    FROM Brand b
    JOIN Receipt r ON b.brand_id = r.brand_id
    WHERE DATE_TRUNC('month', r.createDate) = DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
    GROUP BY b.name
    ORDER BY scanned_count DESC
    LIMIT 5
)
SELECT recent.brand_name, recent.scanned_count, previous.scanned_count AS previous_month_count
FROM recent_month AS recent
LEFT JOIN previous_month AS previous ON recent.brand_name = previous.brand_name;

3. When considering average spend from receipts with 'rewardsReceiptStatus' of 'Accepted' or 'Rejected', which is greater?
SELECT rewardsReceiptStatus, AVG(totalSpent) AS average_spend
FROM Receipt
WHERE rewardsReceiptStatus IN ('Accepted', 'Rejected')
GROUP BY rewardsReceiptStatus
ORDER BY average_spend DESC
LIMIT 1;

4. When considering the total number of items purchased from receipts with 'rewardsReceiptStatus' of 'Accepted' or 'Rejected', which is greater?
SELECT rewardsReceiptStatus, SUM(purchasedItemCount) AS total_items_purchased
FROM Receipt
WHERE rewardsReceiptStatus IN ('Accepted', 'Rejected')
GROUP BY rewardsReceiptStatus
ORDER BY total_items_purchased DESC
LIMIT 1;

5. Which brand has the most spend among users who were created within the past 6 months?
SELECT b.name AS brand_name, SUM(r.totalSpent) AS total_spend
FROM Brand b
JOIN Receipt r ON b.brand_id = r.brand_id
JOIN Users u ON r.userId = u._id
WHERE u.createdDate >= CURRENT_DATE - INTERVAL '6 months'
GROUP BY b.name
ORDER BY total_spend DESC
LIMIT 1;

6. Which brand has the most transactions among users who were created within the past 6 months?
SELECT b.name AS brand_name, COUNT(*) AS transaction_count
FROM Brand b
JOIN Receipt r ON b.brand_id = r.brand_id
JOIN Users u ON r.userId = u._id
WHERE u.createdDate >= CURRENT_DATE - INTERVAL '6 months'
GROUP BY b.name
ORDER BY transaction_count DESC
LIMIT 1;

## 03. Data Quality Issues via Python

**Evaluate Data Quality Issues in the Data Provided**

Since I encountered issues to run the JSON file, the screenshots of which I've uploaded as "Error Screenshots", I've theoretically answered the data quality issues that might arrive. 

1. One common potential data quality issue that can be easily identified using Python is missing or null values in the dataset;

After identifying the missing values, there are several options for handling them:

- remove rows with missing values using the dropna() function.
- fill the missing values with a specified value (e.g., 0) using the fillna() function.
- interpolates missing values based on neighboring values using the interpolate() function.
- use more advanced imputation methods, such as mean, median, or machine learning-based imputation techniques.

2. Another common data quality issue is inconsistent or incorrect data formats within the dataset. Considering this coding challenge, I encountered data format issues due to JSON file not being proper, and file extraction issues from the zipped file downloaded via AWS. For this we could have made use of consistent and robust methods to convert into JSON or chosen CSV for easier flexibility through platforms, and uploaded accurately over the AWS buckets. Or, in some cases, date values that are stored in different formats or numeric values that contain non-numeric characters. We can identify and handle inconsistent data formats using Python;

- making use of various data format conversion functions like I tried using json_to_csv created under the JSON library, and other functions like to_csv under Pandas library
- converting the data to the correct format using functions like pd.to_datetime() to convert date strings to datetime objects, and astype() to convert numeric values to the desired data type
- removing rows with inconsistent data by using the dropna() function, specifying the subset of columns that need to have valid values

_I've also uploaded a theoretical python code set in the repository as Data Quality Issues via Python_ 

## 04. Communicate with Stakeholders 

**
Construct an email or slack message that is understandable to a product or business leader who isn’t familiar with your day to day work. This part of the exercise should show off how you communicate and reason about data with others. Commit your answers to the git repository along with the rest of your exercise.
**

** 
Questions- 
What questions do you have about the data?
How did you discover the data quality issues?
What do you need to know to resolve the data quality issues?
What other information would you need to help you optimize the data assets you're trying to create?
What performance and scaling concerns do you anticipate in production and how do you plan to address them?
** 

Email- 

Subject: Data Quality Assessment and Optimization for Improved Decision Making

Hi [Product/Business Leader's Name],

I hope this email finds you well. I wanted to discuss the recent analysis we conducted on our data assets and highlight some important findings that can significantly impact decision making within our organization.

1. Data Quality Assessment:
During our analysis, we identified a few data quality issues that are worth addressing. These issues include missing or null values in certain fields and inconsistent data formats across the dataset. These issues can potentially impact the accuracy and reliability of our analysis.

To identify these data quality issues, we utilized data profiling techniques, such as examining missing value counts, analyzing data types, and performing exploratory data analysis. These methods helped us gain insights into the overall health of our data and detect areas that require attention.

2. Resolving Data Quality Issues:
To resolve the data quality issues, we have taken the following steps:
- For missing values, we have employed different strategies depending on the context, particularly via Python. In some cases, we removed the rows with missing values, while in other cases, we filled them with appropriate default values or applied interpolation methods to estimate missing values based on neighboring data points.
- Inconsistent data formats, such as different date formats or non-numeric characters in numeric fields, have been addressed by converting the data to the correct format using suitable data transformation techniques.

However, it would be helpful to collaborate with the data engineering team to ensure that these data quality improvements are implemented in the data pipeline to maintain consistency and accuracy over time.

3. Optimization of Data Processes:
In order to optimize the data assets we are creating, we would need additional information and collaboration from various stakeholders. To enhance the quality and usefulness of our data, we plan to implement on the following points and would appreciate insights:
- Clear understanding of the specific business goals and objectives that the data assets should support.
- Identification of key performance indicators (KPIs) and metrics that are most relevant for decision making.
- Collaboration with subject matter experts to ensure the data assets capture all the necessary dimensions and context required for analysis.
- Collaboration with Data Engineering team to smoothen the ETL processes and Data warehouse and decide upon a fast and heavy working consistent data warehouse system along with finalizing on the cloud counterpart and connection as well

4. Performance and Scaling Concerns:
As we move towards production, it is important to anticipate and address any performance and scaling concerns. Some questions we need to consider are:
- What is the expected volume and velocity of incoming data?
- Do we have the necessary infrastructure and resources to handle the increasing data load?
- Are there any specific performance requirements from stakeholder's end regarding important KPI's, or regarding response time or data freshness, that need to be prioritized? 

To address these concerns, we plan to work closely with the data engineering and IT teams to ensure the necessary scalability, performance optimization, and infrastructure planning are in place. Regular monitoring and periodic performance reviews will be conducted to identify any bottlenecks and make proactive adjustments as needed.

I would love to have a discussion with you to further explore these points and align our efforts to maximize the value and reliability of our data assets. Please let me know a convenient time for a meeting or if you have any specific questions or concerns.

Thank you for your attention and support in advancing data-driven decision making within our organization.

Best regards,<br>
Aditi Namdeo <br>
Data Analyst, Fetch

_Answers can also be found in the uploaded PDF in the repository_ 




