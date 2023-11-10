A murder mystery database management system is designed to organize, store, and manage information related to murder investigations, suspects, victims, evidence, and case details. Such a system plays a crucial role in law enforcement, providing investigators with a centralized platform to efficiently track and analyze information throughout the course of a murder investigation. Here is a description of key components and functionalities typically found in a murder mystery database management system:

1. Case Information Management: The system allows law enforcement to create and manage individual cases. Each case entry includes details such as case number, date, location, and a brief description of the incident.

2. Suspect and Witness Profiles: Investigators can input and update information about suspects and witnesses involved in the case. This includes personal details, known associates, criminal history, and any relevant background information.

3. Victim Profiles: Victim profiles contain information about the individuals who have been affected by the crime. This may include personal details, relationships, and any potential connections to suspects.

4. Evidence Tracking: The system facilitates the tracking of physical and digital evidence collected during an investigation. Each piece of evidence is assigned a unique identifier, and details such as the date of collection, location, and chain of custody are recorded.

5. Case Timeline: A chronological timeline of events related to the case helps investigators visualize the sequence of incidents. This feature aids in identifying patterns, potential motives, and key points in the investigation.

6. Document and Media Management: The system allows for the attachment and management of documents, photos, videos, and other media related to the case. This includes police reports, forensic analyses, surveillance footage, and any other relevant files.

7. Investigator Notes and Comments: Investigators can add and share notes, comments, and observations related to the case. This collaborative feature ensures that all team members are informed and can contribute their insights.

8. Case Status and Progress Tracking: The system provides a status update on each case, indicating whether it is open, closed, or in progress. Additionally, it tracks the progress of various tasks within the investigation.

9. Access Control and Security: Given the sensitive nature of the information, access control features ensure that only authorized personnel can view or modify specific details. Security measures such as encryption may also be implemented to protect the integrity of the data.

10. Reporting and Analytics: The system enables the generation of reports and analytics to help investigators analyze trends, patterns, and statistics related to murder investigations. This can assist in decision-making and resource allocation.

Implementing a murder mystery database management system streamlines investigative processes, enhances collaboration among law enforcement personnel, and ultimately contributes to more effective and efficient resolution of murder cases.




1. List the cases with the highest number of clues
   
		SELECT Cases.CaseName, COUNT(Clues.CaseID) AS NumClues
		FROM Cases
		LEFT JOIN Clues ON Cases.CaseID = Clues.CaseID
		GROUP BY Cases.CaseID
		ORDER BY NumClues DESC
		LIMIT 1;




2. find the average age of victims in cases with a description containing the word "murder"
   
		WITH MurderCases AS (
		    SELECT c.CaseID, v.Age
		    FROM Cases c
		    JOIN Victims v ON c.CaseID = v.CaseID
		    WHERE LOWER(c.Description) LIKE '%murder%'
		)
		SELECT AVG(Age) AS AverageVictimAge
		FROM MurderCases;




3. find the cases that involve witnesses with statements containing the word "loud":
   
		WITH LoudWitnesses AS (
		    SELECT c.CaseName, w.Name AS WitnessName, w.Statement
		    FROM Cases c
		    JOIN Witnesses w ON c.CaseID = w.CaseID
		    WHERE LOWER(w.Statement) LIKE '%loud%'
		)
		SELECT CaseName, WitnessName, Statement
		FROM LoudWitnesses;




4. Calculate the average age of victims for cases with clues at 'Abandoned Warehouse'
   
		SELECT AVG(Victims.Age) AS AvgVictimAge
		FROM Victims
		WHERE Victims.CaseID IN (
		    SELECT DISTINCT Clues.CaseID
		    FROM Clues
		    INNER JOIN Locations ON Clues.LocationID = Locations.LocationID
		    WHERE Locations.Name = 'Abandoned Warehouse'
		);



5. Calculate the total number of clues in cases with suspects who are younger than the average age of victims
   
		SELECT COUNT(Clues.ClueID) AS TotalClues
		FROM Clues
		WHERE Clues.CaseID IN (
		    SELECT DISTINCT Suspects.CaseID
		    FROM Suspects
		    INNER JOIN Victims ON Suspects.CaseID = Victims.CaseID
		    GROUP BY Suspects.CaseID
		    HAVING AVG(Victims.Age) > (
		        SELECT AVG(Age)
		        FROM Victims
		    )
		);

	 

6. Find cases with suspects who are also witnesses:

		SELECT DISTINCT Cases.CaseName
		FROM Cases
		WHERE Cases.CaseID IN (
		    SELECT DISTINCT Witnesses.CaseID
		    FROM Witnesses
		    WHERE Witnesses.CaseID = Cases.CaseID
		    AND Witnesses.WitnessID IN (
		        SELECT Suspects.SuspectID
		        FROM Suspects
		        WHERE Suspects.CaseID = Cases.CaseID
		    )
		);



7. Count the number of Events for each Case and truncate the results to one decimal place
   
		SELECT CaseName, TRUNCATE(NumEvents, 1) AS RoundedNumEvents
		FROM (
		    SELECT Cases.CaseName, COUNT(Events.EventID) AS NumEvents
		    FROM Cases
		    LEFT JOIN Events ON Cases.CaseID = Events.CaseID
		    GROUP BY Cases.CaseName
		) AS CaseEventCounts;




8. List the Cases with a truncated description (maximum length of 20 characters) and their respective Suspect's names
    
		SELECT c.CaseName, TRUNCATE(c.Description, 20) AS TruncatedDescription, s.Name AS SuspectName
		FROM Cases c
		JOIN Suspects s ON c.CaseID = s.CaseID;




9. find the suspect's name for Cases where the Victims' ages are below the average age of all Victims:
    
		SELECT s.Name AS SuspectName
		FROM Suspects s
		WHERE s.CaseID IN (
		    SELECT v.CaseID
		    FROM Victims v
		    WHERE v.Age < (SELECT AVG(Age) FROM Victims)
		);




10. Find the average age of Victims for Cases with Suspects who have the word "family" in their motive
    
		SELECT AVG(v.Age) AS AverageVictimAge
		FROM Victims v
		WHERE v.CaseID IN (
		    SELECT s.CaseID
		    FROM Suspects s
		    WHERE s.Motive LIKE '%family%'
		);




11. Identify the top 3 Locations with the most cases
    
		SELECT l.Name AS LocationName, COUNT(c.CaseID) AS NumOfCases
		FROM Locations l
		LEFT JOIN Clues c ON l.LocationID = c.LocationID
		GROUP BY l.LocationID
		ORDER BY NumOfCases DESC
		LIMIT 3;




12. List Cases where the Witnesses' statements contain the word "strange" and "noises"
    
		SELECT c.CaseName
		FROM Cases c
		WHERE EXISTS (
		    SELECT 1
		    FROM Witnesses w
		    WHERE c.CaseID = w.CaseID
		    AND w.Statement LIKE '%strange%'
		    AND w.Statement LIKE '%noises%'
		);



13. Identify the top 3 Suspects with the most common motives across all cases.
    
		SELECT s.Name AS SuspectName, s.Motive AS CommonMotive, COUNT(s.Motive) AS MotiveCount
		FROM Suspects s
		GROUP BY s.Name, s.Motive
		HAVING MotiveCount = (
		    SELECT MAX(MotiveCount)
		    FROM (
		        SELECT COUNT(Motive) AS MotiveCount
		        FROM Suspects
		        GROUP BY Motive
		    ) AS MaxMotiveCount
		);



14. Find the Clues whose descriptions contain the word "mysterious" or "secret" in cases with Victims aged between 30 and 50
    
		SELECT c.Description AS MysteriousOrSecretClue
		FROM Clues c
		WHERE c.Description LIKE '%mysterious%' OR c.Description LIKE '%secret%'
		AND c.CaseID IN (
		    SELECT v.CaseID
		    FROM Victims v
		    WHERE v.Age BETWEEN 30 AND 50
		);




15. List the Victims who are Witnesses in the most cases and their respective CaseIDs
    
		SELECT w.Name AS WitnessName, COUNT(w.CaseID) AS NumOfCases
		FROM Witnesses w
		GROUP BY w.Name
		HAVING COUNT(w.CaseID) = (
		    SELECT MAX(NumOfCases)
		    FROM (
		        SELECT COUNT(w2.CaseID) AS NumOfCases
		        FROM Witnesses w2
		        GROUP BY w2.Name
		    ) AS MaxCases
		);




16. Find the Suspects and their motives in cases with Suspects whose names start with a vowel.
    
		SELECT Name, Motive
		FROM Suspects
		WHERE LOWER(SUBSTRING(Name, 1, 1)) IN ('a', 'e', 'i', 'o', 'u');



17. find the most common motive among suspects
    
		WITH SuspectMotives AS (
		    SELECT LOWER(Motive) AS Motive
		    FROM Suspects
		)
		SELECT Motive, COUNT(*) AS Count
		FROM SuspectMotives
		GROUP BY Motive
		ORDER BY Count DESC
		LIMIT 1;




18. find cases with a suspect and their respective locations
    
		WITH CaseSuspectLocations AS (
		    SELECT c.CaseName, s.Name AS SuspectName, l.Name AS LocationName
		    FROM Cases c
		    JOIN Suspects s ON c.CaseID = s.CaseID
		    JOIN Locations l ON l.LocationID = s.CaseID
		)
		SELECT CaseName, SuspectName, LocationName
		FROM CaseSuspectLocations;




19. find the total number of cases and their respective victims
    
		WITH CaseVictims AS (
		    SELECT c.CaseName, v.Name AS VictimName
		    FROM Cases c
		    LEFT JOIN Victims v ON c.CaseID = v.CaseID
		)
		SELECT CaseName, COUNT(VictimName) AS TotalVictims
		FROM CaseVictims
		GROUP BY CaseName;



20. find the total number of witnesses in each case, ordered by case name
    
		SELECT c.CaseName, COUNT(w.CaseID) AS TotalWitnesses
		FROM Cases c
		LEFT JOIN Witnesses w ON c.CaseID = w.CaseID
		GROUP BY c.CaseName
		ORDER BY c.CaseName;




21. find cases with the most recent events and their descriptions
    
		SELECT c.CaseName, e.EventName, e.Date, e.Description
		FROM Cases c
		JOIN Events e ON c.CaseID = e.CaseID
		WHERE e.Date = (SELECT MAX(Date) FROM Events WHERE CaseID = c.CaseID);



22. find cases with a victim named "Alice Johnson" and list the events related to those cases
    
		SELECT c.CaseName, e.EventName, e.Date
		FROM Cases c
		JOIN Events e ON c.CaseID = e.CaseID
		WHERE EXISTS (
		    SELECT 1
		    FROM Victims v
		    WHERE v.CaseID = c.CaseID AND v.Name = 'Alice Johnson'
		);





23. find cases where the suspect's name contains "John" and display the number of events for each case
    
		SELECT c.CaseName, COUNT(e.CaseID) AS TotalEvents
		FROM Cases c
		LEFT JOIN Events e ON c.CaseID = e.CaseID
		WHERE EXISTS (
		    SELECT 1
		    FROM Suspects s
		    WHERE s.CaseID = c.CaseID AND s.Name LIKE '%John%'
		)
		GROUP BY c.CaseName;




24. find cases with exactly two witnesses and list their names
    
		SELECT c.CaseName, w1.Name AS Witness1, w2.Name AS Witness2
		FROM Cases c
		JOIN Witnesses w1 ON c.CaseID = w1.CaseID
		JOIN Witnesses w2 ON c.CaseID = w2.CaseID
		WHERE c.CaseID IN (
		    SELECT w.CaseID
		    FROM Witnesses w
		    GROUP BY w.CaseID
		    HAVING COUNT(*) = 2
		);




25. find cases that involve both suspects and witnesses and display the total number of people involved in each case
    
		SELECT c.CaseName, COUNT(DISTINCT s.Name) + COUNT(DISTINCT w.Name) AS TotalPeopleInvolved
		FROM Cases c
		LEFT JOIN Suspects s ON c.CaseID = s.CaseID
		LEFT JOIN Witnesses w ON c.CaseID = w.CaseID
		GROUP BY c.CaseName
		HAVING COUNT(DISTINCT s.Name) > 0 AND COUNT(DISTINCT w.Name) > 0;





26. find cases where the victim's name is not found among the suspects
    
		SELECT c.CaseName, v.Name AS VictimName
		FROM Cases c
		JOIN Victims v ON c.CaseID = v.CaseID
		WHERE v.Name NOT IN (
		    SELECT s.Name
		    FROM Suspects s
		    WHERE s.CaseID = c.CaseID
		);





















