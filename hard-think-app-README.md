# hard.think

A eudaimonic knowledge challenge game. Answer intellectually difficult questions using only curated academic sources. Track your growth across knowledge domains through the Wheel of Knowledge.

---

## Overview

hard.think is a game about learning, not winning. Players receive challenging questions across knowledge domains — science, mathematics, philosophy, history, and more — and must find correct answers using only curated, human-authored academic sources. Performance is tracked through the **Wheel of Knowledge**, a multi-axis visualization that reflects engagement, growth, and the boundaries of what you know.

The game is an ode to the vast, freely available body of human knowledge — and an invitation to engage with it seriously.

**This is a passion project, not a commercial product.** It's built to push development skills, explore creative design problems, and contribute something meaningful to the world.

---

## Philosophy

hard.think is designed as a **eudaimonic game** — oriented toward meaning, self-knowledge, and reflective growth rather than compulsion or competition.

**Design principles:**

- No streaks, no dopamine-loop gamification, no artificial urgency
- "Gave up" is not a failure state — it's a discovery moment
- The Wheel of Knowledge is not a leaderboard; it's a mirror
- Difficulty should provoke curiosity, not frustration
- Respect the player's intelligence without being preachy

---

## Gameplay Loop

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   1. Receive Question                                    │
│      ↓                                                   │
│   2. Research using restricted academic sources           │
│      ↓                                                   │
│   3. Submit answer                                       │
│      ↓                                                   │
│   4. Answer graded with confidence scoring                │
│      ↓                                                   │
│   ┌──────────┬──────────────┬─────────────┐              │
│   │ Correct  │  Ambiguous   │  Incorrect  │  Gave Up     │
│   │          │  "Might be   │             │  → Surface   │
│   │          │   right..."  │             │    answer,   │
│   │          │              │             │    source,   │
│   │          │              │             │    domain    │
│   └────┬─────┴──────┬───────┴──────┬──────┘              │
│        └────────────┼──────────────┘                     │
│                     ▼                                    │
│   5. Wheel of Knowledge updates                          │
│      ↓                                                   │
│   6. Next question                                       │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**MVP approach:** Approved sources are provided as clickable links. Players research in those sources and return to submit their answer. The restricted search tool (custom proxy/scoped engine) is a long-term goal.

---

## Wheel of Knowledge

A multi-axis radar chart spanning 6–8 knowledge domains. It is the visual and philosophical centerpiece of the game.

### Domains (Provisional)

| Domain | Scope |
|--------|-------|
| Natural Sciences | Physics, chemistry, biology, earth sciences |
| Mathematics | Pure and applied mathematics, logic, statistics |
| Philosophy | Ethics, epistemology, metaphysics, political philosophy |
| History | World history, civilizations, historical events and figures |
| Technology & Computing | Computer science, engineering, IT |
| Arts & Literature | Visual arts, music, literature, cultural studies |
| Social Sciences | Psychology, sociology, economics, political science |
| Geography & Earth | Physical geography, geopolitics, climate, ecology |

### Scoring Formula

```
score_delta = base_points × difficulty_multiplier × time_factor × attempt_penalty × confidence_modifier
```

**Input signals:**

| Signal | Description | Effect |
|--------|-------------|--------|
| `base_points` | Fixed points per question | Foundation of score |
| `difficulty_multiplier` | Scales with question difficulty (1–10) | Harder questions = more movement |
| `time_factor` | Decays as time to answer increases | Rewards knowledge fluency, but slowly — research should not be penalized harshly |
| `attempt_penalty` | Scales down with incorrect attempts | Multiple wrong guesses reduce score delta |
| `confidence_modifier` | From answer grading confidence score | Ambiguous answers still contribute, but less |

**Design stance:** The Wheel measures *engagement and growth*, not raw correctness. A player who researches deeply and arrives at a correct answer after effort should see meaningful Wheel movement. Quick correct answers and slow-but-thorough correct answers are both valued — the formula should not over-penalize research time.

### Future Scoring Dimensions (Long-Term)

- **Score decay:** Knowledge axes decay slowly over time if not exercised (spaced repetition principle)
- **Cross-domain attribution:** Interdisciplinary questions split scoring across relevant axes
- **Research-depth bonus:** Players who consult multiple sources or deeper sections earn additional credit (requires resource tracking)
- **Difficulty adaptation:** Question difficulty adjusts based on the player's Wheel profile

---

## Event Logging

Every gameplay interaction is logged as raw data:

```json
{
  "event_id": "evt_abc123",
  "event_type": "answer_submitted",
  "player_id": "ident_xyz",
  "question_id": "q_ghi789",
  "player_answer": "Noether's first theorem",
  "time_elapsed_seconds": 145,
  "judgment": "correct",
  "confidence": 0.88,
  "domain": ["mathematics", "physics"],
  "difficulty": 7,
  "wheel_state_before": { "mathematics": 42, "physics": 38, ... },
  "wheel_state_after": { "mathematics": 47, "physics": 41, ... },
  "resources_accessed": [],
  "timestamp": "2025-01-15T22:35:00Z"
}
```

**Other event types:** `question_served`, `gave_up`, `resource_accessed` (future), `session_started`, `session_ended`

Raw logs enable all future scoring refinement, analytics, difficulty calibration, and potential model improvement. **Log everything now, analyze later.**

---

## Architecture

hard.think consumes two independent API services:

```
┌─────────────────────────────────────────────┐
│              hard.think (Next.js)            │
│                                             │
│  Pages & UI        API Routes (Backend)     │
│  ───────────       ────────────────────     │
│  Game view         /api/game/*              │
│  Wheel viz         /api/wheel/*             │
│  Auth screens      /api/auth/* (proxy)      │
│  Community         /api/events/*            │
│  FAQ/About                                  │
│       │                    │                │
│       │         ┌──────────┴──────────┐     │
│       │         ▼                     ▼     │
│       │  Bubble Bath Auth API    Brain Fart│
│       │  (Service 1)          API (Service 2)│
│       │                                     │
└───────┴─────────────────────────────────────┘
```

**The Next.js app owns:**
- All UI/UX
- Game state management
- Scoring logic and Wheel calculations
- Event logging
- Community and social features

**The Next.js app delegates:**
- Identity/authentication → Bubble Bath Auth API
- Question generation and answer grading → Brain Fart API

---

## Pages & Routes

### Core Game
| Route | Description |
|-------|-------------|
| `/` | Landing page — concept introduction, start game |
| `/play` | Main game view — question, research area, answer input |
| `/wheel` | Full Wheel of Knowledge view — detailed domain breakdown |
| `/history` | Player's question history — past answers, scores, sources |

### Auth (Bubble Bath)
| Route | Description |
|-------|-------------|
| `/register` | Bubble Bath registration — number + color picker |
| `/login` | Bubble Bath login — reproduce number + color from memory |

### Community & Info (Long-Term)
| Route | Description |
|-------|-------------|
| `/about` | Project philosophy, credits, open-source links |
| `/faq` | How the game works, scoring explanation, source info |
| `/community` | Player discussion, shared discoveries (future) |
| `/contact` | Contact and media inquiries |

---

## API Routes (Next.js Backend)

### Game
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/game/question` | GET | Fetch a new question (proxies to Brain Fart) |
| `/api/game/answer` | POST | Submit an answer for grading (proxies to Brain Fart) |
| `/api/game/giveup` | POST | Player gives up — returns answer, source, domain |

### Wheel
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/wheel/state` | GET | Current Wheel of Knowledge state for player |
| `/api/wheel/history` | GET | Wheel state over time (for progression visualization) |
| `/api/wheel/update` | POST | Internal — recalculate Wheel after grading event |

### Events
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/events/log` | POST | Log a gameplay event |
| `/api/events/history` | GET | Retrieve player event history |

### Auth (Proxy)
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/register` | POST | Proxy to Bubble Bath Auth API |
| `/api/auth/login` | POST | Proxy to Bubble Bath Auth API |
| `/api/auth/validate` | POST | Proxy to Bubble Bath Auth API |
| `/api/auth/logout` | POST | Proxy to Bubble Bath Auth API |

---

## Data Model

### Player State
```
{
  player_id:        string (from Bubble Bath identity)
  wheel_state: {
    science:        float
    mathematics:    float
    philosophy:     float
    history:        float
    technology:     float
    arts:           float
    social_sciences: float
    geography:      float
  }
  total_questions:  integer
  total_correct:    integer
  total_gave_up:    integer
  current_streak:   integer (internal tracking only — not surfaced as gamification)
  created_at:       timestamp
  last_active:      timestamp
}
```

### Game Session
```
{
  session_id:       string
  player_id:        string
  started_at:       timestamp
  ended_at:         timestamp
  questions_served: integer
  questions_answered: integer
  questions_gave_up: integer
}
```

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js (React + API routes) |
| Language | TypeScript |
| Visualization | Recharts or D3 (Wheel of Knowledge) |
| Styling | Tailwind CSS |
| Database | Netlify DB (production) |
| Deployment | Netlify |
| Auth | Bubble Bath Auth API (Service 1) |
| Questions | Brain Fart API (Service 2) |

---

## Project Structure

```
hard-think/
├── README.md
├── package.json
├── next.config.js
├── tailwind.config.js
├── netlify.toml
├── .env.example
├── public/
│   ├── favicon.ico
│   └── assets/
│       └── ...
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout
│   │   ├── page.tsx                # Landing page
│   │   ├── play/
│   │   │   └── page.tsx            # Main game view
│   │   ├── wheel/
│   │   │   └── page.tsx            # Wheel of Knowledge full view
│   │   ├── history/
│   │   │   └── page.tsx            # Question history
│   │   ├── register/
│   │   │   └── page.tsx            # Bubble Bath registration
│   │   ├── login/
│   │   │   └── page.tsx            # Bubble Bath login
│   │   ├── about/
│   │   │   └── page.tsx            # About the project
│   │   ├── faq/
│   │   │   └── page.tsx            # FAQ
│   │   └── community/
│   │       └── page.tsx            # Community (future)
│   ├── components/
│   │   ├── game/
│   │   │   ├── QuestionDisplay.tsx
│   │   │   ├── AnswerInput.tsx
│   │   │   ├── GradeResult.tsx
│   │   │   ├── GaveUpReveal.tsx
│   │   │   └── SourceLinks.tsx
│   │   ├── wheel/
│   │   │   ├── WheelChart.tsx      # Radar chart component
│   │   │   ├── DomainBreakdown.tsx
│   │   │   └── WheelHistory.tsx
│   │   ├── auth/
│   │   │   ├── NumberInput.tsx
│   │   │   ├── ColorPicker.tsx
│   │   │   └── AuthFlow.tsx
│   │   └── layout/
│   │       ├── Header.tsx
│   │       ├── Footer.tsx
│   │       └── Navigation.tsx
│   ├── lib/
│   │   ├── api/
│   │   │   ├── bubble-bath.ts        # Bubble Bath Auth API client
│   │   │   ├── brainFart.ts   # Brain Fart API client
│   │   │   └── events.ts           # Event logging client
│   │   ├── scoring/
│   │   │   ├── wheelCalculator.ts  # Scoring formula implementation
│   │   │   └── constants.ts        # Scoring weights and thresholds
│   │   ├── game/
│   │   │   ├── gameState.ts        # Game session state management
│   │   │   └── timer.ts            # Answer timer logic
│   │   └── utils/
│   │       └── formatting.ts       # Display formatting helpers
│   ├── hooks/
│   │   ├── useGame.ts              # Game loop state hook
│   │   ├── useWheel.ts             # Wheel state hook
│   │   ├── useAuth.ts              # Auth state hook
│   │   └── useTimer.ts             # Timer hook
│   ├── types/
│   │   ├── game.ts                 # Game-related type definitions
│   │   ├── wheel.ts                # Wheel-related type definitions
│   │   ├── auth.ts                 # Auth-related type definitions
│   │   └── events.ts               # Event type definitions
│   └── styles/
│       └── globals.css
├── tests/
│   ├── scoring/
│   │   └── wheelCalculator.test.ts
│   ├── components/
│   │   └── WheelChart.test.tsx
│   └── api/
│       └── game.test.ts
└── docs/
    ├── scoring-formula.md          # Detailed scoring documentation
    ├── wheel-design.md             # Wheel visualization design decisions
    ├── game-modes.md               # Planned game modes (future)
    └── ux-philosophy.md            # Eudaimonic design guidelines
```

---

## Configuration

```env
# .env.example

# Service URLs
CHROMAKEY_API_URL=http://localhost:4000
QUESTION_ENGINE_API_URL=http://localhost:5000

# Database
DATABASE_URL=<netlify_db_connection_string>

# Session
SESSION_SECRET=<your_secret_here>

# Scoring Defaults
BASE_POINTS=100
TIME_DECAY_RATE=0.95
MAX_ATTEMPTS_BEFORE_PENALTY=3
CONFIDENCE_WEIGHT=0.15

# Feature Flags
RESOURCE_TRACKING_ENABLED=false    # Enable when restricted search is built
SCORE_DECAY_ENABLED=false          # Enable when decay logic is implemented
CROSS_DOMAIN_SCORING=false         # Enable when attribution logic is ready

# Deployment
NEXT_PUBLIC_SITE_URL=https://hard.think
```

---

## UX Design Notes

### The "Gave Up" Experience
This is one of the most important UX moments in the game. When a player gives up:
1. Reveal the correct answer clearly
2. Show the source where the answer could be found (with a direct link)
3. Show which knowledge domain(s) the question belongs to
4. Frame it as discovery, not failure: *"Here's something new to explore"*
5. The Wheel should reflect this as a neutral-to-slight-negative event — not punitive

### Ambiguous Answer Handling
When the grading confidence is low:
- Don't say "Wrong." Say: *"Your answer might be correct — here's what we were looking for: [canonical answer]"*
- Show the player's answer alongside the expected answer
- Let the Wheel update modestly (reduced confidence modifier)
- Log for review and pipeline improvement

### Tone
- Respectful, curious, warm — never condescending
- Avoid "Correct!" / "Wrong!" binary. Prefer nuanced language
- The game should feel like a conversation with a knowledgeable friend, not a quiz show host

---

## Open Questions & Future Exploration

### Restricted Search Tool
The largest engineering challenge in the entire ecosystem. Options range from:
- **Filtered iframe** — Fragile, easily broken by source site changes
- **API proxy** — Route player searches through a backend that only queries approved source APIs
- **Custom search index** — Build a search layer over cached/indexed source content

Each has major tradeoffs in reliability, maintenance, and UX. Deferred to post-prototype.

**Status:** Not yet designed. MVP uses direct source links instead.

### Resource Tracking
Tracking which sources a player consults (and how deeply) would enable research-depth scoring bonuses. Requires the restricted search tool to be in place first.

**Status:** Dependent on restricted search tool.

### Multiple Game Modes
Potential modes under consideration:
- **Standard:** Untimed, full research access
- **Timed sprint:** Quick questions, limited research time
- **Deep research:** Harder questions, bonus for thorough sourcing
- **Domain challenge:** Focus on a single knowledge domain
- **Daily question:** One high-quality question per day

**Status:** Conceptual. Prototype focuses on standard mode only.

### Difficulty Adaptation
Adjusting question difficulty based on the player's Wheel profile — serving harder questions in strong areas and easier questions in weak areas (or vice versa, depending on design stance).

**Status:** Requires sufficient play data and Wheel calibration first.

### Reflection Features
Periodic summaries of what the player has learned, growth over time, knowledge gaps explored, and suggested areas for further study. Aligned with eudaimonic philosophy — the game should help players understand themselves.

**Status:** Long-term feature. Conceptual only.

### Community Features
Player discussion, shared discoveries, collaborative learning. Needs careful design to avoid competitive dynamics that undermine the eudaimonic philosophy.

**Status:** Long-term feature.

---

## Contributing

Contributions are welcome, particularly in:
- Wheel visualization design and animation
- Scoring formula refinement and playtesting
- UX research on eudaimonic game design
- Accessibility improvements
- Game mode design

---

## License

TBD — will be open-source. License selection pending.
