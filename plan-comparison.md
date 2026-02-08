# Rust Learning Plan Comparison

## Student Profile Reminder
- Senior Ruby/Rails developer (years of experience)
- Knows Rust basics already
- Goal: master Rust + gain systems knowledge
- Prefers hands-on learning: small programs + bigger projects
- Uses macOS, Ghostty/Zellij, Neovim
- Learning spans many sessions (needs clear session boundaries and progress tracking)

---

## Plan Summaries

### Plan A: Systems-First (`plan-systems-first.md`)
A bottom-up approach starting from bytes and memory, building toward networking and performance. 20 chapters, each with a "Systems Knowledge" section, exercises, and a mini-project. Concepts are introduced linearly.

### Plan B: Project-Based (`plan-project-based.md`)
13 real-world projects (CLI tool, grep clone, HTTP client, shell, KV store, web scraper, site generator, web framework, SQL database, chat system, macro toolkit, container runtime, Redis clone) with concept checkpoints between groups. ~52 sessions.

### Plan C: Ruby Bridge (`plan-ruby-bridge.md`)
18 chapters, each structured as "Ruby Anchor -> Rust Bridge -> Mental Model Shift -> Aha Moments." Concepts are taught by explicitly contrasting with Ruby equivalents. Includes a Ruby-Rust quick reference table.

### Plan D: Concept Spiral (`plan-concept-spiral.md`)
4 spiral levels (Foundations, Depth, Mastery, Expert), each revisiting the same core concepts at increasing depth. Includes a progression matrix showing how each concept deepens. Bridge projects between spirals.

---

## Evaluation Criteria (1-10 each)

### 1. Pedagogical Soundness (Does the progression make sense?)

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 8 | Excellent logical progression from low-level to high-level. However, frontloading memory/bytes may feel abstract before the student has enough Rust to write real programs. Lifetimes at Chapter 18 (after unsafe!) is questionable ordering. |
| B: Project-Based | 7 | Each project naturally introduces concepts, which is motivating. However, some early projects (note CLI in sessions 1-3) skip over ownership and borrowing fundamentals that will trip up a learner. The concept checkpoints are too brief to fill gaps. |
| C: Ruby Bridge | 8 | Very strong sequencing for someone coming from Ruby. Each chapter builds logically. However, closures get their own chapter (13) separate from iterators (10), which fragments a naturally connected topic. Lifetimes at chapter 11 is a bit late. |
| D: Concept Spiral | 9 | The spiral approach is pedagogically the most sound. Revisiting concepts at increasing depth mirrors how learning actually works. The progression matrix is brilliant for tracking what deepens when. Bridge projects tie spirals together. |

### 2. Engagement (Will it keep a senior Ruby dev motivated?)

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 6 | The systems knowledge is fascinating but the early chapters (bytes, endianness) may bore a senior dev before they build anything meaningful. The hex dump tool in Chapter 1 is not especially exciting. |
| B: Project-Based | 10 | By far the most engaging. Every session builds a real, usable tool with a catchy name. The progression from `rumble` to `prism` (Redis clone) tells a compelling story. A senior dev will feel accomplished after each project. |
| C: Ruby Bridge | 8 | The Ruby comparisons are inherently engaging for a Rubyist ("aha, so THAT'S why!"). But the mini-projects are generic (config parser, shape calculator, task manager). Less exciting than Plan B's real tools. |
| D: Concept Spiral | 7 | The spiral structure keeps things fresh by revisiting concepts in new contexts. But the exercise/project naming is functional rather than inspiring. The bridge projects (minigrep, http-server, mini-redis) are solid but fewer "wow" moments than Plan B. |

### 3. Rust Coverage

Core topics: ownership, lifetimes, traits, generics, async, unsafe, macros, error handling

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 9 | Comprehensive coverage of all core topics. Particularly strong on smart pointers, collections internals, and performance. Each topic gets its own dedicated chapter with depth. |
| B: Project-Based | 8 | All major topics are covered through projects, but coverage depends on project scope. Some topics (lifetimes, generics) may be shallow because they're learned "as needed" rather than thoroughly. Macros get a full dedicated project, which is strong. |
| C: Ruby Bridge | 8 | Good coverage of fundamentals. Slightly weaker on advanced topics -- the capstone is a choice of 3 options rather than covering all areas. Closures split across two chapters adds depth there but macros coverage is light (mentioned only in context of derive). |
| D: Concept Spiral | 10 | The most thorough coverage by design. Each concept is visited 3-4 times at increasing depth. The progression matrix ensures nothing is missed. Includes advanced topics (GATs, Pin, Miri, no_std) that other plans skip. |

### 4. Systems Knowledge Coverage

Core topics: memory, concurrency, networking, OS concepts

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 10 | This is the plan's core strength. Every chapter has a dedicated "Systems Knowledge" section. Covers memory layout, stack frames, heap allocation, file descriptors, syscalls, TCP/IP, CPU caches, branch prediction, and more. Unmatched depth. |
| B: Project-Based | 8 | Systems knowledge is naturally acquired through projects (shell -> processes/pipes, KV store -> storage/fsync, container runtime -> namespaces/cgroups). Good breadth. But systems concepts are learned incidentally rather than systematically. |
| C: Ruby Bridge | 5 | The weakest on systems knowledge. Focus is on Ruby-to-Rust concept mapping, not on understanding the machine. File I/O, networking, and OS concepts are touched but not deeply explored. No coverage of memory layout, cache effects, or syscalls. |
| D: Concept Spiral | 8 | The progression matrix includes a "Memory & Systems" row that deepens from stack/heap to allocators to FFI/mmap. Good coverage but embedded within other concept modules rather than as a standalone focus. |

### 5. Leverage of Ruby Background

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 5 | Minimal Ruby references. The intro mentions Ruby hides the machine, but chapters don't explicitly bridge Ruby knowledge. A Rubyist could follow this plan, but it doesn't capitalize on existing knowledge. |
| B: Project-Based | 6 | The intro mentions Ruby contrasts and some projects echo Ruby tooling (web framework, site generator). But within each project, there's no systematic bridging of Ruby concepts. |
| C: Ruby Bridge | 10 | This is the plan's core strength. Every chapter starts with Ruby, bridges to Rust, and highlights mental model shifts. The Ruby-Rust quick reference table is gold. The "where Ruby intuition helps" and "where it misleads" sections are perfectly targeted. |
| D: Concept Spiral | 6 | Each module has a "Ruby bridge" note at the bottom, but it's a brief paragraph rather than a structural feature. The comparisons are accurate but not as deep or systematic as Plan C. |

### 6. Hands-On Ratio and Project Quality

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 8 | 3 exercises + 1 mini-project per chapter is a solid ratio. Mini-projects are well-designed (hex dump, expression evaluator, CSV parser, plugin system, LRU cache, file sync, chat server, KV store). Good range. |
| B: Project-Based | 10 | The entire plan IS hands-on. Every session produces working code toward a real tool. Projects are creative, well-scoped, and progressively challenging. The extension ideas add replayability. This is the gold standard for hands-on learning. |
| C: Ruby Bridge | 7 | 5 exercises + 1 mini-project per chapter. Exercises are good but mini-projects are somewhat generic (config parser, task manager, shape calculator). Less inspiring than building real tools. |
| D: Concept Spiral | 8 | 3 exercises + 1 mini-project per module, plus bridge projects between spirals. Bridge projects are substantial and well-designed. The module-level mini-projects are functional but smaller in scope than Plans A or B. |

### 7. Session Structure (Can you pick up easily between sessions?)

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 7 | Chapters are self-contained with clear boundaries. Session estimates per chapter are provided. But no explicit progress tracking template or session log format. |
| B: Project-Based | 9 | Each project has numbered sessions with specific goals ("Session 1: do X, Session 2: do Y"). Very easy to pick up where you left off. The concept checkpoints between projects are natural pause points. |
| C: Ruby Bridge | 7 | Chapters are self-contained. The progress tracking table at the end is useful but minimal. No session-level breakdown within chapters. |
| D: Concept Spiral | 9 | Explicit session tracking template with fields for topics, exercises, struggles, and next steps. The spiral structure means each module is a complete unit. Bridge projects are clear milestones. The "Quick Reference: When Stuck" section helps resume after breaks. |

### 8. Completeness and Detail Level

| Plan | Score | Analysis |
|------|-------|----------|
| A: Systems-First | 10 | Exceptionally detailed. Every chapter has learning objectives, systems knowledge section, concepts list, 3 exercises with descriptions, and a mini-project with example output. 915 lines of content. The most thorough single plan. |
| B: Project-Based | 9 | Each project has a clear build plan, Rust concepts list, systems concepts list, step-by-step sessions, and extension ideas. 577 lines. Well-structured but less conceptual depth per topic. |
| C: Ruby Bridge | 9 | Very detailed per chapter with Ruby code examples, Rust equivalents, comparison tables, and 5 exercises each. 936 lines. The Ruby-Rust reference table adds practical value. |
| D: Concept Spiral | 9 | The progression matrix is uniquely valuable. 4 spirals with 6 modules each is comprehensive. The "When Stuck" reference and session tracking template add practical utility. 531 lines but dense. |

---

## Scoring Summary

| Criterion | A: Systems | B: Project | C: Ruby | D: Spiral |
|-----------|-----------|-----------|---------|-----------|
| Pedagogical Soundness | 8 | 7 | 8 | 9 |
| Engagement | 6 | 10 | 8 | 7 |
| Rust Coverage | 9 | 8 | 8 | 10 |
| Systems Knowledge | 10 | 8 | 5 | 8 |
| Ruby Background Leverage | 5 | 6 | 10 | 6 |
| Hands-On / Projects | 8 | 10 | 7 | 8 |
| Session Structure | 7 | 9 | 7 | 9 |
| Completeness / Detail | 10 | 9 | 9 | 9 |
| **Total** | **63** | **67** | **62** | **66** |

---

## Analysis

**No single plan is the clear winner.** Each has distinct strengths:

- **Plan B (Project-Based)** scores highest overall due to exceptional engagement and hands-on quality, but has gaps in systematic concept coverage and systems knowledge depth.
- **Plan D (Concept Spiral)** scores second, with the strongest pedagogical model and Rust coverage, but lower engagement and Ruby leverage.
- **Plan A (Systems-First)** has unmatched systems knowledge depth and detail, but weak engagement and Ruby leverage.
- **Plan C (Ruby Bridge)** has the best Ruby-to-Rust bridging, but weakest systems coverage.

## Recommendation: Hybrid Plan

The ideal plan combines:
1. **Plan D's spiral structure** (pedagogically sound, revisits concepts at depth)
2. **Plan B's real projects** (engaging, practical, builds a portfolio)
3. **Plan C's Ruby bridging** (leverages existing knowledge, accelerates learning)
4. **Plan A's systems knowledge sections** (deep understanding of the machine)

The hybrid uses the spiral structure as the backbone, replaces generic exercises with Plan B's named projects, incorporates Plan C's Ruby comparison format for each module, and adds Plan A's systems knowledge sections to every chapter.
