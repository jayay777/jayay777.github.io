---
title: AI Genre Categorization
layout: post
post-image: /assets/images/library_image.jpg
description: AI prediction of Dewey Decimal Classification with titles and indexes. Made using BERT with pytorch
tags:
- AI
- Python
- Google Colab
---

This university project's goal was to find out the strength of the relation between title and genre, as well as to experiment to find out if an automatic categorization of a book is possible using AI.


This was a project that was done in the second half of 2022. Approximately 3 months were spent. To see the code and results, click [here](https://github.com/jayay777/2022F-Ajou-ML-TEAM3){:target="blank"}

The overall process is as follows: 

1. Crawl the book index, title, and genre from the University's Library API.
2. Process the information into CSV, so that it can processed again with BERT Tokenizer.
3. Train the categorization model with title, index to genre, and try out different hyperparameters

    
# Features I have implemented

* Crawling the information from the library website
* The English title to genre categorization
* merging the KoBERT and index categorization models into one.

For a more specific report(Korean) on the project, click [here](/assets/downloads/AI_Librarian_Report.docx){:target="blank"}