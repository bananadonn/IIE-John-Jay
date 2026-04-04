# Week 7: RAG Security Knowledge Assistant — Evaluation Report

**Name:** Donald Reith
**Date:** 2026-04-04
**Capstone Project:** Automated Threat Intelligence Hub
**My Component:** Feed Collector

---

## 1. Setup Summary

- **LLM:** llama-3.3-70b-versatile via Groq
- **Embeddings:** sentence-transformers/all-MiniLM-L6-v2 via HuggingFace Inference API
- **Vector Store:** In-Memory Vector Store
- **Documents loaded:**
  - mitre-initial-access.txt (~3 pages) — MITRE ATT&CK Initial Access techniques including phishing, drive-by compromise, valid accounts, and supply chain compromise
  - mitre-credential-access.txt (~3 pages) — MITRE ATT&CK Credential Access techniques including brute force, OS credential dumping, input capture, and session cookie theft
  - mitre-lateral-movement.txt (~3 pages) — MITRE ATT&CK Lateral Movement techniques including remote services, pass the hash, internal spearphishing, and lateral tool transfer

**Chat link:** https://cloud.flowiseai.com/chatbot/50c88339-7d1f-4389-bfd6-334919a8cfaf

---

## 2. Test Results

| # | Question | Used Documents? | Quality | Notes |
|---|----------|----------------|---------|-------|
| 1 | What are common techniques attackers use for credential access according to MITRE ATT&CK? | Yes | Good | Correctly pulled NTDS, LSA Secrets, Cached Domain Credentials, and DCSync from the credential access document with accurate technique IDs |
| 2 | How does phishing relate to initial access and what are the different sub-techniques? | Yes | Good | Correctly identified spearphishing attachment and spearphishing link, and also pulled in internal spearphishing from the lateral movement document showing cross-document retrieval working |
| 3 | What is lateral movement and what techniques do adversaries use to move through a network? | Yes | Good | Accurately defined lateral movement and listed Remote Services and Internal Spearphishing with correct tactic ID TA0008 |
| 4 | What is Pass the Hash and how do adversaries use it for lateral movement? | Yes | Partial | The chatbot acknowledged Pass the Hash was not explicitly described in context and fell back to related techniques instead of hallucinating — honest but incomplete |
| 5 | What is the difference between spearphishing attachment and spearphishing link? | Yes | Good | Clearly explained the difference between the two sub-techniques with accurate descriptions matching the uploaded document |

---

## 3. Edge Case Observations

- **Unrelated question ("What is the weather like today?"):** The chatbot responded with "Hmm, I'm not sure" rather than making something up. This is the correct behavior — it recognized the question was outside its knowledge base and did not attempt to hallucinate an answer.

- **Topic not in documents ("What are the latest CVEs discovered in 2026?"):** Same result — "Hmm, I'm not sure." Even though CVEs are a related cybersecurity topic, the chatbot correctly recognized that this specific information was not in the uploaded documents and declined to answer rather than fabricating CVE data. This is important for a security context where made-up vulnerability information could be genuinely harmful.

---

## 4. Settings Experiments

No settings experiments were conducted this week. Future testing would include adjusting temperature, chunk size, and Top K retrieval to observe the effect on answer quality and specificity.

---

## 5. Reflection

What surprised me most about this lab was the ability to upload files that would then be used as the context for the chatbot. With many of the available AI models, such as ChatGPT or Claude, there are limits to the number of files that can be uploaded, the number of questions that can be asked, or the file becomes inaccessible. However, using the RAG system, there is no cap on the number of questions that can be asked or the file accessed by the chatbot. This allows for more context to be pushed into the model, which presents opportunities for improving the responses.
