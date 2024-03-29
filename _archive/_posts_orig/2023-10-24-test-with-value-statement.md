---
layout: post
title:  "Chatting with healthcare data in natural language"
author: sandy
categories: [ ChatGPT, NLP, tutorial ]
image: assets/images/2023-10/shutterstock_2188258735_license_resize.png
---
In September, I had the opportunity to present the 2nd iteration of my talk on Large Language Models (LLMs).  This time I had a chance to go a bit deeper into the demos that covered integrating LLMs with LangChain, and LLMs with SQL.  I have written this tutorial to accompany those demos.    

This tutorial will cover 2 examples:

1) Use LLMs with LangChain to chat with a healthcare document  
2) Use LLMs with SQL to chat with the MIMIC-III database

All resources can be found [here](https://github.com/slsu0424/pmc-patients).

## What's the value of chatting with data?

Healthcare is inundated with data - much of it found in unstructured EHR notes, radiology images, medical device readings, etc.  Focusing on the former, there is a wealth of knowledge buried in pages of text.  From a clinical perspective, how can relevant information be retrieved quickly without explicit programming?

While NLP techniques have been used [since the 1950s](https://en.wikipedia.org/wiki/Natural_language_processing) to process and generate text, the ability to generate text with human-like accuracy is extremely [powerful](https://news.mit.edu/2023/ai-models-astrocytes-role-brain-0815).  

What if you could easily chat with data using natural language?  What if the AI system could better understand your intent?

## Example 1: Use LLMs with LangChain to chat with a healthcare document 

#### Generate an ADE report

I used OpenAI's ChatGPT (GPT-3.5) to generate a synthetic adverse events report for warfarin.  I chose warfarin as it is in the class of drugs that have resulted in [serious adverse drug reactions](https://www.ncbi.nlm.nih.gov/books/NBK519025/).  Hence, there would be more internet information on warfarin that would be fed into the model.   

These were the series of prompts I used to generate the final output:

"Create an adverse event report related to warfarin.  Limit to 250 words."  
"Remove the Reporting Authority section"  
"expand to 500 words"  
"Include the Reporting Authority section"  

A snippet of the document is below:

>On October 20, 2023, at 09:30 AM, the patient, John Doe, experienced a significant adverse event related to the anticoagulant medication warfarin. Mr. Doe, a 68-year-old male with a history of atrial fibrillation, had been taking warfarin (5 mg daily) for the past three years as prescribed by his cardiologist.

*Note:* LLMs can generate [different answers for the same prompt](https://ai.stackexchange.com/questions/32385/why-do-language-models-produce-different-outputs-for-same-prompt) due to the stochastic nature of predicting the next word.  For example, I tried this single prompt:

"Create an adverse event report related to warfarin.  Limit to 500 words." 

which returned a very different [output]().

#### Use LangChain to load documents into a vector store

Langchain is a framework for developing applications powered by LLMs.  It was launched as an open source project in October 2022.  The main idea is that developers can "chain" different components around an LLM to create more powerful use cases.  

Hence, we can "chain" an LLM to another component, such as a document.

LangChain follows these standard steps for creating a Q&A application using text:

1. Load documents
2. Split documents into chunks
3. Create embedding vectors from chunks
4. Store vectors in vector database
5. Retrieve relevant documents from database
6. Pass relevant documents to LLM
7. LLM generates final answer

This workflow help illustrate the steps above:

[]!

LangChain has many different methods to load documents.  We easily use the **PyPDFLoader** to load in the PDF document and create a vector representation using the **VectorStoreIndexCreator**:

```python
# Load PDF document
loaders = PyPDFLoader('/Users/sandysu/Documents/GitHub/OpenAI/docs/ADR11.pdf')

# Create a vector representation of the loaded document
index = VectorstoreIndexCreator().from_loaders([loaders])
```

This [article](https://medium.com/@kbdhunga/enhancing-conversational-ai-the-power-of-langchains-question-answer-framework-4974e1cab3cf) gives a nice explanation of the VectorStoreIndexCreator class.  Basically, this class handles steps 2-4 above (chunking, embedding, and storage). 

The default settings include:

- Chunking - [RecursiveCharacterTextSplitter()](https://api.python.langchain.com/en/latest/text_splitter/langchain.text_splitter.RecursiveCharacterTextSplitter.html) - divides text at specific characters
- Embedding - [OpenAIEmbeddings](https://api.python.langchain.com/en/latest/embeddings/langchain.embeddings.openai.OpenAIEmbeddings.html) - generates embeddings using the OpenAI API
- Storage - [Chroma](https://www.trychroma.com/) - an open-source vector store

#### Query the vector store

The [query()](https://api.python.langchain.com/en/latest/_modules/langchain/indexes/vectorstore.html#VectorStoreIndexWrapper.query) method is used to interact with the vector store.  Thus, when a user passes in a question, the store is queried to retrieve the data that is 'most similar' to the embedded query.

https://python.langchain.com/docs/modules/data_connection/vectorstores/#:~:text=One%20of%20the%20most%20common%20ways%20to%20store,that%20are%20%27most%20similar%27%20to%20the%20embedded%20query.

```python
if prompt:
 # Get the resonse from LLM
 # We pass the model name (3.5) and the temperature (Closer to 1 means creative resonse)
 # stuff chain type sends all the relevant text chunks from the document to LLM

 response = index.query(llm=OpenAI(model_name="gpt-3.5-turbo", temperature=0.2), question = prompt, chain_type = 'stuff')
```

By chain_type = 'stuff', this combines multiple input documents into a single prompt passed to an LLM.

https://python.langchain.com/docs/modules/chains/document/stuff

#### Set up streamlit app to query document

Next, we setup an external UI to allow users to ask questions of the ADE document.  

```python
# Display the page title and the text box for the user to ask the question
st.title('🦜 Query your PDF document ')
prompt = st.text_input("Enter your question to query your PDF documents")
```
#### Test response


## Example 2: Use LLMs with SQL to chat with the MIMIC-III database

Since we saw the limitations with one-hot encoding, a better approach would be to assign each word a unique integer.  The integer encoding for a specific word remains the same across all documents, so this will reduce the size of the corpus to unique words. 

To do this, Keras (neural network library) provides a handy **Tokenizer() API** that can handle multiple documents.  For a deeper understanding of its implementation, see this [tutorial](https://machinelearningmastery.com/prepare-text-data-deep-learning-keras).


![](/assets/images/2023-09/output2.png)

## Conclusion

In this tutorial, we explored how to create word embeddings from scratch, using a neural network to perform a classification task.  By taking sample text from PubMed patient summaries, we were able to train a neural network to classify patients who had COVID-19 and those that did not.  In doing so, we were also able to train the embeddings, such that words with similar meanings were visually placed closer together.  

We can boost the performance of the training accuracy by adding in a different layer, such as a convolution layer.  I will explore these in future posts.


## References
+ <https://medium.com/technology-hits/overview-of-langchain-9f6362707cd0>
+ <https://towardsai.net/p/machine-learning/chat-with-your-healthcare-documents-build-a-chatbot-with-chatgpt-and-langchain>

https://www.techsmartfuture.com/chunking-natural-language-processing/
https://towardsdatascience.com/how-to-chunk-text-data-a-comparative-analysis-3858c4a0997a
https://api.python.langchain.com/en/latest/indexes/langchain.indexes.vectorstore.VectorstoreIndexCreator.html

https://www.wwt.com/blog/a-brief-history-of-nlp
https://dspace.mit.edu/bitstream/handle/1721.1/150502/2023_NLP_JPM.pdf?sequence=1&isAllowed=y

https://www.analyticsvidhya.com/blog/2022/07/the-evolution-of-nlp-from-1950-to-2022/