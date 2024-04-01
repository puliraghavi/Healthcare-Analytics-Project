
# Healthcare-Analytics-Project
## Dialysis of Patients

![dialysis bg](https://github.com/puliraghavi/Healthcare-Analytics-Project/assets/119037510/46a78a8b-69e9-4787-b745-9cd2ba2d875b)

***
## 📚Contents
- [Introduction](#introduction)
- [EDA in SQL](#eda-in-sql)
- [Excel Report](#excel-report)
- [Insights Obtained](#insights-obtained)

# Introduction

Healthcare is a fundamental aspect of human society, encompassing the prevention, diagnosis, treatment, and management of illness and injury. It is an essential service that impacts individuals, families, communities, and entire nations. In recent years, **data analysis has emerged as a critical tool in the healthcare industry**. The vast amount of data generated within healthcare systems, including patient records, medical imaging, genomic data, and administrative data, presents immense opportunities for **improving healthcare delivery and patient outcomes** through data-driven insights.

In this project that I have done during my intership at Ai Variant, I have conducted an analysis focusing on dialysis treatment under various chain organisations, overview of these organisations, their performance over years and addressed the key performance indicators asked by the participating entities thereby providing valuable insights.

- **Dialysis**:
Dialysis is a life-saving medical procedure used to treat patients with kidney failure, also known as end-stage renal disease (ESRD). It is performed to mimic the function of the kidneys by removing waste products and excess fluids from the blood. It helps maintain the body's fluid balance and electrolyte levels, thereby preventing the buildup of toxins.

The dataset consists of the following information :
1. Chain Organisations
2. Facility centers and their details- rating, location, etc.
3. Count of dialysis stations under each facility center
4. Patient summaries - Includes count of patients under various aggregated data points or metrics that provide an overview of various aspects related to patient care or treatment outcomes.
5. Category Text - Describes each facility center by assessing the performance or outcomes based on certain criteria or benchmarks. The values "as expected," "worse than expected," and "better than expected" represent these qualitative judgments.
6. Total Performance score of each facility center
7. Type of the facility center -Profit/ Non-Profit
8. Payment reduction %

**Sample Dataset:**
![image](https://github.com/puliraghavi/Healthcare-Analytics-Project/assets/119037510/2ffef3d5-47ed-4409-b01b-a4cfc1098bca)

***
# EDA in SQL
- The project was executed using SQL for Exploratory Data Analysis and in Excel for creatng an interactive dashboard.
- Following are the MySQL queries for a few EDA questions and KPIs.

1. Total no of dialysis stations  
````sql
select sum(`# of Dialysis Stations`) as count_dialysis_stations
from `project healthcare`;
````

2. Total number of summaries across all categories
````sql
with cte as(
SELECT sum(`No of patients in the transfusion summary`)as transfusion_summaries,
sum(`No of patients in hypercalcemia summary`) as hypercalcemia_summaries ,
sum(`No of patients in Serum phosphorus summary`) as Serum_phosphorus_summaries ,
sum(`No of patients in hospital readmission summary`) as hospital_readmission_summaries,
sum(`Noof patients in hospitalization summary`) as hospitalization_summaries,
sum(`No of Patients in survival summary`) as survival_summaries,
sum(`No of patients in nPCR summary`) as nPCR_summaries,
sum(`No of Patients in fistula summary`) as fistula_summaries,
sum(`No of patients in long term catheter summary`) as long_term_catheter_summaries
FROM dialysis.`project healthcare`)
select concat(round((sum(transfusion_summaries+ hypercalcemia_summaries+ 
Serum_phosphorus_summaries+ hospital_readmission_summaries+ hospitalization_summaries+ survival_summaries+
 nPCR_summaries+fistula_summaries+long_term_catheter_summaries)/1000000))," Million") as Total_summaries
 from cte;
````

3. State wise no of facility centers
````sql
select state, 
       count(`Facility Name`) as count
from `project healthcare`
group by 1
order by 2 desc;
````

4. Year wise count of facility centers (Profit/Non-Profit)
````sql   
   -- Cleaning Date column
update `project healthcare`
set `Certification or Recertification Date` = 
     STR_TO_DATE(`Certification or Recertification Date`, '%m/%d/%Y')
where `Certification or Recertification Date` like '__/__/____';

-- For MM-DD-YYYY format
update `project healthcare`
set `Certification or Recertification Date` = 
     STR_TO_DATE(`Certification or Recertification Date`, '%m-%d-%Y')
where `Certification or Recertification Date` like '__-__-____';

-- Solution:
select year(`Certification or Recertification Date`) as year_ ,
       count(case when `Profit or Non-Profit` = "Profit" then `Facility Name` end) as Profit_count,
       count(case when `Profit or Non-Profit` = "Non-Profit" then `Facility Name` end) as Non_Profit_count
from `project healthcare`
group by year_
order by year_ asc;
````

5. Chain Organizations with 5 star rating
````sql
Select distinct `Chain Organization`
from `project healthcare`
where `Five Star` = 5;
````

- Key performance indicators:
  
1. Number of Patients across various summaries
````sql
SELECT sum(`No of patients in the transfusion summary`)as transfusion_summaries,
sum(`No of patients in hypercalcemia summary`) as hypercalcemia_summaries ,
sum(`No of patients in Serum phosphorus summary`) as Serum_phosphorus_summaries ,
sum(`No of patients in hospital readmission summary`) as hospital_readmission_summaries,
sum(`Noof patients in hospitalization summary`) as hospitalization_summaries,
sum(`No of Patients in survival summary`) as survival_summaries,
sum(`No of patients in nPCR summary`) as nPCR_summaries,
sum(`No of Patients in fistula summary`) as fistula_summaries,
sum(`No of patients in long term catheter summary`) as long_term_catheter_summaries
FROM dialysis.`project healthcare`;
````

2. Profit Vs Non-Profit Stats
````sql
select `Profit or Non-Profit` as Category, 
count(*) as Count, 
concat(round((count(*)/(select count(*) from dialysis.`project healthcare`))*100,2),"%") as Percentage
from dialysis.`project healthcare`
group by `Profit or Non-Profit`;
````

3. Chain Organizations w.r.t. Total Performance Score as No Score
````sql
SELECT DISTINCT(`Chain Organization`) , 
       COUNT(`Dialysis - II.Total Performance Score`) AS Total_Performance_Score
FROM dialysis.`project healthcare`
WHERE `Dialysis - II.Total Performance Score` != 'No Score'
GROUP BY `Chain Organization`
order by 2 desc
limit 10;
````

4. Dialysis Stations Stats
````sql
SELECT DISTINCT(`Chain Organization`) , 
	   SUM(`# of Dialysis Stations`) AS Dialysis_Stations,
       round(avg(`Five Star`),1) as Avg_rating
FROM dialysis.`project healthcare`
GROUP BY `Chain Organization`
order by 2 desc;
````

5. #_of Category Text  - As Expected
````sql
SELECT 
    SUM(CASE WHEN `Patient Transfusion category text` = 'As Expected' THEN 1 ELSE 0 END) AS Transfusion_As_Expected_Count,
    SUM(CASE WHEN `Patient hospitalization category text` = 'As Expected' THEN 1 ELSE 0 END) AS Hospitalization_As_Expected_Count,
    SUM(CASE WHEN `Patient Hospital Readmission Category` = 'As Expected' THEN 1 ELSE 0 END) AS Hospital_Readmission_As_Expected_Count,
    SUM(CASE WHEN `Patient Survival Category Text` = 'As Expected' THEN 1 ELSE 0 END) AS Survival_As_Expected_Count,
    SUM(CASE WHEN `Patient Infection category text` = 'As Expected' THEN 1 ELSE 0 END) AS Infection_As_Expected_Count,
    SUM(CASE WHEN `Fistula Category Text` = 'As Expected' THEN 1 ELSE 0 END) AS Fistula_As_Expected_Count,
    SUM(CASE WHEN `SWR category text` = 'As Expected' THEN 1 ELSE 0 END) AS SWR_As_Expected_Count,
    SUM(CASE WHEN `PPPW category text` = 'As Expected' THEN 1 ELSE 0 END) AS PPPW_As_Expected_Count
FROM dialysis.`project healthcare`;
````

6. Average Payment Reduction Rate
````sql
SELECT CONCAT(round(AVG(`Dialysis - II.Payment Reduction %`)*100,2),"%")AS Payment_redn 
from dialysis.`project healthcare`
where `Dialysis - II.Payment Reduction %` is not null;
````

# Excel Report

- The same dataset was imported into excel to create a dashboard to vizually view the metrics and KPIS.
- The data was cleaned and formatted in the power query editor accordingly.
- Using pivot tables and various charts, the data was summarised into the dashboard below.

![dialysis excel](https://github.com/puliraghavi/Healthcare-Analytics-Project/assets/119037510/66105e48-0a6b-4490-9977-eca2879570a1)

# Insights Obtained
1. Total summaries - 6 million.
2. There are a total of 135k dialysis stations under 116 chain organisations.
3. Upon summarizing the total performance score of all chain organizations, it becomes evident that **Davita, Fresenius Medical Care, and other independent organizations** emerge as the top three performers.
4. In total, there are **7504 facility centers across 56 states** and the growth of this count has increased linearly over years from the 1970s to 2010s.
5. Across the nation, there's considerable diversity in the distribution of dialysis stations. States like **California, Texas, and Florida** boast numerous stations, whereas states like Wyoming, North Dakota, and Alaska have notably fewer. This discrepancy likely arises from factors such as population density, the prevalence of kidney disease, and disparities in healthcare infrastructure.
6. **89%** of these facility centers are classified as profitable and the rest of **11.2%** as non-profitable.
7. Upon analysing various patient summaries it was found the **Survival summary** is the most common summary with 1.93 million patients which implies that a significant portion of the patients in the dataset are being monitored or assessed based on their survival outcomes suggesting that the dataset primarily consists of patients with chronic or life-threatening conditions.
8. The other summaries are primarily based on vaious diagnostic tests and transfusion  procedures.
9. The **nPCR summary has the least patients** indicating that nPCR is not a common test performed in patients with renal diseases.
10. The **"Hospitalization" category** exhibits the highest number of "As Expected" results, totaling to 6,638, followed closely by "Hospital Readmission" with 6,539. This pattern implies that outcomes in these facility centers are often predictable or well-understood.
11. The participating entities were intrested to know the **average payment reduction rate and it was found to be - 0.32%**.







