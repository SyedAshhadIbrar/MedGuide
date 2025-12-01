# 🏥 MedGuide: AI-Powered Medical Learning Companion

> **A multi-agent system for personalized medical education featuring parallel processing, spaced repetition memory, and real-time PubMed integration.**


## 📋 Table of Contents

1. [Problem Statement](#-problem-statement)
2. [Solution Overview](#-solution-overview)
3. [Architecture](#-architecture)
4. [ADK Concepts Demonstrated](#-adk-concepts-demonstrated)
5. [Features](#-features)
6. [Installation & Setup](#-installation--setup)
7. [Usage Examples](#-usage-examples)
8. [Project Structure](#-project-structure)
9. [Technical Implementation Details](#-technical-implementation-details)

---

## 🎯 Problem Statement

Medical students and healthcare professionals face significant challenges in self-directed learning:

| Challenge | Impact |
|-----------|--------|
| **Information Overload** | Medical knowledge is vast (~2 million PubMed articles/year) |
| **Fragmented Resources** | Content scattered across textbooks, papers, guidelines |
| **No Personalization** | Generic materials don't adapt to individual progress |
| **Poor Retention** | Without spaced repetition, 70% forgotten within days |
| **Limited Feedback** | Self-assessment separate from learning materials |

**The Result:** Learners spend excessive time searching, have no systematic progress tracking, and lack efficient retention methods.

---

## 💡 Solution Overview

**MedGuide** is an AI-powered medical learning companion that addresses these challenges through a sophisticated multi-agent system:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MedGuide Solution                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  ✓ Ask ANY medical question → Get comprehensive explanation                 │
│  ✓ Request quiz → Receive MCQs + flashcards                                │
│  ✓ Need study plan → Get personalized 5-7 day schedule                     │
│  ✓ Want research → Real PubMed articles retrieved                          │
│  ✓ Track progress → Spaced repetition remembers what you studied           │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why Agents?

Unlike a single LLM, MedGuide uses **10 specialized agents** that collaborate:

- **Parallel execution** speeds up information gathering by 2-3x
- **Sequential pipelines** ensure content builds logically
- **Intelligent routing** means each agent focuses on its specialty
- **Persistent state** enables truly personalized learning journeys

---

## 🏗️ Architecture

### High-Level System Design

```
                              ┌─────────────────┐
                              │   User Input    │
                              │ "What causes    │
                              │  B12 deficiency"│
                              └────────┬────────┘
                                       │
                                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                        MedGuideRouter (Orchestrator)                         │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                    STEP 1: Intent Classification                        │ │
│  │  ┌──────────────────┐                                                   │ │
│  │  │ Intent Classifier │ → {"intent": "study_concept",                    │ │
│  │  │      Agent        │    "topic": "vitamin B12 deficiency"}            │ │
│  │  └──────────────────┘                                                   │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                       │                                      │
│                    ┌──────────────────┼───────────────────┐                 │
│                    │                  │                   │                 │
│                    ▼                  ▼                   ▼                 │
│  ┌─────────────────────┐  ┌───────────────────┐  ┌──────────────────────┐  │
│  │     PATH A          │  │     PATH B        │  │      PATH C          │  │
│  │   Simple Routes     │  │  Quiz/Plan Only   │  │   Full Pipeline      │  │
│  │                     │  │                   │  │   (mixed_tutor)      │  │
│  │ • chitchat          │  │ • create_quiz     │  │                      │  │
│  │ • study_history     │  │ • make_study_plan │  │  ┌────────────────┐  │  │
│  │ • recommendations   │  │                   │  │  │   PARALLEL     │  │  │
│  │ • pubmed_search     │  │  Sequential:      │  │  │ ┌────────────┐ │  │  │
│  │ • study_concept ◄───┼──┤  Explain(silent)  │  │  │ │ Literature │ │  │  │
│  │                     │  │       ↓           │  │  │ │   Agent    │ │  │  │
│  └─────────────────────┘  │  Quiz/Plan Agent  │  │  │ └────────────┘ │  │  │
│           │               │                   │  │  │ ┌────────────┐ │  │  │
│           │               └───────────────────┘  │  │ │ Guideline  │ │  │  │
│           │                         │            │  │ │   Agent    │ │  │  │
│           │                         │            │  │ └────────────┘ │  │  │
│           │                         │            │  └────────────────┘  │  │
│           │                         │            │          │           │  │
│           │                         │            │          ▼           │  │
│           │                         │            │  ┌────────────────┐  │  │
│           │                         │            │  │   SEQUENTIAL   │  │  │
│           │                         │            │  │                │  │  │
│           │                         │            │  │  Concept       │  │  │
│           │                         │            │  │  Explainer     │  │  │
│           │                         │            │  │      ↓         │  │  │
│           │                         │            │  │  Quiz          │  │  │
│           │                         │            │  │  Generator     │  │  │
│           │                         │            │  │      ↓         │  │  │
│           │                         │            │  │  Study Plan    │  │  │
│           │                         │            │  │  Builder       │  │  │
│           │                         │            │  │      ↓         │  │  │
│           │                         │            │  │  Response      │  │  │
│           │                         │            │  │  Synthesizer   │  │  │
│           │                         │            │  └────────────────┘  │  │
│           │                         │            └──────────────────────┘  │
│           │                         │                       │              │
│           └─────────────────────────┼───────────────────────┘              │
│                                     │                                      │
│                                     ▼                                      │
│  ┌────────────────────────────────────────────────────────────────────────┐│
│  │                    Memory Update (Spaced Repetition)                    ││
│  │  memory_db.json ← Record topic, timestamp, difficulty, next_review     ││
│  └────────────────────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
                              ┌─────────────────┐
                              │  Final Response │
                              │  (Clean output  │
                              │   to user)      │
                              └─────────────────┘
```

### Agent Interaction Matrix

| Agent | Input | Output | Tools | Runs In |
|-------|-------|--------|-------|---------|
| Intent Classifier | User message | JSON intent | None | Always first |
| Literature Searcher | Topic | PubMed summary | `pubmed_search` | Parallel |
| Guideline Explainer | Topic | Guidelines summary | None | Parallel |
| Concept Explainer | Topic + context | Explanation | None | Sequential |
| Quiz Generator | Explanation | MCQs + flashcards | None | Sequential |
| Study Plan Builder | Topic | 5-7 day plan | `save_study_plan` | Sequential |
| Smalltalk Handler | Greeting | Welcome message | None | Direct |
| Study History | Memory data | Progress summary | None | Direct |
| Study Recommender | Memory data | Recommendations | None | Direct |
| Response Synthesizer | All outputs | Combined response | None | Final |

---

## 🎓 ADK Concepts Demonstrated

This project demonstrates **8 key ADK concepts** (requirement: minimum 3):

### 1. Multi-Agent System ✅
```python
# 10 specialized LLM agents with distinct roles
sub_agents=[
    intent_classifier_agent,      # Routes requests
    concept_explainer_agent,      # Primary educator
    literature_search_agent,      # PubMed integration
    guideline_explainer_agent,    # Clinical guidelines
    quiz_generator_agent,         # Assessment creation
    study_plan_builder_agent,     # Study scheduling
    smalltalk_agent,              # Greetings
    study_history_agent,          # Progress tracking
    study_recommender_agent,      # Personalized suggestions
    response_synthesizer_agent,   # Output combining
]
```

### 2. Parallel Agents ✅
```python
async def _run_agents_parallel(self, agents, ctx):
    """Run multiple agents concurrently - 2-3x faster"""
    await asyncio.gather(*[run_one(agent) for agent in agents])

# Usage: Literature + Guidelines run simultaneously
await self._run_agents_parallel(
    [literature_search_agent, guideline_explainer_agent],
    ctx
)
```

### 3. Sequential Agents ✅
```python
# Explain → Quiz → Plan pipeline ensures logical flow
await self._run_agent_silent(concept_explainer_agent, ctx)
await self._run_agent_silent(quiz_generator_agent, ctx)
await self._run_agent_silent(study_plan_builder_agent, ctx)
```

### 4. Custom Tools ✅
```python
# Real PubMed API integration
def pubmed_search(query: str, max_results: int = 5) -> Dict:
    """Search NCBI E-utilities API for peer-reviewed articles"""
    # ESearch → EFetch → Parse XML → Return articles
    
# Study plan persistence
def save_study_plan(user_id: str, plan_content: str) -> Dict:
    """Save generated study plans to markdown files"""
```

### 5. Sessions & State Management ✅
```python
# Inter-agent communication via session state
ctx.session.state["detected_intent"] = intent
ctx.session.state["detected_topic"] = topic
ctx.session.state["concept_explanation"] = explanation
# Downstream agents read from state
```

### 6. Long-Term Memory ✅
```python
class StudyMemoryManager:
    """Persistent JSON storage with spaced repetition"""
    
    def record_study_session(self, topic, intent):
        # Track: times_studied, last_studied, difficulty, next_review
        
    def get_due_topics(self):
        # Return topics where next_review <= today
```

**Memory Structure:**
```json
{
  "vitamin B12 deficiency": {
    "topic": "vitamin B12 deficiency",
    "times_studied": 3,
    "last_studied": "2025-12-01 14:30",
    "difficulty": "medium",
    "next_review": "2025-12-04",
    "notes": ["study_concept on 2025-12-01"]
  }
}
```

### 7. Context Engineering ✅
```python
# Shared system context for consistent behavior
MEDGUIDE_SYSTEM_CONTEXT = """
CORE PRINCIPLES:
1. BE HELPFUL - Answer directly, don't redirect
2. BE EDUCATIONAL - Explain with clinical relevance
...
"""

# Learner context injection for personalization
def _inject_context(self, ctx):
    ctx.session.state["learner_context"] = memory_manager.get_context_for_agents()
```

### 8. Observability ✅
```python
# Structured logging throughout
logging.basicConfig(
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s"
)

logger.info("MedGuide: Processing request")
logger.info(f"Intent: {intent}, Topic: {topic}")
logger.info("Route: Full Pipeline (mixed_tutor)")
```

---

## ✨ Features

### Intelligent Intent Classification
- 10 intent categories with priority-based routing
- Medical questions default to educational response
- Robust JSON parsing handles various LLM outputs

### Real PubMed Integration
- Live NCBI E-utilities API queries
- Returns actual peer-reviewed articles
- Includes PMIDs, abstracts, authors, journals

### Spaced Repetition Memory
- Persistent JSON storage between sessions
- Automatic review scheduling (1/3/7 days)
- Difficulty tracking adapts to performance

### Clean Output Filtering
- Internal agent events hidden from user
- Only final, synthesized response shown
- No backend noise or debug information

---

## 🚀 Installation & Setup

### Prerequisites
- Python 3.10+
- Google ADK installed
- Gemini API key

### Step 1: Clone Repository
```bash
git clone https://github.com/YOUR_USERNAME/medguide-agent.git
cd medguide-agent
```

### Step 2: Install Dependencies
```bash
pip install google-adk
```

### Step 3: Configure API Key
```bash
# Linux/Mac
export GOOGLE_API_KEY="your-gemini-api-key"

# Windows PowerShell
$env:GOOGLE_API_KEY="your-gemini-api-key"
```

### Step 4: Run the Agent
```bash
# Terminal interface
adk run medguide

# Web interface
adk web medguide
```

---

## 📖 Usage Examples

### Learning a Concept
```
User: What are the causes of vitamin B12 deficiency?

MedGuide: ## 📚 Vitamin B12 Deficiency

### Overview
Vitamin B12 (cobalamin) deficiency occurs when the body lacks adequate B12...

### Causes / Etiology
• **Dietary:** Strict veganism, malnutrition
• **Malabsorption:** Pernicious anemia, gastric surgery, Crohn's disease
• **Medications:** Metformin, PPIs, H2 blockers
• **Increased demand:** Pregnancy, hyperthyroidism

### Mechanism / Pathophysiology
B12 requires intrinsic factor (IF) from gastric parietal cells...

### 🎯 Key Points to Remember
1. Most common cause in developed countries: Pernicious anemia
2. Neurological symptoms can precede hematological findings
3. Check methylmalonic acid (MMA) when B12 borderline
4. Schilling test historically used but rarely performed today
```

### Getting a Quiz
```
User: Quiz me on diabetes

MedGuide: ## 📝 Quiz: Diabetes

**Q1.** A 52-year-old obese man presents with polyuria, polydipsia...
A) Type 1 DM  B) Type 2 DM  C) LADA  D) MODY  E) Secondary DM

**Answer:** B - Type 2 DM (classic presentation with obesity, age >45)

**Flashcard 1:**
Front: What is the diagnostic threshold for fasting glucose in diabetes?
Back: ≥126 mg/dL (7.0 mmol/L) on two occasions
```

### Tracking Progress
```
User: What have I studied?

MedGuide: ## 📊 Your Learning Progress

**Topics Studied:** 5
**Due for Review:** 2

• vitamin B12 deficiency: studied 3x, next review: tomorrow
• diabetes mellitus: studied 2x, next review: in 2 days
• hypertension: studied 1x, next review: today (overdue!)
```

---

## 📁 Project Structure

```
medguide/
├── __init__.py              # Module exports (root_agent)
├── agent.py                 # Main implementation (~800 lines, heavily commented)
├── memory_db.json           # Persistent study memory (auto-generated)
├── study_plan_default_user.md  # Saved study plans (auto-generated)
└── README.md                # This documentation
```

---

## 🔧 Technical Implementation Details

### Intent Classification Strategy
```
Priority Order:
1. study_concept (DEFAULT) - Any medical question
2. create_quiz - Explicit quiz request
3. make_study_plan - Explicit plan request
4. mixed_tutor - Multiple requests
5. clinical_question - Case scenarios
6. study_history - Progress inquiry
7. study_recommendation - Next steps
8. pubmed_search - Research request
9. chitchat - Pure greetings only
10. other - Fallback
```

### Spaced Repetition Algorithm
```
Difficulty → Interval:
  hard   → 1 day  (clinical questions)
  medium → 3 days (default)
  easy   → 7 days (studied 5+ times)
```

### Parallel vs Sequential Execution
```
Parallel (asyncio.gather):
  Literature + Guidelines → 2-3x faster

Sequential (await in order):
  Explain → Quiz → Plan → Synthesize
```

---

## 📜 License

MIT License - See LICENSE file for details.

---

## 🙏 Acknowledgments

- **Google ADK Team** - Excellent agent development framework
- **5-Day AI Agents Intensive Course** - Comprehensive training
- **NCBI/PubMed** - Open access to medical literature APIs

---

*Built with ❤️ for medical learners everywhere*

**Author:** Syed Ashhad Ibrar & Syed Muhammad Ayyan Ibrar  
**Track:** Agents for Good (Healthcare/Education)  
**Competition:** Google AI Agents Intensive Capstone Project
