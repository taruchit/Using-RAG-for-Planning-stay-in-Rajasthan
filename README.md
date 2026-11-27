# Using-RAG-for-planning-stay-in-Rajasthan

**Project overview**

Here we build a proof of concept that ingests scanned, image-only PDF document consisting of tariffs of hotels across different cities in Rajasthan, extracts structured tables and text via vision/OCR and answer natural-language queries with page-level citations using Retrieval Augmented Generation (RAG).

**Problem statement**

Standard PDF parsers such as pdfplumber and PyPDF2 fail silently on scanned or rasterized regulatory documents. It means they do not return any error but nor give any results after parsing the document, thus, returning empty strings. 

**Motivation to solve the problem**

To learn and demonstrate about parsing a PDF document which has tables as images, and retrieve the content which can be useful to answer domain specific queries for multi-page document.

**High-level details for parsing the PDF file**
1. Loading the PDF document and fetching its total number of pages.
2. For each page: -
   - We create an in-memory byte stream (virtual file). This is like an in-memory buffer. It is useful to avoid writing temporary .png files to disk for every page and cleaning them up later.
   - Then we convert pixel data into compressed PNG bytes in the virtual file.
   - Using the compressed PNG bytes, we convert them to raw bytes using base64 and then translate them into corresponding string values.
   - The string values are passed to LLM to convert them into readable format which can be use later for Q&A
   - The results for each page are stored in a TEXT file: tariff_summary.txt along with the corresponding page numbers.

**High-level details for using the Text file to return responses**
1. Loading the text file while maintaining the respective page number.
2. Chunking the text while ensuring that table rows and columns remain together for a given page.
3. Then we use OpenAI embedding: text-embedding-3-small to process the raw data and transform it into numeric format which can be understood by the model.
4. We store the chunks and the corresponding embeddings in FAISS vector database.
5. Then we wrap FAISS vector store into LangChain retriever which is configured to return the top 4 most similar chunks based on the query.
6. We define the system prompt to instruct the model about its role and persona.
7. We also define the LLM model: gpt-4o-mini, along with temperature = 0 which indicates the response generated will be highly deterministic.
8. Then we define RAG pipeline to handle the input and fetch results via LLM based on the TEXT file.
9. Finally we define the question and fetch the corresponding response.

**Reference material**

* Approach for parsing the document: https://www.elastic.co/search-labs/blog/alternative-approach-for-parsing-pdfs-in-rag
* Source data: https://www.rtdc.tourism.rajasthan.gov.in/Pdf/Tariff.pdf
