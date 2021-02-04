# Music Streaming App Data Modeling 

>The goal of this project is to model data on user activity of a music streaming app, Sparkify in a NoSQL database, 
Apache Cassandra for testing query performance.

## Table of contents

* [General info](#general-info)
* [Screenshots](#screenshots)
* [Technologies and Libraries](#technologies-and-libraries)
* [Project Setup](#project-setup)
* [Quality Assurance](#quality-assurance)
* [Contact](#contact)

## General info

The project's main aim is to help the analysis team test the query performance for data modeled from a CSV file on
the music streaming app. The analyis team is interesting in understanding what songs users are listening to. An ETL
pipeline is created to transfer data stored in a directory of CSV files `event-datafile-new.csv` to an Apache Cassandra 
database.

## Screenshots

![event_datafile_new csv screenshot](https://user-images.githubusercontent.com/76578061/106852203-e9d19c80-6674-11eb-8b90-1f0ec15c7ba9.png)

*Fig. 1: Denormalized data of `event_datafile_new.csv` for data modeling 

## Technologies and Libraries

* Python - version 3.0
* jupyterlap - version 1.0.9
* Pandas Library
* Cassandra Library
* RegEx Library
* OS Library
* GLOB Library
* Numpy Library
* JSON Library
* CSV Library

## Project Setup

The project is divided into two main parts: Part I involves the development of the ETL pipeline for pre-processing files prior to creating 
data modeling. Part II involves creating tables on Apache Cassandra to load the processed data unto for quering. We shall look at both in detail.

### 1. Part I: ETL Pipeline building

In this part, we pre-processed the files for data modeling with Apache Cassandra by first building an ETL pipeline through iterating throug each 
event file in `event-data` to process and create a new CSV file in Python.

### Datasets

This project has just one dataset - `event-data` - which is a directory of CSV files partitioned by date. Below are examples filepaths to two files 
in the dataset: 

`event_data/2018-11-08-events.csv
event_data/2018-11-09-events.csv`


### 2. Part II: Data Modeling with Apache Cassandra

Here, we designed tables to answer queries given by the analysis team. There are 3 queries with each its own unique table for modeling data for the 
specific query. 

#### Query 1 : Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4

We modeled our data for this query by first creating a table using a `CREATE` statement with 2 **PRIMARY KEYS** - `sessionid` and `iteminsession` - 
to partition the data. We then loaded the data  unto the cassandra table using an `INSERT` statement.

##### Code Examples

`query = "CREATE TABLE IF NOT EXISTS music_app_history"
query = query + "(sessionid int, iteminsession int, artist text, song text, length float, PRIMARY KEY (sessionid, iteminSession))"
try:
    session.execute(query)
except Exception as e:
    print(e)`
    
 `query = "INSERT INTO music_app_history (sessionid, iteminSession, artist, song, length)"
        query = query + "VALUES (%s, %s, %s, %s, %s)"
        session.execute(query, (int(line[8]), int(line[3]), line[0], line[9], float(line[5])))`

#### Query 2 : Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182

Following the same logic as query 1 above, we created a table but this time around with 2 **PARTITIONING KEYS** - `userid` and `sessionid` - as primary keys
for distributing data across nodes in cassandra and 1 **CLUSTERING KEY** - `iteminsession` - as clustering column for sorting data within te partition.

##### Code Example

`query1 = "CREATE TABLE IF NOT EXISTS music_app_history1"
query1 = query1 + "(userid int, sessionid int, iteminsession int, artist text, song text, firstname text, lastname text, \
                    PRIMARY KEY ((userid, sessionid), iteminsession))"`
                    
 ` query1 = "INSERT INTO music_app_history1 (userid, sessionid, iteminsession, artist, song, firstname, lastname)" 
        query1 = query1 + "VALUES (%s, %s, %s, %s, %s, %s, %s)"
        session.execute(query1, (int(line[10]), int(line[8]), int(line[3]), line[0], line[9], line[1], line[4]))`
        
#### Query 3 :  Give me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'

Lastly, we modeled data for query 3 by creating a table with just 2 **PARTITIONING KEYS** - `song` and `userid`. 

##### Code Example

`query2 = "CREATE TABLE IF NOT EXISTS music_app_history2"
query2 = query2 + "(userid int, firstname text, lastname text, song text, PRIMARY KEY (song, userid))"
try:
    session.execute(query2)
except Exception as e:
    print(e)`
    
` query2 = "INSERT INTO music_app_history2 (userid, firstname, lastname, song)"
        query2 = query2 + "VALUES (%s, %s, %s, %s)"
        session.execute(query2, (int(line[10]), line[1], line[4], line[9]))`

### Quality Assurance

We cannot complete this project until, we are completely certain that we are getting right results; that is why we must test the performance
of our query using `SELECT` statements. After testing all 3 queries, we got the right results thus validating the workability our code.

#### Code Example and Output

##### Query 1

`
query = "CREATE TABLE IF NOT EXISTS music_app_history"
query = query + "(sessionid int, iteminsession int, artist text, song text, length float, PRIMARY KEY (sessionid, iteminSession))"
try:
    session.execute(query)
except Exception as e:
    print(e)
`  

![select_statement_query](https://user-images.githubusercontent.com/76578061/106245446-b05ae600-61c9-11eb-88cd-101692d4e50a.png)

##### Query 2

`
query1 = "SELECT artist, song, firstname, lastname FROM music_app_history1 WHERE userid = 10 AND sessionId = 182"
try:
    rows = session.execute(query1)
except Exception as e:
    print(e)
for row in rows:
    print (row.artist, row.song, row.firstname, row.lastname)
`
    
![select_statement_query1](https://user-images.githubusercontent.com/76578061/106245707-24958980-61ca-11eb-8010-617e54742b1b.png)

##### Query 3

`
query2 = "SELECT firstname, lastname FROM music_app_history2 WHERE song = 'All Hands Against His Own'"
try:
    rows = session.execute(query2)
except Exception as e:
    print(e)  
for row in rows:
    print (row.firstname, row.lastname)
`
    
![select_statement_query2](https://user-images.githubusercontent.com/76578061/106245873-776f4100-61ca-11eb-9cf6-458e92b6a513.png)

## Contact
Created by [@peterndiforchu](https://www.linkedin.com/in/peter-ndiforchu-0b8986129) - feel free to contact me!
