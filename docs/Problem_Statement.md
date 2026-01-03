# 📄 Problem Statement: Local-First Semantic Audio Intelligence

## The Vision
In an era of abundant audio content—podcasts, lectures, meetings—finding specific information remains archaic. We rely on scrubbing through timelines or searching for exact keywords (Ctrl+F), which fails when concepts are discussed using synonyms or metaphors (e.g., searching for "sadness" won't find "feeling blue").

**InsightCast** aims to democratize audio intelligence by building a tool that "understands" audio content, allowing users to ask natural language questions like *"What is the speaker's stance on privacy?"* or *"Give me an example of intuition"* and jump instantly to that moment.

## The Constraints
Existing solutions rely on heavy cloud infrastructure (OpenAI Whisper API, Pinecone, Python backends). This introduces:
1.  **Privacy Risks:** Sensitive audio leaves the user's device.
2.  **Latency:** Uploading large files takes time.
3.  **Cost:** Pay-per-minute API usage.

## The Challenge
**Can we build a production-grade Semantic Search engine that runs 100% locally in the browser?**

## Initial Approach
Our initial hypothesis was simple:
1.  Use a small Whisper model in the browser for transcription.
2.  Use a lightweight embedding model to vectorize the transcript.
3.  Perform a simple cosine similarity search.

**Why it wasn't enough:**
Early tests showed that while this worked for literal matches, it failed at "understanding."
* *Fragmentation:* Whisper breaks sentences based on pauses, often slicing a thought in half.
* *Context Loss:* Searching for "examples" failed because the example itself (e.g., "a sad friend") didn't contain the keyword "example" or the main topic "intuition."
* *Blind Spots:* Proper nouns (like author names) were often missed by pure vector search if the model hadn't seen them frequently during training.