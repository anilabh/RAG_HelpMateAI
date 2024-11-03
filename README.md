# RAG_HelpMateAI
HelpMateAI Project Documentation
Project goals
In this project we are given an insurance policy document and the goal is to build a RAG-LLM based response engine that can retrieve the right information for any query on the insurance policy from the document provided. Policy document is a 64-page pdf file. It is very hard for any individual to go over the entire document and be able to remember all the details. The response engine provides a conversational interface to get the right information accurately and quickly.

![image](https://github.com/user-attachments/assets/0ef47292-16d0-4aa2-924b-e16e69d929a4)

Data Sources
Principal-Sample-Life-Insurance-Policy.pdf : is the policy document in pdf format that is the source of truth. This policy document this document needs to be processed, cleaned and chunked for embeddings.
ChromaDB: is used to store embeddings, documents and metadata. This helps is quick semantic searches. Caching layer is implemented for better performance for repeated queries. 
Design choices
 
PDF parsing:
We used pdfplumber to read and process the pdf file. It allows for better parsing of the PDF file as it can read various elements of the PDF apart from the plain text, such as, tables, images, etc. It also offers wide functionaties and visual debugging features to help with advanced preprocessing as well. A lot of pages in PDFs have only a title or are mostly blank. We removed such pages and kept only the pages with at least 10 words.
Chunking:
This concludes the chunking aspect also, as we can see that mostly the pages contain few hundred words, maximum going upto 1000. So, we don't need to chunk the documents further; we can perform the embeddings on individual pages. This strategy makes sense for 2 reasons:
1. The way insurance documents are generally structured, you will not have a lot of extraneous information in a page, and all the text pieces in that page will likely be interrelated.
2. We want to have larger chunk sizes to be able to pass appropriate context to the LLM during the generation layer
Embeddings:
We embed the pages in the dataframe through OpenAI's `text-embedding-ada-002` model, and store them in a ChromaDB collection. The user Queries and documents need to have the same embeddings so that vector Database can find text relevance distance accurately.
Re-ranking:
Re-ranking the results obtained from ChromaDB semantic search can significantly improve the relevance of the retrieved results. This is done by passing the query paired with each of the retrieved responses into a cross-encoder to score the relevance of the response w.r.t. the query. Sentence Transformers (a.k.a. SBERT) is the go-to Python module for accessing, using, and training state-of-the-art text and image embedding models. We use it to calculate similarity scores using Cross-Encoder models.
Retrieval Augmented Generation:
We pass the final top search results to an GPT 3.5 along with the user query and a well-engineered prompt, to generate a direct answer to the query along with citations, rather than returning whole pages/chunks.
Challenges 
While designing the queries it was important to keep in mind the distance that vector DB will calculate with the documents. If the emphasis is not given to the right key words, the end result was inaccurate. For instance, when we first created Query 1, the focus was more on Actively Working viz. “The document mentions that policy is only active for an Actively working member. However, there are conditions under which this requirement is waived. What are those conditions”
With this query, the section about waiver for Actively working requirement was ranked lower and OpenAI API was not able to get the right response. In fact, it responded by “There is no explicit mention of waiver to Actively working requirement”.
We then changed the query to be more focused on Waiver, viz “ What are the conditions under which Actively at Work requirements can be Waived. Extract out information about Waived members”. 
After this change in the Query, we clearly got the right section of the document ranked at the top. And OpenAI accurately listed the waiver conditions.

