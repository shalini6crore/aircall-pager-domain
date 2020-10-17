# aircall-pager-domain
Solution to the Pager service

Instructions

Unit tests are in the package src/test/java
Please right-click and Run as Junit Test (Might differ from IDE to IDE. I have used RAD)

Solution Overview

Assumptions
1. DB has a table UNHEALTHY_SERVICES. Alerts sent and all their details are stored in this table.
2. If a service has an entry in this table, means it is unhealthy. otherwise, it's healthy.
3. Pager Service knows the names of the Escalation Levels. I have assumed there are 5. Level_A, Level_B.. Level_E.
4. Time of the Alert sent is stored in this table. It is used to find out if the alert were sent 15 minutes ago or more. Timeout function.
5. The timer service runs every minute to check which alerts have timed out and got no acknowledgements.

Services
1. Service to send alerts accepts request from all the services and timer service too. When a request reaches this service, it checks if an alert entry for the same service is existing in db. 
        If not, it fetches the targets of Level_A (first escalation level), adds a new alert entry in DB and sends emails and/or sms to the targets. 
        If yes, it fetches the entry from DB and check for it's alert time. 
              If it's 15 or more minutes old, means the request is from the Timer service. So, it fetches the current Level from the entry, increments it to next (or keeps the                   Level same if it's already highest, i.e. Level_E). Then, it calls Escalation Policy Service for targets of the new Level. It updates the DB with new Level, adds                    current time as alert time (reseting the timer) and sends request for emails and sms. 
              If it's less than 15 minutes old, means that this alert is from a service which already exists in our DB table. Means we have to do nothing about it.

2. Service to receive incident closed request from Console. As soon as it receives this request, it deletes the alerts entry from the table in DB. Which signifies that the service is now not in the table and so, is healthy. (Similar to hospitals. If someone is admitted in the hospital means they are sick, otherwise they are healthy. Hospitals don't need to keep trcak of healthy people)

3. Service to receive acknowledgement request from web. When the service receives this request, it update the DB and sets the isAcknowledged flag to true for that service.

4. Service to handle timer request. HandleTimer service receives request from the Timer service. It queries DB for all the entries with FALSE value for isAcknowledged flag. Then it checks which ones are timed out. Gets a list of entries. Calls the Service 1 to send alerts for all those services which are timed out.

