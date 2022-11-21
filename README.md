# Formula1 data analysis on how regulation changes affect the competition of the sport

Interested in Formula 1 for years, I have noticed the changes in the competition as it has evolved.  To set the stage, let me explain a few basics: the Federation Internationale de l'Automobile (FIA), the authority setting the rules of the sport, revises the regulation of the car specs that every team must abide by. A major regulation change usually occurs every few years (3-4). While regulations cannot be changed mid-season, they  might change even year depending on the specifics of the prior season; whether there was lack of safety or violations of certain features. What I noticed was that every time there is a change in the regulations and specs, the former champion loses. In other words, a dominant team under previous regulations, no longer is the winner, making space for a new champion. 

After observing this over several years, I noticed a pattern.  One of the runners up, of the previous season, would be the new champion post change in regulations.This team would generally remain champions until the regulations changed again. Now, one of the runners, would become the champion after the new regulations came into place. Interestingly, it appeared that the runner up team in each instance has mastered how to function best within the new regulations in the following seasons or found loopholes in the upcoming regulations and they managed to have a long term win margin. 

While this was a casual observation, I wanted to verify if this was historically true.  In order to investigate this seeming pattern, I created a few sets of python programs which would use an API key, linked to the Google cloud platform. This platform allowed me to write SQL programs to sort through the data that I found. This data was CSV files that contained information about the sport since its beginning. I gathered various forms of information; for example, I collected the results of each race, the drivers who had won the championship, the teams that won the team championship, lap times, and most importantly the major regulation changes and the timing of these changes in relation to the results since the beginning of the sport. 

In my IDE (Integrated development environment) I wrote a python program where I used the dataframe 
pd.read_csv which reads my CSV file and provides a result.  
In order to put that result into the GCP i used the program below:

from google.cloud import bigquery
from google.oauth2 import service_account
import pandas as pd
# Getting the GCP Cloud Permissions to connect Jupyter and python to Cloud database
    credentials = service_account.Credentials.from_service_account_file('F:\\Abhi\\code\\Vscode\\phyton\\credentials\\config.json')
    client = bigquery.Client('abhinand-playground',credentials)
# CHANGE NAME OF TABLE HERE AFTER RUNNING CODE TO GET DATAFRAME VALUES
    table_id = 'formula1_env.regchanges'

    #write_disposition = 'WRITE_APPEND' will append data into the table.
    #write_disposition = 'WRITE_TRUNCATE' If the table already exists, BigQuery overwrites the table data.
    #write_disposition = 'WRITE_EMPTY' If the table already exists and contains data, a ‘duplicate’ error is returned
    job_config = bigquery.LoadJobConfig(write_disposition='WRITE_TRUNCATE')
    job = client.load_table_from_dataframe(df, table_id, job_config=job_config)  # Make an API request.
    job.result()  # Wait for the job to complete.
    table = client.get_table(table_id)  # Make an API request.
    print(
        "Loaded {} rows and {} columns to {}".format(
            table.num_rows, len(table.schema), table_id
     )
    )

The most important parts of this program are the credentials for my personal account and loading my data into the GCP configuration. This part can be seen in the lines:
 “credentials = service_account.Credentials.from_service_account_file('F:\\Abhi\\code\\Vscode\\phyton\\credentials\\config.json')” 

The credentials is basically the “proof” that I have an account subscribed to GCP which would allow my IDE (visual studio code) to upload my CSV data into the GCP. There is a lot more to this program which I could explain, but essentially, once I added the files to GCP I could use SQL to sort through the data and analyze the changes in the winners of the Championships in Formula 1 over the years. 

In order to verify if the data is uploaded into GCP the program below uses an SQL based program to pull the data of a specific set: 
         from google.cloud import bigquery
         from google.oauth2 import service_account
        import db_dtypes
          # Construct a BigQuery client object.

        sql = """
        select * from formula1_env.regchanges
            """
        df_1 = client.query(sql).to_dataframe()
        df_1.head()
        
The code above the sentence "from google.cloud import bigquery" connects to GCP and the next sentence "from google.outh2 import service_account" connects to the service account or the account that refers to your credentials of your GCP service account. So in the program above when I use the "select * from formula1_env.regchanges" basically takes all the information from the data set "regchanges.csv" which can bee seen in the rest of my files. The results produced is below: 
![image](https://user-images.githubusercontent.com/117139045/201490363-c9680742-6cc1-47d7-88e9-5f4704d40788.png)

In the case I need to remove a table or a set of datasets I would use the program below:

    from google.cloud import bigquery
    from google.oauth2 import service_account

    # Construct a BigQuery client object.
    credentials = service_account.Credentials.from_service_account_file('F:\\Abhi\\code\\Vscode\\phyton\\credentials\\config.json')
    client = bigquery.Client('abhinand-playground',credentials)

    # If the table does not exist, delete_table raises
    # google.api_core.exceptions.NotFound unless not_found_ok is True.
    table_id = 'formula1_env.weather'
    client.delete_table(table_id, not_found_ok=True)  # Make an API request.
    print("Deleted table '{}'.".format(table_id))
    
The difference of code that is differnt is the  "client.delete_table(table_id, not_found_ok=True)  # Make an API request.
    print("Deleted table '{}'.".format(table_id))". This is pretty straight forward, "client.delete_table" would refer to the table entered in table_id, in this case it is 'formula1_env.weather' and it uses that ID to sort through all the data sets. That is why it is included with"(table_id, not_found_ok = True)"


I made the correlations by writing the SQL program such that it collected certain columns from certain data sets and joined them together. For example, I could take the championship winners from the years 1950 to 1955 and I would join the column from the data set with regulation changes from 1950 to 1955. So if there was a significant change in the regulations, then I would see whether there was a change in the Championship winning team, I did this over all the seasons of  Formula 1 since its inception. 
Here is an example of a code I used to join the tables of constructor_standings and constructor results to refer to which team won which title:

    select * from `formula1_env.constructor_standings` as a 
    join `formula1_env.constructor_results` as b on b.constructorId = a.constructorId 
    join `formula1_env.constructors`as c on c.constructorId = b.constructorId


Here are two examples of a set of seasons where a change in regulations resulted in a change in the Championship winners in the following years. During the 2009 Formula 1 Season, Ferrari placed 3rd in the Championship, Red Bull F1 finished in 2nd place and Brawn F1 placed 1st. After the 2009 season,  the FIA changed the  rev-limit (revolutions per minute) to 18,000 rpm, reduction in the ground clearance of the front wing from 150 mm to 50 mm and the introduction of KERS (kinetic energy recovery system) to store some of the energy generated under braking. After this change, in 2010 Red Bull F1 the runner up, placed 1st in the championship and McLarin placed 2nd and Brawn (which became Mercedes in 2010) placed 4th. 

Red Bull dominated the following 4 seasons winning all Championships and 41 out of 77 races until another major regulation change from 2013 to 2014. Mercedes who were 2nd in the Championship in 2013 after a major regulation change in 2014 turned out to dominate the next 8 years winning 8 championships in a row, a period in which no major changes in regulations were made. The regulation change in 2013-2014 was the introduction of the turbo-hybrid v6 engine. This regulation in racing standards was a major change and Redbull could not adapt to the new changes, they won only 3 races in 2014 compared to 13 wins in 2013, that is the year before the regulation changes. These examples illustrate a pattern that held consistent throughout the history of the sport. 

It was exciting to find my initial observation being confirmed by my investigation with the application of a program I had learned to put to use. 

The rest of the data can be seen in the files in my repository. There is a lot more analysis that can be done with the data sets above, but primarily I have used the data to understand how the regulation changes affect the competition of the sport. 
