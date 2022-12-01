






  Solution Design Document 
Replication Mule Project









Version: 1.00
Updated: 01/01/2022
Author: Tanner Hanson
























Overview

		This document is used to explain the replication process, this process includes taking mock data to simulate a salesforce user and inserting it into a mock sql database.  The dataset provided includes a country code that is used to sort the data into the appropriate tables in the database.


Requirement Details


	The requirements listed were a scheduled daily batch job, and replicating the data from the batch job from system A to system B.


Assumptions and Prerequisites

	The main assumptions used is the data taking place, as this is a simulated environment and there is no clear source of the data.  I had to create my own data based on the given information, this is an example of the data I used to mock:


{
  "case": [
    {
      "userId": 63,
      "country": "CAN",
      "favoriteAnimal": "Cat",
      "referenceNumber": "5b5d87be-b954-4d71-89d0-fbf67192fad8"
    },
    {
      "userId": 35,
      "country": "USA",
      "favoriteAnimal": "Dog",
      "referenceNumber": "cc32a328-d973-48fe-9fe0-b509fdf1873d"
    }
  ]
}




High-Level Design

The design process uses a standard Mule integration design.  Including a Xapi layer, a Papi layer and a Sapi layer.  The Xapi layer takes in the data from the daily scheduled batch, the papi layer then takes that data and morphs it as needed, in this specific case, there was no need to morph any data.  Finally, the sapi later then decides which SQL database to insert into by checking the country code associated with the data.  Then, the API returns the amount of unsuccessful entries, along with which ones unsuccessful and the successful entries. 



Low-Level Design 

XAPI LAYER:


This layer starts by having the scheduler kick off the API daily, it then takes the data from the mock and saves it so there is a base variable with it.  Then it leads to a flow reference to the papi layer.  The error handling sends back the error along with a description of it.



PAPI LAYER: 



This layer is usually used to morph data as needed, however for this specific assignment there is no need and that can be done more eloquently in the sapi layer.  However, I did still include the papi layer to future proof this API as there could be more projects down the line that the papi layer would need to call to get more data from, or the data could turn out to be more complex.  The error handling is returning the error with a description.   

SAPI LAYER:




This is the main crutch of the API.  It starts by initializing a few counter variables and the finalMessage that will be returned, it then goes through a “for each” with a try to iterate through the data that is being sent.  It checks to see if the data can be inserted successfully, if it does it returns back a successful message and appends it to the final message.  If the data does not have a matching country code, or one of the database calls fails it moves to an on error continue, where it logs the issue, and adds an unsuccessful entry to the finalMessage, however it does not completely fail out, it keeps processing the other entries.  

Each database has a sql statement similar to this:



INSERT INTO Case(country, userId, referenceNumber favoriteAnimal)
VALUES (:case, :userId, :referenceNumber :favoriteAnimal)



This returns a payload similar to this:



{
  "STATUS": "STATUS OF INSERTIONS BELOW",
  "STATUS": "CAN Data Entry 0 Successfully inserted",
  "STATUS": "USA Data Entry 1 Successfully inserted",
  "STATUS": "AUS Data Entry 2 Successfully inserted",
  "STATUS": "USA Data Entry 3 Successfully inserted",
  "STATUS": "AUS Data Entry 4 Successfully inserted",
  "STATUS": "AUS Data Entry 5 Successfully inserted",
  "STATUS": "CAN Data Entry 6 Successfully inserted",
  "STATUS": "AUS Data Entry 7 Successfully inserted",
  "STATUS": "USA Data Entry 8 Successfully inserted",
  "STATUS": "AUS Data Entry 9 Successfully inserted",
  "STATUS": "CAN Data Entry 10 Successfully inserted",
  "STATUS": "AUS Data Entry 11 Successfully inserted",
  "STATUS": "CAN Data Entry 12 Successfully inserted",
  "STATUS": "USA Data Entry 13 Successfully inserted",
  "STATUS": "USA Data Entry 14 Successfully inserted",
  "STATUS": "AUS Data Entry 15 Successfully inserted",
  "STATUS": "AUS Data Entry 16 Successfully inserted",
  "STATUS": "AUS Data Entry 17 Successfully inserted",
  "STATUS": "CAN Data Entry 18 Successfully inserted",
  "STATUS": "USA Data Entry 19 Successfully inserted",
  "STATUS": "No Unsuccessful Entries"
}



MOCK TESTS:




This is how the api has been tested and how the api runs.  I included three mock tests, one simulating a completely successful run, one simulating a successful run with a few unsuccessful entries and one simulating a complete failure.






The “Successful-test-suite” sends in data that is perfect and just as expected.  It returns all successful entries. 



The “Unsuccessful-test-suite” sends in data that includes country codes that are not expected, thus it returns a message including which entries failed, and the successful entries as well. 




The error mock causes all 3 of the databases to return “HTTP:CONNECTIVITY” errors, thus simulating how the API would handle that.  It returns back the status as no entries have successfully been submitted. 





7. Risks and Mitigation

The main risks of the project involve the connectivity to the database itself, and a change of the data being sent.  If the connectivity to the database goes down, then the data would not be inserted into the database.  Along those lines, if a change in the data being sent occurs then the SQL statements themselves would need to be tweaked in order to reflect those changes. Another limitation that could arise is how the logging is handled.  Currently, any unsuccessful entries get stored in the logs for a user to parse through to figure out where it went wrong, however with a more robust logging system, it could be more efficient then having to sort through thousands of potential issues. 




