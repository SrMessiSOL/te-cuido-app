# TE CUIDO live demo deploy

This path puts the existing demo online with:

- FastAPI agent on Render
- Next.js dashboard on Vercel
- Solana in mock mode first, then devnet mode when the program/wallet are ready

## 1. Pick a demo API key

Generate any long random string and use the same value in both services:

```bash
DEMO_API_KEY=replace-with-a-long-random-value
```

The agent only requires this key for mutation endpoints:

- `POST /api/simulate`
- `POST /api/wellbeing`

`GET /api/status` and `GET /api/health` stay public so the dashboard can poll status.

## 2. Deploy the agent to Render

Create a new Render Blueprint from this repo. Render will read `render.yaml`.

Set these environment variables in Render:

```bash
DEMO_API_KEY=replace-with-the-same-key
GEMINI_API_KEY=optional-for-demo
TELEGRAM_BOT_TOKEN=optional-for-demo
TELEGRAM_CHAT_ID_CONTACT_1=optional-for-demo
TELEGRAM_CHAT_ID_CONTACT_2=optional-for-demo
CORS_ORIGINS=https://your-vercel-app.vercel.app
USE_MOCK_SOLANA=true
```

After deploy, verify:

```bash
curl https://your-render-agent.onrender.com/api/health
```

Trigger a demo event:

```bash
curl -X POST "https://your-render-agent.onrender.com/api/simulate?event_type=low_hr" \
  -H "x-demo-api-key: replace-with-the-same-key"
```

## 3. Deploy the dashboard to Vercel

Create a Vercel project with `dashboard` as the root directory.

Set these environment variables in Vercel:

```bash
NEXT_PUBLIC_AGENT_URL=https://your-render-agent.onrender.com
AGENT_API_KEY=replace-with-the-same-key
NEXT_PUBLIC_SOLANA_RPC=https://api.devnet.solana.com
NEXT_PUBLIC_PROGRAM_ID=
```

Build command:

```bash
npm run build
```

Output directory:

```bash
.next
```

## 4. Solana mode

For the fastest live demo, start with:

```bash
USE_MOCK_SOLANA=true
```

The UI works and shows mock tx hashes.

To use real devnet writes:

1. Deploy `solana/lib.rs` with Solana Playground.
2. Download the generated IDL to `solana/idl.json`.
3. Set Render env vars:

```bash
USE_MOCK_SOLANA=false
PROGRAM_ID=your-program-id
SOLANA_RPC_URL=https://api.devnet.solana.com
```

4. Watch the Render logs on first boot. The agent prints its generated wallet pubkey.
5. Fund that pubkey on devnet.
6. Trigger `/api/simulate` again and open the returned tx hashes in Solana Explorer with `?cluster=devnet`.

## 5. Pitch-day checklist

- Render agent is awake before the pitch.
- Vercel dashboard shows `Agente conectado`.
- `curl /api/simulate` command is ready with the `x-demo-api-key` header.
- Telegram is optional; without tokens the agent logs notifications to stdout.
- Solana can stay mocked if devnet is slow. The product flow still works.
