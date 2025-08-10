# Local Deep Researcher: Code Structure Guide

This document provides a detailed explanation of each file in the Local Deep Researcher project, following a logical sequence to understand how they work together.

## 1. `configuration.py`

**Purpose**: Defines configuration parameters and options for the research assistant.

**Key Components**:
- `SearchAPI` enum: Defines available search providers (Perplexity, Tavily, DuckDuckGo, SearXNG)
- `Configuration` class: A Pydantic model with all configurable fields including:
  - LLM provider settings (Ollama or LMStudio)
  - Model selection and base URLs
  - Search API options and behavior settings
  - Maximum research iterations
  - Output formatting options

**Configuration Loading**:
- The `from_runnable_config` method loads configuration from:
  1. Environment variables (highest priority)
  2. LangGraph UI configuration
  3. Default values in the class (lowest priority)

## 2. `state.py`

**Purpose**: Defines the state structure for the LangGraph workflow.

**Key Components**:
- `SummaryState`: Main dataclass that tracks:
  - `research_topic`: The user-provided topic
  - `search_query`: Current search query
  - `web_research_results`: List of all web search results
  - `sources_gathered`: List of sources collected
  - `research_loop_count`: Current iteration count
  - `running_summary`: The evolving research summary

- `SummaryStateInput`: Input interface for the graph
- `SummaryStateOutput`: Output interface for the graph

The state design uses Python's dataclasses with annotations for list merging behavior.

## 3. `prompts.py`

**Purpose**: Contains system prompts and instructions for the LLM at different stages.

**Key Components**:
- `query_writer_instructions`: Prompts for generating search queries
- `summarizer_instructions`: Guidelines for creating and updating summaries
- `reflection_instructions`: Framework for identifying knowledge gaps
- Format-specific instructions:
  - `json_mode_query_instructions`: For JSON-formatted query output
  - `tool_calling_query_instructions`: For tool-calling query output
  - `json_mode_reflection_instructions`: For JSON-formatted reflection
  - `tool_calling_reflection_instructions`: For tool-calling reflection

The prompts are designed to guide the LLM through different tasks in the research workflow.

## 4. `lmstudio.py`

**Purpose**: Provides integration with LMStudio's OpenAI-compatible API.

**Key Components**:
- `ChatLMStudio`: Custom class extending ChatOpenAI that:
  - Handles connection to LMStudio's API
  - Processes JSON-formatted responses
  - Manages error handling and fallbacks
  - Cleans up LLM output when needed

This allows using locally-hosted models through LMStudio as an alternative to Ollama.

## 5. `utils.py`

**Purpose**: Contains utility functions for web searches and content processing.

**Key Components**:
- Search API Functions:
  - `duckduckgo_search`: Uses DuckDuckGo for web searches
  - `searxng_search`: Uses a SearXNG instance
  - `tavily_search`: Uses Tavily API
  - `perplexity_search`: Uses Perplexity API

- Content Processing:
  - `fetch_raw_content`: Retrieves full web page content and converts to markdown
  - `deduplicate_and_format_sources`: Removes duplicates and formats search results
  - `format_sources`: Creates a bullet-point list of sources
  - `strip_thinking_tokens`: Removes <think></think> tokens from LLM output

These utilities handle external data retrieval and formatting for the research process.

## 6. `graph.py`

**Purpose**: The core orchestration logic using LangGraph's StateGraph.

**Key Components**:
- Helper Functions:
  - `get_llm`: Initializes the appropriate LLM based on configuration
  - `generate_search_query_with_structured_output`: Handles structured output for queries

- Node Functions:
  - `generate_query`: Uses LLM to create search queries
  - `web_research`: Performs web searches using the configured API
  - `summarize_sources`: Creates/updates the research summary
  - `reflect_on_summary`: Analyzes gaps and generates follow-up queries
  - `finalize_summary`: Creates the final report with citations
  - `route_research`: Makes routing decisions based on the iteration count

- Graph Construction:
  - Creates a StateGraph with the defined nodes
  - Connects nodes with edges to form the workflow
  - Adds conditional routing based on iteration count

This file brings together all components to create the research workflow.

## 7. `__init__.py`

**Purpose**: Makes the package importable and may expose key components.

## Workflow Sequence

The research process follows these steps:

1. **Initialization**: User provides a research topic
   - `generate_query` node creates initial search query

2. **First Research Cycle**:
   - `web_research` retrieves information from the web
   - `summarize_sources` creates initial summary
   - `reflect_on_summary` identifies knowledge gaps
   - `route_research` decides to continue if iterations remain

3. **Subsequent Research Cycles**:
   - Process repeats with new queries targeting identified gaps
   - Summary is iteratively updated with new information
   - `research_loop_count` tracks the cycle count

4. **Finalization**:
   - When max iterations are reached, `route_research` directs to `finalize_summary`
   - `finalize_summary` formats the report with all sources as citations
   - Final summary is returned as output

## Dependencies

The project uses several key libraries:
- `langgraph`: For workflow orchestration
- `langchain`: Core components for LLM interactions
- `langchain-ollama`: Integration with Ollama
- `langchain-openai`: Base classes for OpenAI-compatible APIs
- `duckduckgo-search`: For web searches
- `markdownify`: For HTML to markdown conversion
- Various utilities for HTTP requests and formatting

## Configuration Files

- `pyproject.toml`: Defines project metadata and dependencies
- `Dockerfile`: Containerizes the application for deployment
- `langgraph.json`: LangGraph configuration
- `.env.example`: Example environment variables

## Running the Project

The project can be run using:
1. LangGraph Studio UI (recommended for visualization)
2. Direct API calls to the LangGraph server
3. Containerized deployment using Docker

Each approach uses the same core components but offers different interfaces.
