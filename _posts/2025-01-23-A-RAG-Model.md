---
title:      A Basic RAG Model
layout:     post
category:   Notes
tags: 	    [beginner, devops, RAG Models, AI]
---

In the world we live in today, its almost everyday that we hear about a new AI "revolution". Just keeping up with the innvoation and launches, requires a full time job. Due to this, it was very easy to get overwhelmed. I was thinking that I was the only one that feels this way but I was wrong. Many other felt this way as well. So I decided to write to to be able to understanding building A Basic RAG Model. After reading this, you will find a basic understanding on what it takes to build a rag model.
<!--more-->

In order to create a successfull RAG model, you need o first store your data. This data is the data, that will make your AI model know specific stuff that was not part of the planning. Then next the RAG model needs a search capability to search the user query against the data your store. Once the search is complete, you need to augment your prompt to the LLM to use the LLM properly. 

So now for step 1, you want to store the data which is a book into your DB. The book needs to be broken down into chunks that needs to be converted into embeddings which in this context it can be vector embeddings. Creating the chunks of data and creating vector embeddings can be done using LANGCHAIN. Thie step of creating embeddings is important as the same type/method/model needs to be userd to create the embeddings of the user query

Now for step 2, we will have to create embeddings for the users query. We will need this user query because we needs compare the vectors with what is stored in the vector DB which in this case can be CHroma DB. In this comparison, it might not find the exact match of vectors but it will find the closest vector in the db for that of the query. 

Now for step 3, we will need to bring in the LLM so that we can use their reasoning capability with out RAG model. How does this work? How does this happen? For that we need to understand context. What makes the AI model to keep up the conversation going is this thing called context. For example, I want the model to call me Superman. For this I will need to let the model know that my name is Superman and ask it to call me that only. This is called context. You provided it some context to allow the flow of the converstation. So what happens here after the query to the db is that it returns the data/object that is the closest vector. This data or obejct is then included in the API call to the LLM model. It is like prompt + context. So now with the longer, more meaningful prompt, the LLM will be able to reason better. This is where the augemented part of the Retrieval Augement Generation comes it. The prompt is augmented with the context. 
