---
layout: post
title:  "Chat with the MIMIC-III database using natural language"
author: sandy
categories: [ ChatGPT, NLP, tutorial ]
image: assets/images/2023-10/shutterstock_2188258735_license_resize.png
---
What is the value of LLMs in healthcare, specificially when it comes to chat capabilities?  After all, NLP has been around for a long time.  What makes GenAI different?  

https://www.nature.com/articles/s41591-023-02448-8

This was a topic I explored in my September [tech talk] on LLMs, where I had a chance to walk through 2 examples that covered integrating LLMs with LangChain and SQL.  This tutorial explores each example in more detail. 

Pre-requisites:
1. Knowledge of OpenAI
2. Knowledge of Langchain
3. Knowledge of Azure services

Environment:
1. Python: 3.9.0
2. OS: MacOS 11.7.3
3. DB: Azure SQL Database
4. DB Driver: ODBC driver 18 for SQL Server


All resources can be found [here](https://github.com/slsu0424/pmc-patients).

## Load data into Azure SQL DB
The [MIMIC-III](https://physionet.org/content/mimiciii-demo/1.4/) is a publicly available database comprising of de-identified data for > 40,000 CCU patients who stayed at the Beth Israel Deaconess Medical Center between 2001 and 2012.  

Download the database (.csv files), and load the [ADMISSIONS]() table into Azure SQL DB. 

## Connect to Azure SQL DB
We next connect python to Azure SQL DB.  This tutorial provides more details on setting this up.  As there are known issues with the ODBC Driver on MacOS, be sure to follow this [guide](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/known-issues-in-this-version-of-the-driver?view=sql-server-ver16) if errors are encountered.  

In my own experience, I had to make the following changes to my system:

1. brew install openssl@1.1
2. run symlink commands

Get the database connection string variables and create a connection:
```python
# connect python to sql server
conn_str = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver={driver}"

try:
    # create instance of sql database from a given database uri
    db = SQLDatabase.from_uri(conn_str)
```
## Set up SQL Database Agent
LangChain provides an agent that allows the user to interact with SQL databases.  Below are the steps to initialize the agent:

```python
# create new llm model
llm = ChatOpenAI(
    temperature=0, 
    openai_api_key=OPENAI_API_KEY, 
    model_name='gpt-3.5-turbo')

# toolkit for interacting with sql databases
toolkit = SQLDatabaseToolkit(
    db=db,
    llm=llm)

# initialize agent
agent_executor = create_sql_agent(
    llm=llm,
    toolkit=toolkit,
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTION
    )
 ```
 *Note*: The **ChatOpenAI** class shares many similar properties to the **OpenAI** class.  However, it contains more chat-related methods and is better suited for building chatbots.

## Ask questions in natural language
Let's run a simple query using a language prompt:
```python
# query 1
agent_executor.run("how many rows are there?")
```
Response:
```python
> Entering new AgentExecutor chain...

Invoking: `sql_db_list_tables` with `{}`


ADMISSIONS, BuildVersion, ErrorLog
Invoking: `sql_db_query` with `SELECT COUNT(*) FROM ADMISSIONS`


[(129,)]There are 129 rows in the ADMISSIONS table.

> Finished chain.
```
What's interesting here is that the chain exposes the process by which the LLM "thinks through" how to answer the query.  The LLM recognizes that it has to find the list of tables available in the database, and runs a COUNT statement to return the row sum.

Now, let's run a more complex query:
```python
# query 2
agent_executor.run("What is the frequency of ethnicities?")
```
Response:
```python
> Entering new AgentExecutor chain...

Invoking: `sql_db_list_tables` with `{}`


ADMISSIONS, BuildVersion, ErrorLog
Invoking: `sql_db_schema` with `ADMISSIONS`



CREATE TABLE [ADMISSIONS] (
	row_id INTEGER NOT NULL, 
	subject_id INTEGER NOT NULL, 
	hadm_id INTEGER NOT NULL, 
	admittime DATETIME2 NOT NULL, 
	dischtime DATETIME2 NOT NULL, 
	deathtime DATETIME2 NULL, 
	admission_type NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	admission_location NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	discharge_location NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	insurance NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	language NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
	religion NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
	marital_status NVARCHAR(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
	ethnicity NVARCHAR(100) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	edregtime DATETIME2 NULL, 
	edouttime DATETIME2 NULL, 
	diagnosis NVARCHAR(150) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	hospital_expire_flag NVARCHAR(150) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL, 
	has_chartevents_data NVARCHAR(150) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL
)

/*
3 rows from ADMISSIONS table:
row_id	subject_id	hadm_id	admittime	dischtime	deathtime	admission_type	admission_location	discharge_location	insurance	language	religion	marital_status	ethnicity	edregtime	edouttime	diagnosis	hospital_expire_flag	has_chartevents_data
12258	10006	142345	2164-10-23 21:09:00	2164-11-01 17:15:00	None	EMERGENCY	EMERGENCY ROOM ADMIT	HOME HEALTH CARE	Medicare	None	CATHOLIC	SEPARATED	BLACK/AFRICAN AMERICAN	2164-10-23 16:43:00	2164-10-23 23:00:00	SEPSIS	0	1
12263	10011	105331	2126-08-14 22:32:00	2126-08-28 18:59:00	2126-08-28 18:59:00	EMERGENCY	TRANSFER FROM HOSP/EXTRAM	DEAD/EXPIRED	Private	None	CATHOLIC	SINGLE	UNKNOWN/NOT SPECIFIED	None	None	HEPATITIS B	1	1
12265	10013	165520	2125-10-04 23:36:00	2125-10-07 15:13:00	2125-10-07 15:13:00	EMERGENCY	TRANSFER FROM HOSP/EXTRAM	DEAD/EXPIRED	Medicare	None	CATHOLIC	None	UNKNOWN/NOT SPECIFIED	None	None	SEPSIS	1	1
*/
Invoking: `sql_db_query` with `SELECT ethnicity, COUNT(*) AS frequency FROM ADMISSIONS GROUP BY ethnicity ORDER BY frequency DESC LIMIT 10`


Error: (pyodbc.ProgrammingError) ('42000', "[42000] [Microsoft][ODBC Driver 18 for SQL Server][SQL Server]Incorrect syntax near 'LIMIT'. (102) (SQLExecDirectW)")
[SQL: SELECT ethnicity, COUNT(*) AS frequency FROM ADMISSIONS GROUP BY ethnicity ORDER BY frequency DESC LIMIT 10]
(Background on this error at: https://sqlalche.me/e/20/f405)
Invoking: `sql_db_query_checker` with `SELECT ethnicity, COUNT(*) AS frequency FROM ADMISSIONS GROUP BY ethnicity ORDER BY frequency DESC OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY`


SELECT ethnicity, COUNT(*) AS frequency 
FROM ADMISSIONS 
GROUP BY ethnicity 
ORDER BY frequency DESC 
OFFSET 0 ROWS 
FETCH NEXT 10 ROWS ONLY
Invoking: `sql_db_query` with `SELECT ethnicity, COUNT(*) AS frequency FROM ADMISSIONS GROUP BY ethnicity ORDER BY frequency DESC OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY`


[('WHITE', 86), ('HISPANIC/LATINO - PUERTO RICAN', 15), ('UNKNOWN/NOT SPECIFIED', 11), ('BLACK/AFRICAN AMERICAN', 7), ('OTHER', 3), ('HISPANIC OR LATINO', 2), ('ASIAN', 2), ('AMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE', 2), ('UNABLE TO OBTAIN', 1)]The frequency of ethnicities in the database is as follows:

- WHITE: 86
- HISPANIC/LATINO - PUERTO RICAN: 15
- UNKNOWN/NOT SPECIFIED: 11
- BLACK/AFRICAN AMERICAN: 7
- OTHER: 3
- HISPANIC OR LATINO: 2
- ASIAN: 2
- AMERICAN INDIAN/ALASKA NATIVE FEDERALLY RECOGNIZED TRIBE: 2
- UNABLE TO OBTAIN: 1

> Finished chain.
```

We can compare the LLM response to what would normally be generated by [SQL queries]().


## Conclusion
In this tutorial, we explored two examples of how to use LLMs to interact with healthcare data.  First, we looked at integrating LLMs with Langchain to quickly build a chat-enabled application.  Second, we looked at how LLMs can be used a SQL Database to send queries using natural language.  These are powerful use cases by which users can interact with various kinds of healthcare data, without having to write code.  


## References
+ <https://www.ncbi.nlm.nih.gov/books/NBK519025/>
+ <https://github.com/hwchase17/chat-your-data/blob/master/blogpost.md>
+ <https://python.langchain.com/>
+ <https://medium.com/@kbdhunga/enhancing-conversational-ai-the-power-of-langchains-question-answer-framework-4974e1cab3cf>
+ <https://physionet.org/content/mimiciii-demo/1.4/>