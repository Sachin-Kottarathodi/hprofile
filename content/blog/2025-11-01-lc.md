---
title: "LangGraph — Notes"
date: 2025-11-01
tags: ["langgraph","agents","langchain"]
description: "Compact notes on LangGraph"
draft: false
---

Introduction To LangGraph:
    ☐ LG - Framework for multi agent apps. More control for agentic workflows.
    ☐ Chains; reliable.
        * Control flow defined
        * Between Agent defined Flow vs Human flow -> find midway
    ☐  LC - Interactions, Vector Stores
       *  LG -> Orchestrations.
        * LC -> can be used in LG but not necessary.
    ☐ State
        * Pass around in the Graph.
    ☐ Edges
        * Conditional Edges
    ☐ StateGraph
        * add_node
        * add-edge
        * add_condiational_edge with function
    ☐ bind_tools
        * Tool Call
    ☐ Messages as State
        * Since State is overwritten, if we need to append, we need to do something more.
        * Use reducer to annotate and append.
        * Built in MessagesState -> does this above.
    ☐ Router
        * LLM calls Tools for no reason, additional Guard? 
        * Introduce ToolNode.
            * can add multiple tools
        * Conditional edge -> prebuilt -> tools_condition.
        * based on tool received in message -> tools_conditions routes to right tool.
    ☐ Agent
        * ReAct 
            * Act
            * Observe
            * Reason
    ☐ Agent with Memory
            * State is transient in single graph execution with no perisstence.
            * checkpoint for memorySaver
            * configure a Thread id for checkpoint
            - How ?
                * create a graph with checkpointer - Graph Compile
                * invoke the Graph with config - config has thread_id
                * and pass this config wherever state is necessary.
      ☐ Review:
        * act - let the model call specific tools
        * observe - pass the tool output back to the model
        * reason - let the model reason about the tool output to decide what to do next (e.g., call another tool or just respond directly)
        * persist state - use an in memory checkpointer to support long-running conversations with interruptions

        ☐ There are a few central concepts to understand -

            - LangGraph —
            Python and JavaScript library
            Allows creation of agent workflows

            - LangGraph API —
            Bundles the graph code
            Provides a task queue for managing asynchronous operations
            Offers persistence for maintaining state across interactions

            - LangGraph Cloud --
            Hosted service for the LangGraph API
            Allows deployment of graphs from GitHub repositories
            Also provides monitoring and tracing for deployed graphs
            Accessible via a unique URL for each deployment

            - LangGraph Studio --
            Integrated Development Environment (IDE) for LangGraph applications
            Uses the API as its back-end, allowing real-time testing and exploration of graphs
            Can be run locally or with cloud-deployment

            - LangGraph SDK --
            Python library for programmatically interacting with LangGraph graphs
            Provides a consistent interface for working with graphs, whether served locally or in the cloud
            Allows creation of clients, access to assistants, thread management, and execution of runs

     ☐ StateSchema
         * TypedDict
         * Dataclass
             *  Type is not inforced at runtime
         * Pydantic -> data Validation.

     ☐ Reducers
         * default update of state -> override values
         * Diamond Hierarchy of state update -> leads to Invalid Update.
         * Use Reducers -> specify how to perform state update
         * Annotated -> [list[int], <operation>]
         * Custom Reducers
         * Passing the message with an id -> updates the message.
         * RemoveMessage -> delete message by Id.

     ☐ Private State & Overall State
         - You could communicate between graphs in private state but output only the overall state.
         * node(state: PrivateState) -> OverallState
         * node(state: OverallState) -> PrivateState
         - StateGraph (state, input=io, output=oo) -> type hints -> to allow for filtering!

     ☐ Add Messages Reducer can recognize RemoveMessage and act on it.

     ☐ Langchain core trim messages 
         * max_tokens, strategy last.

     ☐ Langgraph checkpoint persitence
         * sqlite 
         * Graph.getstate.

 ☐ Stream and AStream
     * mode = values -> append
     * mode = updates -> update
     * graph.stream.

     - Streaming Tokens
         * Generally with LLMs
         * .astream_events
             - Streams back events as they happen inside nodes
         * Steam mode = messages.
             * messages -> partial -> tokens from LLM.
             * helper function.

 ☐ Breakpoints
     - Approval
         * interrupt_before = ['tools'] -> some node.
         * builder.compile(interrupt_before)
         * state = graph.get_state( thread )
            state.next.
         - Graph 
             - Super-Steps
                 - Checkpoints
                     - Thread
                         - StateSnapshot
              get_state -> most recent checkpoint
              get_state_history -> list of all checkpoints
              stream(None, thread_id) -> Execute graph from current state
          - Approval adds an interrupt.
          - Stream runs from there.

     - Debugging
     - Editing
     - graph.update_state.
         - graph.update_state, as_node='node name'
     - no-op dummy node
     - Dynamic Breakpoints
         - Dynamic Node Interrupt - raise NodeInterrupt.
     - graph.get_state_history -> get all states.
     - graph.stream(None, {checkpoint_id, thread_id}) - Replay and not execute.
     - Forking
         - Create a checkpoint, and run it and re-execute.
         - update_state -> needs to be passed an id to override the existing state - to overide an existing one.
         - reducer on messages is 'append' unless you give it an 'id' -> in which case it will override.

 ☐ Parallelization
     - reduce for hierarchy and diamond
     - use custom reducer and decide on how to update state.

 ☐ Sub Graphs
     - parent state -> same key as output.
     - child graph state -> same key as parent to use
     - Overlapping Keys
         * both subgraph will have same key, unless there is reducer there is collision.
         * Even when state is not changed in hierarchy. 

     - 'Send' -> send to node, generates nodes dynamically -> like dynamic task mapping.

  ☐ Map Reduce
      - Annotated[list, operator.add]
      - 'Send' -> don't have to manually create edges. automatically create nodes.

  ☐ Long Term Memory
      - Personalized Long Term Memory.
      - Short term vs Long term
          * Short term is within a thread or session
          * Long term is across sessions.
          * Short term - using checkpointer
          * Long term - using Store

      - Types of Long term
          * Semantic  - Facts
              * Profile - single long document.
              * List - lot of documents - retrieval difficult.
              * management vs retrieval.
          * Episodic - Past Agent Actions
          * Procedural - Agent's system prompt.

      - Memory Updating
          * Hot Path
          * Background

      - LangGraph InMemoryStore
          - Session
              * namespace - key, values.
          - put and search (by namespace)
          - get
          - TrustCall 
              * Schema*
