---
name: integration-nextjs-app-router
description: Add or review PostHog analytics in a Next.js App Router project. Use for browser or server events, identity, and verification of an instrumented user journey.
---

# Integrate PostHog with Next.js App Router

Inspect the installed Next.js version, package manager, App Router layout, existing analytics wrapper, authentication lifecycle, configuration and consent flow. Start from the product question. Choose a small set of events that can answer it and define when each event counts as having happened.

1. **Plan capture.** List event names, their source, allowed properties and the boundary that confirms success. Reuse existing event names and conventions. When a client initiates work that may fail on the server, decide explicitly whether the event measures intent or completion.
2. **Wire the SDK.** Use the initialization mechanism supported by the project's framework and SDK versions. Initialize once, using the project's environment configuration and regional ingestion host. Keep server credentials on the server. Prefer an existing wrapper when it already handles consent and initialization. Consult the current [PostHog Next.js guide](https://posthog.com/docs/libraries/next-js).
3. **Follow identity.** For an authenticated journey, use a stable application user ID, associate events after sign-in and clear the browser identity on sign-out. Use consistent identity across authorized browser and server events. When correlating sessions, limit propagation to the intended backend. Treat analytics identifiers as telemetry only.
4. **Capture at the source.** Wire an interaction to its handler and a business outcome to its successful transition. Avoid duplicate capture across rerenders, retries or client and server. Keep raw form text, secrets and unrelated personal data out of properties. Where error tracking is requested, capture the useful failure once at a boundary that has context.
5. **Verify a journey.** Use an approved test destination or a capture stub. Exercise success, failure, sign-out and the applicable consent states. Inspect event order, property values and identity. Distinguish local payload verification from confirmed ingestion.

Finish with an event inventory, configuration names, checks performed and any remaining dashboard or deployment steps. For SDK-specific details and current examples, refer to PostHog's maintained documentation.
