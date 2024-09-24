---
title: 'Temporally Persistent Information Retrieval System'
description: 'Designing a temporally persistent '
pubDate: 'Jul 02 2022'
category: 'Data engineering and Data Infrastructure'
heroImage: '../../assets/images/placeholder-hero.jpg'
tags: ['ML']
---

Recently, I’ve joined a research team at Kaggle that focuses on submitting workshop papers to Kaggle Clef conferences. My subteam’s specific conference is CLEF LongEval Lab encourages participating teams to develop temporal information retrieval systems and longitudinal text classifiers that survive through temporal text changes. Essentially the workshop introduces time as a new benchmark for model performance. The workshop is built on top of recent research showing that the performance of the models/systems drops as the data becomes more temporally distant from the data the model was trained on. 
There are two different submission criteria for this CLEF. The first one is to develop an information retrieval system that is temporally persistent across different periods. The second one is to design a sentiment analysis binary classifier that is temporally persistent across different periods
This series will cover our implementation for the information retrieval submission. Our implementation utilized ElasticSearch and Python as a vector database. The reason we did this was to isolate being able to quickly ingest and query data. The rest of this article will cover a basic implementation of our first attempt at creating a temporally persistent information retrieval system.
Elastic Search allows us to rapidly search through our data, scale our clusters to hundreds of nodes, and perform rapid analysis. We utilized Elastic Search by ingesting training documents, vectorizing them then passing in test queries that were also vectorized to then perform vector search. Vector Search uses distances in the embedding space to represent similarity. Embeddings are lists of numbers that describe our text data. We can utilize traditional NLP and machine learning techniques to vectorize text data. We can generate embeddings by the word using methods like Word2Vec, and Glove, or we can generate embeddings by the sentence by utilizing sentenceTransformer models.
 For our research setup, we vectorized the text data and then vectorized the queries. Then we searched for text data that were mathematically close in the embedding space and returned those documents. The rest of this article will focus on explaining our implementation.

In the following code snippet, we created our elastic search client and then created our file path from where we ingest our documents. We also imported our pre-trained sentence transformer model. For this series, we will be utilizing pre-trained models for speed.


#from elasticsearch import Elasticsearch
#from elasticsearch.helpers import bulk
#from elasticsearch import helpers
#import numpy as np
#from random import getrandbits
#logging.basicCOnfig(filename="es.log", level=logging.INFO)
#import requests


#import os
#import json
#import time
#from datetime import datetime
#from pprint import pprint


es = Elasticsearch("http://localhost:9200")


print(es.info())
path = '/publish/English/Documents/Json/'
rootpath = '/publish/English/Documents/Json/'
jsonfilename = 'collector_kodicare_70.txt.json'
directory = path




We then created and initialized our Vector_Search class with our sentence transformer model
class Vector_Search:
def __init__(self):

     # this is a sentence-transformer model that maps sentences & paragraphs to a 384       dimensional dense vector space and can be used for tasks like clustering or semantic search
     self.Minimodel = SentenceTransformer('all-MiniLM-L6-v2')
     self.es = es





Here we created dense vectors and sparse vectors indexes to store our documents and the embeddings we create for them

def create_dense_index(self):
    self.es.indices.create(index='my_documents_dense', mappings={
              'properties': {
                         'embedding': {
                                    'type': 'dense_vector',
                                      }
                            }
                      })






This method allows us to Ingest our documents into the indexes we’ve already created
def Ingesting(self,documents):
    cwd = os.getcwd()
    for filename in os.listdir(cwd+documents):
        with open(cwd+documents+filename, "r") as f:
             arr = json.load(f)
             document_list = []
             operations = []
             for a in arr:
                 dictionaries=arr[a]
                 _id = dictionaries["id"]
                 _content = dictionaries["contents"]
                 embedding = self.get_embedding(self,_content)
                 operations.append({'index': {'_index':'my_documents'}})
                 operations.append({**a, 'embedding':embedding})
                 newDict = { _id:_content}
                 document_list.append(newDict)



   return self.es.bulk(operations = operations)



Here we pass in our queries that we will be vectorizing and using to search our index for relevant documents
def get_query_list(queries):
    q = {}
    cwd = os.getcwd()
    for filename in os.listdir(cwd+queries):
        if filename == 'train.tsv':
           with open(cwd+queries, 'r') as f:
                for line in f:
                    (key, value) = line.split()
                    q[key]=value
    return q


Here we utilized our SentenceTransformed to generate our embeddings
def get_sentenceTransfomrer_embedding(self,text):
    return self.model.encode(text)


This method is where we make our actual runs utilizing the queries that were passed in.
def make_run(self):
    listOfResults = []
    q = self.get_query_list(queries)
    self.create_index(self)
    self.Ingesting(self,documents)
    for value in q:
       results = es.search(
       knn={'field':'embedding',
              'query_vector':self.get_embedding(value),
                   'num_candidates': 100,
                                    'k': 10,
           },
                size = 10
         )






This method allows us to show our results 

def show_results(results):
    for result in results:
        print(f'{result["fields"]["title"]}\nScore: {result["_score"]}\n')




In this article we developed an information retrieval system built on top of elasticsearch and then ran k nearest neighbors against the documents we’ve ingested into our database, ranked documents and returned relevant documents. In the following articles of this series we will focus on different methods to generate embeddings and return relevant documents. We will then show how to compare the temporal performance of these embeddings utilizing temporally distant data.

