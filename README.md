# NovaMart Multi-Agent Customer Support System (Strands + Amazon Bedrock AgentCore)

*Project from the Udacity Future AWS Agent Engineer program.*

## Summary
An enterprise-style customer-support system where an **Orchestrator agent** (Claude Haiku 4.5, fast routing) sends each request to specialist worker agents (Claude Sonnet 4.5):
- **InventoryAgent**: order and customer facts from DynamoDB
- **PolicyAgent**: multi-agent RAG; Returns, Shipping and Warranty retrievers query three Bedrock Knowledge Bases **in parallel**
- **RefundAgent**: refund/return decisions using DynamoDB data
- **CommunicationAgent**: composes the final customer reply

Shared workflow state lives in a DynamoDB `WorkflowStateTable` that each routing step reads and updates.

## Platform features used
- AgentCore Runtime deployment (via the AgentCore CLI)
- Bedrock Guardrails: content, PII, denied topics (competitors, legal threats) and word policies, published as a numbered version
- AgentCore Memory with a session-summary strategy (7-day event expiry)
- Bedrock Knowledge Bases with `amazon.titan-embed-text-v2:0` embeddings and S3 Vectors
- Observability: CloudWatch logs at INFO and X-Ray tracing (100% sampling, Transaction Search)

## Architecture
```
Customer -> OrchestratorAgent -> Inventory | Policy (3 parallel KB retrievers) | Refund | Communication
                     \__ DynamoDB WorkflowState __/
```

## How to run
Copy `.env.example` to `.env` and fill in your own IDs (never commit `.env`).
```bash
python src/agent_orchestrator.py test     # 3 scenarios, traced to X-Ray
python src/agent_orchestrator.py deploy   # guardrail, runtime, memory, observability
python src/agent_orchestrator.py invoke "Where is my order?"
```
Note: `agent_orchestrator.py` imports helper modules from the course starter kit (`config`, `bedrock_kb_retrieval`, `agent_utils`, `agent_observability`), which are not included here.

## Results (from my AWS account, us-east-1)
Course validation tests, Tasks 2–6, all passed ([screenshot](evidence/validation-tasks-2-to-6.png)):

| Task | Area | Score |
|---|---|---|
| 2 | Multi-agent orchestration | 40/40 |
| 3 | AgentCore deployment + guardrails | 20/20 |
| 4 | Memory | 15/15 |
| 5 | Bedrock Knowledge Bases | 25/25 |
| 6 | Observability | 20/20 |

All three Knowledge Bases returned passages, and the combined parallel policy search returned in 7.3 s.

The [X-Ray trace map](evidence/xray-trace-map.png) shows the orchestrator calling every worker agent and all three Knowledge Base retrievers.

## Skills demonstrated
Multi-agent orchestration, parallel RAG, AI guardrails, agent memory, serverless agent deployment, and production observability on AWS.
