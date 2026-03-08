<h1 align="center">🧠 Voyager-VRAM</h1>
<h3 align="center"><em>A Workplace Simulator for Training Memory-Aware Agents</em></h3>

<p align="center">
  <a href="#the-voyager-architecture"><img src="https://img.shields.io/badge/Architecture-Voyager--Lite-blue?style=for-the-badge" alt="Architecture"></a>
  <a href="#system-details"><img src="https://img.shields.io/badge/Model-Qwen2.5--1.5B-orange?style=for-the-badge" alt="Model"></a>
  <a href="#training--results"><img src="https://img.shields.io/badge/Training-Expert_Iteration-green?style=for-the-badge" alt="Training"></a>
</p>

---

## The Problem

Imagine you're managing a 6-week software project. You get 100 Slacks, 50 emails, meeting notes, spreadsheets, calendar invites — information comes at you constantly. Three weeks in, the client asks:

> *"When did we agree to cut the mobile feature?"*

…and you have to remember a decision buried in a Slack thread from week one.

The best project managers have **extraordinary working memory**. They remember who said what, which tasks block which, and what changed since last week.

**Can we train an LLM to develop that same skill?**

That's what we built.

<p align="center">
  <img src="images/HF-space.png" alt="VRAM-PM Hugging Face Space — Live Environment Demo" width="85%" />
  <br />
  <em>VRAM-PM: Live environment on Hugging Face Spaces</em>
</p>

---

## System Details

| Component | Detail |
|---|---|
| **Model** | Qwen2.5-1.5B-Instruct (4-bit quantized via Unsloth) |
| **LoRA Rank** | 16 (18.5M trainable params out of 1.56B = **1.18%**) |
| **Training** | Expert Iteration — Best-of-4 rejection sampling + SFT × 3 rounds |
| **Data** | 32 Voyager-enhanced prompts per round |
| **Environment** | 31 tools across 7 channels, 25-step episodes, `client_brief` scenario |

---

## Environment: 31 Tools × 7 Channels

The agent operates in a realistic workplace simulation with **31 tools** spread across **7 communication channels** — mirroring how a real project manager navigates information.

```mermaid
graph LR
    Agent["🤖 <b>WorkSim Agent</b><br/>Working Memory · Skill Library"]

    Agent --> Email["📧 <b>Email</b> · 5 tools<br/><sub>list_inbox · search · open_thread · send · reply</sub>"]
    Agent --> Slack["💬 <b>Slack</b> · 5 tools<br/><sub>list_channels · search · read_channel · open_thread · send_message</sub>"]
    Agent --> Drive["📁 <b>Shared Drive</b> · 4 tools<br/><sub>list_files · open_file · search · upload</sub>"]
    Agent --> Sheets["📊 <b>Spreadsheets</b> · 5 tools<br/><sub>list · open · read_cell · read_range · write_cell</sub>"]
    Agent --> Calendar["📅 <b>Calendar</b> · 4 tools<br/><sub>list_events · check_conflicts · create_event · search</sub>"]
    Agent --> Notes["📝 <b>Notes</b> · 4 tools<br/><sub>list · read · create · append</sub>"]
    Agent --> Meta["🔍 <b>Meta-Search</b> · 4 tools<br/><sub>search_global · workspace_status · submit_deliverable · workspace_context</sub>"]

    style Agent fill:#1a1a2e,stroke:#e94560,stroke-width:2px,color:#fff
    style Email fill:#0f3460,stroke:#e94560,color:#fff
    style Slack fill:#0f3460,stroke:#e94560,color:#fff
    style Drive fill:#0f3460,stroke:#e94560,color:#fff
    style Sheets fill:#0f3460,stroke:#e94560,color:#fff
    style Calendar fill:#0f3460,stroke:#e94560,color:#fff
    style Notes fill:#0f3460,stroke:#e94560,color:#fff
    style Meta fill:#0f3460,stroke:#e94560,color:#fff
```

The agent can search mail, open threads, read spreadsheet cells, check calendar conflicts, write memos, and submit deliverables — all through structured tool calls within a 25-step episode.

---

## Tool Usage: Pattern Repetition vs. Real Exploration

<p align="center">
  <img src="images/worksim_voyager_dashboard_panelB.png" alt="Panel B — Tool Usage Concentration Heatmap" width="85%" />
  <br />
  <em>Panel B (top-left of bottom section) for tool usage heatmap</em>
</p>

**Look at the difference.** The basic LLM just hammers `mail.list_inbox` over and over. The Voyager agent uses **diverse tools** — it searches, opens threads, reads spreadsheets, checks chat. That's real exploration, not pattern repetition.

### Hidden State: What Makes This Genuinely Hard

But here's what makes it genuinely hard: **the world has hidden state.**

- 📊 There's a **stale budget spreadsheet** — the numbers are outdated
- 💬 A constraint the VP mentioned **in chat but never put in email**
- 📅 A **deadline that changed** without a formal announcement

The agent must **discover** these through tool use — just like a real PM. No information is handed to it; everything must be actively retrieved, cross-referenced, and reconciled.

---

## The Voyager Architecture

On top of the environment, we built a **Voyager-inspired learning layer**. Three components:

### 1. 📚 Skill Library

After successful episodes, the agent extracts **reusable tool-call sequences** as skills. In our runs, it learned two skills:
- A **mail listing** skill
- A **mail-search-then-open** workflow

These get stored and reused in later episodes, compounding capability over time.

<p align="center">
  <img src="images/worksim_voyager_dashboard_panelE.png" alt="Panel E — Facts Discovered & Skill Library Growth" width="85%" />
  <br />
  <em>Panel E — Facts discovered and skill library growth across episodes. Two skills were extracted after the warm-up phase.</em>
</p>

### 2. 🧠 Working Memory

Structured state that **persists within an episode**:

| Slot | Purpose |
|---|---|
| **Current Goal** | What the agent is trying to achieve right now |
| **Active Plan** | Step-by-step plan being executed |
| **Discovered Facts** | Information retrieved from the environment |
| **Pending Subgoals** | Tasks queued for later |
| **Recent Errors** | Failed actions to avoid repeating |

### 3. 🔄 Episodic Memory

**Cross-episode learning** — what worked, what failed, which tools were effective. After 23 episodes, the agent has accumulated patterns that guide future exploration.

---

### The Result?

<p align="center">
  <img src="images/worksim_voyager_dashboard_panelC.png" alt="Panel C — Mean Shaped Reward per Episode" width="85%" />
  <br />
  <em>Panel C — Voyager-lite scores 5.75 shaped reward vs. 4.74 for the basic LLM.</em>
</p>

**Voyager-lite scores 5.75** shaped reward versus **4.74 for the basic LLM**. That's a **21% improvement** — and this is *before any training*. The architecture itself adds value.

---

## Training & Results

For training, we use **Expert Iteration**:

1. Generate **4 trajectories** per prompt
2. Keep the **best** (by shaped reward)
3. **Fine-tune with SFT**
4. Repeat × **3 rounds**, 32 prompts each

All training uses **Unsloth with LoRA** on Qwen 2.5 1.5B (4-bit).

<p align="center">
  <img src="images/worksim_voyager_dashboard_panelF.png" alt="Panel F — SFT Loss Across Rounds" width="85%" />
  <br />
  <em>Panel F — SFT loss drops consistently: 8.78 → 8.60 → 8.50. The model is learning.</em>
</p>

**SFT loss drops consistently** — 8.78 → 8.60 → 8.50. The model is learning.

And look at the capability profile after training:

<p align="center">
  <img src="images/worksim_voyager_dashboard_panelD.png" alt="Panel D — Agent Capability Radar Chart" width="85%" />
  <br />
  <em>Panel D — The Voyager post-train agent has the broadest capability profile: higher tool diversity, more facts discovered, better shaped reward.</em>
</p>

The **Voyager post-train agent** has the broadest capability profile: higher tool diversity, more facts discovered, better shaped reward. It's not just scoring higher — **it's exploring more intelligently**.

---

## Why This Matters

This environment tests exactly what cutting-edge research — **MEM1**, **Memory-R1**, **Mem-alpha** — is trying to solve:

> **Long-horizon information management with tool use.**

We're also building a second environment layer, **VRAM-PM**, that adds:
- 🔻 **Shrinking memory budgets** — forcing consolidation
- 🔍 **Explicit memory probes** — forcing agents to learn *what to forget*


---
