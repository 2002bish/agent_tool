Markdown
# Local RAG Poisoning PoC: Ollama, Chroma, and the Behavior Gap Between DeepSeek-R1 and Mistral

This project provides a local, end-to-end Proof-of-Concept (PoC) demonstrating **Indirect Prompt Injection via Vector Database (RAG) Poisoning**. 

While air-gapped local setups (using tools like Ollama) protect enterprise data from leaking outward, this simulation proves a critical flaw: **vector similarity math does not care about intent.** An attacker who successfully injects a malicious document into an ingestion path can compromise downstream outputs.

---

## 🏗️ Architecture & Attack Flow

```text
[ Attacker Payload ] ──┐
                       ▼
[ Corp Knowledge ] ──> [ Chroma DB Vector Search ] 
                              │
                       (Blind Match)
                              │
                              ▼
                     [ LLM Context Window ] 
                     (Poison + Policy Clash)
Ingestion Layer: Local text files (legit_kb.txt and poisoned_doc.txt) are embedded using nomic-embed-text.

Vector Store: Embeddings are persisted locally via Chroma (langchain-chroma).

Retrieval Phase: A user query matches the closest semantic vector chunks—pulling the attacker's override payload directly into the context window.

Generation Phase: The context is evaluated by a local LLM via Ollama.

🛠️ Tech Stack
Orchestration: LangChain (langchain-core, langchain-chroma, langchain-ollama)

Vector Database: ChromaDB

Local Engine: Ollama (nomic-embed-text, deepseek-r1, mistral)

📂 Project Structure
Plaintext
agent_tool/
├── app.py                # Main execution pipeline script
├── legit_kb.txt          # Authentic corporate security policy
├── poisoned_doc.txt      # Injected attacker payload with system override
└── requirements.txt      # Project dependencies
🚀 Getting Started
1. Clone & Install Dependencies
Bash
git clone https://github.com/2002bish/agent_tool.git
cd agent_tool
pip install -r requirements.txt
(Ensure your requirements.txt contains: langchain, langchain-ollama, langchain-chroma, chromadb)

2. Pull Local Ollama Models
Make sure your Ollama service is running, then pull the required models:

Bash
ollama pull nomic-embed-text
ollama pull deepseek-r1:latest
ollama pull mistral
3. Run the Simulation
Execute the script in your terminal:

Bash
python app.py
🔬 Behavioral Results: Reasoning vs. Instruction-Tuned Models
Because vector retrieval blindly pulls top matching chunks, both models receive the poisoned document in their context window. However, their architectural differences produce drastically different outcomes:

DeepSeek-R1 (Reasoning Model)
Behavior: Utilizes an internal <think> reasoning trace to analyze and weigh conflicting constraints.

Outcome: Successfully filters out the malicious system override and anchors its final response strictly to the legitimate corporate security email (security@company.com).

Mistral (Instruction-Tuned Model)
Behavior: Processes instructions sequentially based on compliance training.

Outcome: Acknowledges the legitimate policy, but ingests, surfaces, and repeats the malicious phishing link (https://malicious-fake-portal.com), demonstrating vulnerability to context blending and indirect prompt injection.