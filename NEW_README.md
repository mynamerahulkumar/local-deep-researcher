# Local Deep Researcher: Step-by-Step Explanation

Local Deep Researcher is a fully local web research assistant that leverages locally-run Large Language Models (LLMs) through either [Ollama](https://ollama.com/search) or [LMStudio](https://lmstudio.ai/). This tool performs comprehensive research on user-provided topics by autonomously generating search queries, retrieving web information, summarizing findings, identifying knowledge gaps, and iteratively refining the research through multiple cycles.

## Project Overview

The core concept of Local Deep Researcher is inspired by the [IterDRAG](https://arxiv.org/html/2410.04343v1) research approach, which decomposes queries into sub-queries, retrieves documents for each one, and builds up answers incrementally.

### How It Works

1. **Research Initialization**: When provided with a research topic, the assistant uses a local LLM to generate an initial search query.
2. **Web Search**: The assistant uses one of several search engines (DuckDuckGo, SearXNG, Tavily, or Perplexity) to gather relevant web content.
3. **Summarization**: The LLM summarizes findings from the web search related to the research topic.
4. **Reflection & Gap Analysis**: The LLM analyzes the current summary, identifies knowledge gaps, and generates follow-up search queries.
5. **Iteration**: Steps 2-4 repeat for a configurable number of iterations, with each cycle building upon previous knowledge.
6. **Final Summary**: After completing all iterations, the assistant produces a comprehensive markdown summary with citations to all sources used.

## Architecture Breakdown

The project is structured around a LangGraph workflow that orchestrates the research process through several key components:

### 1. Configuration System (`configuration.py`)

This file defines the configurable parameters for the research assistant using Pydantic models:

- **SearchAPI Enum**: Defines available search engines (Perplexity, Tavily, DuckDuckGo, SearXNG)
- **Configuration Class**: Contains all configurable fields like:
  - `max_web_research_loops`: Number of research iterations to perform (default: 3)
  - `local_llm`: Name of the LLM model to use (default: "llama3.2")
  - `llm_provider`: Provider for the LLM ("ollama" or "lmstudio")
  - `search_api`: Web search API to use (default: "duckduckgo")
  - `fetch_full_page`: Whether to include full page content in search results
  - Various base URLs and operational settings

Configuration values are loaded with the following priority:
1. Environment variables (highest priority)
2. LangGraph UI configuration
3. Default values in the Configuration class (lowest priority)

### 2. State Management (`state.py`)

Manages the state of the research process using dataclasses:

- **SummaryState**: Main state class tracking:
  - `research_topic`: The user-provided topic
  - `search_query`: Current search query
  - `web_research_results`: List of web research results
  - `sources_gathered`: List of sources collected
  - `research_loop_count`: Current iteration count
  - `running_summary`: The evolving research summary

- **SummaryStateInput/Output**: Classes for input and output interfaces

### 3. Graph Implementation (`graph.py`)

The core orchestration logic using LangGraph's StateGraph:

- **get_llm**: Helper function to initialize the LLM based on configuration
- **Node Functions**:
  - `generate_query`: Generates a search query using the LLM
  - `web_research`: Performs web search using the configured search API
  - `summarize_sources`: Summarizes web research results 
  - `reflect_on_summary`: Identifies knowledge gaps and generates follow-up queries
  - `finalize_summary`: Formats the final report with citations
  - `route_research`: Router function that decides whether to continue research or finalize

- **Graph Structure**: Defines the workflow by connecting nodes:
  ```
  START → generate_query → web_research → summarize_sources → reflect_on_summary 
                                                            ↙                    ↘
                                                   web_research                finalize_summary → END
  ```

### 4. Utility Functions (`utils.py`)

Contains helper functions for various operations:

- **Search API Functions**:
  - `duckduckgo_search`: Performs web searches via DuckDuckGo
  - `searxng_search`: Searches using SearXNG
  - `tavily_search`: Searches using Tavily API
  - `perplexity_search`: Searches using Perplexity API

- **Content Processing**:
  - `fetch_raw_content`: Retrieves full web page content
  - `deduplicate_and_format_sources`: Deduplicates and formats search results
  - `strip_thinking_tokens`: Cleans up LLM output by removing thinking tokens

### 5. Prompt Templates (`prompts.py`)

Contains system prompts for different stages of the research process:

- `query_writer_instructions`: Prompts for generating search queries
- `summarizer_instructions`: Guidelines for summarizing content
- `reflection_instructions`: Framework for identifying knowledge gaps
- Various JSON and tool-calling format instructions

### 6. LMStudio Integration (`lmstudio.py`)

Provides integration with LMStudio's OpenAI-compatible API:

- **ChatLMStudio**: Custom class extending ChatOpenAI to work with LMStudio
- Includes special handling for JSON format responses

## How to Use the Project

### Setup Options

1. **Ollama Setup**:
   - Download and install [Ollama](https://ollama.com/download)
   - Pull a local LLM model: `ollama pull deepseek-r1:8b`
   - Configure the environment or UI settings:
     ```
     LLM_PROVIDER=ollama
     OLLAMA_BASE_URL="http://localhost:11434"
     LOCAL_LLM=deepseek-r1:8b
     ```

2. **LMStudio Setup**:
   - Download and install [LMStudio](https://lmstudio.ai/)
   - Load your preferred model (e.g., qwen_qwq-32b)
   - Start the server with OpenAI-compatible API
   - Configure the environment or UI settings:
     ```
     LLM_PROVIDER=lmstudio
     LOCAL_LLM=qwen_qwq-32b
     LMSTUDIO_BASE_URL=http://localhost:1234/v1
     ```

3. **Search Tool Setup**:
   - Default: DuckDuckGo (no API key needed)
   - Alternatives: SearXNG, Tavily, or Perplexity (require API keys)
   - Configure the environment or UI settings:
     ```
     SEARCH_API=duckduckgo
     TAVILY_API_KEY=xxx  # If using Tavily
     PERPLEXITY_API_KEY=xxx  # If using Perplexity
     ```

### Running the Project

#### Local Development:

1. Clone the repository and create a virtual environment
2. Launch LangGraph server using the provided commands
3. Access the LangGraph Studio UI at the provided URL
4. Enter a research topic in the UI and observe the research process

#### Docker Deployment:

1. Build the Docker image: `docker build -t local-deep-researcher .`
2. Run the container with the necessary environment variables
3. Access the LangGraph Studio UI at the provided URL

## Key Features and Benefits

1. **Privacy-Focused**: All processing happens locally on your machine
2. **Flexible LLM Options**: Works with any model supported by Ollama or LMStudio
3. **Multiple Search Engines**: Supports various search providers
4. **Iterative Research**: Automatically refines research through multiple cycles
5. **Structured Output**: Provides formatted markdown report with citations
6. **Visual Process**: LangGraph UI shows the research process step-by-step
7. **Configurable**: Extensive options to customize behavior
8. **Fallback Mechanisms**: Handles models with limitations in producing structured output

## Technical Details

- Built with LangGraph for workflow orchestration
- Uses Pydantic for configuration management
- Implements both JSON mode and tool calling for structured output
- Supports various local LLM providers (Ollama, LMStudio)
- Includes Docker support for containerized deployment

## Sequence of Operation

1. **User Input**: User provides a research topic
2. **Initial Query**: LLM generates an initial search query
3. **Web Search**: System retrieves information from the web
4. **Summarization**: LLM summarizes the findings
5. **Gap Analysis**: LLM identifies knowledge gaps
6. **Iterative Refinement**: Process repeats with new queries targeting gaps
7. **Final Report**: System produces a comprehensive report with citations

## Conclusion

Local Deep Researcher demonstrates how modern LLM workflows can be orchestrated to create a powerful research assistant that operates entirely on local hardware. The modular design, configurable settings, and iterative approach make it a versatile tool for in-depth exploration of various topics without relying on remote API services.
