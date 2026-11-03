# 🛡️ Guardrails — Crash Course

A practical Jupyter Notebook demonstrating how to implement **AI guardrails in LangChain agent systems**. The notebook explores input filtering, PII protection, human approval for sensitive tools, output validation, and layered safety controls.

## 📚 What You'll Learn

1. What guardrails are and why they matter in AI agent systems
2. Deterministic vs. model-based guardrails
3. Built-in PII detection and handling with `PIIMiddleware`
4. Human approval workflows with `HumanInTheLoopMiddleware`
5. Custom input filtering with the `before_agent()` hook
6. Custom output validation with the `after_agent()` hook
7. Combining multiple guardrails into a layered middleware stack
8. A healthcare chatbot example using multiple safety layers

## 🧠 What Are Guardrails?

Guardrails are safety mechanisms that control what enters and leaves an AI agent and help ensure that the agent:

- Processes safe and appropriate inputs
- Performs only approved actions
- Returns validated and policy-compliant outputs

In this notebook, guardrails are implemented as middleware that can intercept agent execution before or after the agent runs, and around model or tool operations.

### Common Use Cases

| Use case | Example |
|---|---|
| PII protection | Redact email addresses or credit-card details |
| Prompt-injection/content filtering | Detect and block suspicious or disallowed inputs |
| Harmful-content filtering | Block unsafe requests |
| Business-rule enforcement | Require approval before sensitive operations |
| Output validation | Check responses before returning them to users |

## ⚖️ Guardrail Approaches

### 1. Deterministic Guardrails

Rule-based checks such as regular expressions, keyword matching, and explicit conditions.

- **Advantages:** Fast, predictable, and cost-effective
- **Limitations:** May miss nuanced or context-dependent violations

### 2. Model-Based Guardrails

Use an LLM or classifier to evaluate content semantically.

- **Advantages:** Can identify subtle or context-dependent issues
- **Limitations:** Typically slower and more expensive than simple rule-based checks

The notebook demonstrates both approaches, including a Groq-hosted model check.

## 🔒 Built-in Middleware

### PII Detection — `PIIMiddleware`

The notebook demonstrates handling these PII categories:

| PII type | Example |
|---|---|
| `email` | `user@example.com` |
| `credit_card` | `5105-1051-0510-5100` |
| `ip` | `192.168.1.1` |
| `mac_address` | `00:1A:2B:3C:4D:5E` |
| `url` | `https://secret-site.com` |

Strategies shown:

- `redact` — replaces detected data with a redaction marker
- `mask` — obscures sensitive characters
- `hash` — replaces the value with a hash
- `block` — blocks processing by raising an exception

### Human Approval — `HumanInTheLoopMiddleware`

Pauses execution before configured sensitive tool operations and waits for human approval.

Example operations in the notebook include:

- Sending an email
- Deleting records
- Other operations with significant business impact

**Important:** Human-in-the-loop execution requires a checkpointer so the agent state can be persisted across interruptions. The notebook uses LangGraph's `InMemorySaver` for its demonstration.

## ⚙️ Custom Middleware

### Before-Agent Input Filter

The custom `ContentFilterMiddleware` uses `before_agent()` to inspect requests before LLM processing begins.

Example uses:

- Keyword/content filtering
- Authentication checks
- Rate limiting
- Blocking disallowed request categories

### After-Agent Output Validator

The custom `SafetyGuardrailMiddleware` uses `after_agent()` to inspect the agent's final response before it is shown to the user.

Example uses:

- Model-based safety evaluation
- Compliance checks
- Response-quality validation
- Detecting or removing sensitive information

## 🧱 Layered Guardrails

The notebook combines multiple middleware components to provide defense in depth:

```text
User Input
    |
    v
[1] ContentFilterMiddleware
    |  Deterministic input filtering
    v
[2] PIIMiddleware
    |  Input PII handling
    v
[3] HumanInTheLoopMiddleware
    |  Approval for sensitive tool calls
    v
[4] PIIMiddleware
    |  Output PII handling
    v
[5] SafetyGuardrailMiddleware
    |  Model-based output safety check
    v
User Response
```

The combined example configures middleware through the `middleware=[]` argument when creating a LangChain agent.

## 🏥 Healthcare Chatbot Example

The final example applies several guardrails to a healthcare chatbot:

1. Blocks harmful or off-topic requests.
2. Redacts patient email addresses and masks credit-card details in inputs.
3. Requires human approval before booking appointments.
4. Adds a medical disclaimer and validates the chatbot's output.

The example includes tools for symptom search, appointment booking, and medication information. It is an educational demonstration, not a clinically validated medical system.

## 🧰 Tech Stack

- Python
- Jupyter Notebook
- LangChain Agents and Middleware
- LangGraph
- Groq (`ChatGroq`)
- `python-dotenv`

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd <your-repository-folder>
```

### 2. Create and activate a virtual environment

**Windows PowerShell:**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install jupyter langchain langgraph langchain-groq python-dotenv
```

Use compatible, current package versions for the LangChain and LangGraph APIs used by the notebook.

### 4. Configure the Groq API key

The model-based guardrail example uses `ChatGroq` with the model `openai/gpt-oss-20b`.

Create a `.env` file in the project directory:

```env
GROQ_API_KEY=your_groq_api_key
```

Keep your API key private. Do not commit `.env` to GitHub. Add it to `.gitignore`:

```gitignore
.env
.venv/
.ipynb_checkpoints/
__pycache__/
```

### 5. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open `13_guardrails_crash_course.ipynb` and run the cells in order. The model-based examples require a valid Groq API key; the rule-based examples do not require an LLM API call.

## 📂 Project Structure

```text
.
├── 13_guardrails_crash_course.ipynb
├── README.md
└── .env                  # Create locally; do not commit
```

The notebook also references illustrative images under an `images/` directory. Include those image files in the repository if you want the notebook's embedded illustrations to render.

## 🔑 Key Takeaways

1. **Guardrails are implemented as middleware** using the `middleware=[]` parameter in `create_agent()`.
2. **Layering guardrails provides defense in depth.** Combine input checks, privacy controls, approval steps, and output checks.
3. **Use deterministic checks early** for fast, inexpensive filtering; use model-based checks when semantic evaluation is needed.
4. **Human-in-the-loop workflows need a checkpointer.** `InMemorySaver` is used for development demonstrations; production systems should use an appropriate persistent checkpointer.
5. **Custom middleware provides control** through hooks such as `before_agent()` and `after_agent()`.

## 📖 References

- [LangChain Guardrails](https://docs.langchain.com/oss/python/langchain/guardrails)
- [LangChain Middleware Overview](https://docs.langchain.com/oss/python/langchain/middleware/overview)
- [LangChain Human-in-the-Loop](https://docs.langchain.com/oss/python/langchain/human-in-the-loop)
- [LangSmith Observability](https://docs.langchain.com/oss/python/langchain/observability)

---

**Note:** This repository is intended for learning and experimentation. Review middleware behavior, test edge cases, and adapt the policies to your application's requirements before using guardrails in production.
