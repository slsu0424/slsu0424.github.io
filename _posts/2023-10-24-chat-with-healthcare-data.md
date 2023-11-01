---
layout: post
title:  "Chat with healthcare data in natural language"
author: sandy
categories: [ ChatGPT, NLP, tutorial ]
image: assets/images/2023-10/shutterstock_2188258735_license_resize.png
---
What is value LLMs in healthcare, specificially when it comes to chat capabilities?  After all, NLP has been around for a long time.  What makes GenAI different?  

https://www.nature.com/articles/s41591-023-02448-8

This was a topic I also explored in my September [tech talk] on LLMs, where I had a chance to go deeper into the demos that covered integrating LLMs with LangChain and SQL.  This tutorial explores those demos in more detail. 


Pre-requisites:
1. Intermediate knowledge of OpenAI
2. Azure SQL DB
3. Python 3.11


## Example 1: LLMs and LangChain to chat with a healthcare document 

All resources can be found [here](https://github.com/slsu0424/pmc-patients).

#### Generate an ADE report

I used OpenAI's ChatGPT (GPT-3.5) to generate a synthetic adverse events report for warfarin.  I chose warfarin as it is in the class of drugs that have resulted in [serious adverse drug reactions](https://www.ncbi.nlm.nih.gov/books/NBK519025/).

These were the series of prompts I used to generate the final [output]():

"Create an adverse event report related to warfarin.  Limit to 250 words."  
"Remove the Reporting Authority section"  
"expand to 500 words"  
"Include the Reporting Authority section"  

A snippet of the document is below:

>On October 20, 2023, at 09:30 AM, the patient, John Doe, experienced a significant adverse event related to the anticoagulant medication warfarin. Mr. Doe, a 68-year-old male with a history of atrial fibrillation, had been taking warfarin (5 mg daily) for the past three years as prescribed by his cardiologist.

#### Q&A application overview
This LangChain [blog post](https://github.com/hwchase17/chat-your-data/blob/master/blogpost.md) provides a high-level overview for building a text-based Q&A application.  

The main steps include:

1. Load documents
2. Split documents into chunks
3. Create embedding vectors from chunks
4. Store vectors in vector database
5. Retrieve relevant documents from database
6. Pass relevant documents to LLM
7. LLM generates final answer

Ingest Data:

![langchain1](/assets/images/2023-10/langchain1.png)

Query Data:

![langchain2](/assets/images/2023-10/langchain2.png)


#### Use LangChain to load documents into a vector store
[Langchain](https://docs.langchain.com/docs/) is a framework for developing applications powered by LLMs.  The main idea is that developers can "chain" different components around an LLM to create more powerful use cases.  

Hence, we can "chain" an LLM to another component, such as a document.

LangChain has many different methods to load documents.  **PyPDFLoader** is used to load in a PDF document and create a vector representation using the **VectorStoreIndexCreator**:

```python
# Load PDF document
loaders = PyPDFLoader('/Users/sandysu/Documents/GitHub/OpenAI/docs/ADR11.pdf')

# Create a vector representation of the loaded document
index = VectorstoreIndexCreator().from_loaders([loaders])
```

The VectorStoreIndexCreator handles steps 2-4 above (chunking, embedding, and storage).  This [article](https://medium.com/@kbdhunga/enhancing-conversational-ai-the-power-of-langchains-question-answer-framework-4974e1cab3cf) goes into more detail about the class, including customization.  

The default settings to note are: 

- Chunking - uses **RecursiveCharacterTextSplitter()** to divide text at specific characters
- Embedding - uses **OpenAIEmbeddings**  to generate embeddings
- Storage - embeddings are stored in **Chroma**, an open-source vector store 

#### Set up Streamlit app to query document
Next, we set up a simple UI to allow users to ask questions of the ADE document.  

```python
# Display the page title and the text box for the user to ask the question
st.title('🦜 Query your PDF document')
prompt = st.text_input("Enter your question to query your PDF documents")
```

#### Query the vector store
When a user passes in a question, the store is queried to retrieve the data that is 'most similar' to the embedded query.

```python
response = index.query(
  llm=OpenAI(model_name="gpt-3.5-turbo", temperature=0.2), 
  question = prompt, 
  chain_type = 'stuff')
```

Under the hood, we pass in the OpenAI model (gpt-3.5-turbo), and set the **temperature**.  The temperature controls the randomness of the output generated (closer to 1 will generate a more creative response).  For **chain_type = 'stuff'**, this will combine the question and relevant document chunks into a single prompt to pass to the LLM.

This visual shows the overall workflow:

<a href="https://python.langchain.com/docs/modules/data_connection/vectorstores/#:~:text=One%20of%20the%20most%20common%20ways%20to%20store,that%20are%20%27most%20similar%27%20to%20the%20embedded%20query">
  <img src="/assets/images/2023-10/langchain3.png" alt="langchain3" width="750" height="311">
</a>


## Let's ask some questions
With the Streamlit app loaded, we can ask a question of the ADE document:



## Example 2: LLMs and SQL to chat with a healthcare database
All resources can be found [here](https://github.com/slsu0424/pmc-patients).

#### Load data into Azure SQL DB
[MIMIC-III](https://physionet.org/content/mimiciii-demo/1.4/) is a publicly available database comprising of de-identified data for > 40,000 CCU patients who stayed at the Beth Israel Deaconess Medical Center between 2001 and 2012.  

Downloaded the database (.csv files), and load the [ADMISSIONS]() table into Azure SQL DB. 

#### Connect to Azure SQL DB
We connect python to Azure SQL DB via pyodbc.  Next, get the database connection string variables and create a connection:
```python
# connect python to sql server
conn_str = f"mssql+pyodbc://{username}:{password}@{server}/{database}?driver={driver}"
```
#### Set up SQL Database Agent
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
 *Note*: The **ChatOpenAI** class shares many similar properties to **OpenAI**.  However, it contains more chat-related methods and is better suited for building chatbots.

#### Ask queries in natural language
Let's run a simple query:
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
What I find interesting here is that the chain exposes the process by which the LLM "thinks through" how to answer the query.  The LLM recognizes that it has to find the list of tables available in the database, and runs a COUNT statement to return the row sum.

We can compare the LLMs response to what would normally be generated by [SQL queries]().


## Conclusion
In this tutorial, we explored two examples of how to use LLMs to interact with healthcare data.  First, we looked at integrating LLMs with Langchain to quickly build a chat-enabled application.  Second, we looked at how LLMs can be used a SQL Database to send queries using natural language.  These are powerful use cases by which users can interact with various kinds of healthcare data, without having to write code.  


## References
+ <https://www.ncbi.nlm.nih.gov/books/NBK519025/>
+ <https://github.com/hwchase17/chat-your-data/blob/master/blogpost.md>
+ <https://python.langchain.com/>
+ <https://medium.com/@kbdhunga/enhancing-conversational-ai-the-power-of-langchains-question-answer-framework-4974e1cab3cf>
+ <https://physionet.org/content/mimiciii-demo/1.4/>