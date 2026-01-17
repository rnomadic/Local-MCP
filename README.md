# Building a Local MCP-Powered RAG

<img width="1100" height="733" alt="image" src="https://github.com/user-attachments/assets/4002ae57-0c07-4664-a572-8845a9bf1e09" />

## What we are trying to build
We will be build a MCP setup that can query over 200+ different sources — all from a single chat interface — using:

1. mcp-use (a handy tool from YC S25) for the local MCP client.
2. MindsDB for hooking into all those data sources.
3. Ollama to run open-source LLMs locally.

## Workflow
```
You → MCP Client → MindsDB MCP Server → Your Data Sources → LLM → Answer
```
## STEPS
### Step 1: Run Mindsdb locally
We’ll run MindsDB in Docker. Assuming you have docker installed in your desktop <br> 
Open your terminal and run:
```
docker run -it -p 47334:47334 mindsdb/mindsdb
```
### Step 2: Spinup your Mindsdb GUI
In your browser type
```
http://localhost:47334
```
MindsDB editor — kind of like a SQL workbench will pop up. Now you can connect to hundreds of different sources: Slack, Gmail, Salesforce, MySQL.

### Step 3: Connect your Data
```
-- Slack
CREATE DATABASE slack_db
WITH ENGINE = "slack",
PARAMETERS = {
  "token": "your-slack-api-token"
};

-- Gmail
CREATE DATABASE gmail_db
WITH ENGINE = "gmail",
PARAMETERS = {
  "credentials": "gmail_credentials.json"
};
-- GitHub
CREATE DATABASE github_db
WITH ENGINE = "github",
PARAMETERS = {
  "token": "your-github-token"
};
-- Hacker News
CREATE DATABASE hn_db
WITH ENGINE = "hacker_news";

```
You can fetch data with regular sql type query.

```
SELECT * FROM gmail_db.inbox WHERE is_unread = true;
```
### Step 4: Connecting MCP server and LLM
MindsDB server can talk to our data. But how will AI agent can connect to this server.
We need to make a server config like below JSON structure and pass it to mscp client.

```
{
  "servers": {
    "mindsdb": {
      "url": "http://localhost:47334",
      "tools": ["list_databases", "query"]
    }
  }
}
```
### Step 5: MCP client and LLM
Below is the example with llama3 using ollama but we can swap the LLM by changing "llama3" to "mistral" or "gemma2".
```
from mcp_use import MCPClient
from ollama import ChatModel

# 1. Load MCP client config
client = MCPClient.from_config("mcp_config.json")
# 2. Connect local LLM
llm = ChatModel("llama3")
# 3. Create agent
agent = llm.bind(client)
# 4. Run a query
response = agent.ask("Show me all unread Slack messages from this week.")
print(response)
```





