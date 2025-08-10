# Local Deep Researcher: Architecture Diagram

Below is the architecture diagram showing the components and flow of the Local Deep Researcher system. The diagram illustrates how different components interact and how data flows through the research process.

```
+-------------------------------------------------------------------------------------------------------------------+
|                                        LOCAL DEEP RESEARCHER ARCHITECTURE                                          |
+-------------------------------------------------------------------------------------------------------------------+

                   +-------------------+
                   |                   |
                   |   User Interface  |
                   |   (LangGraph UI)  |
                   |                   |
                   +--------+----------+
                            |
                            | Research Topic
                            v
+--------------------------------------------------------+
|               LANGGRAPH WORKFLOW ENGINE                 |
+--------------------------------------------------------+
                            |
                            |
   +------------------------+-------------------------+
   |                        |                         |
   v                        v                         v
+--------+        +-------------------+     +-------------------+
|        |        |                   |     |                   |
| Config |        |  StateGraph Flow  |     |  State Management |
|        |        |                   |     |                   |
+--------+        +-------+-----------+     +-------------------+
                          |
                          |
                          v
+--------------------------------------------------------+
|                       NODES                             |
+--------------------------------------------------------+
   |           |           |             |           |
   |           |           |             |           |
   v           v           v             v           v
+--------+ +--------+ +--------+   +--------+  +--------+
|        | |        | |        |   |        |  |        |
|Generate| |  Web   | |Summarize|  |Reflect |  |Finalize|
| Query  | |Research| |Sources  |  |   on   |  |Summary |
|        | |        | |        |   |Summary |  |        |
+--------+ +--------+ +--------+   +--------+  +--------+
               |
               v
      +-------------------+
      |                   |      +-------------------+
      |  Search Provider  |<---->| Local LLM Provider|
      |                   |      |                   |
      +--------+----------+      +--------+----------+
               |                          |
               v                          v
     +-------------------+      +-------------------+
     |                   |      |                   |
     |  Search Services  |      |   Ollama/LMStudio |
     |                   |      |                   |
     +-------------------+      +-------------------+
     | - DuckDuckGo      |      | - Various Models  |
     | - Tavily          |      | - JSON Mode       |
     | - Perplexity      |      | - Tool Calling    |
     | - SearXNG         |      |                   |
     +-------------------+      +-------------------+

+-------------------------------------------------------------------------------------------------------------------+
|                                             DATA FLOW                                                              |
+-------------------------------------------------------------------------------------------------------------------+

+---------------+    +---------------+    +---------------+    +---------------+    +---------------+
| Research      |    | Search        |    | Web           |    | Updated       |    | Final         |
| Topic         |--->| Query         |--->| Results       |--->| Summary       |--->| Report        |
|               |    |               |    |               |    |               |    | with Sources  |
+---------------+    +---------------+    +---------------+    +---------------+    +---------------+
                           ^                                          |
                           |                                          |
                           +------------------------------------------+
                                   Follow-up Query (Iteration)
```

## Component Descriptions

### User Interface
- **LangGraph UI**: The web interface where users input research topics and view results
- Visualizes the workflow and allows configuration adjustments

### LangGraph Workflow Engine
- **StateGraph Flow**: Defines the research process flow and routing logic
- **State Management**: Tracks research state across iterations using the `SummaryState` class
- **Config**: Handles configuration through environment variables and UI settings

### Nodes
- **Generate Query**: Creates optimized search queries for the research topic
- **Web Research**: Performs web searches using the configured provider
- **Summarize Sources**: Synthesizes information from search results
- **Reflect on Summary**: Identifies knowledge gaps and generates follow-up queries
- **Finalize Summary**: Creates the final report with citations

### External Services
- **Search Provider**: Interface to various search engines
  - DuckDuckGo, Tavily, Perplexity, SearXNG
- **Local LLM Provider**: Interface to locally-run language models
  - Ollama or LMStudio with various model options

### Data Flow
The system follows an iterative process:
1. User provides a research topic
2. LLM generates an initial search query
3. System retrieves web search results
4. LLM summarizes the findings
5. LLM reflects on the summary and identifies knowledge gaps
6. LLM generates a follow-up query
7. Process repeats for configured number of iterations
8. Final summary with sources is produced

## Key Features

- **Fully Local**: All processing happens on the user's machine
- **Multiple Search Options**: Supports various search providers
- **Flexible LLM Integration**: Works with Ollama or LMStudio
- **Iterative Research**: Automatically refines queries to fill knowledge gaps
- **Structured Output**: Produces formatted reports with citations
- **Configurable**: Extensive options to customize behavior
