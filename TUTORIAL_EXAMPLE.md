# Local Deep Researcher: Tutorial Example

This tutorial walks through a complete example of using Local Deep Researcher to investigate a sample topic. We'll follow each step of the process and explain what's happening behind the scenes.

## Topic: "Modern applications of transformer neural networks"

Let's use this topic to demonstrate how the Local Deep Researcher works through each phase of its research process.

## Prerequisites

Ensure you've completed the setup as described in the USER_GUIDE.md:
1. Installed Ollama or LMStudio
2. Pulled a suitable LLM model
3. Created and configured your `.env` file
4. Launched the LangGraph server

## Step 1: Start the Research Process

1. Open the LangGraph Studio UI:
   - Navigate to the URL provided when you launched the server
   - You should see the graph visualization interface

2. Enter the research topic:
   - Type "Modern applications of transformer neural networks" in the input field
   - Click "Run" to begin the research process

## Step 2: Initial Query Generation

The first node in the graph to run is `generate_query`:

1. **What happens**:
   - The system prompts the LLM with `query_writer_instructions`
   - The LLM generates an optimized search query for the topic
   - The query is stored in the state as `search_query`

2. **Example output**:
   ```json
   {
     "query": "recent applications transformer neural networks NLP computer vision 2024",
     "rationale": "This query targets the most recent applications of transformer models across major domains including natural language processing and computer vision."
   }
   ```

3. **Behind the scenes**:
   - The `generate_query` function in `graph.py` processes the user's topic
   - It uses either JSON mode or tool calling based on configuration
   - The current date is included to help get recent information

## Step 3: Web Research

The `web_research` node executes next:

1. **What happens**:
   - The system uses the configured search API (e.g., DuckDuckGo)
   - It searches the web with the generated query
   - Results are fetched, potentially including full page content

2. **Example output**:
   - Multiple search results including:
     - Research papers on transformer applications
     - Recent articles on transformer innovations
     - Code repositories and examples
     - Industry applications of transformers

3. **Behind the scenes**:
   - The `web_research` function in `graph.py` calls the appropriate search function
   - Search results are formatted and deduplicated
   - Results are stored in the state's `web_research_results` list
   - Source URLs are added to `sources_gathered`
   - The `research_loop_count` is incremented

## Step 4: Summarize Sources

The `summarize_sources` node processes the search results:

1. **What happens**:
   - The system provides the search results to the LLM
   - The LLM synthesizes the information into a coherent summary
   - This becomes the initial `running_summary`

2. **Example output**:
   ```
   Transformer neural networks have evolved significantly since their introduction in 2017, with applications spanning multiple domains. In natural language processing (NLP), transformers power modern systems like GPT-4, PaLM, and Claude through their self-attention mechanism which enables parallel processing of sequential data. Recent NLP applications include advanced language translation, document summarization, and conversational AI.
   
   In computer vision, Vision Transformers (ViT) have challenged traditional convolutional neural networks by treating images as sequences of patches. Applications include image classification, object detection, and image generation. The flexibility of transformers has enabled multimodal models that can process both text and images, leading to systems like DALL-E and Midjourney.
   
   Healthcare applications have emerged where transformers analyze medical imaging, predict protein structures (as in AlphaFold), and assist in drug discovery. In audio processing, transformers have improved speech recognition, music generation, and audio enhancement.
   ```

3. **Behind the scenes**:
   - The `summarize_sources` function uses the `summarizer_instructions` prompt
   - For the first iteration, it creates a new summary
   - In later iterations, it updates the existing summary with new information

## Step 5: Reflection and Gap Analysis

The `reflect_on_summary` node analyzes the current summary:

1. **What happens**:
   - The system prompts the LLM to identify knowledge gaps
   - The LLM generates a follow-up query to address those gaps
   - This query becomes the new `search_query`

2. **Example output**:
   ```json
   {
     "knowledge_gap": "The summary lacks information about transformer applications in time-series forecasting and reinforcement learning",
     "follow_up_query": "transformer neural networks applications time-series forecasting reinforcement learning finance"
   }
   ```

3. **Behind the scenes**:
   - The `reflect_on_summary` function uses `reflection_instructions`
   - It asks the LLM to think about what's missing from the current summary
   - The generated query is designed to fill those knowledge gaps

## Step 6: Route Research Decision

The `route_research` function decides what happens next:

1. **What happens**:
   - System checks if `research_loop_count` has reached `max_web_research_loops`
   - If not, it routes back to `web_research` for another iteration
   - If yes, it routes to `finalize_summary`

2. **Behind the scenes**:
   - This conditional routing creates the iterative research loop
   - The configuration setting `max_web_research_loops` controls the depth of research

## Step 7: Second Research Iteration

The second cycle repeats steps 3-5:

1. **New web search**:
   - Uses the follow-up query from reflection
   - Gathers new information about time-series and RL applications

2. **Summary update**:
   - Integrates new information with the existing summary
   - Adds details about the identified knowledge gaps

3. **New reflection**:
   - Identifies additional gaps, like transformer efficiency or limitations
   - Generates another follow-up query

## Step 8: Third Research Iteration

The third cycle again repeats steps 3-5:

1. **Final web search**:
   - Targets remaining knowledge gaps
   - Gathers information on efficiency improvements, limitations, etc.

2. **Final summary update**:
   - Creates a comprehensive summary incorporating all research
   - Covers multiple domains and applications

3. **Final reflection**:
   - If configured for more iterations, would generate another query
   - Otherwise, proceeds to finalization

## Step 9: Finalize Summary

After reaching the maximum research iterations, the `finalize_summary` node runs:

1. **What happens**:
   - The system formats the final research report
   - It deduplicates all sources gathered
   - It combines the running summary with all sources

2. **Example output**:
   ```markdown
   ## Summary
   Transformer neural networks have revolutionized multiple domains since their introduction in the 2017 "Attention is All You Need" paper. In natural language processing (NLP), transformers power systems like GPT-4, PaLM, and Claude, enabling advanced language translation, document summarization, and conversational AI. Their self-attention mechanism allows parallel processing of sequential data, overcoming limitations of previous recurrent neural networks.

   In computer vision, Vision Transformers (ViT) have challenged convolutional neural networks by treating images as sequences of patches, showing strong performance in image classification, object detection, and generation tasks. Multimodal applications combine text and visual processing in systems like DALL-E and Midjourney.

   Transformers have expanded to time-series forecasting in finance and economics, where their ability to capture long-range dependencies helps predict stock prices and economic trends with higher accuracy than traditional models. In reinforcement learning, transformer-based architectures like Decision Transformer and Trajectory Transformer reframe sequential decision-making as a sequence modeling problem.

   Healthcare applications include medical imaging analysis, protein structure prediction (AlphaFold), and drug discovery. Audio applications span speech recognition, music generation, and enhancement.

   Recent efficiency improvements address transformer limitations through techniques like sparse attention, knowledge distillation, and quantization. Researchers are working on reducing the quadratic complexity of self-attention and developing more parameter-efficient architectures like MLP-Mixers and Mamba models.

   ### Sources:
   * Transformers in NLP: Recent Advances and Applications : https://towardsdatascience.com/transformers-in-nlp-recent-advances-and-applications-b42c9a01e7d8
   * Vision Transformers: The Future of Computer Vision? : https://ai.plainenglish.io/vision-transformers-the-future-of-computer-vision-db4599b9c7e4
   * Transformers for Time Series Forecasting : https://arxiv.org/abs/2202.07125
   * Transformers in Reinforcement Learning: A Survey : https://paperswithcode.com/paper/transformers-in-reinforcement-learning-a
   * Efficient Transformer Models: Methods and Applications : https://huggingface.co/blog/efficient-transformers
   ```

3. **Behind the scenes**:
   - The `finalize_summary` function formats all sources as citations
   - The final output is stored in `running_summary`
   - The graph completes and returns the final result

## Viewing the Results

In the LangGraph Studio UI:

1. **Graph Visualization**:
   - Shows the complete flow through all research iterations
   - Highlights the path taken through the nodes

2. **Node Details**:
   - Click on any node to see its inputs and outputs
   - Examine how the summary evolved over time

3. **Final Output**:
   - The complete markdown summary with citations
   - Ready to copy and use elsewhere

## What to Try Next

Now that you've seen how Local Deep Researcher works, try:

1. **Different Topics**:
   - Technical: "Quantum computing error correction"
   - Business: "Sustainable supply chain innovations"
   - Science: "CRISPR gene editing applications"

2. **Configuration Adjustments**:
   - Increase `max_web_research_loops` for deeper research
   - Try different search providers
   - Test different LLM models to compare results

3. **Advanced Features**:
   - Enable `fetch_full_page` for more detailed analysis
   - Try `use_tool_calling` if your model struggles with JSON output

This example demonstrates the power of iterative research using local LLMs. The system not only gathers information but actively identifies gaps in its knowledge and seeks to fill them, resulting in comprehensive research on any topic.
