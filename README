# SQL_Potfolio_Project
Manufacturing Data Analysis with SQL and Power BI Dashbord.

SELECT * FROM potfoli_project.production_data;

/* 
	Total number of records and Defect vs Non-Defect count

    First, we check the dataset size and the
   distribution of Defect vs Non-Defect records. This helps us
   understand whether the dataset is balanced or not.
*/
SELECT 
    DefectStatus,
    COUNT(*) AS total_records,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM production_data), 2) AS percentage
FROM production_data
GROUP BY DefectStatus;

   /* 
   Average ProductionCost, DefectRate, QualityScore 
-
    Calculate the overall average production cost,
   defect rate, and quality score for the entire dataset. This
   serves as a baseline/benchmark for further analysis.
*/

SELECT 
    ROUND(AVG(ProductionCost), 2)  AS avg_production_cost,
    ROUND(AVG(DefectRate), 2)      AS avg_defect_rate,
    ROUND(AVG(QualityScore), 2)    AS avg_quality_score
FROM production_data;

/* 
Min/Max DeliveryDelay and SafetyIncidents
  
    Find the minimum and maximum values of delivery
   delay and safety incidents - identifying extreme values helps
   spot outliers or problematic batches.
*/
SELECT 
    MIN(DeliveryDelay)   AS min_delivery_delay,
    MAX(DeliveryDelay)   AS max_delivery_delay,
    MIN(SafetyIncidents) AS min_safety_incidents,
    MAX(SafetyIncidents) AS max_safety_incidents
FROM production_data;

/* 
    Average QualityScore and ProductionCost by DefectStatus
  
   Compare quality score and production cost
   between Defect and Non-Defect batches. This helps us
   understand whether a low quality score is the main cause
   of defects.
*/
SELECT 
    DefectStatus,
    ROUND(AVG(QualityScore), 2)   AS avg_quality_score,
    ROUND(AVG(ProductionCost), 2) AS avg_production_cost,
    ROUND(AVG(SupplierQuality), 2) AS avg_supplier_quality
FROM production_data
GROUP BY DefectStatus;


/* 
	Defect rate by SupplierQuality range (CASE WHEN bucket)

    SupplierQuality is divided into buckets
   (Low/Medium/High) to see which bucket has the highest defect
   rate. This gives an important insight for supplier selection.
*/
SELECT 
    CASE 
        WHEN SupplierQuality < 70 THEN 'Low (< 70)'
        WHEN SupplierQuality BETWEEN 70 AND 85 THEN 'Medium (70-85)'
        ELSE 'High (> 85)'
    END AS supplier_quality_bucket,
    COUNT(*) AS total_batches,
    SUM(CASE WHEN DefectStatus = 'Defect' THEN 1 ELSE 0 END) AS defective_batches,
    ROUND(SUM(CASE WHEN DefectStatus = 'Defect' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS defect_rate_percent
FROM production_data
GROUP BY supplier_quality_bucket
ORDER BY defect_rate_percent DESC;


/* 
Relationship between DeliveryDelay and DefectRate
  
    Check whether DefectRate increases as
   DeliveryDelay increases - the average defect rate is shown
   for each delay value, revealing the trend/pattern.
*/
SELECT 
    DeliveryDelay,
    COUNT(*) AS total_batches,
    ROUND(AVG(DefectRate), 2) AS avg_defect_rate
FROM production_data
GROUP BY DeliveryDelay
ORDER BY DeliveryDelay;


/* 
Top 10 production batches with the highest SafetyIncidents
  
    Identify the riskiest 
   batches - these can be prioritized for safety audits.
*/
SELECT 
    ProductionVolume,
    SafetyIncidents,
    MaintenanceHours,
    WorkerProductivity,
    DefectStatus
FROM production_data
ORDER BY SafetyIncidents DESC
LIMIT 10;


/* 
    Defect rate % per SupplierQuality bucket using CTE
   
   Same idea as Q5, but written using a CTE
   (Common Table Expression) for cleaner, more readable code.
   CTEs make large queries more readable and reusable.
*/
WITH supplier_bucket AS (
    SELECT 
        *,
        CASE 
            WHEN SupplierQuality < 70 THEN 'Low'
            WHEN SupplierQuality BETWEEN 70 AND 85 THEN 'Medium'
            ELSE 'High'
        END AS quality_bucket
    FROM production_data
)
SELECT 
    quality_bucket,
    COUNT(*) AS total_batches,
    SUM(CASE WHEN DefectStatus = 'Defect' THEN 1 ELSE 0 END) AS defective_batches,
    ROUND(SUM(CASE WHEN DefectStatus = 'Defect' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS defect_rate_percent
FROM supplier_bucket
GROUP BY quality_bucket
ORDER BY defect_rate_percent DESC;


/* 
   Defect rate by WorkerProductivity quartile

    Using the NTILE(4) window function, all records
   are divided into 4 groups (quartiles) based on
   WorkerProductivity, then the defect rate is shown for each
   quartile. This shows whether lower worker productivity is
   associated with higher defect rates.
*/
WITH productivity_quartile AS (
    SELECT 
        *,
        NTILE(4) OVER (ORDER BY WorkerProductivity) AS productivity_quartile
    FROM production_data
)
SELECT 
    productivity_quartile,
    COUNT(*) AS total_batches,
    ROUND(AVG(WorkerProductivity), 2) AS avg_worker_productivity,
    SUM(CASE WHEN DefectStatus = 'Defect' THEN 1 ELSE 0 END) AS defective_batches,
    ROUND(SUM(CASE WHEN DefectStatus = 'Defect' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS defect_rate_percent
FROM productivity_quartile
GROUP BY productivity_quartile
ORDER BY productivity_quartile;


/* 
    Defective records with above-average ProductionCost 
  
   Find all defective batches whose production cost
   is higher than the overall average production cost. These
   batches represent a "double loss" for the company - higher
   cost AND a defective product.
*/
SELECT 
    ProductionVolume,
    ProductionCost,
    DefectRate,
    QualityScore,
    DefectStatus
FROM production_data
WHERE DefectStatus = 'Defect'
  AND ProductionCost > (SELECT AVG(ProductionCost) FROM production_data)
ORDER BY ProductionCost DESC;


/* 
    Does lower MaintenanceHours lead to higher DowntimePercentage?
    
   MaintenanceHours is grouped into buckets to check
   whether the low-maintenance group has a higher average
   downtime percentage. This helps validate whether increasing
   maintenance reduces downtime.
*/
SELECT 
    CASE 
        WHEN MaintenanceHours < 5 THEN 'Low Maintenance (< 5 hrs)'
        WHEN MaintenanceHours BETWEEN 5 AND 15 THEN 'Medium Maintenance (5-15 hrs)'
        ELSE 'High Maintenance (> 15 hrs)'
    END AS maintenance_bucket,
    COUNT(*) AS total_batches,
    ROUND(AVG(DowntimePercentage), 2) AS avg_downtime_percent,
    ROUND(AVG(DefectRate), 2) AS avg_defect_rate
FROM production_data
GROUP BY maintenance_bucket
ORDER BY avg_downtime_percent DESC;
