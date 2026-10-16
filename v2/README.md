# LexFis AI Mini App v2 — secure staged integration

## Status

**Code and isolated Make workflows prepared. Not live.** Current production `@LexFisAI_bot` and the GitHub Pages Mini App remain unchanged until backend setup and an end-to-end test succeed.

## What v2 adds

- Questions answered within the Mini App (a new isolated Make AI Agent scenario).
- Preliminary document analysis: PDF, DOCX, TXT, JPG, PNG (up to 3 MiB).
- Court-dispute preliminary assessment.
- Requests to a lawyer, saved to `LEXFIS_LEADS` and delivered to the lawyer through the existing bot.
- Signed Telegram Mini App auth on the Worker: Telegram `initData` HMAC-SHA256, signed auth date <= 1 hour, user ID from the validated signature, not from the client.
- Separate webhook secrets **only on the server**, per-user limits using KV, explicit consent.

## Existing Make scenarios (created inactive)

- Chat: scenario `8004808` (webhook -> Make AI Agent -> response).
- Document: scenario `8004922` (webhook -> Make AI Agent with `toBinary(base64)` file -> response).
- Leads: scenario `8004931` (webhook -> `LEXFIS_LEADS` -> response).

All are currently **inactive and untested with real webhook deliveries**. The original `LexFis AI Bot — Production` scenario `6754813` has not been modified.

The Make-generated webhook URLs must be placed in Worker secrets, **not in browser JavaScript, wrangler.jsonc, public GitHub files, or query strings**. Obtain URLs from the Make scenario triggers.

## Deployment prerequisites (Cloudflare Workers)

1. Cloudflare account with permission to deploy Worker and create Worker KV namespace.
2. Create KV namespace `LEXFIS_MINIAPP_RATELIMIT`, copy its namespace ID into `wrangler.jsonc` in place of `REPLACE_WITH_YOUR_KV_NAMESPACE_ID` (this ID is public, not a secret).
3. Install dependencies (`npm install`), authenticate Wrangler (`npx wrangler login`).
4. Deploy: `npm run deploy`. The Worker will refuse intake until the required secrets are configured.
5. Using **Wrangler's secret prompt**, configure separately:

   ```text
   npx wrangler secret put BOT_TOKEN
   npx wrangler secret put ADMIN_CHAT_ID
   npx wrangler secret put MAKE_CHAT_WEBHOOK_URL
   npx wrangler secret put MAKE_DOCUMENT_WEBHOOK_URL
   npx wrangler secret put MAKE_LEAD_WEBHOOK_URL
   ```

   Never paste the token, chat ID or webhook URLs into a public repository, this chat or a screenshot.

6. Activate only the **three new** Make scenarios after configuration, privacy review and rate limit validation. The chatbot's Production scenario must remain unchanged.
7. Smoke-test `/api/health`: each `enabled` value should be `true`. Open the Worker URL through Telegram, test with an account owned by the operator and **non-sensitive fictional data**. Test question, small sample file, and lead end-to-end. Verify the lead in `LEXFIS_LEADS` and delivery to the Telegram administrator. Check failures/quotas.
8. Only then set the BotFather Main App URL to the Worker-hosted HTTPS URL. Do not switch production to an untested deployment.

## Do not skip before public use

- Publish and link to a compliant personal-data handling policy, confirm operator details, legal grounds and cross-border transfer implications under Russian law, and establish retention/deletion procedures. Avoid sending privileged materials until an approved secure channel exists.
- Configure Cloudflare WAF / account-level rate limits and monitor abuse. KV read/write limits in this code are best-effort; they are not atomic and do not provide strong global DDoS protection. Further protective measures may be required before accepting untrusted public users.
- Confirm Make AI model billing/credit limits and decide a per-day budget. The new AI scenarios incur existing Make/AI credits when they run; no unbounded tests or automatic retries should be enabled.
- Check Make's file processing and response latency with a genuine test request. `toBinary(file_data; "base64")` is the documented Make method for loading an AI agent's input file, but it is not proven in this user's scenario until tested.
- Never treat an AI-generated judicial citation as confirmed without opening the decision in a primary source.

## Local testing

```sh
npm test
```

The current 21 tests use fake Telegram signatures, an in-memory KV double and stub HTTP endpoints; they are not production integration tests.