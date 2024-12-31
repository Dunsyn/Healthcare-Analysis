# Healthcare-Analysis
In today's fast-evolving world, data is not just an asset; it is the key to unlocking unparalleled efficiencies and transformative insights. 
As a Business Intelligence Analyst, I recently had the opportunity to analyze a healthcare dataset comprising patient demographics, treatment outcomes, admission insights, billing details, and hospital performance. 
This experience underscored the vital role of data-driven insights in shaping the future of healthcare delivery. 

SELECT * FROM Patient_Table;

SELECT * FROM Admission_Table;

SELECT * FROM Doctor_Table;

SELECT * FROM Hospital_Table;

SELECT * FROM Insurance_Table;


			--PATIENT DEMOGRAPHICS
--1) Age Distribution

SELECT 
    COUNT(patient_id) AS id_count,
    CASE 
        WHEN age BETWEEN 18 AND 24 THEN 'Young Adults'
        WHEN age BETWEEN 25 AND 34 THEN 'Adult'
        WHEN age BETWEEN 35 AND 49 THEN 'Middle Adult'
        WHEN age BETWEEN 50 AND 64 THEN 'Senior Adult'
        WHEN age BETWEEN 65 AND 85 THEN 'Elder' 
    END AS age_bracket
FROM Patient_Table
GROUP BY 
    CASE 
        WHEN age BETWEEN 18 AND 24 THEN 'Young Adults'
        WHEN age BETWEEN 25 AND 34 THEN 'Adult'
        WHEN age BETWEEN 35 AND 49 THEN 'Middle Adult'
        WHEN age BETWEEN 50 AND 64 THEN 'Senior Adult'
        WHEN age BETWEEN 65 AND 85 THEN 'Elder' 
    END
ORDER BY id_count DESC;

--2) Gender Breakdown

--a) Total number of patients by Gender

SELECT 
    Gender,
    COUNT(*) AS Patient_Count
FROM 
    Patient_Table
GROUP BY 
    Gender;

---b) Patient distribution by Gender, medical condition and medication

SELECT gender, medical_condition,  COUNT(patient_id) AS no_of_patient
FROM Patient_Table
GROUP BY gender, medical_condition
ORDER BY no_of_patient DESC;


SELECT gender, medical_condition, medication, COUNT(patient_id) AS no_of_patient
FROM Patient_Table
GROUP BY gender, medical_condition, medication
ORDER BY no_of_patient DESC;

--3) Blood Type Distribution 

-----Total Patients by Blood type
SELECT blood_type,  COUNT(patient_id) AS id_count
FROM Patient_Table
GROUP BY blood_type 
ORDER BY id_count DESC; 

------Blood Type by Medication

SELECT blood_type, medical_condition, COUNT(patient_id) AS id_count
FROM Patient_Table
GROUP BY blood_type, medical_condition
ORDER BY medical_condition, id_count DESC;

				--MEDICAL INSIGHTS

--1) Most Common Medical Conditions

SELECT medical_condition, COUNT(patient_id) AS id_count
FROM Patient_Table
GROUP BY medical_condition
ORDER BY id_count DESC;

--2) Condition by Age
SELECT 
    medical_condition,
    age_bracket,
    COUNT(patient_id) AS id_count
FROM (
    SELECT 
        medical_condition,
        patient_id,
        CASE 
            WHEN age BETWEEN 18 AND 24 THEN 'Young Adults'
            WHEN age BETWEEN 25 AND 34 THEN 'Adult'
            WHEN age BETWEEN 35 AND 49 THEN 'Middle Adult'
            WHEN age BETWEEN 50 AND 64 THEN 'Senior Adult'
            WHEN age BETWEEN 65 AND 85 THEN 'Elder'
            ELSE 'Unknown'
        END AS age_bracket
    FROM Patient_Table
) AS Age_Categorized
GROUP BY 
    medical_condition, age_bracket
ORDER BY 
     id_count DESC;

--3) Condition by Gender
SELECT medical_condition, gender, count(patient_id) AS patient_count
FROM Patient_Table
GROUP BY medical_condition, gender
ORDER BY medical_condition, patient_count DESC;

	--3 ADMISSION INSIGHTS
--a. Total number of admissions by hospital and admission type

SELECT admission_type, COUNT(*) AS no_of_admission
FROM Admission_Table
GROUP BY admission_type
ORDER BY admission_type DESC;

--b. Duration of Stay by Condition and Admission Type 

SELECT p.medical_condition, a.admission_type,
DATEDIFF(DAY, a.date_of_admission, a.discharge_date) * 24 AS length_of_stay_in_hours
FROM Admission_Table AS a
INNER JOIN Patient_Table AS p ON a.Patient_ID = p.Patient_ID
ORDER BY length_of_stay_in_hours DESC;

--c. Admissions by Hospital - the number of paient admitted by each hospital.

SELECT h.hospital, a.admission_type, COUNT(h.Hospital_ID) AS admission_count
FROM Admission_Table AS a
JOIN Hospital_Table AS h 
ON a.hospital_id = h.hospital_id
GROUP BY h.hospital, a.Admission_Type
ORDER BY admission_count DESC;


SELECT h.hospital, a.admission_type, COUNT(a.Admission_ID) AS admission_count
FROM Admission_Table AS a
JOIN Hospital_Table AS h 
ON a.hospital_id = h.hospital_id
GROUP BY h.hospital, a.Admission_Type
ORDER BY a.Admission_Type, admission_count DESC;

--d. Year by Year

SELECT p.medical_condition, DATEPART(YEAR, a.date_of_admission) AS Year, COUNT(*) AS Admissions,
LAG(COUNT(*)) OVER (PARTITION BY p.medical_condition ORDER BY DATEPART(YEAR, a.date_of_admission)) AS Prev_Year_Admissions,
CASE
   WHEN LAG(COUNT(*)) OVER (PARTITION BY p.medical_condition ORDER BY DATEPART(YEAR, a.date_of_admission)) IS NOT NULL
   THEN (COUNT(*) - LAG(COUNT(*)) OVER (PARTITION BY p.medical_condition ORDER BY DATEPART(YEAR, a.date_of_admission))) / LAG(COUNT(*)) 
   OVER (PARTITION BY p.medical_condition ORDER BY DATEPART(YEAR, a.date_of_admission))* 100 ELSE NULL
END AS YoY_Growth_Percent
FROM Admission_Table AS a
INNER JOIN Patient_Table AS p ON a.patient_id = p.patient_id
GROUP BY p.medical_condition, DATEPART(YEAR, a.date_of_admission)
ORDER BY p.medical_condition, DATEPART(YEAR, a.date_of_admission);


--e. Percentage of Abnormal Test Results per Condition and Admission Type:

SELECT medical_condition, admission_type, COUNT(*) AS total_tests,
    SUM(CASE WHEN test_results = 'Abnormal' THEN 1 ELSE 0 END) AS Abnormal_Count,
    ROUND(SUM(CASE WHEN test_results = 'Abnormal' THEN 1 ELSE 0 END) * 100.0 / COUNT(*),2) AS abnormal_percent
FROM Admission_Table AS a
JOIN  Patient_Table AS p
ON a.patient_id = p.patient_id
GROUP BY p.medical_condition, a.admission_type
ORDER BY abnormal_percent DESC;


		--Billing & Cost Analysis
		
--a) Average Billing Amount by Medical Condition - 

SELECT medical_Condition, ROUND(AVG(Billing_Amount),2) AS Avg_Billing_Amount
FROM Admission_Table AS a
JOIN Patient_Table AS p 
ON a.Patient_ID = p.Patient_ID 
GROUP BY p.Medical_Condition 
ORDER BY Avg_Billing_Amount DESC;


--b).Top Costliest Conditions per Insurance Provider 

SELECT i.Insurance_Provider, Medical_Condition, ROUND(AVG(Billing_Amount),2) AS Avg_Billing,
ROW_NUMBER() OVER (PARTITION BY i.Insurance_Provider ORDER BY AVG(Billing_Amount) DESC) AS Rank
FROM Admission_Table AS a
JOIN Insurance_Table AS i
ON a.Insurance_ID = i.Insurance_ID 
JOIN Patient_Table AS p
ON p.patient_id = a.patient_id
GROUP BY i.Insurance_Provider, Medical_Condition
ORDER BY i.Insurance_Provider;


--c. Cost per Admission Type - Average billing amount for each admission type.

SELECT Admission_Type, ROUND(AVG(Billing_Amount),2) AS Avg_Billing_Amount
FROM Admission_Table 
GROUP BY Admission_Type
ORDER BY Avg_Billing_Amount DESC;


			--Treatment Outcome

--a. Medical Usage

SELECT medical_condition,medication, COUNT(medical_condition) AS no_of_medication
FROM Patient_Table
GROUP BY medication, medical_condition
ORDER BY medical_condition,no_of_medication DESC;

--b. Test Results Trends

SELECT test_results, COUNT(test_results) AS no_of_results, medical_condition, medication
FROM Patient_Table
GROUP BY medical_condition, medication, test_results
ORDER BY medical_condition, test_results;


			--Hospital-Specific Insights

--ai). Hospital Performance
SELECT TOP 5 h.hospital, ROUND(AVG(billing_amount),2) AS avg_cost,
		DATEDIFF(DAY, Date_of_Admission, Discharge_Date) AS Length_of_Stay, test_results
FROM Hospital_Table AS h
JOIN Admission_Table AS a
ON h.hospital_id = a.hospital_id
join Patient_Table p
ON p.patient_id = a.patient_id
GROUP BY h.Hospital,DATEDIFF(DAY, Date_of_Admission, Discharge_Date), test_results
ORDER BY avg_cost DESC; 

----aii). ----Hospital by Average Billing Amount and  Patient Outcomes
SELECT h.hospital, ROUND(AVG(billing_amount),2) AS avg_cost, test_results
FROM Hospital_Table AS h
JOIN Admission_Table AS a
ON h.hospital_id = a.hospital_id
join Patient_Table p
ON p.patient_id = a.patient_id
GROUP BY h.Hospital, test_results
ORDER BY test_results, avg_cost DESC; 

----aiii). Distribution of Hospial by Distribution of Stay and  Patient Outcomes
SELECT  h.hospital, DATEDIFF(DAY, Date_of_Admission, Discharge_Date)  AS Length_of_Stay, test_results
FROM Hospital_Table AS h 
JOIN Admission_Table AS a
ON h.hospital_id = a.hospital_id
join Patient_Table p
ON p.patient_id = a.patient_id
GROUP BY h.Hospital,DATEDIFF(DAY, Date_of_Admission, Discharge_Date), test_results
ORDER BY  Length_of_Stay DESC; 

--b. Room Utilization
SELECT COUNT(room_number) AS no_of_room, h.hospital
FROM Admission_Table AS a
JOIN Hospital_Table AS h
ON a.hospital_id = h.hospital_id
GROUP BY h.hospital
ORDER BY no_of_room DESC

