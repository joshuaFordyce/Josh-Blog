---
title: 'A Comparative study of reccurent neural networkss, support vector machines and knn models'
description: 'I analyzed the performance of several different models for a simple heart disease classification problem '
pubDate: 'Jul 02 2022'
heroImage: '../../assets/images/placeholder-about.jpg'
category: 'Machine Learning'
tags: ['JavaScript', 'css', 'HTML5', 'GitHub']
---

1 Summary
The first classification problem I choose to solve is the classificaitno problem of predicting what factors cause heart disease using a dataset that had a ton of indicators including Diabetes, obesity and high alcohol use. This data came from the cdc and was part of the Behavioral Risk Factor Surveillance System which uses surveys to colette data on the health status of US citizens. I felt that the dataset was perfect for showing the clear difference between the algorithms that we were using.
2 Data Preprocessing
The only cleaning required for this datasets was to strip the dataset of nulls and naans. The scond step of my preprocessing was to encode categorical values as numeric for easier analysis. I Then moved onto splitting the training and testing data. I divided the datasest randomly into training and testing data with 40 percent data as testing data and the rest as training data.
3 Support Vector Machine
The first type of model Im going to be evauluating is the SVM model. This model was first created in 1963 and offers two main advantages of other models. This is the fact that they have diffrent kernels and the fact that the actual formula the computer uses is a simple easy to understand quadratic formula. After doing additional research I focused on the core three kernels for a comparative analysis. I kept C and gamma as the default to isolate the kernel as my dependent factor. As you can see from my Table 1 the rbf kernel out performed the polynomial and linear kernel. After further analysis, We suspect that the ability to work with non-separable datasets is what led to the difference in performance
4 K Nearest Neighbors
The second type of model Im going to be evauluating is the KNN model. This model is a non-parametric which allows us to use proximity between data points to make predictions of the different groupings of data points within a dataset. I ran three different experiments using the knn. The first was using a fixed number of k that was 10 and the second was using a fixed number of k that was 120. I picked these at random simply to discover if there was a massive difference between the performance when using different amounts of neighbhors. The third experiment was a range of k from 0 to 50 and identify which number of k performed the best. The third experiment proved to be more helpful to identify what number of k was better for this particular dataset. After further analysis, We suspect that the overfitting that we see in our k=10 graph was due to the low amount of k. and the underfitting that we see in our k=120 graph was due to the high amount of k. We can also see in Figure 6 that the accuracy did increase from k = 10 to k = 50.


5 Artificial Neural Network
The third type of model that I’m going to be evaluating is the ANN model. This model is inspired by biological neural networks and they use perceptrons which operate like neurons in that that they calculate outputs from input singles. For this model I kept things very simple and simply used a multi layer perceptron from the sklearn library. I focused on analyzing the difference between the acti- vation functions. I examined models built with the tanh, identity, logistic, and default relu function. I thought that the data being non-linear and multiclass would lend itself to the relu function. Instead for some reason the activation function didn’t affect the accuracy at all since I ended up getting the exact same accuracy score.
 6 Conclusion
This was a really interesting experiment and it came with some really interesting results. It was clear that the svm-rbf did the best job of classification which was different than my hypothesis of the nn-relu model.
[]



 
