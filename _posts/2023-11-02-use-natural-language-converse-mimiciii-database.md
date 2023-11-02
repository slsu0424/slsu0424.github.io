---
layout: post
title:  "Interact with the MIMIC-III database without writing SQL"
author: sandy
categories: [ ChatGPT, NLP, tutorial ]
image: assets/images/2023-11/mimic_resize.png
---
If someone were to tell me that I would someday be able to query a database without writing SQL statements, I would have said its impossible.  However, thanks to LLMs, a user can now dialogue with a database using natural language.  This is an powerful capability that will be a game-changer for anyone that works with data, especially in healthcare.

This tutorial continues on from the previous tutorial on using LLMs with Langchain.  We will explore using LLMs and LangChain to interact with MIMIC-III (critical care data) stored in a SQL Database. 

Pre-requisites:
1. Knowledge of OpenAI
2. Knowledge of Langchain
3. Knowledge of Azure services

Environment:
1. Python: 3.9.0
2. OS: MacOS 11.7.3
3. DB: Azure SQL Database
4. DB Driver: ODBC driver 18 for SQL Server


All resources can be found [here](https://github.com/slsu0424/langchain-sql-public).

## Load data into Azure SQL DB
[MIMIC-III](https://www.nature.com/articles/sdata201635) is a publicly available database comprising of de-identified data for > 40,000 critical care patients who stayed at the Beth Israel Deaconess Medical Center between 2001 and 2012.  

Download the database [files](https://physionet.org/content/mimiciii-demo/1.4/), and load the [ADMISSIONS]() table into Azure SQL DB. 

## Connect to Azure SQL DB
Next, we connect python to Azure SQL DB.  This [tutorial](https://learn.microsoft.com/en-us/azure/azure-sql/database/connect-query-python?view=azuresql) provides more details on the setup.  As there are known issues with the ODBC Driver on MacOS, be sure to follow this [guide](https://learn.microsoft.com/en-us/sql/connect/odbc/linux-mac/known-issues-in-this-version-of-the-driver?view=sql-server-ver16) if errors are encountered.  

In my own experience, I had to make the following changes via Terminal:

```bash
brew install openssl@1.1

rm -rf $(brew --prefix)/opt/openssl
version=$(ls $(brew --prefix)/Cellar/openssl@1.1 | grep "1.1")
n -s $(brew --prefix)/Cellar/openssl@1.1/$version $(brew --prefix)/opt/openssl
```

Get the [odbc connection string](https://azurelessons.com/azure-sql-database-connection-string/) and create an instance of the SQL database.
```python
# connect python to sql server
conn_str = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver={driver}"

try:
    # create instance of sql database from a given database uri
    db = SQLDatabase.from_uri(conn_str)
```
## Set the OpenAI API Key
Navigate to [OpenAI](https://platform.openai.com/) to obtain your API key.  This will be needed to authenticate requests to the API.

```python
# set API keys to authenticate requests to the API
OPENAI_API_KEY = "<KEY>"
```

## Set up SQL Database Agent
LangChain provides an agent that allows the user to interact with SQL databases.  Below are the steps to initialize it:

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
Chain execution:
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

Response:
```

```
What's interesting here is that the chain exposes the process by which the LLM "thinks through" how to answer the question.  The LLM recognizes that it has to create the Admissions table in order to aggregate results list of tables available in the database, and generates SQL statements to do so.  The next statement performs a COUNT on the rows, and returns the result in natural language.  

Without having the user write explicit SQL statements, the LLM understands the user's intent, and generates SQL statements dynamically.

We can also compare the LLM response to what would normally be generated by [SQL queries]().


## Conclusion
In this tutorial, we explored two examples of how to use LLMs to interact with healthcare data.  Using the Langchain framework, we enabled the OpenAI model to interact with the database, enablinWe looked at how LLMs can be used a SQL Database to send queries using natural language.  This is a very powerful use case that  by which users can interact with various kinds of healthcare data, without having to write code.  


## References
+ <https://www.ncbi.nlm.nih.gov/books/NBK519025/>
+ <https://github.com/hwchase17/chat-your-data/blob/master/blogpost.md>
+ <https://python.langchain.com/>
+ <https://medium.com/@kbdhunga/enhancing-conversational-ai-the-power-of-langchains-question-answer-framework-4974e1cab3cf>
+ <https://physionet.org/content/mimiciii-demo/1.4/>