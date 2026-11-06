# Preview deploys fail with `unexpected_failure`: eve agents + Next.js middleware

A minimal reproduction. A Next.js app hosts 12 eve agents through `withEve` from `eve/next` and has a `proxy.ts` (middleware). Preview deployments to Vercel fail at **"Deploying outputs..."** with `errorCode: unexpected_failure`. Production deployments of the same code succeed.

## Versions

- `next` 16.4.5 (stock `create-next-app --yes`)
- `eve` 1.0.0-rc.6
- Node.js 26.x (`engines.node`)
- pnpm 12.18.0
- Vercel CLI 65.11.0

## What's in the repo

Everything is stock scaffold output except:

- `agents/agent-01` … `agents/agent-12`: 12 identical agents from `eve init agent-NN`, which make this an eve agent workspace.
- `next.config.ts`: the stock config wrapped with `withEve(nextConfig)`.
- `proxy.ts`: a trivial middleware that sets one response header.
- `pnpm-workspace.yaml`: `minimumReleaseAge: 0`, so the eve release can be installed while it's newer than pnpm's default cutoff.

## Steps to reproduce

```bash
pnpm install
vercel link
vercel deploy        # preview
vercel deploy --prod # production
```

## Expected

Both deployments reach `READY`.

## Actual

The preview build completes, then fails while deploying outputs:

```
Deploying outputs...
Error: Unexpected error. Please try again later. ()
```

The deployment ends with `readyState: ERROR` and `errorCode: unexpected_failure`. The production deploy of the same commit reaches `READY`.

## Notes

- Removing `proxy.ts`, or dropping to 10 or fewer agents, makes the preview deploy succeed. Around 11 agents it's intermittent.
- `withEve` copies the host's middleware function into every agent service's output. With middleware present, each additional agent adds another copy of the middleware to the deployment. That may be what crosses a limit or timeout in the preview deploy step, but we haven't confirmed it.
