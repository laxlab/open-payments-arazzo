# open-payments-arazzo

Arazzo Specification 1.0.1 workflow files documenting the [Open Payments](https://openpayments.dev) API flows.

Submitted as part of the [Interledger Foundation SDK Grant Program](https://interledger.org/grant/open-payments-sdk) — Arazzo documentation track.

---

## What is this?

Open Payments has three well-documented OpenAPI specifications but no Arazzo workflow files. A developer reading the specs knows what each individual endpoint does — but not the order to call them, which outputs feed which inputs, or how the GNAP grant system connects to the resource operations. This repo fills that gap.

Each file here documents one complete Open Payments flow as a machine-readable, validated Arazzo 1.0.1 workflow. The files can be used by documentation tools, SDK generators, test runners, and AI agents that consume Arazzo descriptions.

---

## What is Arazzo?

[Arazzo](https://www.openapis.org/arazzo-specification) is an OpenAPI Initiative standard for describing sequences of API calls as machine-readable workflows. Where OpenAPI describes what a single endpoint does, Arazzo describes how multiple endpoints work together to achieve a real outcome — which calls to make, in what order, and how data flows from one response into the next request.

---

## Source specifications

The `specs/` directory contains the three official Open Payments OpenAPI specifications, sourced from [interledger/open-payments-specifications](https://github.com/interledger/open-payments-specifications).

| File | Server | Purpose |
|------|--------|---------|
| `wallet-address-server.yaml` | Wallet Address Server | Public account info and JWKS |
| `auth-server.yaml` | Authorization Server | GNAP grant requests and continuations |
| `resource-server.yaml` | Resource Server | Incoming payments, quotes, outgoing payments |

All workflow files reference these specs via relative paths in their `sourceDescriptions` blocks.

---

## Workflows

| File | Flow | Steps | Grant type |
|------|------|-------|------------|
| `01-create-incoming-payment.arazzo.yaml` | Create incoming payment | 3 | Non-interactive |
| `02-create-quote.arazzo.yaml` | Create quote | 3 | Non-interactive |
| `03-end-to-end-payment.arazzo.yaml` | Full end-to-end payment | 9 | Interactive (step 7) |
| `04-list-and-read-resources.arazzo.yaml` | List and read resources | 7 | Non-interactive |
| `05-complete-incoming-payment.arazzo.yaml` | Complete incoming payment | 3 | Non-interactive |

### Flow 1 — Create incoming payment

Resolves the payee wallet address, requests a non-interactive incoming-payment grant, and creates the incoming payment resource on the payee resource server. The `incomingPaymentUrl` output is the input that Flow 2 and Flow 3 require.

### Flow 2 — Create quote

Resolves the payer wallet address, requests a non-interactive quote grant, and creates the quote resource on the payer resource server. The quote locks in the ILP exchange rate for a short window. The `quoteId` output feeds into the outgoing payment creation step in Flow 3.

### Flow 3 — Full end-to-end payment

The complete Open Payments protocol in a single workflow. Combines all sub-flows across both wallets: creates an incoming payment on the payee side (steps 1-3), creates a quote on the payer side (steps 4-6), requests an interactive outgoing payment grant requiring explicit payer approval (step 7), continues the grant after approval (step 8), and creates the outgoing payment that moves funds (step 9).

Step 7 is the only interactive step in the Open Payments protocol. The authorization server returns a redirect URL that the payer must visit to approve the payment before the workflow can proceed to step 8.

### Flow 4 — List and read resources

Documents all read operations in a single workflow using one non-interactive grant that requests list and read access across all three resource types. Covers paginated listing of incoming payments and outgoing payments, and individual resource reads for incoming payments, outgoing payments, and quotes.

### Flow 5 — Complete incoming payment

Requests a non-interactive grant with the complete action and marks an incoming payment as finished. Used after fixed-send outgoing payments where no `incomingAmount` cap was set, and at the end of streaming Web Monetization sessions. Signals to the payee ASE that no further payments are coming so it can begin post-payment processing immediately.

---

## Flow connections

The workflows are designed to be used in sequence. Outputs from earlier flows become inputs to later flows:

| Output | Source flow | Consumer flow | Field |
|--------|------------|---------------|-------|
| Incoming payment URL | Flow 1 | Flow 2, Flow 3 | `incomingPaymentUrl` → `incomingPaymentUrl` input |
| Incoming payment UUID | Flow 1 | Flow 4, Flow 5 | Parse UUID from `incomingPaymentUrl` → `incomingPaymentId` input |
| Quote URL | Flow 2 | Flow 3 | Flow 3 creates its own quote internally |
| Outgoing payment UUID | Flow 3 | Flow 4 | Parse UUID from `outgoingPaymentId` → `outgoingPaymentId` input |
| Quote UUID | Flow 2, Flow 3 | Flow 4 | Parse UUID from `quoteId` → `quoteId` input |

---

## Known limitations of Arazzo 1.0.1

These are honest limitations of the specification itself, not gaps in this documentation. They are called out here so implementers understand where the Arazzo runtime ends and the client implementation must begin.

**Server URL resolution.** Each workflow discovers `authServer` and `resourceServer` URLs in step 1 by calling the wallet address endpoint. Arazzo 1.0.1 has no mechanism to dynamically override the server URL for a subsequent step using a runtime expression. The discovered URLs are extracted as step outputs and documented in step descriptions, but the actual request routing must be handled by the client implementation or Arazzo runtime.

**URL path segment extraction.** The GET and POST operations for individual resources (`get-incoming-payment`, `get-outgoing-payment`, `get-quote`, `complete-incoming-payment`) are at paths like `/incoming-payments/{id}` where `{id}` is the UUID segment only — not the full resource URL. Arazzo 1.0.1 runtime expressions cannot split a URL string and extract a path segment. Affected inputs (`incomingPaymentId`, `outgoingPaymentId`, `quoteId`) are documented as requiring the UUID only, and the client implementation is responsible for parsing the full resource URL to extract it.

**Interactive grant pause.** The outgoing payment grant in Flow 3 step 7 requires the payer to approve the payment in the authorization server's interface before an access token is issued. Arazzo 1.0.1 has no mechanism to model asynchronous human interactions. The workflow documents this honestly: execution must pause after step 7, the client must redirect the payer to `interact.redirect`, and two inputs (`interactRef` and `continueId`) carry the values produced by the approval back into the workflow for step 8 to consume.

---

## Validation

All workflow files are validated against the Arazzo 1.0.1 schema using [Redocly CLI](https://redocly.com/docs/cli/).

```bash
npm install
npm run validate
```

To validate a single file:

```bash
npx redocly lint workflows/03-end-to-end-payment.arazzo.yaml
```

---

## Author

Victor — [Lax Lab](https://github.com/laxlab)
