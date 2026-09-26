# QBIT — Quantum Digital Signature Threat Detection Framework

**Built for Hacktivate 2.0**

A live, working simulation of a teleportation-based Quantum Digital Signature (QDS) protocol, paired with a real-time statistical engine that detects forgery, impersonation, replay, and quantum channel manipulation attacks — using pure math and statistics, with no AI or machine learning involved.

---

## The Problem

Classical digital signatures (RSA, ECC) rely on computational hardness — math problems that are hard to solve *today*. Shor's Algorithm, run on a sufficiently powerful quantum computer, breaks that assumption entirely. Quantum Digital Signatures fix the underlying cryptography using physics instead of math, but almost nothing exists to demonstrate how you'd actually *detect* an attack against one in real time.

## What We Built

QBIT simulates the full lifecycle of a teleportation-based QDS exchange between two parties — Alice (signer) and Bob (verifier) — using real quantum circuit simulation (Qiskit), then runs a deterministic statistical engine on top to catch tampering. Every decision the system makes is explainable with an exact, calculable number — no black-box AI involved anywhere in the detection logic.

### Core Capabilities

- **Quantum Teleportation Simulation** — Bell-state entanglement, joint Bell measurement, and Pauli correction, implemented and verified in Qiskit.
- **Quantum Digital Signature Scheme** — signing and verification using independently-derived quantum fingerprints per trial copy.
- **Four Attack Simulations** — Forgery, Impersonation, Replay, and Channel Manipulation, each using a distinct, appropriate detection mechanism.
- **Statistical Detection Engine** — error-rate (QBER-style) calculation, threshold-based deterministic accept/reject decisions, and forgery probability scoring.
- **Live Interactive Dashboard** — real-time Alice → Bob visualization, live error-rate telemetry graph, verdict banner, attack controls, and session run history.

---

## Tech Stack

**Backend**
- Python 3.11
- Qiskit 2.5.2 + Qiskit Aer (local quantum circuit simulation)
- FastAPI (REST API layer)

**Frontend**
- Next.js 14 + TypeScript
- Tailwind CSS
- Recharts (live telemetry graph)
- lucide-react (icons)

**Deployment**
- Backend: Render
- Frontend: Vercel

---

## Architecture

```
hacktivate-2.0/
├── shared/
│   └── api_contract.json      # Single source of truth for API request/response shape
├── backend/
│   ├── main.py                 # FastAPI endpoints
│   ├── quantum_core.py         # Qubit measurement, Bell states, teleportation circuit
│   ├── signatures.py           # QDS signing & verification logic
│   ├── attacks.py              # Forgery, impersonation, channel manipulation simulations
│   ├── detection.py            # Error rate, threshold, forgery probability engine
│   └── requirements.txt
└── frontend/
    ├── src/
    │   ├── app/
    │   │   ├── globals.css      # Global styles
    │   │   ├── layout.tsx       # Root layout
    │   │   └── page.tsx         # Main dashboard — all sections composed here
    │   └── lib/
    │       ├── api.ts           # Typed API layer
    │       └── types.ts         # TypeScript interfaces matching the API contract
    └── package.json
```

---

## How It Works

1. **Entanglement Distribution** — Alice and Bob each receive one half of a pre-shared entangled Bell-state qubit pair.
2. **Signing** — Alice derives a set of independent quantum fingerprints from her message and private key, and teleports them to Bob.
3. **Bell Measurement & Transmission** — Alice performs a joint Bell-state measurement and sends Bob two classical bits per trial.
4. **Pauli Correction** — Bob applies the corresponding correction (I, X, Z, or XZ) to reconstruct the original signed state.
5. **Statistical Verification** — the detection engine computes an error rate from the measurement outcomes and compares it against a fixed threshold. If the error rate exceeds the threshold, the signature is rejected as an attack.

Replay attacks are the one exception to this statistical flow — since a replayed signature is quantum-mechanically valid (it was legitimately signed once already), it's instead caught via a classical reused-signature check, not error-rate analysis.

---

## API

Two endpoints, both returning the same response shape (see `shared/api_contract.json` for the full spec):

**`POST /simulate/clean`** — runs a legitimate signing and verification cycle.

**`POST /simulate/attack`** — body: `{ "attack_type": "forgery" | "impersonation" | "replay" | "channel_manipulation" }`

**Response fields:**
```json
{
  "status": "clean | attack_detected",
  "attack_type": "string or null",
  "error_rate": "number, 0 to 1",
  "threshold": "number, 0 to 1",
  "verdict": "human-readable string",
  "forgery_probability": "number, 0 to 1",
  "trial_count": "integer",
  "timestamp": "ISO 8601 string",
  "latency_ms": "real measured backend computation time",
  "session_hash": "genuine SHA3-512-derived fingerprint of this result"
}
```

---

## Running It Locally

**Backend**
```bash
cd backend
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # macOS/Linux
pip install -r requirements.txt
uvicorn main:app --reload
```
Runs on `http://127.0.0.1:8000`. Interactive API docs at `http://127.0.0.1:8000/docs`.

**Frontend**
```bash
cd frontend
npm install
```
Create `.env.local`:
```
NEXT_PUBLIC_API_URL=http://127.0.0.1:8000
```
```bash
npm run dev
```
Runs on `http://localhost:3000`.

---

## Why No AI/ML

Every detection decision in this system is deterministic: the same measured error rate always produces the same accept/reject verdict, and every flagged attack comes with an exact, calculable statistic behind it — not a learned prediction from a black-box model. This makes the system fully auditable, which matters for a security-critical use case like signature verification.

---

## Security Note: A Real Vulnerability We Found and Fixed

During development, our first signature design encoded an entire signature into a single qubit's rotation angles. Stress-testing it against 200 tampered messages showed a **10% false-accept rate** — a genuine collision vulnerability, not a coding bug. We redesigned the scheme to derive an independent quantum fingerprint per trial copy instead of one shared fingerprint, and retested against 500 tampered messages: false accepts dropped to **0.2%**.

---

## What This Is (and Isn't)

This is a software simulation and proof-of-concept — not a production cryptographic service. Real-world deployment of teleportation-based QDS requires physical quantum hardware for entanglement distribution, which doesn't exist at usable civilian scale yet. What's built here is a complete, honestly-tested validation of the detection approach: the mathematical modelling, attack simulation, and statistical security analysis needed to demonstrate it works, built on the standard Qiskit SDK so it could extend to real quantum hardware as that infrastructure matures.

---

## Team

Built for **Hacktivate 2.0**.

---

## License

This project was built for hackathon purposes. Add a license of your choice if open-sourcing further.

## Recent Updates & Additions

The following functionality has been added to the existing QBIT implementation without changing the original architecture or detection approach.

## Quantum Teleportation Telemetry

QBIT now exposes detailed results from the actual Qiskit teleportation simulation instead of only returning the final QDS security verdict.

Each teleportation run provides:

Number of teleportation shots/trials
Teleportation success rate
Bell-state measurement results
Measured classical bits
Pauli correction applied by Bob
Verification result
Per-trial pass/fail status
Backend execution latency

Example teleportation response:
``` bash
{
  "protocol": "quantum_teleportation",
  "shots": 20,
  "teleportation_success_rate": 1.0,
  "bell_measurement": {
    "m1": 1,
    "m2": 0,
    "bits": "10"
  },
  "pauli_correction": {
    "gate": "Z",
    "label": "Pauli-Z",
    "operations": ["Z"]
  },
  "verification": {
    "bit": 0,
    "passed": true
  },
  "latency_ms": 78.373
}
```
## Trial-Level Teleportation Results

The dashboard now displays individual teleportation trials rather than only an aggregate result.

Each trial contains:

Trial number
Bell measurement bits
Pauli correction
Verification output
PASS / FAIL status

The dashboard initially displays the first three trials and provides a View More control when additional trials are available. The control can be used to expand the complete trial history and then collapse it using View Less.

## Teleportation Protocol Visualization

The dashboard now visualizes the complete teleportation sequence:
``` bash
01 Prepare
      ↓
02 Entangle
      ↓
03 Bell Measurement
      ↓
04 Pauli Correction
      ↓
05 Verification
```
The interface also exposes the actual Qiskit-generated telemetry for the teleportation process, including:

Bell measurement
Pauli correction
Verification
Teleportation success rate

This allows the user to inspect the underlying quantum simulation rather than seeing only the final attack verdict.

## Attack + Teleportation Integration

Teleportation data is now integrated into the attack simulation workflow.

When an attack simulation produces teleportation data, the frontend displays the corresponding teleportation measurements alongside the QDS security result.

The supported attack types remain:
```bash
Forgery
Impersonation
Replay
Channel Manipulation
```
Each attack continues to use its corresponding detection mechanism rather than a single generic detector.

## Channel Manipulation

Channel Manipulation is now fully integrated as one of the four attack options in the dashboard and API.

The API accepts:
```bash
{
  "attack_type": "channel_manipulation"
}
```
The frontend also provides a dedicated Channel Manipulation state in the QDS flow visualization.

## Clean Teleportation Simulation

A standalone teleportation simulation has also been integrated into the backend/frontend workflow.

The teleportation endpoint is:
``` bash
POST /simulate/teleportation
```
This endpoint runs the Qiskit teleportation circuit and returns the teleportation-specific telemetry, including trial results and verification information.

## Updated API Integration

The API layer now supports the teleportation simulation in addition to the existing QDS clean and attack simulations.

Current simulation routes include:
```bash
POST /simulate/clean
POST /simulate/attack
POST /simulate/teleportation
```
The attack endpoint continues to accept:
```
forgery
impersonation
replay
channel_manipulation
```
The frontend API layer has been updated to consume the teleportation response and expose the returned data to the dashboard.

Frontend Teleportation Data Model

The frontend now maintains dedicated teleportation state so that the quantum simulation results can be displayed independently from the normal attack result.

The teleportation data includes:
```bash
protocol
shots
teleportation_success_rate
bell_measurement
pauli_correction
verification
trials
latency_ms
```
The frontend also maintains the visibility state of the trial table so users can switch between the abbreviated and complete trial history.

## Updated Project Structure

The teleportation implementation is now represented by a dedicated backend module:
``` bash
backend/
├── main.py
├── teleportation.py
├── signatures.py
├── attacks.py
├── detection.py
└── requirements.txt
```
teleportation.py contains the Qiskit teleportation simulation and produces the teleportation telemetry consumed by the API and frontend.

A corresponding teleportation test module is also included:
```bash
backend/
└── test_teleportation.py
```
