# 🏗️ ForgeAI — Cursor for CAD

**Describe it. Design it. Manufacture it.**

ForgeAI is an AI-native CAD workspace where engineers design physical products through natural language conversation. Stop clicking through menus for hours. Describe what you want in 10 seconds, get a parametric 3D model instantly, iterate by talking to it.

> "Create a cylindrical enclosure, 120mm diameter, 2mm wall thickness, with four M3 screw bosses on the base and a snap-fit lid."
>
> Done. Editable. Parametric. Exportable. Manufacturable.

---

## The Problem

Engineers know exactly what they want. They spend 5% of their time thinking and 95% fighting CAD software — clicking through nested menus, setting constraints manually, extruding, filleting, assembling. SolidWorks crashes. Fusion 360 keeps changing pricing. CATIA costs $20K/seat. All were designed in the 90s.

**We're flipping the ratio.** Think more. Click less. Ship faster.

---

## How It Works

```
Engineer speaks/types → LLM interprets intent → Parametric code generated → 3D model renders → Engineer iterates
```

### Core Loop

1. **Describe** — Type or speak what you want in plain English
2. **Generate** — AI produces parametric CAD geometry (CadQuery/OpenSCAD under the hood)
3. **Iterate** — "Make the walls thinner." "Add ventilation holes." "Fillet all edges 2mm."
4. **Validate** — Manufacturability checks, wall thickness analysis, draft angle verification
5. **Export** — STEP, STL, OBJ — ready for 3D printing, CNC, injection molding

### Interaction Examples

```
> "Design a phone case for iPhone 15 Pro with a 1.5mm lip around the screen"
> "Add a cutout for the camera module with 0.5mm tolerance"  
> "Make it 2mm thick on the back, add a ribbed texture"
> "Is this injection moldable?"
> "Export as STEP"
```

```
> "Create an L-bracket, 50x50x30mm, 3mm steel, with two M5 through-holes on each face"
> "Add a gusset for rigidity"
> "What's the estimated weight in aluminum?"
```

```
> "Design a gear with 24 teeth, module 1.5, 10mm face width, 8mm bore with keyway"
> "Now make a mating gear with 36 teeth"
> "Show them meshed together"
```

---

## Tech Stack

### Core
- **Frontend:** React + Three.js (3D viewport rendering)
- **3D Engine:** CadQuery (Python parametric CAD kernel) via API
- **LLM:** Claude / GPT-4 for natural language → CadQuery code generation
- **Backend:** FastAPI (Python)
- **Real-time:** WebSocket for live model updates

### Why CadQuery
CadQuery is a Python-based parametric CAD library built on the Open CASCADE (OCCT) kernel — the same kernel that powers FreeCAD and parts of CATIA. It generates real B-Rep (boundary representation) geometry, not mesh approximations. This means:
- True parametric models (editable history tree)
- Boolean operations (union, subtract, intersect)
- Fillets, chamfers, shells, lofts, sweeps
- STEP/STL/OBJ export
- Manufacturable output

The LLM generates CadQuery Python code. We execute it server-side and stream the resulting mesh to the browser for rendering.

### Architecture

```
┌─────────────────────────────────────────────┐
│                   Frontend                   │
│         React + Three.js Viewport            │
│         Chat Interface + Model View          │
├─────────────────────────────────────────────┤
│                  WebSocket                   │
├─────────────────────────────────────────────┤
│                   Backend                    │
│               FastAPI Server                 │
│                                             │
│  ┌───────────┐  ┌───────────┐  ┌─────────┐ │
│  │ LLM Agent │→ │ CadQuery  │→ │ Mesher  │ │
│  │ (Claude)  │  │ Executor  │  │ (STL)   │ │
│  └───────────┘  └───────────┘  └─────────┘ │
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │  Validation Engine                    │  │
│  │  - Wall thickness check               │  │
│  │  - Draft angle analysis               │  │
│  │  - Overhang detection                 │  │
│  │  - Estimated cost/weight              │  │
│  └───────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│              Export Pipeline                  │
│          STEP / STL / OBJ / 3MF              │
└─────────────────────────────────────────────┘
```

---

## MVP Scope (Hackathon Build)

### ✅ In Scope
- Chat-based interface with 3D viewport
- Natural language → CadQuery code → 3D model pipeline
- Basic primitives: boxes, cylinders, spheres, extrusions, revolutions
- Boolean operations: union, subtract, intersect
- Modifications: fillets, chamfers, shells
- Iterative editing via conversation ("make it taller", "add holes")
- Model history (undo/redo through conversation)
- STL and STEP export
- Basic manufacturability feedback (wall thickness, overhangs)

### ❌ Out of Scope (Post-Hackathon)
- Full assembly modeling (multiple parts with constraints)
- Drawing/blueprint generation
- Simulation (FEA, CFD)
- Supplier integration / cost estimation
- Team collaboration
- Version control for models
- Import existing CAD files
- GD&T (geometric dimensioning and tolerancing)

---

## Project Structure

```
forge-ai/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Viewport.jsx          # Three.js 3D viewport
│   │   │   ├── Chat.jsx              # Chat interface
│   │   │   ├── ModelHistory.jsx       # Conversation/edit history
│   │   │   ├── PropertyPanel.jsx      # Model dimensions & properties
│   │   │   └── ExportPanel.jsx        # Export options
│   │   ├── hooks/
│   │   │   ├── useWebSocket.js        # Real-time model updates
│   │   │   └── useModelState.js       # 3D model state management
│   │   └── App.jsx
│   └── package.json
│
├── backend/
│   ├── app/
│   │   ├── main.py                    # FastAPI entry point
│   │   ├── agent/
│   │   │   ├── cad_agent.py           # LLM agent — NL to CadQuery
│   │   │   ├── prompts.py             # System prompts & few-shot examples
│   │   │   └── validator.py           # Code safety checks before execution
│   │   ├── engine/
│   │   │   ├── executor.py            # CadQuery code execution (sandboxed)
│   │   │   ├── mesher.py              # Convert B-Rep → mesh for rendering
│   │   │   └── exporter.py            # STEP/STL/OBJ export
│   │   ├── analysis/
│   │   │   ├── manufacturability.py   # Wall thickness, draft, overhang checks
│   │   │   └── properties.py          # Volume, weight, bounding box
│   │   └── ws/
│   │       └── handler.py             # WebSocket connection handler
│   ├── requirements.txt
│   └── Dockerfile
│
├── prompts/
│   ├── system_prompt.md               # Core system prompt for CAD agent
│   └── examples/                      # Few-shot CadQuery examples
│       ├── enclosure.py
│       ├── bracket.py
│       ├── gear.py
│       └── snap_fit.py
│
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## Setup & Installation

### Prerequisites
- Python 3.10+
- Node.js 18+
- Docker (recommended for CadQuery dependencies)

### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# CadQuery has heavy dependencies (OCCT kernel)
# Docker recommended:
docker-compose up backend
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Environment Variables

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...      # or OPENAI_API_KEY
MODEL=claude-sonnet-4-20250514
