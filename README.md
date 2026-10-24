# LA4 — CSRF Attack Demonstration

**Course:** Cyber Security Lab — Assignment 4  
**Author:** Aryan Srivastava (IIT2024501)

---

## Overview

This repository demonstrates a **Cross-Site Request Forgery (CSRF)** attack and its mitigation using a two-site setup:

| Folder | Role |
|---|---|
| `profile/` | Legitimate Next.js web app — a user profile page with a login system and a sensitive `change-email` API endpoint |
| `attacker-site/` | Malicious static HTML page that silently triggers the `change-email` endpoint on the victim's behalf using a hidden `<img>` tag |

### How the Attack Works

1. The victim logs into the **profile app** at `http://localhost:3000`. A session cookie is set in the browser.
2. The victim is lured into visiting the **attacker site** (e.g., `http://localhost:5500`), which presents a fake prize claim page.
3. A hidden zero-pixel `<img>` on the attacker page fires a GET request to `http://localhost:3000/api/change-email?email=hacked@attacker.com`.
4. Because the browser automatically sends the session cookie with every request, the server accepts it — and the victim's email is silently changed.

### The Fix (CSRF Mitigation)

The `change-email` route contains commented-out code that blocks cross-origin requests by checking the `Sec-Fetch-Site` and `Origin`/`Referer` headers. Uncommenting that block makes the endpoint reject any request not originating from the same site.

---

## Project Structure

```
la4-iit2024501/
├── profile/               # Victim web application (Next.js)
│   ├── app/
│   │   ├── api/
│   │   │   ├── change-email/route.ts   # Vulnerable (& fixable) endpoint
│   │   │   ├── login/route.ts          # Session-based login
│   │   │   └── logout/route.ts         # Clears session cookie
│   │   ├── login/page.tsx              # Login page
│   │   └── profile/page.tsx            # Protected profile dashboard
│   ├── lib/session.ts                  # Cookie-based session helpers
│   └── proxy.ts                        # Route protection middleware
│
└── attacker-site/
    └── index.html         # Malicious page with hidden CSRF payload
```

---

## How to Run

### Prerequisites

- [Node.js](https://nodejs.org/) v18 or later
- npm (comes with Node.js)
- Any static file server for the attacker site (e.g., VS Code **Live Server** extension, or `npx serve`)

---

### 1. Start the Profile App (Victim Site)

```bash
cd profile
npm install
npm run dev
```

The app will be available at **`http://localhost:3000`**.

**Login credentials:**

| Field | Value |
|---|---|
| Username | `aryan` |
| Password | `aryan123` |

---

### 2. Serve the Attacker Site

Open `attacker-site/index.html` using any static server. The easiest options:

**Option A — VS Code Live Server:**  
Right-click `attacker-site/index.html` → *Open with Live Server* (default port: `5500`).

**Option B — npx serve:**
```bash
npx serve attacker-site
```

The attacker page will be at **`http://localhost:5500`** (or whichever port is shown).

---

### 3. Reproduce the Attack

1. Open `http://localhost:3000/login` and log in.
2. **Without logging out**, navigate to the attacker site in the same browser.
3. Observe that the hidden `<img>` fires a request to the profile app — the email is changed silently.
4. Go back to `http://localhost:3000/profile` to confirm the email has been updated to `hacked@attacker.com`.

### 4. Test the Fix

In [`profile/app/api/change-email/route.ts`](profile/app/api/change-email/route.ts), uncomment the CSRF-check block (lines 11–23), restart the dev server, and repeat step 3. The request will now be blocked with **HTTP 403**.
