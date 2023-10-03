---
layout: post
title:  "Create word embeddings from PubMed patient summaries"
author: sandy
categories: [ PubMed, NLP, tutorial, SQL, Azure ]
image: assets/images/2023-09/OIP_resize.jpg
---
Building upon my previous [tutorial](https://slsu0424.github.io/encoding-pubmed-abstracts-for-nlp-tasks/) on one-hot encoding, this tutorial will review the concept of word embeddings and apply this to real-life data.  

For our example, we will extract patient summaries (documents) from PubMed.  From these patient summaries, we will label those that had COVID-19, and those that did not.  Portions of text from each document will be selected to create word embeddings.  These embeddings will be "trained" as part a neural network model to perform a classification task (COVID-19 vs. non-COVID-19).  By training the word embeddings, the computer will learn if there are any meaningful relationships between the words in the text. 

## A primer - Word Embeddings

Word Embeddings were a bit of a complex concept to grasp, until I got into the weeds of building one.  I've come to learn they are an important concept in deep learning, for the very reason that semantic meaning of words can be approximated mathematically.

There are a number of techniques available to build a word embedding.  As a quick note, LLMs use word embeddings, but the technique used to build them are proprietary.  

To start, I wanted to backtrack the origins of LLMs, which are a type of neural network.  A neural network is a type of machine learning approach that attempts to mimic the way the brain works (biological neural network).  This approach has been shown to perform better on NLP tasks than previous methods.  For a great overview of NLP, check out this [guide](https://www.deeplearning.ai/resources/natural-language-processing).

The goal of NLP is to enable computers to "understand" natural language in order to perform some task, such as sentiment analysis.  In order to do so, natural language (text format) has to be converted (encode) into a numerical format.

## Let's get data

I selected a dataset of ~167K PubMed patient summaries via [HuggingFace](https://huggingface.co/datasets/zhengyun21/PMC-Patients/tree/main).  To demonstrate a low-code approach, SQL Server and Azure Data Studio will be used.

This dataset will be loaded into a SQL Server on a Mac.  For further instructions on how to set this up, check out this [tutorial](https://builtin.com/software-engineering-perspectives/sql-server-management-studio-mac).  

*Note*: this requires Docker to be run on your desktop.  

SQL server can then be started via the terminal giving the username and password:

```
$ mssql -u <sql server username> -p <sql server password>
```

Azure Data Studio will be used to access and query the data.  To connect to a SQL Server using Azure Data Studio, review this [tutorial](https://www.sqlshack.com/sql-server-data-import-using-azure-data-studio/).  

Attached are screenshots to load the dataset correctly.

1. Create a new connection:
![AzureDataStudio](/assets/images/2023-09/azstudio_setup1.png)

2. Use the Import flat file wizard:
![AzureDataStudio](/assets/images/2023-09/azstudio_setup2.png)

3. Modify the columns as follows:
![AzureDataStudio](/assets/images/2023-09/azstudio_setup3.png)

We can run a few queries to inspect the data, and create the desired dataset.

```
SELECT TOP (100) [patient_id]
  ,[patient_uid]
  ,[PMID]
  ,[file_path]
  ,[title]
  ,[patient]
  ,[age]
  ,[gender]
  ,[similar_patients]
  ,[relevant_articles]
FROM [test].[dbo].[PMC-Patients]
```

```
SELECT count(*)
FROM [test].[dbo].[PMC-Patients] WHERE patient LIKE '%covid-19%'
```

![](/assets/images/2023-09/azstudio_query1.png){width=1742px}(/assets/images/2023-09/azstudio_query1.png)


We will execute the first query to get the top 100 records.  The results can be exported from Azure Data Studio as a .csv.

While this represents a bit of a longer approach to loading and manipulating the dataset, you may wish to explore other approaches:
1. Load dataset into pandas dataframe
2. Load dataset into a cloud database, such as Azure SQL Database.  Please keep in mind there are costs associated with running the database in the cloud, as well as querying costs.

## Define class labels

Since this is a binary classification task, we will label the dataset as:
- patients with COVID-19 = '1'
- patients without COVID-19 = '0'

We set up our python code as follows:

```python
labels = []

df['label'] = ''

for index, row in df.iterrows():
    if 'COVID-19' in row['patient']:
        row['label'] = '1'
    else:
        row['label'] = '0' 

    labels.append(row['label'])

# convert list to array
labels_arr = np.array(labels).astype(float)

print(labels_arr)
```

Output array:

```
[1. 0. 0. 0. 0. 0. 0. 0. 1. 1. 1. 0. 0. 0. 1. 0. 0. 0. 0. 0. 0. 1. 1. 0.
 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 1. 0. 0. 0. 0. 0. 0.
 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.
 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 1. 1. 0.
 0. 0. 1. 0.]
```

## Create corpus of documents

Now that we have our labeled dataset, we will create a corpus of documents.  We'll take the first 3 sentences from each document (100 PubMed articles).  The resulting corpus will look something like this:

```
[This 60-year-old male was hospitalized due to moderate ARDS from COVID-19 with symptoms of fever, dry cough, and dyspnea. We encountered several difficulties during physical therapy on the acute ward. First, any change of position or deep breathing triggered coughing attacks that induced oxygen desaturation and dyspnea
We describe the case of a 55-year-old male who presented to the emergency department via emergency medical services for the chief complaint of sudden onset shortness of breath that woke him from his sleep just prior to arrival. He reported three days of non-radiating lumbar back pain and two episodes of non-bloody emesis leading up to this event. His medical history included hypertension and type 2 diabetes mellitus
A 20-year-old Caucasian male (1.75 m tall and 76 kg (BMI 24.8)), was admitted to the medical department for persistent hyperpyrexia, severe sore throat, dyspnea, and impaired consciousness with stupor]

```


## Convert text to integers

As explored in the previous tutorial, categorical variables (text) have to be converted to numerical variables in order to be processed by a computer.  Hence, the document texts have to be converted into ther integer equivalents.  One approach would be to one-hot encode each word, but this would result in a bunch of one-hot vectors that would demonstrate no meaning between the words, and be computationally expensive.  

A better approach would be to integer encode each word.  To do this, we will use Keras (a deep learning framework)




## Pad the documents

## Create an embedding 

## Visualize intial embeddings






## Let's convert a few PubMed abstracts to one-hot vectors

There are 15 total words in the corpus:

>Doc 1: "The difference between actual and"  
Doc 2: "Cardiovascular disease (CVD) is a"  
Doc 3: "To evaluate awareness about cardiovascular"   

Next, we need to find the unique words.  We have 14 unique words ('cardiovascular' is repeated twice):

>Doc 1: The(1) difference(2) between(3) actual(4) and(5)  
Doc 2: Cardiovascular(6) disease(7) (CVD)(8) is(9) a(10)  
Doc 3: To(11) evaluate(12) awareness(13) about(14) cardiovascular(6)  

We represent these unique words in a vocabulary:
    
>Vocabulary = {'The', 'difference', 'between', 'actual', 'and', ‘Cardiovascular', 'disease', '(CVD)', 'is', 'a', 'To', 'evaluate', 'awareness', 'about'}

Each word is represented as a one-hot vector with a length equal to the size of the vocabulary (N = 14).  We will have something that looks like this:

![encoding](/assets/images//2023-07/encoding.png)

## Limitations of one-hot encoding

One challenge seen with the one-hot encoding approach is that there is no information about the words the vectors represent, or how the words relate to each other. That is, it tells us nothing about the similarities or differences between words.  If we were to graph each vector and calculate a similarity measure (e.g., cosine similarity) between them, we would see that there is 0 similiarity between any two vectors.  Refer to this [article](https://towardsdatascience.com/word-embeddings-intuition-behind-the-vector-representation-of-the-words-7e4eb2410bba) for a mathematical overview of the concept.


A second challenge is sparse, highly dimensional data.  When each word in a document is replaced with a one-hot vector, we start to get a large feature space with a lot of zeroes.  In fact, when increase our vocabulary size (for example, double the vocabulary size N = 28 words), the feature space will morph into something like this:

![encoding_sparse](/assets/images/2023-07/encoding2.png)


This dataset is:
- sparse (majority of the elements are zeroes)
- highly-dimensional (a larger vocabulary creates a larger feature space, which requires more computational power and data)
- hard-coded (machine does not learning anything from the data)
  
Given these limitations, this technique makes it difficult for a computer to detect any sort of meaningful patterns to make a prediction.


## Conclusion

In this tutorial, we explored one-hot encoding, a very simple way to convert categorical variables for natural language processing tasks.  By taking a subset of PubMed abstracts, we were able to see how this approach becomes highly inefficient with larger vocabularies.  Some of these inefficiencies are due to a lack of understanding relationships between words, as well as a sparse and highly-dimensional feature space.

There have been different techniques that have improved upon the limitations of one-hot encoding, and they have worked increasingly well with neural networks.  I will explore these in future posts.


## References

Data/SQL Server:
+ <https://huggingface.co/datasets/zhengyun21/PMC-Patients/tree/main>
+ <https://builtin.com/software-engineering-perspectives/sql-server-management-studio-mac>
+ <https://www.sqlshack.com/sql-server-data-import-using-azure-data-studio/>


+ <https://machinelearningmastery.com/prepare-text-data-deep-learning-keras>
+ <https://machinelearningmastery.com/use-word-embedding-layers-deep-learning-keras>
+ <https://medium.com/analytics-vidhya/understanding-embedding-layer-in-keras-bbe3ff1327ce>
+ <https://medium.com/@naidubhavya06/detailed-explanation-of-keras-embedding-layer-afe4c3a596a>
+ <https://towardsdatascience.com/neural-network-embeddings-explained-4d028e6f0526>

Concepts:
+ <http://colah.github.io/posts/2014-07-NLP-RNNs-Representations>
+ <https://towardsdatascience.com/similarity-metrics-in-nlp-acc0777e234c>
+ <https://stackoverflow.com/questions/42762849/keras-embedding-layers-how-do-they-work>
+ <https://stackoverflow.com/questions/45649520/explain-with-example-how-embedding-layers-in-keras-work>
+ <https://stats.stackexchange.com/questions/270546/how-does-keras-embedding-layer-work?rq=1>
+ <https://github.com/keras-team/keras/issues/3110>

Visualization:
+ <https://towardsdatascience.com/how-to-visualize-text-embeddings-with-tensorboard-47e07e3a12fb?gi=c4a391cf1a39>
+ <https://branyang.gitbooks.io/tfdocs/content/get_started/embedding_viz.html>
+ <https://www.tensorflow.org/tensorboard/tensorboard_projector_plugin>
+ <https://towardsdatascience.com/visualizing-your-embeddings-4c79332581a9>
+ <https://medium.com/analytics-vidhya/how-to-visualize-word-embeddings-7ed0fb047089>


## Full HTML

Perhaps the best part of Markdown is that you're never limited to just Markdown. You can write HTML directly in the Markdown editor and it will just work as HTML usually does. No limits! Here's a standard YouTube embed code as an example:

<p><iframe style="width:100%;" height="315" src="https://www.youtube.com/embed/Cniqsc9QfDo?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe></p>