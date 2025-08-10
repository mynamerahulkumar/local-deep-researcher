# Local Deep Researcher: Architecture Diagram

```
+----------------------------------------------------------------------+
|                    LOCAL DEEP RESEARCHER ARCHITECTURE                 |
+----------------------------------------------------------------------+

+------------------+
| User Interface   |
| (LangGraph UI)   |
+--------+---------+
         |
         | Research Topic
         v
+------------------+     +------------------+     +------------------+
| Configuration    |     | State Management |     | LangGraph Engine |
| System           |<--->| (SummaryState)   |<--->| (StateGraph)     |
+------------------+     +------------------+     +--------+---------+
                                                          |
                                                          |
                +---------------------------------------+-+
                |                                       |
                v                                       v
        +---------------+                       +---------------+
        | Research Flow |                       | Conditional   |
        |               |                       | Routing       |
        +-------+-------+                       +-------+-------+
                |                                       |
                |                                       |
+---------------+---------------------------------------+---------------+
|                                                                       |
|                         RESEARCH NODES                                |
|                                                                       |
+---------+---------+-----------+-----------+-----------+---------------+
          |         |           |           |           |
          |         |           |           |           |
          v         v           v           v           v
    +-----------+ +-----------+ +-----------+ +-----------+ +-----------+
    | Generate  | | Web       | | Summarize | | Reflect   | | Finalize  |
    | Query     | | Research  | | Sources   | | on        | | Summary   |
    |           | |           | |           | | Summary   | |           |
    +-----------+ +-----+-----+ +-----------+ +-----------+ +-----------+
                        |
                        v
          +-------------------------+
          |                         |
    +-----+------+           +------+------+
    | Search     |           | Local LLM    |
    | Providers  |           | Providers    |
    +------------+           +------+-------+
    | DuckDuckGo |                  |
    | Tavily     |           +------+------+
    | Perplexity |           | Ollama      |
    | SearXNG    |           | LMStudio    |
    +------------+           +-------------+


+----------------------------------------------------------------------+
|                              DATA FLOW                                |
+----------------------------------------------------------------------+

  Research       Search        Web          Summary       Knowledge      Final
  Topic    -->   Query   -->   Results -->  Creation  --> Gaps     -->  Report
                   ^                                       |
                   |                                       |
                   +---------------------------------------+
                              Follow-up Queries
                              (Iterative Process)
```

## Architecture Components

### 1. User Interface
- **LangGraph UI**: Web interface for inputting research topics and viewing results
- Provides visualization of the research workflow and configuration options

### 2. Core System
- **Configuration System**: Manages settings from environment variables and UI
- **State Management**: Tracks research progress using SummaryState dataclass
- **LangGraph Engine**: Orchestrates the workflow using StateGraph

### 3. Research Flow
- **Research Nodes**: The key processing stages in the workflow
- **Conditional Routing**: Determines whether to continue research or finalize

### 4. Processing Nodes
- **Generate Query**: Creates search queries based on the research topic or knowledge gaps
- **Web Research**: Performs web searches using configured search provider
- **Summarize Sources**: Creates or updates summary based on search results
- **Reflect on Summary**: Identifies knowledge gaps for further research
- **Finalize Summary**: Prepares final report with sources and citations

### 5. External Services
- **Search Providers**: 
  - DuckDuckGo (default, no API key needed)
  - Tavily (requires API key)
  - Perplexity (requires API key)
  - SearXNG (requires local instance)
  
- **Local LLM Providers**:
  - Ollama: Simple interface for running various LLM models
  - LMStudio: Alternative with OpenAI-compatible API

### 6. Data Flow
The system follows an iterative process:
1. User provides a research topic
2. LLM generates a search query
3. System performs web search
4. LLM summarizes search results
5. LLM identifies knowledge gaps
6. LLM generates follow-up queries
7. Process repeats for configured number of iterations
8. Final research report is produced with citations

## Key Technical Features

- Uses JSON mode or tool calling for structured LLM output
- Handles multiple search providers with unified interface
- Deduplicates and formats sources for citation
- Tracks research state across iterations
- Provides visualization of research process
- Runs entirely locally for privacy and control
