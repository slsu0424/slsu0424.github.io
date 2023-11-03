---
layout: post
title:  "Interact with the MIMIC-III database without writing SQL"
author: sandy
categories: [ azure, langchain, ChatGPT, python, SQL, tutorial ]
image: assets/images/2023-11/mimic_resize.png
---
If someone were to tell me that I would someday be able to query a database without writing SQL statements, I would have said it's impossible.  However, thanks to LLMs, a user can now dialogue with a database using natural language.  This is an exciting and powerful new capability for anyone that works with healthcare data.

This tutorial continues on from the previous [tutorial](https://slsu0424.github.io/chat-adverse-events-report-natural-language/) on using LLMs with Langchain.  We will explore using LLMs and LangChain to interact with a SQL Database. 

Pre-requisites:
1. Knowledge of OpenAI
2. Knowledge of Langchain
3. Knowledge of Azure SQL Database

Environment:
1. Python: 3.9.0
2. OS: MacOS 11.7.3
3. DB: Azure SQL 12.0.2000.8
4. DB Driver: ODBC driver 18 for SQL Server


All resources can be found [here](https://github.com/slsu0424/langchain-sql-public).

## Load data into Azure SQL DB
[MIMIC-III](https://www.nature.com/articles/sdata201635) is a publicly available database comprising of de-identified data for > 40,000 critical care patients who stayed at the Beth Israel Deaconess Medical Center between 2001 and 2012.  

Download the database [files](https://physionet.org/content/mimiciii-demo/1.4/), and load the [ADMISSIONS]() table into Azure SQL DB. 

## Connect to Azure SQL DB
Next, we connect python to Azure SQL DB.  This [tutorial](https://learn.microsoft.com/en-us/azure/azure-sql/database/connect-query-python?view=azuresql) provides more details on the setup.  As there are known issues with the ODBC Driver on MacOS, be sure to follow this [guide](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/known-issues-in-this-version-of-the-driver?view=sql-server-ver16) if errors are encountered.  

In my own experience, I had to make the following changes via Terminal:

```bash
$ brew install openssl@1.1
$ rm -rf $(brew --prefix)/opt/openssl version=$(ls $(brew --prefix)/Cellar/openssl@1.1 | grep "1.1")
$ ln -s $(brew --prefix)/Cellar/openssl@1.1/$version $(brew --prefix)/opt/openssl
```

Get the odbc connection string and create an [instance](https://api.python.langchain.com/en/latest/utilities/langchain.utilities.sql_database.SQLDatabase.html) of the SQL database:
```python
# connect python to sql server
conn_str = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver={driver}"

try:
    db = SQLDatabase.from_uri(conn_str)
```
## Set the OpenAI API Key
Navigate to [OpenAI](https://platform.openai.com/) to obtain the API key.  This will be needed to authenticate requests to the API.

```python
# set API keys to authenticate requests to the API
OPENAI_API_KEY = "<KEY>"
```

## Set up SQL Database Agent
LangChain provides an [agent](https://python.langchain.com/docs/integrations/toolkits/sql_database) that allows the user to interact with SQL databases.  Below are the steps to initialize it:

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
Let's run a query to aggregate some data:

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
...

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

What's interesting here is that the chain reveals the LLM's reasoning process.  For example, the LLM recognizes that it has to create the ADMISSIONS table in order to aggregate the results.  Without having the user write SQL statements, the LLM generates it dynamically and returns the result in natural language.  

We can also compare the LLM response to what would normally be generated by [SQL queries]().


## Conclusion
In this tutorial, we explored an example of how to use LLMs to interact with the MIMIC-III database.  Using the Langchain framework, we were able to integrate an OpenAI model with an Azure SQL Database.  This allows users to ask questions about MIMIC-III Admissions data in natural language.  This is a very powerful use case in which users can interact with healthcare data, without having to write code.  


## References
+ <https://www.nature.com/articles/sdata201635>
+ <https://physionet.org/content/mimiciii-demo/1.4/>
+ <https://learn.microsoft.com/en-us/azure/azure-sql/database/connect-query-python>
+ <https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/known-issues-in-this-version-of-the-driver>
+ <https://api.python.langchain.com/en/latest/utilities/langchain.utilities.sql_database.SQLDatabase.html>
+ <https://platform.openai.com/>
+ <https://python.langchain.com/docs/integrations/toolkits/sql_database>