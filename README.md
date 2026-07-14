# MAPK(Simulator) - Pancreatic Cancer Drug Response Explorer
> Why do cancer cells with the same mutation respond differently to the same drug?
> This explores one answer: stochastic noise at the molecular level.


🔗 **[Live Demo](#)** · 📄 **[Preprint (coming soon)](#)** · 🧪 **[Notebooks](./notebooks/)**

![UI Screenshot](./data/figures/screenshot.png)

---

## Overview

MAPK Simulator is an interactive web tool that simulates how pancreatic cancer cells carrying the **KRAS G12D mutaion** respond to **trametinib**, a **MEK inhibitor** used in clinical treatment.

The core question this tool addresses: even when cells share the same oncogenic mutation, individual cells within a population respond differently to the 

Two computational models run side by side:

- **ODE model** (deterministic): treats the cell population as a homogeneous average. Fast and analytically tractable, but blind to cell-to-cell differences.
- **Gillespie SSA model** (stochastic): simulates each cell individually, capturing the natural randomness of molecular reactions when molecule counts are low.

By comparing the two, the tool shows *when* and *how much* the deterministic average diverges from what individual cells actually experience.

---

## Features

- Adjust drug concentration, initial molecule counts, and KRAS G12D expression level via sliders
- Run ODE and Gillespie simulations simultaneously and compare outputs in real time
- Switch between wild-type and G12D conditions to see how the mutation rewires MAPK signaling
- View population-level drug response distribution across 100 simulated cells
- See what percentage of cells "respond" to the drug at each concentration

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python) |
| Simulation | scipy, numpy |
| Frontend | React |
| Visualization | Recharts |
| Deployment | Vercel (frontend) + Railway (backend) |

---

## Background

### KRAS G12D & Pancreatic Cancer
KRAS is mutated in over 90% of pancreatic ductal adenocarcinomas (PDAC). The G12D substitution impairs GTP hydrolysis, locking RAS in a constitutively active state and driving sustained downstream MAPK signaling — even without external growth signals.

### MAPK Signaling Pathway
The RAS–RAF–MEK–ERK cascade amplifies extracellular signals into cell fate decisions (proliferation, differentiation, survival). In G12D cells, this cascade is constitutively active, making MEK an attractive drug target.

### ODE vs. Gillespie
Ordinary differential equations (ODEs) model signaling as continuous concentration changes — accurate for large molecule numbers, but unable to capture variability between individual cells. The Gillespie Stochastic Simulation Algorithm (SSA) treats reactions as discrete probabilistic events, naturally reproducing cell-to-cell heterogeneity. This divergence becomes especially significant at low molecule counts, a condition relevant in early-stage tumors or single-cell contexts.

---

## How to Run Locally

### Prerequisites
- Python 3.10+
- Node.js 18+

### Backend
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
# API available at http://localhost:8000
# Docs at http://localhost:8000/docs
```

### Frontend
```bash
cd frontend
npm install
npm start
# App available at http://localhost:3000
```

---

## Repository Structure

```
mapk-simulator/
├── backend/              # FastAPI simulation server
│   ├── main.py           # API endpoints
│   ├── ode_model.py      # ODE implementation
│   ├── gillespie.py      # Gillespie SSA implementation
│   └── requirements.txt
├── frontend/             # React web app
│   ├── src/
│   │   ├── components/   # Sliders, charts, comparison view
│   │   └── App.jsx
│   └── package.json
├── notebooks/            # Jupyter notebooks (model development)
│   ├── 01_ode_model.ipynb
│   ├── 02_gillespie.ipynb
│   ├── 03_drug_model.ipynb
│   └── 04_validation.ipynb
├── data/
│   ├── parameters_literature.md   # Kinetic parameters with citations
│   ├── trametinib_parameters.md   # Drug parameters with citations
│   └── figures/                   # Output figures
└── README.md
```

---

## Limitations & Future Work

**Current limitations**
- Model covers the core MAPK cascade only (RAS–RAF–MEK–ERK); PI3K/AKT bypass signaling is not included
- Kinetic parameters are drawn from literature and carry uncertainty; the model is not calibrated to a specific patient dataset
- Drug resistance mechanisms beyond stochastic noise (e.g., feedback reactivation, bypass pathway upregulation) are out of scope

**Future directions**
- Incorporate PI3K/AKT crosstalk to model adaptive resistance
- Add combination therapy simulation (MEK inhibitor + AKT inhibitor)
- Extend to other KRAS variants (G12C, G12V) for comparative analysis

---

## Citation

If you use this tool or reference this work, please cite:

```
[Preprint citation — coming soon]
```

---

## References

> 📌 Full references will be added after literature review (Issue #2 & #7).  
> Parameter sources are documented in [`data/parameters_literature.md`](./data/parameters_literature.md) and [`data/trametinib_parameters.md`](./data/trametinib_parameters.md).

Key papers this work builds on:

- <!-- MAPK pathway kinetics: Author et al., Year, Journal, DOI -->
- <!-- KRAS G12D pancreatic cancer: Author et al., Year, Journal, DOI -->
- <!-- Trametinib PANC-1 IC50: Author et al., Year, Journal, DOI -->
- <!-- Gillespie SSA original: Gillespie, D.T. (1977). J. Phys. Chem. -->

---

## License

MIT License — see [LICENSE](./LICENSE) for details.