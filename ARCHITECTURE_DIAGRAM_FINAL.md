# Local Deep Researcher: Architecture Diagram (Final Version)

```
+------------------------------------------------------------------+
|                LOCAL DEEP RESEARCHER ARCHITECTURE                 |
+------------------------------------------------------------------+

                    +-------------------+
                    |  User Interface   |
                    |  (LangGraph UI)   |
                    +---------+---------+
                              |
                              | Research Topic
                              v
                    +---------+---------+
                    |  LangGraph Flow   |
                    |  (StateGraph)     |
                    +---------+---------+
                              |
                              v
+------------------------------------------------------------------+
|                       RESEARCH WORKFLOW                           |
+------------------------------------------------------------------+
          |                   |                      |
          v                   v                      v
  +---------------+   +---------------+      +---------------+
  | Configuration |   | State Manager |      | Routing Logic |
  +---------------+   +---------------+      +---------------+

+------------------------------------------------------------------+
|                       PROCESSING NODES                            |
+------------------------------------------------------------------+

  +----------+    +----------+    +----------+    +----------+    +----------+
  | Generate |    |   Web    |    |Summarize |    | Reflect  |    |Finalize  |
  |  Query   |--->| Research |--->| Sources  |--->|    on    |--->| Summary  |
  |          |    |          |    |          |    | Summary  |    |          |
  +----------+    +----+-----+    +----------+    +-----+----+    +----------+
                       |                                |
                       |                                |
                       v                                |
                 +-----+------+                         |
                 |  External  |                         |
                 |  Services  |                         |
                 +-----+------+                         |
                       |                                |
        +--------------|----------------+               |
        |              |                |               |
        v              v                v               |
  +-----------+  +-----------+  +------------+         |
  |  Search   |  |   Local   |  | Optional   |         |
  | Providers |  |    LLM    |  | Services   |         |
  +-----------+  +-----------+  +------------+         |
  |DuckDuckGo |  | Ollama    |  | Tavily API |         |
  |SearXNG    |  | LMStudio  |  | Perplexity |         |
  +-----------+  +-----------+  +------------+         |
                                                       |
                       +-----------------------------+ |
                       |         Iteration Loop      | |
                       |      (Follow-up Queries)    |<+
                       +-----------------------------+


+------------------------------------------------------------------+
|                          DATA FLOW                                |
+------------------------------------------------------------------+

  +-----------+   +-----------+   +-----------+   +-----------+   +-----------+
  | Research  |   | Search    |   |   Web     |   | Updated   |   |   Final   |
  |  Topic    |-->|  Query    |-->| Results   |-->| Summary   |-->|  Report   |
  |           |   |           |   |           |   |           |   |           |
  +-----------+   +-----------+   +-----------+   +-----------+   +-----------+
                      ^                                 |
                      |                                 v
                      |                          +-----------+
                      |                          | Knowledge |
                      +--------------------------+   Gaps    |
                                                 +-----------+
```

## Architecture Overview

### 1. User Interface Layer
- **LangGraph UI**: Web-based interface where users:
  - Enter research topics
  - Configure research parameters
  - Visualize the research workflow
  - View final research reports

### 2. Workflow Management Layer
- **LangGraph Flow**: Orchestrates the research process using:
  - **StateGraph**: Defines node connections and routing
  - **Configuration**: Manages settings from environment or UI
  - **State Manager**: Tracks progress and data across iterations
  - **Routing Logic**: Controls iteration flow and termination

### 3. Processing Nodes Layer
A sequential pipeline of specialized functions:

- **Generate Query**:
  - Takes research topic or knowledge gaps
  - Produces optimized search queries
  - Uses structured output (JSON mode or tool calling)
  
- **Web Research**:
  - Executes search queries through configured provider
  - Retrieves relevant web content
  - Formats and prepares search results
  
- **Summarize Sources**:
  - Creates initial summary or updates existing summary
  - Integrates new information coherently
  - Maintains focus on research topic
  
- **Reflect on Summary**:
  - Analyzes current knowledge
  - Identifies information gaps
  - Formulates follow-up queries
  
- **Finalize Summary**:
  - Prepares final research report
  - Formats all sources as citations
  - Organizes information logically

### 4. External Services Layer
- **Search Providers**:
  - DuckDuckGo (default, no API key required)
  - SearXNG (self-hosted option)
  - Tavily (with API key)
  - Perplexity (with API key)
  
- **Local LLM Providers**:
  - Ollama: Easy local LLM hosting
  - LMStudio: Alternative with OpenAI-compatible API
  
### 5. Data Flow
1. User provides a research topic
2. System generates search query
3. Web search retrieves information
4. Information is summarized
5. Knowledge gaps are identified
6. Follow-up queries target gaps
7. Process repeats for configured iterations
8. Final report is produced with sources

### 6. Key Technical Features
- Entirely local operation (no remote APIs required for core functionality)
- Flexible search provider options
- Structured LLM outputs (JSON mode or tool calling)
- Source deduplication and citation
- Iterative knowledge refinement
- Configurable research depth
