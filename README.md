# Healthcare Patient Database Analysis

This report looks closely at the Healthcare Patient database. The database has information about patients, doctors, locations, and hospital departments.
Problem Statement
•	Fetch patients with the same age.
•	Find the second oldest patient and their doctor and department.
•	Get the maximum age per department and the patient name.
•	Doctor-wise count of patients sorted by count in descending order.
•	Fetch only the first name from the PatientName and append the age.
•	Fetch patients with odd ages.
•	Create a view to fetch patient details with an age greater than 50.
•	Create a procedure to update the patient's age by 10% where the department is 'Cardiology' and the doctor is not 'Dr. Smith'.
•	Create a stored procedure to fetch patient details along with their doctor, department, and state, including error handling.
# Data Source
Healthcare industry
# Tools Used
•	SSMS
# Query
CREATE DATABASE HealthcarePatientDb;
USE HealthcaretientDb;
CREATE TABLE Patient(
PatientID VARCHAR(20) PRIMARY KEY,
PatientName VARCHAR(20),
Age INT,
Gender VARCHAR(20),
DoctorID INT,
FOREIGN KEY (DoctorID) REFERENCES Doctor(DoctorID), 
StateID INT,
FOREIGN KEY (StateID) REFERENCES Statemaster(StateID));

CREATE TABLE Doctor(
DoctorID INT PRIMARY KEY,
DoctorName VARCHAR(20),
Specialization  VARCHAR(20));

CREATE TABLE Statemaster(
StateID INT PRIMARY KEY,
StateName VARCHAR(20));

CREATE TABLE MDepartment (
MDepartmentID INT PRIMARY KEY,
MDepartmentName VARCHAR(20));

INSERT INTO Patient(PatientID,PatientName,Age,Gender,DoctorID,StateID)
VALUES 
('PT01', 'John Doe', 45, 'M', 1, 101),
('PT02', 'Jane Smith', 30, 'F', 2, 102),
('PT03', 'Mary Johnson', 60, 'F', 3, 103),
('PT04', 'Michael Brown', 50, 'M', 4, 104),
('PT05', 'Patricia Davis', 40, 'F', 1, 105),
('PT06', 'Robert Miller', 55, 'M', 2, 106),
('PT07', 'Linda Wilson', 35, 'F', 3, 107),
('PT08', 'William Moore', 65, 'M', 4, 108),
('PT09', 'Barbara Taylor', 28, 'F', 1, 109),
('PT10', 'James Anderson', 70, 'M', 2, 110);

INSERT INTO Doctor(DoctorID,DoctorName,Specialization)
VALUES
(1, 'Dr. Smith', 'Cardiology'),
(2, 'Dr. Adams', 'Neurology'),
(3, 'Dr. White', 'Orthopedics'),
(4, 'Dr. Johnson', 'Dermatology');

INSERT INTO Statemaster(StateID,StateName)
VALUES 
(101, 'Lagos'),
(102, 'Abuja'),
(103, 'Kano'),
(104, 'Delta'),
(105, 'Ido'),
(106, 'Ibadan'),
(107, 'Enugu'),
(108, 'Kaduna'),
(109, 'Ogun'),
(110, 'Anambra');

INSERT INTO MDepartment(MDepartmentID, MDepartmentName)
VALUES 
(1, 'Cardiology'),
(2, 'Neurology'),
(3, 'Orthopedics'),
(4, 'Dermatology');

SELECT * FROM Patient;
--1. Fetch patients with the same age.
SELECT Age, COUNT(*) AS PatientCount 
FROM Patient
GROUP BY Age
HAVING COUNT(*) > 0;

-- 2. Find the second oldest patient and their doctor and department.
SELECT p.PatientName, p.Age, d.DoctorName, dept.MDepartmentName
FROM Patient p
JOIN Doctor d ON p.DoctorID = d.DoctorID
JOIN MDepartment dept ON d.Specialization = dept.MDepartmentName
ORDER BY p.Age DESC
OFFSET 1 ROW
FETCH NEXT 1 ROW ONLY;

-- 3. Get the maximum age per department and the patient name.
SELECT 
  dept.MDepartmentName,
  p.Age AS MaxAge,
  p.PatientName
FROM 
  Patient p
  JOIN Doctor d ON p.DoctorID = d.DoctorID
  JOIN MDepartment dept ON d.Specialization = dept.MDepartmentName
WHERE 
  (dept.MDepartmentName, p.Age) IN (
    SELECT 
      dept.MDepartmentName,
      MAX(p.Age)
    FROM 
      Patient p
      JOIN Doctor d ON p.DoctorID = d.DoctorID
      JOIN MDepartment dept ON d.Specialization = dept.MDepartmentName
    GROUP BY 
      dept.MDepartmentName
  );

-- 4. Doctor-wise count of patients sorted by count in descending order.
SELECT 
  dept.MDepartmentName,
  p.Age AS MaxAge,
  p.PatientName
FROM 
  Patient p
  JOIN Doctor d ON p.DoctorID = d.DoctorID
  JOIN MDepartment dept ON d.Specialization = dept.MDepartmentName
WHERE 
  (dept.MDepartmentName, p.Age) IN (
    SELECT 
      dept.MDepartmentName,
      MAX(p.Age)
    FROM 
      Patient p
      JOIN Doctor d ON p.DoctorID = d.DoctorID
      JOIN MDepartment dept ON d.Specialization = dept.MDepartmentName
    GROUP BY 
      dept.MDepartmentName
  );

-- 5. Fetch only the first name from the PatientName and append the age.
SELECT 
  LEFT(PatientName, CHARINDEX(' ', PatientName) - 1) AS FirstName,
  Age
FROM 
  Patient;

-- 6. Fetch patients with odd ages.
SELECT *
FROM Patient
WHERE Age % 2 != 0;

-- 7. Create a view to fetch patient details with an age greater than 50.
CREATE VIEW PatientDetailsOver50 AS
SELECT *
FROM Patient
WHERE Age > 50;
SELECT * FROM PatientDetailsOver50;
-- 8. Create a procedure to update the patient's age by 10% where the department is 'Cardiology' and the doctor is not 'Dr. Smith'.
CREATE PROCEDURE UpdatePatientAge
AS
BEGIN
    UPDATE P
    SET Age = Age * 1.10
    FROM Patient P
    JOIN Doctor D ON P.DoctorID = D.DoctorID
    JOIN MDepartment M ON D.Specialization = M.MDepartmentName
    WHERE M.MDepartmentName = 'Cardiology' AND D.DoctorName != 'Dr. Smith';
END;
EXEC UpdatePatientAge;

-- 9. Create a stored procedure to fetch patient details along with their doctor, department, and state, including error handling.
CREATE PROCEDURE GetPatientDetails
AS
BEGIN
    BEGIN TRY
        SELECT 
            P.PatientName, 
            P.Age, 
            D.DoctorName, 
            M.MDepartmentName, 
            S.StateName
        FROM 
            Patient P
        JOIN 
            Doctor D ON P.DoctorID = D.DoctorID
        JOIN 
            MDepartment M ON D.Specialization = M.MDepartmentName
        JOIN 
            Statemaster S ON P.StateID = S.StateID;
    END TRY
    BEGIN CATCH
        DECLARE @ErrorMessage NVARCHAR(4000);
        SET @ErrorMessage = ERROR_MESSAGE();
        RAISERROR (@ErrorMessage, 16, 1);
    END CATCH;
END;

# Image
 ![11](https://github.com/user-attachments/assets/2d0bfe1d-9827-4742-9caf-109a7b9c9472)

# Data Analysis
Before analyzing the data, I set up databases and tables in the SQL system and added healthcare data to them. This allowed me to connect and combine the data. Next, I identified the questions we needed to answer and stored the data in a way that makes it easy to access and use again in the future, using tools like views and stored procedures.
# Findings
1. Age distribution: Patients' ages range from 28 to 70, with a diverse age distribution.
2. Gender balance: The gender distribution is relatively balanced, with 5 male and 5 female patients.
3. Doctor specialization: Doctors specialize in Cardiology, Neurology, Orthopedics, and Dermatology.
4. State representation: Patients come from various states, including Lagos, Abuja, Kano, Delta, and others.
5. Departmental distribution: Patients are distributed across different departments, with Cardiology having the most patients.

# Recommendations
1. Age-specific healthcare initiatives: Develop targeted healthcare programs for specific age groups, such as geriatric care for patients above 60.
2. Gender-sensitive healthcare: Ensure equal access to healthcare for both male and female patients.
3. Specialized care: Continue to provide specialized care through various departments, such as Cardiology and Neurology.
4. Geographic expansion: Consider expanding healthcare services to more states and regions.
5. Departmental resource allocation: Allocate resources effectively across departments to ensure quality patient care.

# Conclusion
The analysis of the Healthcare Patient database reveals a diverse patient population with varying ages, genders, and geographic locations, and a well-structured healthcare system with specialized departments and doctors. The findings highlight the need for targeted healthcare initiatives, resource allocation, and expansion of services to improve patient care. Overall, the data analysis provides valuable insights that can inform data-driven decisions to enhance patient care and services, and regular monitoring and analysis will be essential to identify trends and areas for improvement.
