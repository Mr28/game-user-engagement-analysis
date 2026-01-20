*Data columns description:
- sessionid: identifier for each gaming session, unique, though shared across records for multi-days gaming sessions, or in case of repeated logged data of the same session. It is an integer, uniformly distributed around 0.
- playerid: self-explanatory, player identifier, integer umiformly distributed around 0
- serverdate_first/serverdate_last: timestmps. I guess they show the start/end of the logging session on the server for each gaming session on a single day. Though there are few records (faulty or not) with server dates covering multiple days. The data seems to be collected based on these timestamps, to cover a 6 months period.
- clientdate_first/clientdate_last: timestamps. my speculation is that they show the logging session on the user's device for each gaming session. These seem to be the true references for session up-time calculations.
- dateid: Integer, seems to be representing the date extracted from server related timestamps.
- countrycode: String, Self explanatory, country codes based on ISO 3166-1 alpha-2 convention. 
- gameversion: Categorical, identical for the entire dataset, thus removed for the modelling.
- playersessions_index: integer, representing the rank in the que for the records of each player sorted based on the record date and time
- playdays_index: integer, representing the rank in the sequence of records representing a single gaming session.
- sessionuptime_duration: integer, seems to be the difference of clientdate_first and clientdate_last, in seconds.
- cum_playtime_duration: integer, often equal to session_playtime_duration, but sometimes bigger and sometimes smaller. Was disregarded due to being unexplainble.
- session_playtime_duration: integer, self explanatory, the actual play-time of the player during the preiod associated to the record
- population: string, categorical, binary, seems to be indicating 2 populations used for an A/B test.





*Anomalies:
Logical consistency checks:
Due to faulty device date or long-postponed sync with servers, some records date back to up to 2 years before the server dates according to dates recorded based on the users' devices. It's even possible to be indicator of attempts to bypass the licensing system if the game is subscription based. There are 359 records with ove 1 week of difference beetween client and server dates. Kept in the data, since there's not enough evidence of harm to the modelling to be remoevd.

First and last server dates are always matching (same day) except for 16 records, which differ up to 6 weeks, while the session up-times are no more than 10 hours, thus, suggesting an error in the logging system of the servers. Kept in the data, since there's not enough evidence of harm to the modelling to be remoevd.

There were 307 records with an abnormal country code ('RU5514', presumably meaning Russia!) with session_playtime_duration=-1. It could be hinting at malicious attempts to access to the servers or malware in the system trying to modify the database. Thery were removed for the current modeling task.
Also there were a couple of records with no registry of the country. Since the according player had no other records in the sample, the records were dropped due to isolate nature of the records and not impacting the other record series.

sessionuptime_duration was 0 in some cases while session_playtime_duration was greater than 0. It was reconstructed in such cases using the difference of clientdate_first and clientdate_last.

There were instances of session_playtime_duration=-1. But it turned out to be exclusivly for the records with countrycode='RU5514', which were droped due to strange value and and since they were few (307) isolated records, not contaminating other record sequences,

Further investigations revealed that in some cases, multiple records were mistakenly created for the same period of single gaming sessions, sharing same playerid and clientdate_first ,often also similar other values (though not necessarily), except for the server datetimes. These records were combined. If the values of a field were dissimilar across the combining reocrds (apparently because of incomplete transmission/recording of data, or error in aggregating the data from which the current datset is possibly constructed), max, min or mode was used according to the nature of the field.





*Insights:

- Seasonlity:
This figure shows sum of play-time across all players per day. It suggest fluctuations, especially a jump in early May and the second one early in July. It could be nice if we had more data to see if they are resuly of internal forced from within the company (campaigns, feature releases, etc.) or external contributers such as school summer breaks. 

Moreover, the general trend seems to be similar in both populations 1 and 2, though population 1 seems to have maintained higher play-time compared to population 2 during the first 2 weeks of June. It can be an area of further investigation to see if this difference is also statiscally significant, or merely just some noise due to the random nature of the data.

- Global distribution:

The plot in the left shows the distribution of total number of players across countries, and the plot in the right shows the distribution of number of players per capita. While US is home to most players followed by China, we learn from the plot in the right that Some European countries, Canada and Australia have higher numper of players per population. 

This can be an area of further investigation to see what may have resulted such pattern (e.g. effectiveness of the campaigns, stronger fan-base, etc.)




*Data preprocessing:
Other than data cleaning (described in the anomalies section), I took the following steps
- Feature Creation: Added 'time_since_joined' (tenure in seconds), 'play_ratio' (playtime/uptime), and 'gap' (time between sessions).
- Time-Series Features: Computed expanding and rolling statistics (mean, std) for 'gap' and 'play_ratio' per player(expanding means mean/std observed from player data up to the record, rolling means calculation over the last 3 records of the player sorted by date).
- Target Definition: Defined 'last_session' as True for the final session if the gap to max date exceeds statistical thresholds(according to historical gap sata for the player).


*Statistical Analysis and Feature Validation

- Column Definitions: Listed numerical (e.g., durations, ratios) and categorical (countrycode, population) features.
- Correlation Analysis: Computed point-biserial correlations with 'last_session'; key findings: negative correlations for 'playersessions_index' (-0.113) and 'time_since_joined' (-0.099), indicating experienced players are less likely to churn.
- Association Tests: Performed chi-square tests; strong associations for 'countrycode' (chi2=780.56) and 'population' (chi2=80.35), confirming geographical and demographic influences.
- Feature Importance: Used logistic regression to rank features; top predictors included country codes and session metrics.
- Class Imbalance Check: Confirmed imbalance (7.2% 'last_session' = True), necessitating balanced modeling.
- Data Splitting: Split unique players into train (60%), validation (20%), and test (20%) sets to prevent leakage, maintaining class distributions.
Modeling Preparation
- Preprocessing: Encoded categoricals, imputed missing values, and aligned feature sets across splits.
- Benchmark Model: Trained logistic regression with class weights; validation AUC=0.652, F1=0.186.
- Comparison Model: Trained random forest; validation AUC=0.687, F1=0.016, with top features like 'playersessions_index'.
- Key Insights: The pipeline transformed raw session data into player-level features for churn prediction. Statistical tests validated features, highlighting tenure and geography as key drivers. Models outperformed baselines, but imbalance affected minority class recall. Data quality improvements (e.g., outlier handling) could enhance performance. Total players: 4,823; final dataset size: ~58,150 rows.