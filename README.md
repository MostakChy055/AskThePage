# AskThePage
Attempt to find correct information from right site!



## What is an Intelligent Crawler?

A traditional crawler (or "spider") follows a rigid set of rules: "Go to this URL, find all `<a>` tags, and download the HTML."

An **Intelligent Crawler** uses AI (often LLMs) to navigate and extract data more like a human would. Instead of just grabbing raw code, it understands the **context** and **structure** of a page. In the context of RAG, an intelligent crawler:

*   **Identifies Relevant Content:** It can distinguish between the actual article and "noise" like ads, headers, footers, or legal disclaimers.
*   **Handles Navigation:** It can figure out that it needs to click "Load More" or "Next Page" to get the full dataset.
*   **Adapts to Layout Changes:** If a website changes its design, a traditional scraper breaks. An intelligent crawler recognizes the *intent* (e.g., "Find the price") and keeps working.

---

## What is a Headless Browser (Playwright)?

To understand **Playwright**, you first need to understand the **Headless Browser**.

*   **Headless Browser:** This is a web browser (like Chrome or Firefox) that runs without a graphical user interface (GUI). It does everything a normal browser does—downloads files, executes JavaScript, renders CSS—but it does it in the background without a window popping up on your screen.
*   **Playwright:** This is an open-source automation library built by Microsoft. Think of it as the "remote control" for the headless browser. It allows you to write code to tell the browser to "Click this button," "Wait for that image to load," or "Type this into the search bar."



---

## 3. Why is this recommended for JS-heavy sites?

This is the "aha!" moment for RAG developers. 

Many modern websites (built with React, Vue, or Angular) are **JavaScript-heavy**. When you visit these sites, the initial HTML file sent by the server is almost empty—it’s just a skeleton with a bunch of script tags. The actual content is generated *inside your browser* after the JavaScript runs.

### The Problem: Traditional Scraping
If you use a basic tool like Python's `requests` or `BeautifulSoup`, you are only downloading that initial "skeleton" HTML. Because these tools can't "run" JavaScript, they see an empty page. Your RAG system ends up indexed with "Loading..." text instead of actual data.

### The Solution: Playwright
Because Playwright drives a real browser engine, it **waits for the JavaScript to execute.** It renders the page exactly as a human would see it. 

### How Playwright solves this
Playwright doesn't just "download" the text. **Playwright launches an actual instance of Chromium (the engine behind Chrome).**

1.  **The Request:** Playwright tells Chromium to go to your URL.
2.  **The Execution:** Chromium downloads the HTML and the JS.
3.  **The Wait:** Chromium executes the JS, fetches data from APIs, and populates the tags.
4.  **The Hand-off:** Playwright waits until the page looks "finished" (you can even tell it to wait for a specific button to appear) and *then* gives you the HTML.

## Chunking Strategies

If fixed-size chunking is a **chainsaw** (effective but messy), and recursive chunking is a **scalpel**, then **Semantic** and **Agentic** chunking are the "AI-powered" methods that actually try to understand what the text is saying before they cut.


## Semantic Chunking
**The Core Idea:** "Split the text only when the topic changes."

Instead of counting characters, Semantic Chunking uses **embeddings** to measure the relationship between sentences. It looks for "meaning breaks."

### How it works:
1.  **Sentence Splitting:** It breaks the document into individual sentences.
2.  **Vectorization:** It turns every sentence into an embedding (a vector).
3.  **Distance Calculation:** It looks at the "distance" (similarity) between Sentence A and Sentence B.
4.  **The Cut:** If the distance between two consecutive sentences exceeds a certain threshold (meaning the topic just shifted significantly), it creates a new chunk.

*   **Pros:** Chunks are highly coherent. You won't have a chunk that starts with the end of a recipe and ends with the beginning of a weather report.
*   **Cons:** It’s computationally expensive because you have to generate embeddings for *every* sentence in your dataset before you even start indexing.

---

## Agentic Chunking
**The Core Idea:** "Use an LLM as an editor to decide where the boundaries should be."

This is the most advanced (and "expensive") form of chunking. Instead of using math (vectors), you use an **Agent** (an LLM with a specific prompt) to "read" the document and perform the layout.

### How it works:
The Agent is given a task: *"You are an expert document editor. Read the following text and break it into logical sections that can stand alone as independent pieces of knowledge."*

1.  **Initial Proposition:** The agent looks at a piece of text and proposes a chunk.
2.  **Contextual Check:** It looks at the next few sentences. It asks itself, "Does this new info belong with the previous part, or is this a new concept?"
3.  **Adjustment:** The agent can grow, shrink, or merge chunks based on the **intent** of the content, not just the word similarity.

*   **Pros:** Incredible accuracy. It understands nuance, sarcasm, and complex transitions that mathematical "semantic" chunking might miss.
*   **Cons:** **Very slow and very expensive.** You are essentially paying for an LLM to read your entire database just to prepare it for another LLM to read later.

