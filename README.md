# SATARK.
**Verify before you transfer.**

Live demo: https://satark-neural-nexus.dheermehta48.chatgpt.site

A working investor-protection prototype for Team Neural Nexus at SANGYAN, IIT (BHU). Built for Ramesh, 62, in Gorakhpur, and first-time investors who receive financial claims through messaging groups. Independent hackathon prototype; no regulator endorsement.

## Problem and solution
Financial messages can combine promises, pressure and payment requests. SATARK lets a user paste or speak a message, inspect the exact warning signals, understand uncertainty, and listen to a plain-language explanation before acting.

## Implemented
- Responsive React/TypeScript interface with English/Hindi UI, three demo scenarios, message checking, detailed evidence and transparent scores.
- Deterministic bilingual rules shared by a TypeScript runtime and Python FastAPI. No API keys required.
- Browser speech recognition and speech synthesis, with visible unavailable/permission-error states. Hindi voice availability depends on device installation.
- Image preview with clearly labeled manual transcription fallback on the hosted site. Local Python backend provides actual Tesseract OCR and deletes temporary images. Review OCR text before checking.
- Basic-phone simulation that reuses the risk engine through `/api/telephony/simulate`. No real telephone number or call provider.
- PWA manifest, SVG icon and minimal service worker. No offline cache of authenticated pages or user messages. Full offline reload and universal installation support are not claimed.
- Backend Dockerfile, reproducible dependency lockfile, unit/API tests and CI workflow.

## Architecture
`PWA / simulated call → typed text or browser ASR → risk rules → template explainer → browser TTS`

The hosted site runs React with a Worker-compatible TypeScript API. The separately runnable FastAPI service is supplied for Python deployment and local OCR. Set `SATARK_BACKEND_URL` to route server requests to it. Both engines consume `shared/rules.json`. This host does not run Python.

## Risk engine
Rules match contextual message spans in English and Hindi, once per signal. Weights: guaranteed return 25, short-term percentage claim 17, urgency 15, scarcity 5, personal payment 25, upfront/crypto payment 15, secret information 25, registration claim 15, credential request 50. Total capped at 100. The primary demo totals 87.

0–24 no major red flags, 25–49 caution, 50–74 high caution, 75–100 high risk. A registration claim with score below 25 displays NEEDS VERIFICATION. All current registry results are UNABLE TO VERIFY. Status contracts also allow VERIFIED and NOT VERIFIED for a future evidence-backed connector; these are never fabricated today.

These weights are heuristic, not calibrated fraud probabilities. A time-bound percentage claim is a scrutiny signal, not a prediction. Simple negation handling reduces some false positives but is not semantic understanding. The engine can miss scams, unsupported wording, image manipulation, links, fake identities and unsupported languages. Low score does not certify safety.

## AI and voice layers
The MVP uses deterministic extraction (sentence splitting), rules and bilingual templates. It does not use an LLM or trained XGBoost model. Future optional explainers must validate a structured schema and cannot change evidence or registry findings. Browser ASR/TTS can be replaced by Bhashini or Whisper adapters. No such integration is currently live.

## Privacy and guardrails
No database, analytics, permanent message history, SMS access or credential collection. Do not paste real OTPs, PINs, passwords or financial account details. The app handles messages in memory; screenshots remain in the page unless local OCR is configured. Backend OCR uses temporary files removed after processing. Browser speech services may process audio remotely. Hosting providers may retain ordinary connection metadata. No stock tips, recommendations, price predictions, paid upsells or broker promotion.

## Setup
Node 24+, pnpm, Python 3.12+. Use the package manager version declared in package.json.

```sh
pnpm install --frozen-lockfile
python -m venv .venv
. .venv/bin/activate
pip install -r backend/requirements.txt
python -m uvicorn backend.main:app --port 8000 --no-access-log
```
In another terminal:
```sh
pnpm dev
```
Open the local address printed by the server. The frontend works without Python via its own API. For local OCR, install Tesseract and English/Hindi language data, then copy `.env.example` to `.env` before starting the frontend. In a managed Sites environment, use its supervised preview command instead of starting a second frontend server.

## Environment variables
`SATARK_BACKEND_URL` is optional, server-only, pointing to the Python backend. `ALLOWED_ORIGINS` configures FastAPI browser CORS, default `http://localhost:3000`. No secrets or API keys are required. Do not expose a private backend URL in frontend code.

## Tests
```sh
node tests/risk.mjs
python -m unittest discover -s tests -v
pnpm exec tsc --noEmit
```
Covers hero score, registration uncertainty, education, Hindi, credentials, urgency, payment, negation, capping, empty/malformed input and API validation. See `docs/validation.md` for actual run evidence and manual QA limits.

## Deployment
Hosted React/TypeScript build is published through Sites. For the backend:
```sh
docker build -f backend/Dockerfile -t satark-api .
docker run --rm -p 8000:8000 satark-api
```
Docker includes Tesseract English/Hindi. Serve behind HTTPS with ingress request-size limits and rate limits before public production use. Docker image build is not verified in this workspace.

## Demo and pitch
Open Demo center → first scenario → Analyze → inspect 87/100 and each reason → switch to Hindi → Listen (requires an installed Hindi voice) → educational example → basic-phone simulation. Supporting scripts are in `docs/`.

## Roadmap
Evaluate rules against labeled bilingual data; improve negation and transliteration; audited official-registry connector; production voice/IVR with consent and signed provider webhooks; reliable browser-local OCR assets; additional verified language translations; accessibility and senior-user studies; production abuse controls. No accuracy percentage or real-world loss-prevention result is claimed.

## Team Neural Nexus
Aryan Tiwari — Machine Learning · Dheer Mehta — Full Stack · Devansh Upadhyay — DevOps
