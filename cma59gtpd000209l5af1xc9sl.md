---
title: "Under the Hood of PDF Insight Assistant: Streamlit, Gemini AI & FAISS"
seoTitle: "AI-Powered PDF Insight with Streamlit"
seoDescription: "PDF Insight Assistant uses AI to transform PDFs into interactive formats via Streamlit, Gemini AI, and FAISS for enhanced document interaction"
datePublished: Thu May 01 2025 11:07:45 GMT+0000 (Coordinated Universal Time)
cuid: cma59gtpd000209l5af1xc9sl
slug: under-the-hood-of-pdf-insight-assistant-streamlit-gemini-ai-and-faiss
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1746090261277/a71ac3b4-74f8-40e0-bb83-f9baa1eccd81.png
tags: ai, system-design, pdf, modern, awesome, gemini

---

Ever wished you could talk to your PDF files and get instant answers instead of scrolling endlessly? With the rise of AI-powered assistants, that’s no longer a dream — it’s a reality.

The **PDF Insight Assistant** is a Streamlit-based app that brings conversational intelligence to your documents using **Google Gemini AI**, **FAISS vector search**, and seamless PDF processing. Whether you're a developer, AI enthusiast, or just curious about the future of smart interfaces, this post will take you behind the scenes — exploring how this assistant is built and how you can take inspiration to create your own.

[Github Code](https://github.com/shakibkhandev/pdf-insight-assistant)

## Step-by-Step Guide

1. Prerequisites
    
2. Get the PDF
    
3. Extract PDF Text
    
4. Split the Text into Chunks
    
5. Generate Embeddings Locally
    
6. Create and Store in FAISS
    
7. Take a User Question & Find Similar Chunks
    
8. Provide the extracted PDF text in semantically similar chunks to the Gemini AI model for answer generation
    
9. That’s all
    

## **Prerequisites 📋**

Before running the application, make sure you have:

```python
pip install streamlit pdfplumber sentence-transformers faiss-cpu langchain
```

Also you need GEMINI API KEY for this Project

[Get it here](https://aistudio.google.com/app/apikey)

Also, use a short PDF—preferably 2 to 3 pages—if you only need to extract text, as larger files may take more time to process.

## Get the PDF

Use Streamlit functionality to upload and process a PDF file.

```python
import streamlit as st
pdf_file = st.file_uploader("Choose a PDF file", type="pdf")
```

## Extract PDF Text

Extract text from the uploaded PDF

Using `pdfplumber`:

```python
import pdfplumber

def extract_text_from_pdf(uploaded_file):
    text = ""
    with pdfplumber.open(uploaded_file) as pdf:
        for page in pdf.pages:
            text += page.extract_text() + "\n"
    return text
```

## Split the Text into Chunks

Use simple logic or Langchain’s splitter:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

def split_text(text):
    splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
    return splitter.split_text(text)
```

## Generate Embeddings Locally

Use SentenceTransformers :

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

def get_embeddings(chunks):
    return model.encode(chunks).tolist()
```

## Create and Store in FAISS

Use FAISS for fast similarity search:

```python
import faiss
import numpy as np

def create_faiss_index(embeddings):
    dim = len(embeddings[0])
    index = faiss.IndexFlatL2(dim)
    index.add(np.array(embeddings).astype('float32'))
    return index
```

## Take a User Question & Find Similar Chunks

Convert the question to an embedding and search:

```python
def search_similar_chunks(question, index, chunks):
    question_embedding = model.encode([question])
    D, I = index.search(np.array(question_embedding).astype('float32'), k=3)
    return [chunks[i] for i in I[0]]
```

## Provide the extracted PDF text in semantically similar chunks to the Gemini AI model for answer generation

It will generate an answer from the chunks

```python
def generate_answer_with_gemini(question, similar_chunks, model):
    try:
        # Extract just the text from the chunks and their scores
        chunks_text = [f"Context {i+1} (relevance score: {score:.2f}):\n{chunk}" 
                     for i, (chunk, score) in enumerate(similar_chunks)]
        
        # Construct a prompt with context
        prompt = f"""
        You are an intelligent assistant answering questions about a PDF document.
        Based on the following contexts extracted from the document, answer the user's question.
        If the answer cannot be found in the contexts, say "I don't have enough information to answer this question based on the provided document."
        
        User Question: {question}
        
        Document Contexts:
        {''.join(chunks_text)}
        
        Your answer should be:
        1. Directly answering the question based on the document
        2. Well-structured and concise
        3. Including references to specific parts of the document when relevant
        4. Not making up information that is not in the document
        
        Answer:
        """
        
        response = model.generate_content(prompt)
        return response.text
    except Exception as e:
        logger.error(f"Error generating answer with Gemini: {e}")
        return f"Error generating answer: {str(e)}"
```

Display the answer returned by the function at the desired location in the Streamlit app.

## Github Code

%[https://github.com/shakibkhandev/pdf-insight-assistant] 

The PDF Insight Assistant is more than just a chat interface — it’s a demonstration of how AI, when combined with the right tools, can unlock knowledge trapped in static documents. By integrating **Streamlit**, **Gemini AI**, and **FAISS**, this project bridges the gap between passive data and active interaction.

Whether you're looking to extend its capabilities, adapt it for other file formats, or embed it in a larger workflow, the foundation is here — open, adaptable, and ready for innovation.

Feel free to explore the [GitHub repository](https://github.com/shakibkhandev/pdf-insight-assistant), give it a ⭐️ if you found it useful, and join the conversation if you have ideas or questions. The future of intelligent document interaction is just getting started.

👉 **Subscribe to our newsletter** to stay updated on upcoming AI projects, tutorials, and innovation in document intelligence.

💬 Have thoughts, suggestions, or just want to connect? Feel free to reach out — I’d love to hear from you.

> A great developer isn't defined by how much code they write, but by how effectively they solve problems with it.