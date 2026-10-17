# Lanewatch

**A freight-fraud checker for brokers that gets smarter with every confirmed call.**

## The problem

Double brokering and carrier identity theft cost the US freight industry hundreds of millions of dollars a year. Scammers buy fresh MC numbers, use Gmail and VoIP lines, underbid the market, and push brokers onto WhatsApp or ask to "update payment details." Today's vetting tools are static rule lists. They miss subtle scams, and once a scammer is caught, nothing stops them from calling the next broker with a new MC and the same phone.

## How it learns

1. **Score.** Each call is turned into 8 signals: new MC (<90 days), free email domain, VoIP phone, bid 15%+ below market, payment/factoring change request, urgency words, moving the call to text/WhatsApp, and an identifier already flagged by the network. A logistic model turns them into a fraud probability.
2. **Explain.** The verdict shows the %, "Hold the load" or "Safe to book", and the reasons ranked by weight.
3. **Confirm.** When the broker confirms fraud or legit, Lanewatch runs one online gradient step on the log-loss (`w += lr · (label − p) · x`), so signals that showed up in a missed scam gain weight and signals on a false alarm lose weight.
4. **Remember.** Confirmed fraud adds the phone (digits only), email, and MC to the network blocklist. The next call that reuses any of them lights up the strongest signal in the model.
5. **Show it.** The Network memory panel shows calls learned, flagged identifiers, accuracy over the last 20 calls, and live model weights. Every update is logged in plain words.

## The $1B path

1. **Per-call pricing.** Brokers pay per checked call. One prevented theft ($50k+ load) pays for years of checks.
2. **Network-effect moat.** Every confirmed call from every broker trains the shared model and grows the blocklist. A scammer caught by one broker is blocked for all of them. More brokers mean better detection, and better detection brings in more brokers.
3. **Carrier-trust API.** The same score, sold to shippers, factoring companies, and cargo insurers who need to know whether a carrier is real before they pay, fund, or underwrite.

## Demo steps

1. Click **1 · Legit carrier** → about **5%**, Safe to book.
2. Click **2 · Subtle scam** (Gmail, VoIP, WhatsApp) → about **38%**, Safe to book. The model misses it.
3. Click **Confirm fraud** → weights for free email, VoIP and WhatsApp go up, and the phone, email and MC go on the network blocklist. The log explains each change.
4. Click **3 · Repeat scammer** (same phone, new MC, clean company email) → about **89%**, Hold the load. Top reason: phone flagged by the network.
5. Optional: Click **Reset** in Network memory to start over.

## Tech

- One static `index.html`: vanilla JS, no backend, no API keys, no build step.
- Logistic regression with online SGD, written in about 40 lines.
- State (weights, blocklist, history, log) is kept in `localStorage` and wrapped in try/catch.
- Responsive layout, light and dark mode, visible keyboard focus.
- Deployed on Vercel.

Run locally: open `index.html` in a browser, or `python -m http.server`.
