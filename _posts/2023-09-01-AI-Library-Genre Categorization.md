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

This university project's goal was to see find out the strength of relation between title and genre, as well as experimenting if an automatic categorization of a book is possible using AI.


This was project that was done on the second half of 2022. Approximately 3 months were spent. To see the code and results, click [here](https://github.com/jayay777/2022F-Ajou-ML-TEAM3){:target="blank"}

The overall process is as follows: 

1. Crawl the book index, title, and genre from the University's Library API.
2. Process the information into csv, so that it can processed again with BERT Tokenizer.
3. Train the categorization model with title, index to genre, try out different hyperparameters

	
# Features that I have implemented

* Crawling the information from the library website
* The english title to genre categorization
* merging the KoBERT and index categorization model into one.

For a more specific report(Korean) on the project, click [here](/assets/downloads/AI_Librarian_Report.docx){:target="blank"}
