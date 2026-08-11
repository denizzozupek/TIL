**Tarih:** 15.07.2026
**Etiketler:** #RAG #chunking #semanticchunking

---

Kaynaklar: 
	https://www.pinecone.io/learn/chunking-strategies/
	https://www.kaggle.com/code/shedai/llm-rag-1-intro-and-chunking-strategies/notebook

**What is Chunking?**
Chunking is the process of break down large text into small pieces called chunks. 

**Why do we need chunking for our applications?**
Chunking is neccessary for fit data in the models context window and finding true information from database correctly. 

**Chunking’s role in semantic search**
Chunking size is has to be optimal lenght for model's semantic search. If chunks are too small or too large it effects  related search badly. 

**Chunking’s role for agentic applications and retrieval-augmented generation**
If an agent misinformed because of bad chunking strategy, it may waste tokens, hallucinations or calling wrong tools.
	
Even context windows enough, if chunks are too long, the LLM models suffer from[ the lost-in-the-middle problem](https://arxiv.org/abs/2307.03172)

**What effects chunking strategy**

Data type, embedding model, user query lenght and expectations etc.

### Chunking Methods

**Fixed Size Chunking:** Most common chunking, we decide number of tokens in our chunk and use this number to break up our document into fixed size of chunks. 

**Recursive Character Splitting:** It attempts to split text at natural breakpoints (like paragraphs) using a list of delimiters. If a chunk is still too large, it moves to the next smaller delimiter (sentences, then words).

**Document Structure Based Chunking:** Leverages the native formatting of structured documents (HTML, Markdown, or JSON). It splits content based on logical elements like headers, sections, or tables

**Semantic Chunking:**

**Contextual Chunking:**

___

# Greg Kamradt's "5 Levels Of Text Splitting"

Source: https://github.com/FullStackRetrieval-com/RetrievalTutorials/blob/main/tutorials/LevelsOfTextSplitting/5_Levels_Of_Text_Splitting.ipynb

---

**The Chunking Commandment:**  Your goal is not to chunk for chunking sake, our goal is to get our data in a format where it can be retrieved for value later.

## Level 1: Character Splitting

Basic form of splitting. Not recommended but good for understand basics.

- **Pros:** Easy & Simple
- **Cons:** Very rigid and doesn't take into account the structure of your text

**Chunking Overlap (Örtüşme):** Chunk Overlap refers to the **number of characters or tokens shared between consecutive chunks**. Overlapping ensures that important context is not lost when diving the text into smaller parts.

**Character Splitting Python From Scratch**

```python 
text = "Benim adım Deniz Eren Özüpek. Ben chunking mantığını anlamaya çalışıyorum."

chunk_size = 7
chunk_overlap = 3

def chunking_with_overlap(text, chunk_size, chunk_overlap):

    chunks = []
  
    if chunk_overlap >= chunk_size:

        raise ValueError("Overlap can't be bigger than chunk_size")

    step = chunk_size - chunk_overlap

    for i in range(0, len(text), step):
        chunk = text[i : i + chunk_size]
        chunks.append(chunk)

        if i + chunk_size >= len(text):
            break
    return chunks
```


```python 
def chunking_with_words(text,chunk_size, chunk_overlap):

    chunks = []
    words = text.split()
    step = chunk_size - chunk_overlap

    for i in range(0 , len(words), step):
        chunk = words[i: i + chunk_size]
        chunk_text = " ".join(chunk)
        chunks.append(chunk_text)

    return chunks
resultt = chunking_with_words(text, chunk_size=1, chunk_overlap=0)
print(resultt)
```

Character Splitting with LangChain

firstly 
```bash
pip install langchain or pip install pip install langchain-text-splitters
```

```python
from langchain_text_splitters import CharacterTextSplitter

def chunking_with_langchain(text, chunk_size, chunk_overlap):
    text_split = CharacterTextSplitter(chunk_size=chunk_size, chunk_overlap=chunk_overlap, separator='', strip_whitespace=False)

    return text_split.create_documents([text])


result_langchain = chunking_with_langchain(text, chunk_size=chunk_size, chunk_overlap=chunk_overlap)

print(result_langchain)

```

-----

## Level 2: Recursive Character Text Splitting

Recursive Character Text Splitting is a method that breaks long documents into smaller chunks with a series of seperators. 

You can see the default separators for LangChain [here](https://github.com/langchain-ai/langchain/blob/9ef2feb6747f5a69d186bd623b569ad722829a5e/libs/langchain/langchain/text_splitter.py#L842). Let's take a look at them one by one.

- "\n\n" - Double new line, or most commonly paragraph breaks
- "\n" - New lines
- " " - Spaces
- "" - Characters

**Pros:** 
- **Maintains Context:** It keeps related paragraphs and sentences together, preserving the flow of reading.

- **Avoids Broken Words:** It prioritizes natural breaks, ensuring words are rarely cut in half.

**Cons:** Blind to Meaning, Requires Clean formatting texts. 

**Recursive Character Text Splitting Python From Scratch**

```python
"""
Recursive Character Splitting Without Framework
(Langchain Recursive Character Text Splitter with pure Python implementation)
Two main functions are provided:
- merge_with_overlap: Merges a list of text chunks with a specified overlap.
- split_text: Splits a given text into chunks of a specified size with a specified overlap.
"""

def merge_with_overlap(chunks, separator, chunk_size, chunk_overlap):
    result_chunks = []
    buffer = []

    for chunk in chunks:
        candidate_buffer = buffer + [chunk]
        candidate_text = separator.join(candidate_buffer)

        if len(candidate_text) > chunk_size and buffer:
            result_chunks.append(separator.join(buffer))
            
	       #-----overlap handling-----

            overlap_buffer = []

            for p in reversed(buffer):
                candidate_overlap = [p] + overlap_buffer
                overlap_text = separator.join(candidate_overlap)
                combined_with_new = separator.join([overlap_text, chunk])

                if len(overlap_text) > chunk_overlap or len(combined_with_new) > chunk_size:
                    break
                overlap_buffer = candidate_overlap
            buffer = overlap_buffer + [chunk]
        else:
            buffer = candidate_buffer

    if buffer:
        result_chunks.append(separator.join(buffer))
    return result_chunks

def split_text(text, chunk_size, chunk_overlap, separator_index = 0):
    separators = ["\n\n", "\n", " ", ""]
    separator = separators[separator_index]

    if separator == "":
        step = chunk_size - chunk_overlap
        return [text[i:i + chunk_size] for i in range(0, len(text), step)]

    chunks = [c for c in text.split(separator) if c.strip() != ""]

    good_chunks = []
    final_chunks = []

    for chunk in chunks:
        if len(chunk) <= chunk_size:
            good_chunks.append(chunk)
        else:
            if good_chunks:
                final_chunks.extend(merge_with_overlap(good_chunks, separator, chunk_size, chunk_overlap))
                
                good_chunks = []
                
            if separator_index + 1 < len(separators):
                final_chunks.extend(split_text(chunk, chunk_size, chunk_overlap, separator_index + 1))
            else:
                final_chunks.append(chunk)
                
    if good_chunks:
        final_chunks.extend(merge_with_overlap(good_chunks, separator, chunk_size, chunk_overlap))

    return final_chunks

if __name__ == "__main__":

    text = """Harry, Ron, and Hermione squeezed inside. A long line wound right to the back of the shop, where Gilderoy Lockhart was signing his books. They each
grabbed a copy of The Standard Book of Spells, Grade 2 and sneaked up the
line to where the rest of the Weasleys were standing with Mr. and Mrs.
Granger.
“Oh, there you are, good,” said Mrs. Weasley. She sounded breathless and
kept patting her hair. “We’ll be able to see him in a minute. . . ."""

    chunks = split_text(text, chunk_size=300, chunk_overlap=80)
    for i, chunk in enumerate(chunks):
        print(f"Chunk {i + 1}:\n{chunk}\n{'-' * 40}")
```

**Recursive Character Text Splitting With LangChain:**

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

def chunking_with_recursive_splitter(text, chunk_size, chunk_overlap):
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        length_function=len
    )

    chunks = text_splitter.split_text(text)
    return chunks

result_recursive = chunking_with_recursive_splitter(text, chunk_size=80, chunk_overlap=20)

print(result_recursive)
```


------

## Level 3: Document Specific Splitting

Document Specific Chunking adapts chunking strategy for different data formats ( Markdown, Python, JS etc.)

Pros: Keeps structurally related concepts like paragraphs or headings intact. Metadata allows for deep hierarchical tagging.

Cons: Requires custom parsers and complex logic to handle different file types. It also results in variable chunk sizes and a strong source-file dependecy.


**Document Specific Splitting With Langchain for Markdown**

```python
from langchain_text_splitters import MarkdownTextSplitter

splitter = MarkdownTextSplitter(chunk_size=100, chunk_overlap=20)

markdown_text = """**What is Chunking?**
Chunking is the process of break down large text into small pieces called chunks.  
**Why do we need chunking for our applications?**
Chunking is neccessary for fit data in the models context window and finding true information from database correctly.  
**Chunking’s role in semantic search**
Chunking size is has to be optimal lenght for model's semantic search. If chunks are too small or too large it effects  related search badly.
**Chunking’s role for agentic applications and retrieval-augmented generation**
If an agent misinformed because of bad chunking strategy, it may waste tokens, hallucinations or calling wrong tools.
Even context windows enough, if chunks are too long, the LLM models suffer from[ the lost-in-the-middle problem](https://arxiv.org/abs/2307.03172)"""

print(splitter.create_documents([markdown_text]))
```

## Level 4: Semantic Chunking

Semantic Chunking focuses on semantic context rather than fixed token limits. It divides text into meaningful chunks by converting senteces into embeddings, calculating their vector similarity (cosine similarity etc.) and grouping contextually related sentences together. 

**Pros:**
- Improved Retrieval - which delivers highly relevant.
- Smarter segmentation
- Contex Preservation

**Cons:** 
- Implemantation Complexity
- High Computational Cost
- Variable sizes ( unpredictable lenghts )

**Semantic Chunking From Scratch:**

```python
import re
from openai import OpenAI
from sklearn.metrics.pairwise import cosine_similarity
from dotenv import load_dotenv
import numpy as np

load_dotenv()
client = OpenAI()

with open("makale.txt", "r", encoding="utf-8") as f:
    essay = f.read()

# Notes for re: ?<=[.!?] is a positive lookbehind assertion that matches any position in the string that is preceded by a period, exclamation mark, or question mark. The \s+ matches one or more whitespace characters (spaces, tabs, newlines, etc.).

single_sentences_list = [

    s for s in re.split(r"(?<=[.!?])\s+", essay) if s.strip() != ""

]

sentences = [

    {"sentence": sentence, "index": index}

    for index, sentence in enumerate(single_sentences_list)

]

def combine_sentences(sentences, buffer_size=1):

    for i in range(len(sentences)):
        combined_sentence = ""

        for j in range(i - buffer_size, i):
            if j >= 0:
                combined_sentence += sentences[j]["sentence"] + " "

        combined_sentence += sentences[i]["sentence"] + " "

        for j in range(i + 1, i + buffer_size + 1):
            if j < len(sentences):
                combined_sentence += sentences[j]["sentence"] + " "
        sentences[i]["combined_sentence"] = combined_sentence

    return sentences

sentences = combine_sentences(sentences)

text_list = [item["combined_sentence"] for item in sentences]

response = client.embeddings.create(model="text-embedding-3-small", input=text_list)

for i, sentence in enumerate(sentences):
    sentence["combined_sentence_embedding"] = response.data[i].embedding

def calculate_cosine_similarity(sentences):

    distances = []

    for i in range(len(sentences) - 1):
    
        embedding_current = sentences[i]["combined_sentence_embedding"]
        embedding_next = sentences[i + 1]["combined_sentence_embedding"]
        similarity = cosine_similarity([embedding_current], [embedding_next])[0][0]

        distance = 1 - similarity
        distances.append(distance)
        sentences[i]["distance_to_next"] = distance

    return distances, sentences

def treshold_and_chunk_sentences(distances, sentences, breakpoint_percentile_treshold=95):

    breakpoint_distance_treshold = np.percentile(distances, breakpoint_percentile_treshold)

    indices_above_tresh = [i for i, distance in enumerate(distances) if distance > breakpoint_distance_treshold]

    start_index = 0
    chunks = []

    for index in indices_above_tresh:
        group = sentences[start_index:index + 1]
        combined_text = " ".join([item["sentence"] for item in group])
        chunks.append(combined_text)
        start_index = index + 1

    if start_index < len(sentences):

        combined_text = " ".join([item["sentence"] for item in sentences[start_index:]])

        chunks.append(combined_text)

    return chunks

distances, sentences = calculate_cosine_similarity(sentences)
chunks = treshold_and_chunk_sentences(distances, sentences)


for i , chunk in enumerate(chunks):
    print(f"\n---- Chunk {i + 1} ---\n")
    print(chunk)
```


