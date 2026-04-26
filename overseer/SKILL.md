# Overseer — Error Tracking & Session Replay

You are integrating Codmir Overseer into this application. Overseer provides error tracking, 15-second DVR session replay, and AI-powered incident management.

## Step 1: Install the SDK

```bash
npm install @codmir/sdk
```

## Step 2: Environment Variables

Add these to your `.env` or `.env.local`:

```env
OVERSEER_DSN="<your-dsn-url>"
OVERSEER_KEY="<your-client-key>"
```

Get these values from your project's Overseer setup page at `https://codmir.com/project/<projectId>/issues/overseer/setup`.

## Step 3: Initialize the SDK

### Next.js (App Router)

Create or update `instrumentation.ts` in your project root:

```typescript
import { Overseer } from "@codmir/sdk";

export function register() {
  Overseer.init({
    dsn: process.env.OVERSEER_DSN!,
    environment: process.env.NODE_ENV,
    release: process.env.NEXT_PUBLIC_VERCEL_GIT_COMMIT_SHA,
  });
}
```

For client-side, create or update `instrumentation-client.ts`:

```typescript
import { Overseer } from "@codmir/sdk/browser";

Overseer.init({
  dsn: process.env.NEXT_PUBLIC_OVERSEER_DSN!,
  environment: process.env.NODE_ENV,
  replay: {
    enabled: true,
    bufferSeconds: 15,
  },
});
```

### React (Vite / CRA)

In your entry file (e.g., `main.tsx` or `index.tsx`):

```typescript
import { Overseer } from "@codmir/sdk/browser";

Overseer.init({
  dsn: import.meta.env.VITE_OVERSEER_DSN,
  environment: import.meta.env.MODE,
  replay: {
    enabled: true,
    bufferSeconds: 15,
  },
});
```

Add an error boundary wrapper around your app:

```tsx
import { OverseerErrorBoundary } from "@codmir/sdk/react";

function App() {
  return (
    <OverseerErrorBoundary fallback={<ErrorFallback />}>
      <YourApp />
    </OverseerErrorBoundary>
  );
}
```

### Express / Node.js

```typescript
import { Overseer } from "@codmir/sdk";
import express from "express";

Overseer.init({
  dsn: process.env.OVERSEER_DSN!,
  environment: process.env.NODE_ENV,
});

const app = express();

// Your routes here

// Error handler — must be last
app.use(Overseer.expressErrorHandler());
```

### Vue / Nuxt

```typescript
import { Overseer } from "@codmir/sdk/browser";

Overseer.init({
  dsn: import.meta.env.VITE_OVERSEER_DSN,
  environment: import.meta.env.MODE,
  replay: {
    enabled: true,
    bufferSeconds: 15,
  },
});

app.config.errorHandler = (err, instance, info) => {
  Overseer.captureException(err, { extra: { info } });
};
```

## Step 4: Verify

After setup, trigger a test error to verify events appear in the Overseer dashboard:

```typescript
Overseer.captureException(new Error("Test error from Overseer setup"));
```

## Best Practices

- Set `environment` to distinguish between development, staging, and production
- Set `release` to your git SHA or version tag for release tracking
- Use `Overseer.setUser({ id, email })` after authentication to link errors to users
- Enable replay only in production to minimize overhead
- Add source maps upload to your build pipeline for readable stack traces

## Configuration Reference

```typescript
Overseer.init({
  dsn: string,                    // Required: Data Source Name from project setup
  environment: string,            // Recommended: "production" | "staging" | "development"
  release: string,                // Recommended: git SHA or version
  replay: {
    enabled: boolean,             // Enable 15-second DVR session replay
    bufferSeconds: number,        // Buffer size (default: 15)
  },
  sampleRate: number,             // Error sample rate 0-1 (default: 1.0)
  tracesSampleRate: number,       // Performance traces sample rate 0-1 (default: 0.1)
  beforeSend: (event) => event,   // Transform or filter events before sending
});
```
