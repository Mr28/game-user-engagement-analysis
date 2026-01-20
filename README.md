*Data columns description:
- sessionid: identifier for each gaming session, unique, though shared across records for multi-days gaming sessions, or in case of repeated logged data of the same session. It is an integer, uniformly distributed around 0.
- playerid: self-explanatory, player identifier, integer umiformly distributed around 0
- serverdate_first/serverdate_last: timestmps. I guess they show the start/end of the logging session on the server for each gaming session on a single day. Though there are few records (faulty or not) with server dates covering multiple days. The data seems to be collected based on these timestamps, to cover a 6 months period.
- clientdate_first/clientdate_last: timestamps. my speculation is that they show the logging session on the user's device for each gaming session. These seem to be the true references for session up-time calculations.
- dateid: Integer, seems to be representing the date extracted from server related timestamps.
- countrycode: String, Self explanatory, country codes based on ISO 3166-1 alpha-2 convention. 
- gameversion: Categorical, identical for the entire dataset, thus removed for the modelling.
- playersessions_index: integer, representing the rank in the que for the records of each player sorted based on the record date and time
- playdays_index: integer, 
- sessionuptime_duration
- cum_playtime_duration
- session_playtime_duration
- population





*Anomalies:
2- Logical consistency checks:
Due to faulty device date or long-postponed sync with servers, some records date back to up to 2 years before the server dates according to dates recorded based on the users' devices. It's even possible to be indicator of attempts to bypass the licensing system if the game is subscription based. There are 359 records with ove 1 week of difference beetween client and server dates. Kept in the data, since there's not enough evidence of harm to the modelling to be remoevd.

First and last server dates are always matching (same day) except for 16 records, which differ up to 6 weeks, while the session up-times are no more than 10 hours, thus, suggesting an error in the logging system of the servers. Kept in the data, since there's not enough evidence of harm to the modelling to be remoevd.

There were 307 records with an abnormal country code ('RU5514', presumably meaning Russia!) with session_playtime_duration=-1. It could be hinting at malicious attempts to access to the servers or malware in the system trying to modify the database. Thery were removed for the current modeling task.
Also there were a couple of records with no registry of the country. Since the according player had no other records in the sample, the records were dropped due to isolate nature of the records and not impacting the other record series.

