# agentproto BoF Frequently Asked Questions

This FAQ is aimed at answering some common questions that have been raised ahead of the agentproto BoF. It explains why this work is needed, why the IETF is the right venue, and what is in scope. It also describes how the effort relates to other existing work in and outside the IETF. 

Please read it alongside the primary materials:

- BoF datatracker page: <https://datatracker.ietf.org/group/agentproto/about>
- Approved BoF request: <https://datatracker.ietf.org/doc/bofreq-krishnan-agent-communication-protocols/>
- Draft charter: <https://github.com/jdrosen/aiproto-wg/blob/main/Charter_Merge.md>
- Mailing list: <https://www.ietf.org/mailman/listinfo/agent2agent> 

## Background Info

### What is this BoF about?

AI agents increasingly need to communicate with each other and with tools across the Internet. Today they do so over a patchwork of vendor- and framework-specific mechanisms. The agentproto BoF intends to define open, interoperable protocol building blocks for agent-to-agent and agent-to-tool communication, so that agents and tools from different vendors can work together across administrative and trust boundaries.

### What is an "AI agent" in this context?

An AI agent is an autonomous, adaptive intelligent software system that uses AI models to complete a specific task. Agents often interact with users via chat or voice and act based on the flow of the conversation. To complete tasks on behalf of a human user or another agent, they can independently make decisions, execute actions, and interact with other agents and tools. This BoF is focused on how agents communicate with users, other agents and tools


## Why should we do this work?

### What problem are we trying to solve?

Agentic systems are moving from isolated assistants and single applications toward agents and services that interact at Internet scale. Multiple ecosystem efforts are emerging in parallel, which creates a real risk of fragmentation due to a lack of a shared protocol foundation. The goal of this effort is to facilitate interoperability so that tools and agents can work together across the Internet within multiple trust domains.

### Don't protocols for this already exist (MCP, A2A, etc. )?

Yes, there are already several widely used protocols and frameworks, including the Model Context Protocol (MCP), the Agent2Agent (A2A) protocol, and the AGNTCY framework. This effort is not trying to replace them. 

One drawback is that each of these application layer protocols currently implements its own context handling, session management, security etc. Without a common standardized substrate interoperability between these frameworks becomes difficult. The goal of this effort is to provide shared, building blocks that these protocols can reuse instead of each reinventing a non-interoperable version of the same thing.

### Can you provide a concrete example of a limitation with the current protocols?

One example that has been repeatedly brought up is a multi-step, multi-modal agent task (e.g., booking a business trip across several cities, then changing the request mid-task). Walking through it exposes some gaps with using some of the existing protocols

- HTTP is stateless and it has no session lifecycle to hold a long-running, evolving task together.
- WebSockets does not offer multiplexing, priority, or partial teardown and hence cannot cleanly manage many concurrent streams of different urgency.
- MoQ is promising for low-latency, multiplexed media, but it does not know what an agent is or how to manage context across interrupts and reconnects.

None of the existing IETF transports natively manage agent context across interrupts and reconnects, or coordinate multi-entity (agent/tool/user) interactions. That is the gap the proposed session work targets.

### Why is multimodal support required?

A single agent interaction can mix very different traffic with very different latency needs simultaneously:

- Realtime media: voice, video, screen share.
- Semi-realtime interactive text: chat, streaming LLM output.
- Non-realtime bulk data: context transfer, tool calls, file transfers.

These modalities need to stay synchronized. e.g. a cancellation on a control channel needs to be able to preempt a bulk transfer on another channel. Rather than have each application protocol (MCP, A2A, and others) invent its own non-interoperable way to do this, it would be good to develop a common substrate that can be reused across them.


## Why are we doing this at the IETF?

The core of the proposal, a session layer for agent communication, fits nicely with the IETF's area of expertise. The IETF already provides several of the core underlying pieces this work would build on, such as QUIC, WebRTC, Oauth etc.

Defining an interoperable protocol substrate on top of existing IETF transports, reusable across vendors and frameworks, is exactly the kind of work the IETF does well.

## How does this relate to other IETF work

### How does this work fit alongside other IETF groups?

The framework deliberately treats identity, authorization, discovery, and transport as building blocks that are owned by **other working groups**. agentproto's goal is to bind those blocks into a coherent agent communication scenario, identify gaps, and define the session layer that does not exist. Specifically, 

- **Transport** : MoQ, QUIC, WebTransport, WebRTC, etc. provide the real-time and reliable transport substrate. agentproto consumes these and builds on top.
- **Security & authorization** : wimse (workload/agent identity), oauth (authorization) and potentially webbotauth provide identity and finegrained, behavior-driven authorization. agentproto consumes these specs and incorporates them into a framework.
- **Discovery & operations** : the DAWN BoF covers agent/workload/entity discovery and operational considerations.
- **Application frameworks** : MCP and A2A (and AGNTCY) sit on top the potential outputs of agentproto and are out of scope. They are the targeted consumers for the reuse of the common substrate we intend to define.

### What is the relationship to the DAWN BoF?

DAWN is anothet WG-forming BoF at this IETF meeting that is focused on agent, workload, and entity discovery. Discovery of AI agents is explicitly out of scope for agentproto. We reference and coordinate with DAWN's work (and incorporate it into the framework), but actual work on discovery is deferred to dawn.

### What about WIMSE and OAuth?

Agent identity and authorization are expected to build on work coming out of WIMSE (workload/agent identity) and OAuth. The goal is to enable an agent to create an independent identity and to obtain and exchange access tokens with fine-grained, behavior-driven scopes bound to the specific operations it is permitted to perform on a user's behalf. Importantly, any resulting changes or extensions to OAuth and WIMSE are expected to be worked on **in those groups**, not inside agentproto. This group would raise requirements; those groups decide.


### How does agentproto coordinate with these other groups in practice?

The intended coordination model is unidirectional: if agentproto participants identify the need for changes to another group's protocols, they need to bring these requests to the relevant WGs and the other WG decides how to deal with those requests. 

The expected steps are (1) identify a gap, (2) present the requirement to the relevant WG, (3) that WG owns the decision and the work, (4) agentproto consumes the output. 

### How does this relate to work outside the IETF, like MCP and A2A?

MCP, A2A, and the AGNTCY framework are being developed in other venues (including the Linux Foundation). This work is not intended to compete with them but rather to provide an interoperable, standardized substrate that such frameworks can build on.


