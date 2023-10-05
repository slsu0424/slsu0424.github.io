---
layout: post
title:  "Create word embeddings from PubMed patient summaries"
author: sandy
categories: [ PubMed, NLP, tutorial, SQL, Azure ]
image: assets/images/2023-09/OIP_resize.jpg
---
Building upon the previous [tutorial](https://slsu0424.github.io/encoding-pubmed-abstracts-for-nlp-tasks/) on one-hot encoding, this tutorial will review the concept of word embeddings and apply this to real-life data.  

For our example, we will extract patient summaries (documents) from PubMed.  From these patient summaries, we will label those that had COVID-19, and those that did not.  Portions of text from each document will be selected to create word embeddings.  These embeddings will be "trained" as part a neural network model to perform a classification task.  By training the word embeddings, the computer will learn if there are any meaningful relationships between the words in the text. 

## A short intro to Word Embeddings

Word Embeddings were a bit of a complex concept to grasp, until I got into the weeds of building one.  I've come to learn they are an important concept in deep learning, for the very reason that semantic meaning of words can be approximated mathematically.

To best understand the intuition behind word embeddings, I highly recommend Colah's blog post of this topic.  In summary 

There are a number of techniques available to build a word embedding... 

As a quick note, LLMs use word embeddings, but the technique used to build them are proprietary.  

## Let's get data

I selected a dataset of ~167K PubMed patient summaries via [HuggingFace](https://huggingface.co/datasets/zhengyun21/PMC-Patients/tree/main).  To demonstrate a low-code approach, SQL Server and Azure Data Studio will be used.

The data is loaded into SQL Server on a Mac.  For further instructions on how to set this up, check out this [tutorial](https://builtin.com/software-engineering-perspectives/sql-server-management-studio-mac).  

SQL Server is started via the terminal giving the username and password (Docker must be run on your machine):

```
$ mssql -u <sql server username> -p <sql server password>
```

Azure Data Studio is used to access and query the data.  To connect to a SQL Server using Azure Data Studio, review this [tutorial](https://www.sqlshack.com/sql-server-data-import-using-azure-data-studio/).  

Attached is a screenshot to modify the columns before importing the data.

![AzureDataStudio](/assets/images/2023-09/azstudio_setup3.png)

We can run a few [queries](https://github.com/slsu0424/pmc-patients/blob/develop/pmc-patients.sql) to inspect the data, and create the desired dataset.

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

Execute the first query to get the top 100 records.  The results can be exported from Azure Data Studio as a .csv.

*Note:* While this represents a longer approach to loading and manipulating the dataset, you may wish to explore other approaches:
1. Load dataset into pandas dataframe.
2. Load dataset into a cloud database, such as Azure SQL Database.  Please keep in mind there are costs associated with running the database in the cloud, as well as querying costs.

## Define class labels

Since this is a binary classification task, we will label the dataset as:
- patients with COVID-19 = '1'
- patients without COVID-19 = '0'

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

Output:

```
[1. 0. 0. 0. 0. 0. 0. 0. 1. 1. 1. 0. 0. 0. 1. 0. 0. 0. 0. 0. 0. 1. 1. 0.
 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 1. 0. 0. 0. 0. 0. 0.
 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0.
 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 0. 1. 1. 0.
 0. 0. 1. 0.]
```

## Create a corpus

Now that we have our labeled dataset, the next step is to create a corpus (collection of documents).  We take the first 3 sentences from each document (100 PubMed articles).  The resulting corpus is a python dictionary; a sample of the output is shown below:

Doc 1 = <span style="color: red;">red</span>  
Doc 2 = <span style="color: blue;">blue</span>  
Doc 3 = <span style="color: green;">green</span>  


[<span style="color: red;">'This 60-year-old male was hospitalized due to moderate ARDS from COVID-19 with symptoms of fever, dry cough, and dyspnea. We encountered several difficulties during physical therapy on the acute ward.'</span>, <span style="color: blue;">'We describe the case of a 55-year-old male who presented to the emergency department via emergency medical services for the chief complaint of sudden onset shortness of breath that woke him from his sleep just prior to arrival. He reported three days of non-radiating lumbar back pain and two episodes of non-bloody emesis leading up to this event.'</span>, <span style="color: green;">'A 20-year-old Caucasian male (1.75 m tall and 76 kg (BMI 24.8)), was admitted to the medical department for persistent hyperpyrexia, severe sore throat, dyspnea, and impaired consciousness with stupor. Persistent symptoms started at home 4 days before and he assumed clarithromycin as empiric antibiotic therapy.'</span>, ...] 

For this example, there are a total of X words in the corpus.


## Convert text to integers

As explored in the previous tutorial, categorical variables (text) must be converted into numerical variables.  One approach would be to one-hot encode each word, but this would result in a bunch of one-hot vectors that would demonstrate no meaning between the words, and be computationally expensive.  

A better approach would be to tag each word with a unique integer.  The nice thing about this approach is that the integer encoding for a specific word remains the same across all documents.  For example, ... 

To do this, Keras (a neural network library) provides a handy **Tokenizer() API** that can handle multiple documents.  For a deeper understanding of how to implement this, see this [tutorial](https://machinelearningmastery.com/prepare-text-data-deep-learning-keras).

```python
encod_corp = []

#for i, v in enumerate(corp):
#    print("Document", i+1, "words:", v)

# fit tokenizer on docs
t = Tokenizer()
t.fit_on_texts(corp) 
encod_corp = t.texts_to_sequences(corp) # convert docs to num sequence

# get unique words
vocab = t.word_index

print("vocab:")
for i in vocab:
    print(i)

vocab_size = len(vocab) # input into embedding layer
print('Vocab size = %s unique words' % vocab_size)

```

Output:

```
vocab:
a
and
the
of
with
was
to
year
old
in
...

Vocab size = 1842 unique words

``` 
The size of the vocabulary (1842) will be important as an input for the embedding layer.

## Pad the documents

The next thing that Keras requires is that all documents must be of the same length.  As some of documents have more words than others, padding (zeroes) will be added to make the document lengths even.

```python
maxlen=-1

# find max number of words per doc
for i, v in enumerate(corp):
    #print(i,v)
    print("Document", i+1, "words:", len(v.split()))
    if (maxlen<len(v.split())):
        maxlen=len(v.split())

print("Max number of words in any doc is:", maxlen)
```

## Create an embedding 

## Visualize intial embeddings



## Limitations of one-hot encoding

One challenge seen with the one-hot encoding approach is that there is no information about the words the vectors represent, or how the words relate to each other. That is, it tells us nothing about the similarities or differences between words.  If we were to graph each vector and calculate a similarity measure (e.g., cosine similarity) between them, we would see that there is 0 similiarity between any two vectors.  Refer to this [article](https://towardsdatascience.com/word-embeddings-intuition-behind-the-vector-representation-of-the-words-7e4eb2410bba) for a mathematical overview of the concept.


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
