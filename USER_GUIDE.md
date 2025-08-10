# Local Deep Researcher: User Guide

This guide will walk you through setting up and using the Local Deep Researcher project step-by-step. This tool allows you to perform comprehensive research on any topic using local LLMs (Large Language Models) without relying on remote API services.

## Prerequisites

Before starting, you'll need:

1. A computer with sufficient resources to run a local LLM
   - Recommended: 16GB+ RAM for smaller models
   - 32GB+ RAM for larger models

2. One of the following LLM providers:
   - [Ollama](https://ollama.com/download) (recommended for ease of use)
   - [LMStudio](https://lmstudio.ai/)

3. Optional: API keys for advanced search providers
   - [Tavily](https://tavily.com/)
   - [Perplexity](https://www.perplexity.ai/)

## Step 1: Clone the Repository

```bash
git clone https://github.com/langchain-ai/local-deep-researcher.git
cd local-deep-researcher
```

## Step 2: Set Up Your Environment

### Create a Virtual Environment

For Mac/Linux:
```bash
python -m venv .venv
source .venv/bin/activate
```

For Windows:
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### Create Configuration File

```bash
cp .env.example .env
```

Now edit the `.env` file to customize your setup.

## Step 3: Choose and Configure an LLM Provider

### Option A: Using Ollama (Recommended)

1. Download and install [Ollama](https://ollama.com/download) for your platform

2. Pull a model (choose one based on your hardware capabilities):
   ```bash
   # For powerful machines (32GB+ RAM):
   ollama pull llama3.2
   
   # For mid-range machines (16GB RAM):
   ollama pull deepseek-r1:8b
   
   # For lower-end machines:
   ollama pull deepseek-r1:1.5b
   ```

3. Configure your `.env` file:
   ```
   LLM_PROVIDER=ollama
   OLLAMA_BASE_URL="http://localhost:11434"
   LOCAL_LLM=llama3.2  # or whatever model you pulled
   ```

### Option B: Using LMStudio

1. Download and install [LMStudio](https://lmstudio.ai/)

2. In LMStudio:
   - Download a model of your choice (e.g., qwen_qwq-32b)
   - Go to the "Local Server" tab
   - Start the server with the OpenAI-compatible API

3. Configure your `.env` file:
   ```
   LLM_PROVIDER=lmstudio
   LOCAL_LLM=qwen_qwq-32b  # Use the exact model name as shown in LMStudio
   LMSTUDIO_BASE_URL=http://localhost:1234/v1
   ```

## Step 4: Choose a Search Provider

### Basic Option: DuckDuckGo (No API Key Required)

Configure your `.env` file:
```
SEARCH_API=duckduckgo
FETCH_FULL_PAGE=true  # Set to false if you want faster but less detailed results
```

### Advanced Options: 

For Tavily:
```
SEARCH_API=tavily
TAVILY_API_KEY=your_tavily_api_key_here
```

For Perplexity:
```
SEARCH_API=perplexity
PERPLEXITY_API_KEY=your_perplexity_api_key_here
```

For SearXNG (if you have a local instance):
```
SEARCH_API=searxng
SEARXNG_URL=http://your-searxng-instance:8888
```

## Step 5: Configure Research Depth

Set the number of research iterations in your `.env` file:
```
MAX_WEB_RESEARCH_LOOPS=3  # Default is 3, increase for more thorough research
```

## Step 6: Launch LangGraph Server

For Mac/Linux:
```bash
# Install uv package manager
curl -LsSf https://astral.sh/uv/install.sh | sh
uvx --refresh --from "langgraph-cli[inmem]" --with-editable . --python 3.11 langgraph dev
```

For Windows:
```powershell
# Install dependencies
pip install -e .
pip install -U "langgraph-cli[inmem]"            

# Start the LangGraph server
langgraph dev
```

## Step 7: Use the LangGraph Studio UI

1. When the server starts, you'll see a URL like:
   ```
   LangGraph Studio Web UI: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
   ```

2. Open that URL in your browser (Firefox recommended)

3. In the UI:
   - Navigate to the "configuration" tab if you want to adjust settings
   - Enter your research topic in the input field
   - Click "Run" to start the research process

4. View the Research Process:
   - The graph visualization shows each step of the research
   - Click on nodes to see inputs and outputs
   - The final summary appears in the output when complete

## Step 8: Interpret the Results

The final output includes:
- A comprehensive summary of your research topic
- Citations to all sources used
- Information organized in a logical structure

You can:
- Copy the markdown output for use elsewhere
- Click on the URLs in the sources to visit original pages
- Review the state graph to understand how the research evolved

## Troubleshooting

### Common Issues:

1. **Model Output Errors**: Some models struggle with JSON output
   - Solution: In configuration, set `use_tool_calling=true`

2. **Browser Security Warnings**: Safari may block mixed content
   - Solution: Use Firefox or Chrome instead

3. **Memory Issues**: LLM crashes during operation
   - Solution: Try a smaller model or reduce `MAX_WEB_RESEARCH_LOOPS`

4. **Slow Performance**: Research takes too long
   - Solution: Set `FETCH_FULL_PAGE=false` for faster (but less detailed) results

5. **Connection Errors**: Can't connect to LLM provider
   - Check that Ollama or LMStudio is running
   - Verify the base URLs in your configuration

## Advanced Usage

### Docker Deployment

Build and run the Docker container:
```bash
docker build -t local-deep-researcher .
docker run --rm -it -p 2024:2024 \
  -e SEARCH_API="duckduckgo" \
  -e LLM_PROVIDER=ollama \
  -e OLLAMA_BASE_URL="http://host.docker.internal:11434/" \
  -e LOCAL_LLM="llama3.2" \
  local-deep-researcher
```

Then access the UI at:
https://smith.langchain.com/studio/thread?baseUrl=http://127.0.0.1:2024

### Customizing Prompts

For advanced users who want to modify the system prompts:
1. Edit the prompts in `src/ollama_deep_researcher/prompts.py`
2. Restart the LangGraph server to apply changes

## Example Research Topics

Try researching these topics to test the system:
- "Quantum computing recent advances"
- "Climate change mitigation strategies"
- "Modern web development frameworks comparison"
- "Machine learning explainability techniques"
- "Urban planning innovations"

Each topic will demonstrate the assistant's ability to gather diverse information, synthesize it, and identify knowledge gaps for further exploration.
