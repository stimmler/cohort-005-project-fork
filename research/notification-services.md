# Which third-party notification services fit Cadence's constraints?

Research for [#4](https://github.com/stimmler/cohort-005-project-fork/issues/4), part of map [#2](https://github.com/stimmler/cohort-005-project-fork/issues/2) (in-app Notifications).
All sources are vendor-owned (docs, pricing, legal pages), accessed **2026-11-25** unless noted. This file reports facts and disqualifiers only. It does not recommend a service; that decision is made separately.

## Constraints checked

1. EU data residency **or** an EU-compliant DPA (SCCs or equivalent).
2. Free or low cost at Cadence's current (small) scale.
3. Recipients are identified by an opaque id only: no names or emails sent.
4. A React in-app feed/inbox component, or a headless API.
5. A Node server SDK.
6. Workable in local development, where the DevUI panel switches between seed people (each seed person must map to a recipient id, and the client must be able to authenticate as whoever is selected).

## Comparison

| | Knock | Novu Cloud | Novu self-hosted | MagicBell | SuprSend | Courier |
|---|---|---|---|---|---|---|
| EU data residency | **No** today; "expect to support … by the end of 2026" | **Yes**, Frankfurt, all plans incl. Free | Wherever you host it | **Not documented** | US or EU workspace per DPA; pricing lists custom region as Enterprise-only (unclear) | **Yes**, Ireland, **paid plans only** (Business/Enterprise) |
| DPA / transfer mechanism | DPA with SCCs (Modules 2/3) + UK Addendum | DPA + SCC template at novu.co/dpa | N/A (no processor; DPA listed as cloud-only) | "We provide data processing agreements"; terms, hosting and transfer mechanism not published | DPA: EU-US DPF, fallback SCCs; support staff access from India | Not checked beyond residency |
| Free tier | 10,000 messages/mo | 10,000 workflow runs/mo, 2 environments, 24 h activity-feed retention | Free (MIT core) but you run MongoDB, Redis, S3, API/worker/WS/dashboard | 1,000 deliveries/mo | 10,000 notifications/mo | 10,000 sends/mo (US only) |
| First paid step | Starter $250/mo | Pro from $30/mo | Infra cost; Enterprise self-host unpriced | Startup $249/mo | Essentials $100/mo | $0.005/send for **all** sends, incl. the first 10k |
| Opaque id only | Yes, `id` is the only required user attribute | Yes, `subscriberId` | Yes | Yes, `user_external_id` (email optional) | Yes, `distinct_id` | Yes, `user_id` |
| React in-app UI | `@knocklabs/react` (feed popover, bell, hooks), real-time | `@novu/react` `<Inbox>`, real-time over WebSocket | Same `@novu/react` | `@magicbell/react` (`Inbox`, `FloatingInbox`) | `@suprsend/react` + headless feed | `@trycourier/courier-react` (`CourierInbox`), real-time |
| Node SDK | `@knocklabs/node` | `@novu/api` | `@novu/api` | `magicbell-js` (browser/Node) | `@suprsend/node-sdk` | Courier Node SDK (`baseURL` for EU) |
| Client auth | Public key + optional RS256 user token ("enhanced security", off by default) | App id + optional HMAC `subscriberHash` | Same | HS256 user JWT signed with project secret (required) | Public key + optional ES256 user JWT ("enhanced security") | Backend-minted JWT via `/auth/issue-token` (required) |
| Local dev with DevUI switching | Fine: dev env with enhanced security off, or sign a token per selected person | Fine: dev environment; HMAC hash per selected person | Runs locally but heavy infra | Fine: sign a JWT per selected person | Fine, same as Knock | Fine: mint a JWT per selected person; EU tokens only work against EU hosts |

## Disqualifiers and caveats

- **MagicBell: fails, or cannot be verified against, the residency/DPA constraint.** Its pricing, docs and Trust Centre say nothing about hosting region or EU residency. The privacy policy says data "is processed at the Owner's operating offices" (San Francisco, US) and names no transfer mechanism. A DPA is "provided" but not published. It also has the smallest free tier (1,000 deliveries/mo). It would only stay in the running if MagicBell confirmed EU hosting or SCCs in writing.
- **Knock: no EU data residency today.** It can only satisfy the "EU-compliant DPA" half of the constraint (SCCs + UK Addendum). The DPA's wording ("by signing this DPA") leaves it unclear whether the DPA applies to Free accounts automatically. Knock's subprocessor page already says "EU processing locations apply only to EU-hosted accounts", which conflicts with the security docs' "expect to support EU data residency by the end of 2026". Ask Knock which is current.
- **Courier: EU residency is not free.** The EU datacenter is Business/Enterprise only, and Business charges $0.005 for every send including the first 10,000 (e.g. 2,000 sends/mo = $10). That is low cost, but not free. The free Developer plan is US-only.
- **SuprSend: residency terms conflict.** The DPA says data is stored in "the hosting region of your workspace, the United States or the European Union". The pricing page lists "Custom Data Center Region" as Enterprise-only, and the security page gives no plan details. Confirm with SuprSend whether a Free/Essentials workspace can be created in the EU. Without EU hosting, the DPA (DPF + SCCs) still meets the DPA half of the constraint. Note that support staff may access data from India.
- **Novu self-hosted: no DPA needed, but operational cost.** Novu's comparison page lists DPAs, RBAC, SSO and >2 environments as cloud-only, and limits self-hosted to 1 team member. Novu's minimum single-VM spec is 4 vCPU / 8 GB plus MongoDB, Redis (AOF) and S3-compatible storage. The core is MIT; the `enterprise` folder is under a commercial licence.
- **Novu Cloud Free retention:** the 24-hour figure on the pricing page is for the *activity feed* (logs), not Inbox messages. The page does not clearly state the Free plan's "Inbox feed retention" value. Check it before relying on Novu Free for stored Notifications.
- **Security in production (all services):** every option either requires or recommends server-signed per-person credentials. Knock's docs warn that without enhanced security "a request with your public key can act as any user", and Novu says another person's `subscriberId` could be guessed without HMAC. In the DevUI, Cadence's server signs for whichever seed person is selected, so the constraint holds as long as that is done server-side.

### Alternatives considered

- **Pingram (formerly NotificationAPI):** has US/EU/CA regions on every plan, and NotificationAPI shipped a React in-app SDK. However, Pingram's current pricing page only covers email/SMS/voice and does not mention in-app. Not pursued further. [pingram.io/pricing](https://www.pingram.io/pricing), [NotificationAPI EU data center post (Jan 2025)](https://notificationapi.com/blog/introducing-notificationapi-s-european-data-center)
- **OneSignal:** moved its data centres to the EU in 2025 (per a Courier blog post, a secondary source). Not investigated further because it is push/in-app-message oriented and not a notification feed/inbox. Verify if it is reconsidered.

## Per-candidate notes

### Knock
- Pricing: Free $0, 10,000 messages/mo, unlimited profiles/workflows, 30-day log retention, Knock branding. No hard cap: Knock contacts you to add billing. Starter $250/mo with 50,000 messages, then $0.005/message. https://knock.app/pricing
- Residency: "We expect to support EU data residency by the end of 2026." https://docs.knock.app/security
- Subprocessors: AWS, MongoDB Atlas, ClickHouse, etc., listed as "USA; EU", with "EU processing locations apply only to EU-hosted accounts". https://knock.app/legal/subprocessors
- DPA: SCCs Module 2/3 (§9.2), UK Addendum (§9.3); no DPF. https://knock.app/legal/data-processing-addendum
- Users: only `id` is required; inline identification on trigger is supported. https://docs.knock.app/concepts/users
- React: `@knocklabs/react` with `KnockProvider`, `KnockFeedProvider`, `NotificationIconButton`, `NotificationFeedPopover`, plus hooks; real-time feed. https://docs.knock.app/in-app-ui/react/overview, https://docs.knock.app/in-app-ui/react/feed
- Auth: enhanced security is off by default. The RS256 JWT (`sub` = user id) is signed with `signUserToken` from `@knocklabs/node`. The docs suggest keeping it off in development and enabling it before release. https://docs.knock.app/in-app-ui/security-and-authentication

### Novu (Cloud)
- Pricing: Free 10,000 workflow runs/mo, 5 workflows, 2 environments, 3 team members, "Activity feed retention: 24 hours"; US **and EU** residency on Free. Pro from $30/mo (30k runs, 7-day retention); Team from $250/mo; extra runs $1.20/1k. https://novu.co/pricing
- Residency: EU region in Frankfurt, all plans, chosen at account creation (cannot be switched by URL; contact sales to move). API `https://eu.api.novu.co`, WS `wss://eu.socket.novu.co`; keys are region-bound. DPA + SCC template at novu.co/dpa; Trust Center trust.novu.co. https://docs.novu.co/platform/additional-resources/security
- React: `@novu/react` `<Inbox applicationIdentifier subscriber backendUrl socketUrl />` (React 18/19). https://docs.novu.co/platform/inbox/setup-inbox
- Auth: optional HMAC. `subscriberHash = HMAC-SHA256(secretKey, subscriberId)`; once enabled, the "Inbox will not load" without it. https://docs.novu.co/platform/inbox/prepare-for-production
- Node: `@novu/api`, `novu.trigger({ workflowId, to: { subscriberId }, payload })`, `serverURL: "https://eu.api.novu.co"`. https://docs.novu.co/platform/sdks/server/typescript

### Novu (self-hosted Community)
- MIT core, `enterprise` folder commercial. https://github.com/novuhq/novu
- Inbox, Bell and preferences are available self-hosted. Cloud-only: snooze, branding removal, >2 environments, translations, SSO/RBAC/MFA, DPAs/SOC 2; 1 team member. https://docs.novu.co/community/self-hosted-and-novu-cloud
- Requirements: MongoDB, Redis with AOF, S3-compatible storage. Single VM 4 vCPU / 8 GB. https://docs.novu.co/community/self-hosting-novu/overview

### MagicBell
- Pricing: Builder $0 with 1,000 deliveries/mo (each channel per user = 1 delivery); Startup $249/mo with 50,000 deliveries, then $0.0025 each. Builder overage rate not listed. https://www.magicbell.com/pricing
- Security: "fully compliant with GDPR", "We provide data processing agreements"; no region given. https://www.magicbell.com/security
- Privacy policy: MagicBell, Inc., San Francisco; "processed at the Owner's operating offices"; no transfer mechanism named. https://www.magicbell.com/privacy-policy
- Users: email or external id; external id recommended. https://www.magicbell.com/docs/primitive/user
- Auth: HS256 user JWT with `user_external_id` (email only required if no external id). https://www.magicbell.com/docs/api/authentication/user
- React: `@magicbell/react` (`Inbox`, `FloatingInbox`, `ContextProvider` with required `token`). https://www.magicbell.com/docs/libraries/magicbell-react

### SuprSend
- Pricing: Free 10,000 notifications/mo; Essentials $100/mo (50k, then $2/1k); Business $250/mo; Enterprise custom with "Custom Data Center Region", self-hosted and BYOC (beta). https://www.suprsend.com/pricing
- Security: "We support EU-data residency options"; links to DPA and subprocessors. https://docs.suprsend.com/docs/security
- DPA: SuprStack Inc (Delaware). Data stored in the workspace's hosting region (US or EU, §6(a)); staff access from India (§6(b)); DPF, then SCCs, UK and Swiss variants (§10). https://www.suprsend.com/data-processing-addendum
- React: `@suprsend/react` (`@suprsend/react-inbox` is end-of-support); headless feed available; JWT auth. https://docs.suprsend.com/docs/inbox-react
- Auth: public key plus optional "enhanced security" ES256 JWT (`entity_id` = distinct id), "For production workspaces public API Keys alone isn't enough". https://docs.suprsend.com/docs/client-authentication
- Node: `@suprsend/node-sdk`, `trigger_workflow` with recipients by `distinct_id`. https://docs.suprsend.com/docs/node-trigger-workflow-from-api

### Courier
- Pricing: Developer free (10,000 sends/mo); Business $0.005/send pay-as-you-go including EU Data Residency (all sends billed once you are on Business); Enterprise custom. https://www.courier.com/pricing
- Residency: EU datacenter in Ireland, "available on every paid plan: Business and Enterprise". Separate EU console/API/inbox hosts (`api.eu.courier.com`, `inbox.eu.courier.io`, `wss://realtime.eu.courier.io`); the EU workspace has its own keys. https://www.courier.com/docs/security/data-residency
- Inbox auth: backend calls `POST /auth/issue-token` with scope `user_id:<id> inbox:read:messages inbox:write:events`; React `@trycourier/courier-react` `signIn({ userId, jwt, apiUrls: getCourierApiUrlsForRegion("eu") })`; no automatic token refresh. https://www.courier.com/docs/platform/inbox/authentication
- Not verified: whether an Inbox message counts as one "send" (the inbox overview page does not say). https://www.courier.com/docs/platform/inbox/inbox-overview

## Open questions to put to vendors

- Knock: is an EU-hosted account available now (the subprocessor page implies it), and on which plan? Does the DPA cover Free accounts?
- SuprSend: can a Free/Essentials workspace be hosted in the EU?
- MagicBell: hosting region and DPA transfer mechanism.
- Courier: does each Inbox message count as one send?
- Novu: the Free plan's Inbox (not activity) feed retention.
