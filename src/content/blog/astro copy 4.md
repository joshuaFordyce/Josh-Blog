---
title: 'Comparative Study2: Sentiment Analysis classification'
description: 'This is a continuation of my comparative study of different recurrent neural networks, support vector machines and knn models'
pubDate: 'Jul 02 2022'
heroImage: '../../assets/images/banner.jpg'
category: 'Machine Learning'
tags: ['JavaScript', 'css', 'HTML5', 'GitHub', 'Ordenador']
---

1 Summary
The second classification problem I chose to solve was the classification problem of sentinemnt analysis of textual data. I utilized a dataset of tweets scraped for a workshop competition called clef:longeval. The CLEF 2024 LongEval lab encourages participants to develop text classifiers that survive through time.
2 Data Preprocessing
First I checked to see if I had any Null or NaN values in the dataset. Secondly, I encoded the labels numerically for easier modeling. Thirdly, I used lemming and stemming to process the textual data to make them easier to analyze. I then vectorized the data using countVectorizer and the bag of word algorithm.
3 Support Vector Machine
The first type of model Im going to be evaluating is the SVM model. I felt that the SVm would work well for this classification problem since most SVMs have builtin protection from overfitting. I also felt that given the fact that this dataset seemed more linearly separable than most the svm model would perform well. I was surprised that all of the svm algorithms performed poorly on this data. I think what might of happened is the way we preprocessed the textual data made it more difficult for the algorithm to make separations within the dataset.
4 K Nearest Neighbors
The second type of model I’m going to be evaluating is the KNN model. This model accomplishes text classification by polling the k nearest neighbors of the data point its trying to classify. This poll is determined by measures such as the Euclidean distance. I experimented with a high amount of k and a low amount k as well as a range of k from 10 to 50. I found that the low amount of k that I used actually worked well for this classificiton problem compared to the high amount of k. I think further research focusing on which specific number of K Nearest Neighbors might help the accuracy of the models.
5 Artificial Neural Network
The third type of model that Im going to be evaluating is the ANN model. This model is inspired by biological neural networks and they use perceptrons which operate like neurons in that that they calculate outputs from input singles. For this model I kept things very simple and simply used a multi layer perceptron from the sklearn library. I focused on comparing the different activation func- tions. An activation function does the actual work of deriving output from a set of input values fed to the node. The activation functions that I compared were the default relu, tanh, logistic and identity. I thought that the tanh fucntion would do the best for this classification problem.
