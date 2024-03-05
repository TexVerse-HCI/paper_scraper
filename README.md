# Data Source
- __Raw data__. Access at `data/paper_data_final.json`. All data were stored in [Firebase](https://firebase.google.com/) __Firestore__.
    - I used firebase because I want to deploy my app directly on GitHub Page, so I want to have a pure front-end app. While Azure doesn’t support data access from front-ends, Firebase supports these sorts of actions.
- __Basic info__. All the papers were scraped from dl.acm.org, with the full query syntax: `"query": { Title:("Textile" OR "Fabric") }  "filter": { E-Publication Date: (01/01/2000 TO 12/31/2024), ACM Content: DL }`. Then these papers were mannually filtered to rule out irrelevant computer science papers (e.g. "fabric" can also be a CS network term/FPGA term. Somehow the "NOT" command in the ACM query is not working for me at that time). Source code in `scraper.ipynb`.
    - Field scraped: title, short abstract, full abstract, citation number, download number, publication conference, author list, content type, doi, date, and year.
    - Final number of papers after filtering is 220.
- __Image__. Using the doi, I used a scraper to download all pdf files (if applicable) of the scraped papers. The purpose of this is that I want to extract representative images of each paper (in HCI Fabrication research, images are CRUCIAL!). With [PyMuPDF](https://pymupdf.readthedocs.io/en/latest/recipes-images.html), I extracted the first image from the pdf. Image size filter was applied to avoid extracting some logos. Then I mannually went over all the images and replace inappropriate ones. The images were stored in Firebase __storage__.
- __Keywords__. I have a list of aspects that I'm interested in for each paper, including "fabrication methods", "tools/machines used", "materials used", etc. Therefore, I used Gemini and carefully adjusted prompt to generate these keywords and add them to the metadata of each paper.

# RAG
This is the funnies part. I want to embed an AI research agent into my application. And I needed to provide context for the LLM to generate more relavant answers. I used [RAG](https://www.pinecone.io/learn/retrieval-augmented-generation/) to achieve this. Source code in `RAG.ipynb`.

- I used the free pod-based index on [pinecone](https://www.pinecone.io/) to store embeddings. I tried to convert the json file to pdf and directly upload the pdf via the UI interface on pinecone, but the chunk result was not good: some papers were split in the middle, while I want each paper to be a chunk. So I turned to the coding method, where I used `text-embedding-3-small` from OpenAI as the embedding model, converts full abstracts of every paper into embeddings and store in the pinecone along with other important metadata (full abstract, year, author, doi, publication conference; maybe later these fields can be used as metadata filter on pinecone).
- In another storage namespace in Pinecone called "_pdf_", I stored the vector representations of the __actual pdf content of each paper__! I firstly used Python to extract paragraphs from the pdf document, took each paragraph as a text chunk, converted it to vector, and stored in the Pinecone database. In academic context, taking paragraphs as chunks makes perfect sense since usually one paragraph conveys one key idea.
- For each user question, it is firstly converted to embeddings using `text-embedding-3-small` and query in the pinecone. Pinecone will return most relevant text strings, serving as the context and was sent as message to the ChatGPT API along with the question to get the answer.

# Lessons Learned
- How to use Firebase __Firestore__ (for text data) and __storage__ (for multi-media data) in Python (write & read).
- How to extract images from pdfs using Python.
- How to embed LLMs in the application, including Gemini and ChatGPT.
- How RAG works and how to use it (including the usage of embedding models, vector storage and query).
- How to use ChatGPT API in Python.