# Week 4: Model Comparison

Tested 4 AI models on 5 cybersecurity text samples to evaluate their suitability for the feed collector component of our Automated Threat Intelligence Hub.

## Models Tested

- HF Sentiment (distilbert-sst-2)
- HF Zero-Shot (bart-large-mnli)
- HF NER (bert-large-NER)
- Groq Llama 3 8B

## Finding

Recommended Groq Llama 3 8B for the feed collector because it correctly classified all 5 threat intelligence entries with appropriate severity levels (CRITICAL/HIGH/INFORMATIONAL) and human-readable reasoning, making it the most suitable model for normalizing and triaging incoming threat feed data.

See `report.md` for full analysis.
