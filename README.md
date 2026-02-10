 Multi-Agent Financial RAG & Portfolio Optimization System

## 1. Executive Summary
**This** is a *self-improving* Financial RAG system that synthesizes **real-time news** with **historical macro scenarios** to produce optimized **asset allocation guidance**.  
Unlike standard retrieval-only pipelines, OptiQ uses a **Multi-Agent Debate framework (Bull vs. Bear)** to surface market divergences, then converts those insights into **quantitative parameter adjustments**, and finally runs **constrained numerical optimization (SLSQP)** to output an end-to-end investment strategy.

---

## 2. Core Architecture & Algorithms

### 2.1 Multi-Agent Inference Workflow (LangGraph)
To reduce single-LLM bias, OptiQ uses a **stateful, agentic graph** built with **LangGraph**:

- **Debate Nodes**
  - **Bullish Strategist**: constructs the most compelling pro-risk / upside thesis from the current news context.
  - **Bearish Analyst**: constructs the strongest downside / risk thesis using the same evidence.

- **Judge Node (CIO)**
  - Synthesizes Bull vs. Bear into a **Consensus** view.
  - Produces interpretable scores such as:
    - **Market Trend Score**
    - **Risk Aversion Score**
  - Explicitly notes **price–news divergence** signals (e.g., “negative headlines, but credit spreads tightening”).

- **Evaluator Node (Feedback Loop)**
  - Validates the reasoning against:
    - the raw retrieved evidence
    - the original user query
  - If the **logic score** falls below a threshold (e.g., **8/10**), OptiQ triggers a **self-refinement loop**, routing agents back to the debate stage via conditional edges.

---

### 2.2 Hybrid RAG Strategy
- **Vector DB (FAISS)**
  - Indexes a **Scenario Knowledge Base (KB)** of historical macro regimes/analogues (e.g., *Deleveraging*, *Goldilocks*, *Liquidity Crunch*).

- **Dynamic Search Querying**
  - Instead of embedding the raw user question, OptiQ embeds the **CIO Consensus** and uses it as the search vector.
  - This retrieves the most statistically relevant historical **Anchor Scenario** and supporting metadata.

---

### 2.3 Quant Engine & Optimization
- **Parameter Tuning**
  - The LLM adjusts scenario-derived statistics based on the **Divergence Note** and current conditions:
    - $$\mu$$ (expected returns)
    - $$\sigma$$ (volatility)
    - correlations / covariances

- **SLSQP Optimizer (SciPy)**
  - Solves for optimal weights $$w^*$$ by maximizing Sharpe Ratio under constraints:

  - **Weight Bounds**
    - $$0 \le w_i \le 0.45$$

  - **Full Investment**
    - $$\sum_i w_i = 1.0$$

---

## 3. Implementation Details

| Requirement | Implementation Detail |
|---|---|
| Data preprocessing | Structured financial scenario KB with embedding vectorization |
| Vector DB indexing | High-speed retrieval via `langchain_community.vectorstores.FAISS` |
| Relevant doc search | Scenario metadata extraction via `similarity_search` |
| LLM generation | Local Llama integration with Pydantic-based structured output |
| Accuracy improvement | Iterative refinement via Evaluator node + conditional edges |
| REST API | FastAPI endpoints: `/analyze` (analysis) and `/metrics` (performance) |
| Performance tracking | `PerformanceTracker` for latency, eval score, retry rate |

---

## 4. System Algorithm (Workflow)

**Input:** User financial query  
> Example: “Impact of Fed rate cut expectations on the KOSPI option market”

1. **Debate**: Bull and Bear agents produce adversarial arguments from the latest news.
2. **Judgement**: CIO agent forms **Consensus**, scoring sentiment/trend and detecting price–news divergence.
3. **Evaluation**: Critically reviews logic.  
   - **Pass** → proceed  
   - **Fail** → loop back to Debate
4. **Retrieval**: Search FAISS using **Consensus embedding** to fetch historical analogues (Anchor Scenarios).
5. **Estimation**: Combine anchor scenario stats + current indicators to estimate:
   - $$\mu$$ (expected returns)
   - $$\Sigma$$ (covariance matrix)
6. **Optimization**: Run SLSQP to compute $$w^*$$.
7. **Output**: Deliver a “Manager View” plus actionable portfolio guidance.

---

## 5. Tech Stack
- **Language**: Python 3.10+
- **Frameworks**: LangChain, LangGraph, FastAPI
- **LLM**: Local Llama (OpenAI-compatible API)
- **Vector DB**: FAISS
- **Math/Stats**: NumPy, SciPy (SLSQP), Pydantic
- **Infra**: Uvicorn, python-dotenv

---

## 6. Performance & Metrics
A `PerformanceTracker` monitors system health and quality:

- **Latency**
  - Multi-agent debate increases response time but improves reasoning depth vs. zero-shot prompting.
- **Retry Rate**
  - Tracks how often the Evaluator detects inconsistencies and triggers refinement (proxy for reliability).
- **Eval Score**
  - Self-scoring using financial expert personas to support continuous prompt/system optimization.

---

## Appendix: Quick Start

1. Configure environment variables in `.env`.
2. Ensure the local LLM server is running on **port 8090**.
3. Run:
   - `python main.py`
4. Open API docs:
   - `http://localhost:8088/docs`

### Example Request
````json
{
  "question": "How will the nomination of Kevin Warsh affect the Korean option market?"
}
