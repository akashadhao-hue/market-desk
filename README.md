# Market Desk

A CopilotKit chat app grounded in the supplied NEM Parquet data and six internal policy documents.
The CopilotKit v2 runtime streams answers from an OpenAI-backed agent with server-side DuckDB and
policy retrieval tools. Tool inputs, executed SQL, results, and retrieved document sections are
rendered inline in the conversation.

## Requirements

- Node.js 20.9 or newer and npm
- An OpenAI API key
- The project `data/` and `docs/` directories (the app reads them; it does not modify them)

## Install and run

From the repository root:

```bash
cd track1_qna_copilotkit/app
npm ci
cp .env.example .env.local
```

Replace `OPENAI_API_KEY` in `.env.local` with your own key. Do not commit this file.

```bash
npm run dev
```

Open <http://localhost:3000>. Set `OPENAI_MODEL` in `.env.local` to select another model supported
by the OpenAI API. The default is `gpt-4o-mini`.

The application expects to run from this directory so the API routes can resolve `../../data` and
`../../docs` from `process.cwd()`.

## Verify

```bash
npm test
npm run lint
npm run build
```

The focused tests check DuckDB data access, SQL write/multi-statement/external-file rejection,
result caps, policy-section ranking, and FCAS limitation retrieval. A labelled manual evaluation set
is in `../evaluation/questions.json`.

## Trust and scope notes

- The SQL endpoint accepts one `SELECT`/`WITH` statement, requires a reference to an allowlisted
  table, rejects write operations and external file functions, interrupts long queries, and caps
  visible output at 100 rows.
- The runtime serves CopilotKit v2's multi-route API below `/api/copilotkit`; an OpenAI key is only
  needed when sending a chat message. `OPENAI_MODEL` selects the model and defaults to `gpt-4o-mini`.
- Retrieval tokenizes each Markdown heading section and ranks by inverse document frequency,
  heading match, and adjacent query phrases. There is no vector database or embedding dependency.
- The LLM is instructed to use the tools before reporting figures and to cite retrieved sections.
  Review the visible audit trail for each response; model instructions are not a formal proof that a
  particular answer used every required source.
- Forecast answers must use only eligible vintages (`RUN_DATETIME <= τ`) and the sample is
  energy-only: FCAS revenue is not available.

See `../DESIGN.md` for the architecture and evaluation strategy, and `../AGENT_LOG.md` for the
implementation and verification record.
# market-desk
