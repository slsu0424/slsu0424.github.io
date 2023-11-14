---
layout: post
title:  "Advancing analysis of FDA Adverse Events using LLMs"
author: sandy
categories: [ langChain, ChatGPT, python, tutorial ]
#image: assets/images/2023-10/meds.jpg
image: assets/images/2023-10/shutterstock_2269334607_license_resize.png
---
Patient safety continues to be an issue in 2023, with nearly [1 in 4 patients](https://www.nbcnews.com/health/health-news/nearly-1-4-us-hospital-patients-experience-harmful-event-study-finds-rcna65119) experiencing an adverse event upon admission.  Up to 40% of adverse events were related to medications given in the hospital, underscoring the importance of post-market drug surveillance and appropriate bedside staffing.  

For the former, consumers and healthcare professionals are encouraged to report adverse events to the FDA.  The information is captured in the [FDA Adverse Events Reporting System](https://www.fda.gov/drugs/surveillance/questions-and-answers-fdas-adverse-event-reporting-system-faers), which is used to improve product safety and protect public health.  Over the years, [NLP techniques](https://www.lifescienceleader.com/doc/how-the-fda-views-natural-language-processing-0001) have been used on FDA data to improve case analysis, such as identifying causal relationships between products and adverse events.   AI continues to play an important role at the FDA, including this latest initiative to apply [LLMs](https://www.fda.gov/about-fda/nctr-research-focus-areas/bertox-initiative) to improve efficiency and accuracy.  

From a technical lens, this was a topic I explored in my September [talk](https://www.meetup.com/cloud-data-driven/events/294617896/) on LLMs.  I covered 2 examples of using LLMs with LangChain, and LLMs with SQL databases, for healthcare use cases.  

This tutorial will demonstrate using LLMs and LangChain to quickly summarize an adverse events report.

Pre-requisites:
1. Basic knowledge of OpenAI

Environment:
1. OS: MacOS 12.7.1
2. Python: 3.11
3. VS Code: 1.84.1

All resources can be found [here](https://github.com/slsu0424/langchain-ade-public).


## Generate an ADE report
Individual Case Reports for adverse events are typically requested through the <a href="https://www.fda.gov/drugs/questions-and-answers-fdas-adverse-event-reporting-system-faers/instructions-requesting-individual-case-reports" target="_blank">FDA</a>.  

A sample report would look like this:

<p style="text-align: center;">
<a href="https://www.researchgate.net/publication/271223596_Automatically_Recognizing_Medication_and_Adverse_Event_Information_From_Food_and_Drug_Administration's_Adverse_Event_Reporting_System_Narratives?_tp=eyJjb250ZXh0Ijp7ImZpcnN0UGFnZSI6Il9kaXJlY3QiLCJwYWdlIjoiX2RpcmVjdCJ9fQ" target="_blank">
  <img src="/assets/images/2023-10/sample-AERS-Report.png" alt="aers" width="750" height="670" alig>
</a>
</p>

For the purpose of this tutorial, I decided to use OpenAI's <a href="https://chat.openai.com/" target="_blank">ChatGPT</a> (GPT-3.5) to generate a synthetic adverse events report.  I chose warfarin, as it is in the class of drugs that have resulted in <a href="https://www.ncbi.nlm.nih.gov/books/NBK519025/" target="_blank">serious adverse drug reactions</a>.

These were the series of prompts I used to generate the final [report](https://github.com/slsu0424/langchain-ade-public/blob/main/ade.pdf):

"Create an adverse event report related to warfarin.  Limit to 250 words."  
"Remove the Reporting Authority section"  
"expand to 500 words"  
"Include the Reporting Authority section"  

A snippet of the document is below:

>On October 20, 2023, at 09:30 AM, the patient, John Doe, experienced a significant adverse event related to the anticoagulant medication warfarin. Mr. Doe, a 68-year-old male with a history of atrial fibrillation, had been taking warfarin (5 mg daily) for the past three years as prescribed by his cardiologist.


## Q&A application overview
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


## Set the OpenAI API Key
Navigate to [OpenAI](https://platform.openai.com/) to obtain the API key.  This will be needed to authenticate requests to the API.

```python
# set API keys to authenticate requests to the API
API_KEY = "<API_KEY>"

os.environ["OPENAI_API_KEY"] = API_KEY
```


## Use LangChain to load documents into a vector store
[LangChain](https://docs.langchain.com/docs/) is a framework for developing applications powered by LLMs.  The main idea is that developers can "chain" different components around an LLM to create more powerful use cases.  

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

## Set up Streamlit app to query document
With [Streamlit](https://streamlit.io/), we set up a simple web app to allow users to ask questions of the ADE document.  

```python
# Display the page title and the text box for the user to ask the question
st.title('🦜 LangChain: Chat with Adverse Events Report')
prompt = st.text_input("Enter your question")
```


## Query the vector store
When a user passes in a question, the store is queried to retrieve the data that is 'most similar' to the embedded query.

```python
response = index.query(
  llm=ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0.2), 
  question = prompt, 
  chain_type = 'stuff')
```

Under the hood, we pass in the **ChatOpenAI** model (gpt-3.5-turbo) and set the **temperature**.  The temperature controls the randomness of the output generated (closer to 1 will generate a more creative response).  **chain_type = 'stuff'** combines the question and relevant document chunks into a single prompt to pass to the LLM.

This visual shows the overall workflow:

<a href="https://python.langchain.com/docs/modules/data_connection/vectorstores/#:~:text=One%20of%20the%20most%20common%20ways%20to%20store,that%20are%20%27most%20similar%27%20to%20the%20embedded%20query">
  <img src="/assets/images/2023-10/langchain3.png" alt="langchain3" width="750" height="311">
</a>


## Let's ask some questions
With the Streamlit app loaded, we can ask a question of the adverse events document:





## Conclusion
In this tutorial, we explored using LLMs and Langchain to retrieve information from an adverse events report.  Langchain is a framework that enables building of LLM-based applications, such as a chatbot.  We then used Streamlit as a web app front-end to ask questions of the document.  This is a powerful use case by which users can dialogue with a healthcare document using natural language.

I would be interested to explore with different types of documents (i.e., Standards of Care guidelines for [Diabetes](https://diabetesjournals.org/care/issue/46/Supplement_1)), and building out an advanced UI with Streamlit.  I will explore these in a future post.


## References
+ <https://www.nature.com/articles/s41591-023-02448-8>
+ <https://chat.openai.com/>
+ <https://www.ncbi.nlm.nih.gov/books/NBK519025/>
+ <https://github.com/hwchase17/chat-your-data/blob/master/blogpost.md>
+ <https://platform.openai.com/>
+ <https://python.langchain.com/>
+ <https://medium.com/@kbdhunga/enhancing-conversational-ai-the-power-of-langchains-question-answer-framework-4974e1cab3cf>
+ <https://streamlit.io/>