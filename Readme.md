# CryptoPulse Live Terminal

<img width="1469" height="884" alt="image" src="https://github.com/user-attachments/assets/89f226f4-9564-4fe1-b48f-c4d8a88b5d84" />

An ultra-lightweight, real-time cryptocurrency dashboard designed for traders who need sub-second updates on market prices without loading down their machines. Built with React, TypeScript, and Vite, it delivers instant market tracking with a footprint of less than 10MB of RAM.

This project is a submission for DEV's Summer Bug Smash Challenge: Clear the Lineup powered by Sentry.

## The Production-Stopping Bug
To maintain a zero-server footprint, the client polls free public crypto pricing APIs (like CoinGecko) directly. However, during high-volatility events, these third-party APIs can return rate-limit errors (HTTP 429 Too Many Requests) or empty/malformed structures.

Because TypeScript types are checked at compile-time rather than runtime, our state was blindly set with an unexpected payload. On re-render, the React component attempted to read nested values on an undefined object:


`const price = marketData.bitcoin.usd; // 💥 TypeError: Cannot read properties of undefined (reading 'usd')`

This unhandled error bubbled up, bypassed our React lifecycle, and rendered a blank white screen for the trader.

## The Resilient Defensive Architecture
I resolved this by applying a multi-layered defensive React model that completely isolates client-side rendering from unstable external APIs:

1. **HTTP Status Verification (res.ok):** We actively check the HTTP status before attempting to parse the JSON payload. If the external API rate-limits us (HTTP 429) or encounters an internal failure (HTTP 500), we intercept the cycle immediately and throw a descriptive error.

2. **Schema and Node Validation:** We ensure that the essential object branches (bitcoin, ethereum) exist before modifying the React state. This guarantees that any unexpected schema drift is gracefully caught as an error rather than a runtime crash.

3. **Optional Chaining and Fallback UI:** Even if an edge-case slips past our initial boundaries, cryptoData?.bitcoin?.usd guarantees the component evaluates to undefined and renders "N/A" safely. A clean, visual error-boundary screen allows the user to click "Retry Connection" to recover gracefully.

<img width="512" height="512" alt="defensive_react_shield" src="https://github.com/user-attachments/assets/e5e7f991-e12b-4f89-8798-a323f6656d18" />

## Getting Started & Installation
Follow these instructions to run the project locally and inspect the codebase.

### Prerequisites

- Node.js (v18.0.0 or higher)

- npm (v9.0.0 or higher) or yarn / pnpm

### Step-by-Step Installation

1. Clone the repository:
```
git clone https://github.com/YOUR_GITHUB_USERNAME/cryptopulse-terminal.git
cd cryptopulse-terminal
```
2. Install project dependencies:
```
npm install
```

Verify Sentry SDK installation: Ensure the core React Sentry SDK is present:
```
npm install @sentry/react --save
```

## Sentry Setup & Configuration
To enable premium error monitoring, session replays, and performance tracing, configure Sentry in your local environment.

1. SDK Initialization (src/main.tsx)
Initialize Sentry in your React application entrypoint. Open src/main.tsx and configure Sentry as follows:

```
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'
import * as Sentry from "@sentry/react";

Sentry.init({
  // Replace this with your actual Sentry project DSN:
  dsn: "https://7704bae58ce801c0ab5146f4af728b7c@o4511455073206272.ingest.us.sentry.io/4511902207442944",

  integrations: [
    // 🎥 Session Replay integration
    Sentry.replayIntegration({
      maskAllText: false,
      blockAllMedia: false,
    }),
    // 🌐 Performance & Browser Tracing
    Sentry.browserTracingIntegration(),
  ],

  // 📈 Tracing & Replay Sample Rates
  tracesSampleRate: 1.0,           // Capture 100% of performance transactions
  replaysSessionSampleRate: 1.0,   // Record 100% of standard user sessions
  replaysOnErrorSampleRate: 1.0,   // Record 100% of sessions with unhandled errors
});

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

2. Sentry AI Developer Assistant Integration (Claude Code)
We instrumented our developer environment using Sentry's AI tracking CLI tool:
```bash
npx @sentry/ai install "Please enable Sentry tracing in my app."
```
<img width="1470" height="956" alt="Screenshot 2026-08-13 at 4 20 28 PM" src="https://github.com/user-attachments/assets/8e7c4bb9-ffed-4e83-ae25-e4c09cff0bc3" />

Once installed, restart your Claude Code terminal agent (claude) to let Sentry monitor commands, LLM token usages, and file modifications in real-time.

<img width="1470" height="885" alt="image" src="https://github.com/user-attachments/assets/0e77054a-1e6d-4518-93f1-9d69bd366f0c" />


## Local Testing Instructions
Ensure Sentry is receiving errors and transaction telemetry from your local workspace:

1. Launch the Development Server
```
npm run dev
```
Open your browser to http://localhost:5173.

<img width="1470" height="885" alt="image" src="https://github.com/user-attachments/assets/3758fdb5-0c8b-4fc6-aee1-dc1ca37cb6a7" />

2. Disable Ad-Blockers & Privacy Shields (Crucial)
Sentry outbound network packets (envelopes) can be classified as tracking. To guarantee Sentry telemetry reaches your dashboard:

- Turn off uBlock Origin, AdBlock, or similar extensions for localhost:5173.
- If using Brave Browser, lower your shield (toggle the Orange Lion icon off).

3. Verify Error Tracking (Simulate Crash)
To test if Sentry is connected properly:

- Temporary testing code with a throw action (throw new Error("This is your first error!")) can be run via click events.
- Check Sentry's Issues dashboard to verify the unhandled trace has successfully created issue CRYPTO-BUGSMASH-1.

4. Verify Performance Tracing & Outgoing Requests

- Open Chrome DevTools (Cmd + Option + I or Ctrl + Shift + I) and check the Network tab.

- Click around the CryptoPulse application to trigger CoinGecko API calls.

- Inspect the outgoing HTTP requests inside Sentry's Explore ➡️ Traces dashboard to watch transaction spans representing page loads and API lookups.
Built with passion, debugged with Sentry, and engineered for resilience.
