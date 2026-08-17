---
version: 0.1
---

# The Harness — a Primer

The word "harness" is used constantly and defined rarely. This primer defines it across three
architecture layers, in that order, because the usual explanations start at the last one and
therefore explain nothing.

| Layer | Question | Who works here |
|---|---|---|
| [Conceptual](#1-conceptual-harness) | WHY does a harness need to exist? | architects |
| [Logical](#2-logical-harness) | WHAT does it consist of? | product owners, business analysts |
| [Physical](#3-physical-harness-app-architecture) | HOW is one built? | engineers |

See [Vocabulary](#vocabulary) for terms.

---

## 1. The Harness Concept

**WHY does a harness need to exist?**

![alt text](../assets/harness-conceptual-human-model-conversation.png)

Conceptually the model is global multi user machinery answering general purpose questions, in the from of textual prompts. Replying with the single text to the remote caller.  

The first obvious problem it has is, it must be **stateless**. Conversation state management would be practically impossible for hundreds of millions of simultanoeus conversations.

The sole purpose of Harness existence is a model conversations management simulation.  Simulation is achieved by composing a (small) book of pages around each new user prompt arrived. Those pages describe the full conversation context. "Everything" on the caller side.

Harness on first reply from the model, will keep composing a new book for next call. Using again the calling context and crucially the bits from the previous reply it just received. And so on in a "loop" until the model reply has no enough feedback for the next book to be made. At which moment harness decides to compose the final reply for the caller to be used.

So the whole "play" is full of assumptions and non deterministic events. But. It still works somehow. Evidently. 

---

## 2. Logical Architecture 

**WHAT does it consist of?**

![Logical: four roles](../assets/harness-logical.png)

> Architecture is the formal description of a system *and its parts*.

The conceptual layer is where the requirements started forming. This layer names the parts that make the architecture manageable and must act together to meet them.

Each part gets two halves. **Why it exists** is the requirement inherited from the concept. **What
it does** is its role — and roles are what this layer adds.

### Calling Site — the calling environment, the user's habitat

**Why it exists** — to start the conversation, and to use whatever benefit the Model can deliver.
The concept says nothing about where the caller sits; the moment we build, it has to sit somewhere.

**What it does** — issues the prompt, supplies whatever local context exists, receives the final
answer. It also owns the refusal: effects are real, and the calling site is where a human can say no.

### Harness — the functional point relying on the Model's presence and abilities

**Why it exists** — the Model is stateless, so the conversation has to be simulated by somebody.
That is the requirement the concept hands over, and nothing else in the picture can take it.

**What it does** — keeps the book, sends it, reads the reply, carries out what the reply asks for,
and decides whether to go round again.

Note the word **functional**. The Harness is a process: prompt and context in, final answer out.
Nothing about it requires a human to be watching. That is precisely what makes automation possible
— the Harness can be invoked by a script, a schedule, or another Harness, and the shape does not
change. The human at the calling site is optional. That is the whole opening for agentic automation,
and it exists because the Harness is a function, not a conversation partner.

### Model

**Why it exists** — it is the only part that can answer. Without it there is no conversation to
simulate.

**What it does** — reads the book, returns a reply, forgets. Nothing else. It cannot act, and it
does not know it is in a loop.

### Tools

A managed appearance of the calling environment's functionality, composed as a controlled set.

**Why it exists** — the Model produces text and text alone. A conversation that changes nothing is
not work, so something must convert text into real effects. Note *controlled*: because the effects
are real, the set of what may be done is deliberately bounded.

**What it does** — ordinary local operations: write the file, run the command, fetch the page.
Reports back what happened, as text, because text is the only thing that can go into the next book.

### The shape that results

Only the Harness touches everything. The Calling Site never reaches the Model. The Model never
reaches a Tool. Everything passes through the middle, because the middle is the only part that
remembers anything.

And one responsibility lands there that nobody assigned: **deciding when to stop**. The calling site
is not in the loop. The Model does not know there is one. So the Harness judges each reply and
decides whether the task is finished — a decision made on assumptions, by the one part that was
never told what "finished" means.

---

## 3. Physical Harness App Architecture

**HOW is one built?**

![](../assets/the-harness-app-arch.png)


The base of the diagram above is the widely circulated one. It is included because it is what people have seen, and it needs caveats, because it was drawn without a conceptual or logical layer before it.

### Caveats on the original

1. **"Claude Code CLI is the Harness."** No. It is *an implementation* of a harness. The harness
   is the concept; this is one product realising it. Others exist and more will.

2. **"Turning a model into an agent."** Marketing. The model is unchanged throughout. What is
   assembled is a conversation loop around it.

3. **"Agent behavior (emergent)."** Nothing in the application implements *being an agent*. The
   model emits text, the app dispatches and feeds results back. Run that loop enough times and it
   resembles something working on a task. "Emergent" is a real term doing promotional work here —
   it dresses up a while-loop.

4. **The model is drawn inside the machine.** It is not. It is a remote endpoint. Inside the
   application there is only a call to it and a wait for the reply.

5. **Tools appear to be parts of the application.** They are not. They are call sites. The app
   shells out to programs already installed on the desktop — bash, an editor, a browser — and
   reads back their output. No tool lives inside the app.

6. **The illustration implies engineering.** Gears, pistons, linkages. The actual mechanism is a
   loop over a text protocol.

### What the physical layer actually contains

Read left to right, the honest version of the same diagram:

1. **Context assembly** — builds the payload for every turn: user prompt, `CLAUDE.md` files,
   `memory/*.md`, conversation so far, previous tool results, tool schemas. Concatenated into one
   text blob. Repeated in full each turn, because the model is stateless. (This component has no
   official name; the label is descriptive.)

2. **Remote call** — sends that payload to the model endpoint and waits.

3. **Reply parser** — reads the returned text. It may contain a structured `tool_use` block naming
   a tool and its arguments. The model never calls anything; it emits a name. The parser looks that
   name up in the application's dispatch table.

4. **The decision** — `tool_use` present? If yes, the loop continues. If no, the loop exits and the
   reply is the answer. Note the assumption buried here: *absence of a tool_use block means done*.
   That is the application deciding, not the model saying so.

5. **Guardrails / permission system** — the boundary of what may be executed, and the point where a
   human can refuse. This is the accountability requirement from the conceptual layer, made real.

6. **Tool call loop** — dispatch to external programs, collect stdout, exit codes, file contents.
   All of it text, appended to the payload for the next turn.

The tool names and their parameter schemas were sent to the model in the payload. That is how the
model knows which names exist. There is no magic in the arrangement — only a text protocol,
a dispatch table, and a loop.

---

## Vocabulary

**Harness** — the interlocutor that keeps the conversation state on behalf of a stateless model,
and turns its replies into effects. A concept, not a product.

**Stateless** — retains nothing between calls. Every request to the model must carry the entire
context it needs.

**Context / payload** — the full text sent to the model on a single call. Instructions, known
material, tool schemas, conversation history, prior results, and the new input.

**tool_use** — a structured block the model may emit inside its reply, naming a tool and its
arguments. The model emits it; the harness executes it.

**Tool schema** — the machine-readable description of a tool's name and parameters, sent to the
model so it knows what it may ask for.

**Guardrails / permission system** — the mechanism deciding which requested effects are allowed,
and where a human may intervene.

**Emergent** — used in the original diagram to mean "agent-like behaviour that no single component
implements". Accurate in the narrow sense, promotional in effect.

**Conceptual / Logical / Physical** — the [DBJ Taxonomy](https://method.dbj.org/taxonomy_core.html)
layers used to structure this document: why, what, and how.

---

(c) 2026 by dbj@dbj.org | MIT license
