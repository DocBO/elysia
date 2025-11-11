Within the Elysia package, you can either call the FastAPI server (API usage) or import Elysia as a Python library (in‑project usage). Choose the path that fits your stack and deployment model.

## API Usage Workflow
- Run the server: `elysia start --host 0.0.0.0 --port 8000 --reload`.
- Initialise: `POST /init/user/{user_id}` then `POST /init/tree/{user_id}/{conversation_id}`.
- Execute queries via WebSocket: connect to `/ws/query` and send JSON with `user_id`, `conversation_id`, `query`, `query_id`, and optional `collection_names`. Streamed messages include `ner`, tool outputs, optional `title`, and a final `completed` payload.
- Manage data and settings using REST:
  - Collections: list/view/metadata under `/collections/...`
  - Config: `/user/config/...` for user defaults, `/tree/config/...` for per‑conversation
  - Feedback: `/feedback/...`, Saved trees: `/db/...`

Example (list collections):
```bash
curl -s localhost:8000/collections/alice/list | jq .
```

## In‑Project Usage Workflow
- Install: `pip install elysia-ai`
- Create a tree and (optionally) tools; configure via `.env` or programmatically.
```python
from elysia import Tree, tool

tree = Tree()

@tool(tree=tree)
async def add(x: int, y: int) -> int:
    return x + y

response, objects = tree(
    "Top 10 expensive items?",
    collection_names=["Ecommerce"],
)
```
- For advanced control: adjust `style/agent_description/end_goal`, switch models, or use `await tree.get_follow_up_suggestions_async()`.

Notes
- API usage is best for frontend/web integrations and multi‑language clients.
- In‑project usage is best for Python services, batch jobs, or custom pipelines.
