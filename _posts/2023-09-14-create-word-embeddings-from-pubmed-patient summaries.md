---
layout: post
title:  "Create word embeddings from PubMed patient summaries"
author: sandy
categories: [ PubMed, NLP, tutorial ]
image: assets/images/2023-09/OIP_resize.jpg
---
Building upon my previous tutorial on one-hot encoding, this tutorial will review the concept of word embeddings and apply this to real-life data.  

For our example, we will extract patient summaries of diabetic and non-diabetic patients from PubMed.  From these patient summaries, portions of text will be selected to create word embeddings.  These embeddings will be "trained" in a neural network model to perform a task, such as classification (diabetic vs. non-diabetic).  By training word embeddings, the computer will learn if there are any meaningful relationships between the words in the text. 

## A primer - Word Embeddings

Word Embeddings were a bit of a complex concept to grasp, until I got into the weeds of building one.  I've come to learn they are an important concept in deep learning, for the very reason that semantic meaning of words can be approximated mathematically.

There are a number of techniques available to build a word embedding.  As a quick note, LLMs use word embeddings, but the technique used to build them are proprietary.  

To start, I wanted to backtrack the origins of LLMs, which are a type of neural network.  A neural network is a type of machine learning approach that attempts to mimic the way the brain works (biological neural network).  This approach has been shown to perform better on NLP tasks than previous methods.  For a great overview of NLP, check out this [guide](https://www.deeplearning.ai/resources/natural-language-processing).

The goal of NLP is to enable computers to "understand" natural language in order to perform some task, such as sentiment analysis.  In order to do so, natural language (text format) has to be converted (encode) into a numerical format.

There are numerous approaches to encode text, with more advanced approaches surfacing over the years.  I will start with one-hot encoding, as it is one of the most familiar data pre-processing techniques for ML.  However, we will also begin to see the limitations of such a technique through the example below. 

For NLP, one of the simplest techniques for encoding text is to represent each categorical variable (a word) as a binary vector.  A one-hot vector can be thought of as a type of binary vector where the index of the word is denoted as '1', and all other words are denoted as '0'.  The size/dimension of the vector is determined by the total number of unique words (vocabulary) found in the text.

Consider the following text: 'I have a fever'.  The vocabulary consists of 4 unique words (I, have, a, fever), and each word is represented as a one-hot vector.

![PubMed](/assets/images/2023-07/fever.png)

## Let's get data

I found a really great dataset of patient profiles extracted from PubMed articles via [HuggingFace](https://huggingface.co/datasets/zhengyun21/PMC-Patients/tree/main).  I downloaded this dataset and loaded it into a local SQL Server database on a Mac.  For further instructions on how to set this up, check out this [tutorial](https://builtin.com/software-engineering-perspectives/sql-server-management-studio-mac). 

Another option would be to load the data into a cloud database, such as Azure SQL DB.  

## Define the dataset and labels

I wanted to run queries to extract profiles of diabetic patients.  For the purpose of training a neural network to perform a classification task, the query will return a set of patient profiles that are diabetic vs. non-diabetic (labels).

To do this, I used Azure Data Studio to access the data.  To connect to a SQL database using Azure Data Studio, check out this tutorial..... Attached are a few screenshots to get the dataset loaded correctly.

Now we can run a few queries to inspect the data, and create our desired dataset.



## Convert text to integers

As explored in the previous tutorial, categorical variables (text) have to be converted to numerical variables in order to be processed by a computer.  Hence, the document texts have to be converted into ther integer equivalents.  One approach would be to one-hot encode each word, but this would be a meaningless and potentially expensive approach.  An alternative approach would be to integer encode each word.   


## Pad the documents

## Create an embedding 

## Visualize intial embeddings






## Let's convert a few PubMed abstracts to one-hot vectors

To put the above tutorial into practice, I thought I would give this a try with a few abstracts.  Let's say that we want to perform an NLP task on papers that discuss cardiovascular disease (CVD) risk factors.  This is what my query and results look like in PubMed:

![PubMed](/assets/images/2023-07/PubMed.png)

Each abstract represents a single document in the corpus.  Let's take the first 5 words from each abstract.  For simplicity purposes, we will ignore header terms such as 'Context', 'Introduction', 'Purpose'.

![PubMed](/assets/images//2023-07/PubMed2.png)

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

Data/Tutorials:
+ <https://huggingface.co/datasets/zhengyun21/PMC-Patients/tree/main>
+ <https://builtin.com/software-engineering-perspectives/sql-server-management-studio-mac>
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