# Loan Performance Analysis

## Table of Content
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning](#data-cleaning)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [The SQL Query for the Analysis](#the-sql-query-for-the-analysis)
- [Results and Findings](#results-and-findings)
- [Recommendations](#recommendations)
- [Reference](#references)

### Project Overview
----
This analysis aims to provide valuable insights into a loan application's performance for a loan company for the years 2021 and 2022, By analyzing various aspects of the loan data, we seek to identify trends, make data-driven recommendations, and gain a deeper understanding of the company's loan application performance.

![image](https://github.com/abutunde/Loan-Performance-Analysis/assets/113314795/52910140-9dae-4dfe-b405-5f892c4a7ca1)

### Data Sources

Loan Data: The primary dataset used for this analysis is the 'QWE_RAW_DATA' file, containing detailed information about each loan application by the company.

### Tools
###  Microsoft Excel
- #### For the Cleaning, Sorting, and Formatting on Microsoft Excel, I used the following functions:
1. Match Function
2. Index Function
3. Vlookup
4. Date Function
5. Concatenate
6. IF, CountIF, CountA, CountIFS
7. Average, Sumif, Max, and Min
   
    -  Data Cleaning [Click here](https://onedrive.live.com/edit?id=340C72DA9142E4B0!669&resid=340C72DA9142E4B0!669&ithint=file%2Cxlsx&nav=MTVfezhENTQ5QTE2LUJDMkQtNDkxRS05Q0FCLTVBQzYwMTc4MjRDN30&redeem=aHR0cHM6Ly8xZHJ2Lm1zL3gvcyFBckRrUXBIYWNndzBoUjFBMFNqTUlPOTZfZXZtP2U9UlJnYnNmJm5hdj1NVFZmZXpoRU5UUTVRVEUyTFVKRE1rUXRORGt4UlMwNVEwRkNMVFZCUXpZd01UYzRNalJETjMw&migratedtospo=true&wdo=2&cid=340c72da9142e4b0)

### SQL - Data Analysis 
Reference - [The SQL Query for the Analysis](#the-sql-query-for-the-analysis)

### Power BI
#### The Power BI was used for the following:
- Creation of Project report dashboard
- Dax formula for calculation of :
1. Pull-through rate
2. Average of Age
3. Turn Around Time
   
### Data Cleaning
In the initial data preparation phase, I performed the following tasks using Microsoft Excel:
-  Cleaned Data [Click here](https://onedrive.live.com/edit?id=340C72DA9142E4B0!669&resid=340C72DA9142E4B0!669&ithint=file%2Cxlsx&nav=MTVfezhENTQ5QTE2LUJDMkQtNDkxRS05Q0FCLTVBQzYwMTc4MjRDN30&redeem=aHR0cHM6Ly8xZHJ2Lm1zL3gvcyFBckRrUXBIYWNndzBoUjFBMFNqTUlPOTZfZXZtP2U9UlJnYnNmJm5hdj1NVFZmZXpoRU5UUTVRVEUyTFVKRE1rUXRORGt4UlMwNVEwRkNMVFZCUXpZd01UYzRNalJETjMw&migratedtospo=true&wdo=2&cid=340c72da9142e4b0)
1. Data loading and Inspection
2. Sorting and Handling Missing Values
3. Data Cleaning and Formatting

### Exploratory Data Analysis
EDA involved exploring the Loan data to answer key questions, such as:
1. What was the pull-through rate in 2021 and 2022 respectively?
2. What is the distribution of loan applications by continent?
3. Which country had the highest rejection rate?
4. What is the monthly trend of loan applications by loan amount since inception?
5. What is the average age of the special applicants?
6. What is the distribution of loan completion by continent?
7. Which regions had the highest applications by loan amount for each continent?
8. "Using the following age grouping, which age group recorded the highest number of applications?
- 20-30 - Young professionals
- 31-40 - Managers
- 41-50 - Executive Managers
- 51-60 - C-Suite"
9. Which country(or countries) had the highest application by loan amount and by number of applicants?
  
### The SQL Query for the Analysis
```SQL
 ---QST 1
Select Round(
           CAST((   Select Count(Status)
                      from ['Loan Profile$']
                     where Status = 'Disbursed'
                       and Year   = 2022) AS FLOAT)
           / (Select count(Loan_Amount) from ['Loan Profile$'] Where Year = 2022),
           2) * 100 as PULL_THROUGH_RATE;

Select Round(
           CAST((   Select Count(Status)
                      from ['Loan Profile$']
                     where Status = 'Disbursed'
                       and Year   = 2021) AS FLOAT)
           / (Select count(Loan_Amount) from ['Loan Profile$'] Where Year = 2021),
           2) * 100 as PULL_THROUGH_RATE;

--QST 2
Select Sum(Loan_Amount_USD3) 'Total_Amount_Disbursed_between_Aug2021&Mar2022'
  from ['Loan Profile$']
 where [Start Date]       >= '2021/08/01'
   and [End/Current Date] <= '2022/03/31'
   and Loan_Amount_USD3   > 10
   and Status             = 'Disbursed'

--QST3 ?
Select Continent,
       count(Loan_Amount_USD3) as loan_application,
       (Count(*) / count(Loan_Amount_USD3) * 100) / (Select count(*) from ['Loan Profile$'])
  FROM ['Loan Profile$']
 group by continent

--QST 4
Select count(country_count) AS Count_of_Country
  from (   Select Count(country) as country_count,
                  country,
                  Avg(Loan_Amount_USD3) as Avg_Loan_Amount
             FROM ['Loan Profile$']
            Group by country
           Having Avg(Loan_Amount_USD3) > 200000) asd;

--QST 4 with CTE
WITH CTE_Count
  AS (SELECT COUNT(country) AS country_count,
             country,
             AVG(Loan_Amount_USD3) AS avg_loan_amount
        FROM ['Loan Profile$']
       GROUP BY country
      HAVING avg(Loan_Amount_USD3) > 200000)
SELECT COUNT(country_count) AS country_count_above_threshold
  FROM CTE_Count;

--QST 5
Select Top 5 Country,
       max(Count_of_Rejection) as Highest_Rejection_Rate
  from (   Select country,
                  max(Status) as rejection,
                  Count(Status) as Count_of_Rejection
             from ['Loan Profile$']
            Where Status = 'Rejected'
            group by Country) Rjt
 Group by country,
          Count_of_Rejection
 order by Count_of_Rejection desc

--QST 6
Select Distinct (MONTH),
       Sum(Loan_Amount_USD3) AS Loan_Trend
  from ['Loan Profile$']
 Where Year = 2021
 group by Month
 ORDER BY (Month)

--QST 7
Select Avg(Avg_Age) Age_of_Special_Applicant
  from (Select Avg(Age) Avg_Age FROM ['Special applicants$'] group by Age) age

--QST 8 (A)
Select Max(Loan_Amount_USD3) Highest_Loan_Amount_SpecialApplicant
  From ['Loan Profile$']
  join ['Special applicants$']
    on ['Loan Profile$'].Name = ['Special applicants$'].[Full name]

---QST 8 (B)
Select Min(Loan_Amount_USD3) Lowest_Loan_Amount_SpecialApplicant
  From ['Loan Profile$']
  join ['Special applicants$']
    on ['Loan Profile$'].Name = ['Special applicants$'].[Full name]

--QST 9
Select Continent,
       Count(Loan_Amount_USD3)
  From ['Loan Profile$']
 where Status = 'Disbursed'
 group by Continent

--QST 10
Select --TOP 3
       Continent,
       Max(Loan_Amount) Amount
  From (   Select Continent,
                  Sum(Loan_Amount_USD3) Loan_Amount
             FROM ['Loan Profile$']
            group by Continent) Loan
 group by Continent
 Order by Amount desc

--QST 11 A
Select AVG(No_of_Days) Avg_Turn_Around_Time_By_Days_2021
  From (   Select DATEDIFF(day, [Start Date], [End/Current Date]) No_of_Days
             from ['Loan Profile$']
            where Year   = 2021
              and Status = 'Disbursed') day_
--QST 11 B
Select AVG(No_of_Days) Avg_Turn_Around_Time_By_Days_2022
  From (   Select DATEDIFF(day, [Start Date], [End/Current Date]) No_of_Days
             from ['Loan Profile$']
            where Year   = 2022
              and Status = 'Disbursed') day_

---QST 12?
Select Round(CAST((   Select Sector,
                             Count(Status)
                        from ['Loan Profile$']
                       where Status = 'Disbursed'
                         and Sector = 'Health') AS FLOAT) / (Select count(Loan_Amount_USD3) from ['Loan Profile$']),
             2) * 100 as PULL_THROUGH_RATE

--QST13
Select [Age Range],
       COUNT([Age Range]) Age_Grouping
  from ['Loan Profile$']
 group by [Age Range]

--QST 14
Select continent,
       Count(EnSport) Merged_Application
  from (   Select Continent,
                  Year,
                  Sector,
                  Case
                       When Sector = 'Energy' then 'Energy_Sport'
                       When Sector = 'Sports' then 'Energy_Sport'
                       Else Sector End EnSport
             FROM ['Loan Profile$']
  --where Year between 2021 AND 2022
  ) New_Sector_Share
 Where EnSport <> [Sector]
   and YEAR    = 2022
 group by Continent

--QST 15
Select Top 1 Country,
       Max(Loan_Amount) Highest_Loan_Amount
  From (   Select Country,
                  count(Loan_Amount_USD3) as Loan_Application,
                  Sum(Loan_Amount_USD3) AS Loan_Amount
             from ['Loan Profile$']
            Group by Country) APP
 group by Country
 Order by Highest_Loan_Amount desc;
--15 B
Select Top 1 Country,
       Max(Loan_Application) Highest_Application_Rate
  From (   Select Country,
                  count(Loan_Amount_USD3) as Loan_Application,
                  Sum(Loan_Amount_USD3) AS Loan_Amount
             from ['Loan Profile$']
            Group by Country) APP
 group by Country
 Order by Highest_Application_Rate desc
```
### Results and Findings
The Analysis results are summarized as follows:
1. The pull-through rate in 2021 and 2022 respectively was 56% for 2021 and 78% for 2022
2. Distribution of loan applications by continent		
![image](https://github.com/abutunde/Loan-Performance-Analysis/assets/113314795/109554e1-ffd1-4ff4-8dcf-0544cd4933d5)

3. The country with the highest rejection rate is 
    - The SyrianArab Republic

4. Monthly trend of loan applications by loan amount since inception
    - ![image](https://github.com/abutunde/Loan-Performance-Analysis/assets/113314795/c6d2c217-cee9-406e-8cad-28895a62ccbb)

5. The average age of the special applicants  
    - 38.52
6. The distribution of loan completion by continent
   - ![image](https://github.com/abutunde/Loan-Performance-Analysis/assets/113314795/a46229da-21ff-4b92-b20d-1b430daca879)

7. Regions with the highest applications by loan amount for each continent
    - Europe with $44,028,469.09
8. The age group with the highest number of applications
  - 20-30 - Young professionals
  - 31-40 - Managers
  - 41-50 - Executive Managers
  - 51-60 - C-Suite"
  - ![image](https://github.com/abutunde/Loan-Performance-Analysis/assets/113314795/5e330604-b63b-41e9-8603-73c688dceced)


9. Countries with the highest applications by loan amount and by number of applicants		
    - By Number of Applications - Côted’Ivoire
    - By Loan Amount - Andorra
   
### Recommendations
Based on the analysis results, here are some recommendations for the loan company:

1. #### Enhance Pull-Through Rate:
   - Investigate the factors contributing to the increased pull-through rate in 2022 compared to 2021.
   - Identify the successful strategies or changes made during 2022 and consider implementing similar practices to improve the pull-through rate further.
2. #### Focus on Africa:
   - Given the high distribution of loan applications from Africa, consider tailoring marketing and outreach efforts to this region.

3. Address Rejection Rates in Syria:
   - Investigate the reasons behind the high rejection rate in The Syrian Arab Republic.
   - Analyze whether specific risk factors or other issues are contributing to the higher rejection rate and implement strategies to mitigate these concerns.
4. #### Optimize Loan Application Trends:
   - Capitalize on the observed monthly trend of rising loan applications from June through August.
   - Consider launching targeted marketing campaigns or promotions during these months to capitalize on the increased demand.
5. #### Maximize Completion Rates in Africa:
   - Leverage the high distribution of completed loans in Africa by continuing to provide excellent customer service and support to clients in this region.
6. #### Tailor Strategies for Specific Countries:
   - Focus attention on Côte d'Ivoire for the highest number of applications and Andorra for the highest loan amount.
7. #### Targeted Engagement Based on Age Groups:
   - Tailor communication and marketing strategies based on the age group demographics.
   - Consider creating specific loan products or features that appeal to the identified age groups, such as professional development loans for young professionals and executive perks for C-Suite individuals.
8. #### Continuous Monitoring and Adaptation:
   - Regularly monitor trends and performance metrics to identify changing patterns in loan applications.
   - Be flexible and adaptive, adjusting strategies based on ongoing data analysis to meet evolving customer needs and market dynamics.
   
These recommendations aim to improve overall performance, address specific challenges, and capitalize on identified opportunities in different regions and demographic groups. Regular monitoring and adaptability are emphasized for sustained success.

### References
1. Emmanuel Onyeachor (Data Analyst at Henley Business School)- Data Source
   - [Linkedin](https://linkedin.com/in/emmanuel-onyeachor)
2. Datamine Analysis
   💻🖱️
