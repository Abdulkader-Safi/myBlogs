# Next.js 15.5: A Developer’s Guide to Turbocharged Builds, Full-Node Middleware & Smarter TypeScript

Published August 18, 2025, Next.js 15.5 delivers powerful enhancements that accelerate builds, modernize middleware, elevate TypeScript, refine linting, and pave the path to Next.js 16. ￼

## Why It Matters

Next.js 15.5 isn’t just another version—it’s a strategic milestone. It empowers developers to build faster, more reliably, and with stronger type safety, all while preparing for the upcoming 16 release.

---

## Turbopack Builds in Beta: Build Speed, Reinvented

- Feature: The next build `--turbopack` command now supports production builds in beta.
- Benefit: Vercel sites like nextjs.org are already powered by Turbopack, handling over 1.2 billion requests, and delivering 2–5× faster builds. ￼ ￼

Sample Script (package.json):

```json
{
  "scripts": {
    "build": "next build --turbopack",
    "start": "next start"
  }
}
```

---

## Node.js Middleware: Stable, Powerful, Full-Stack Capable

- Feature: Middleware now runs on the Node.js runtime—no more edge-only limitations.
- Benefit: You can now use fs, crypto, and other npm packages directly within middleware logic. ￼ ￼

Example: middleware.ts

```typescript
import { NextRequest, NextResponse } from "next/server";

export const config = { runtime: "nodejs" };

export function middleware(req: NextRequest) {
  const fs = require("fs");
  const crypto = require("crypto");

  // Simulated token validation
  const token = req.headers.get("authorization");
  if (!isValid(token)) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
  return NextResponse.next();
}

function isValid(token: string | null) {
  // Validate using crypto
  return Boolean(token);
}
```

---

## TypeScript Improvements: Typed Routes, Code Integrity, Productivity

### Features

- Typed Routes: Compile-time path validation in `<Link />` and router.push().
- Route Export Validation: Ensures consistency across layout and handler exports.
- Global Types: Auto-available PageProps, LayoutProps, and RouteContext.
- next typegen: New CLI for manual type generation in CI workflows. ￼ ￼

Enable typed routes:

```typescript
// next.config.js
module.exports = {
  typedRoutes: true,
};
```

Run type generation:

```bash
npx next typegen
```

### Code Example: Typed Routes in Action

Enable typed routes in your next.config.js:

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    typedRoutes: true, // ✅ Enable typed route validation
  },
};

module.exports = nextConfig;
```

Example project structure:

```bash
app/
 ├─ dashboard/
 │   ├─ page.tsx
 │   ├─ [id]/
 │   │   └─ page.tsx
 ├─ settings/
 │   └─ page.tsx
```

Using `<Link />` with typed routes:

```typescript
// app/page.tsx
import Link from "next/link";

export default function HomePage() {
  return (
    <main>
      <h1>Welcome</h1>

      {/* ✅ Valid route, compiles fine */}
      <Link href="/dashboard">Go to Dashboard</Link>

      {/* ❌ Invalid route: typo will throw a compile-time error */}
      <Link href="/dashbord">Broken Link</Link>

      {/* ✅ Dynamic route with typed param */}
      <Link href="/dashboard/123">Dashboard Item</Link>
    </main>
  );
}
```

Using router.push() with typed routes:

```typescript
"use client";

import { useRouter } from "next/navigation";

export default function DashboardButton() {
  const router = useRouter();

  function goToSettings() {
    // ✅ Correct usage
    router.push("/settings");

    // ❌ Wrong usage: This will cause a compile error
    router.push("/settingz");
  }

  return <button onClick={goToSettings}>Go to Settings</button>;
}
```

Typed routes in API calls:

```typescript
"use client";

import { useRouter } from "next/navigation";

export default function DashboardRedirect({ id }: { id: string }) {
  const router = useRouter();

  function viewDashboard() {
    // ✅ Safe dynamic navigation
    router.push(`/dashboard/${id}`);
  }

  return <button onClick={viewDashboard}>View Dashboard {id}</button>;
}
```

### ✅ Why This Matters

- No more silent 404s → Typos in routes are caught at build time.
- Dynamic safety → Even parameterized routes (/dashboard/[id]) are type-checked.
- Consistency → Works across `<Link />`, router.push(), and even API routes.

---

## Next lint is Deprecated: Own Your Linting with ESLint or Biome

- Change: The next lint command is now deprecated.
- Action: Switch to explicit linter setups like ESLint or Biome using your preferred tooling. ￼ ￼

Example package.json:

```json
{
  "scripts": {
    "lint": "eslint . --ext .js,.ts,.jsx,.tsx"
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "eslint-config-next": "latest"
  }
}
```

---

## Deprecation Warnings for Next.js 16: Get Ahead of the Curve

To ease your upcoming upgrade to Next.js 16, 15.5 already flags deprecated features:

| Deprecated Feature                   | Replacement/Action                          |
| ------------------------------------ | ------------------------------------------- |
| legacyBehavior (in `<Link>`)         | Use the modern `<Link>` syntax              |
| AMP (useAmp, amp: true)              | Migrate to SSR/DSG approaches               |
| `<Image quality>` prop               | Replace with default or configured fallback |
| Local image paths with query strings | Conform to images.localPatterns config      |

---

## Developer Workflow & Real-World Impact

Here’s what to do today to get the most out of Next.js 15.5:

1. Build faster: Run next build `--turbopack` and measure improvements.
2. Enhance middleware: Leverage Node’s full API surface in middleware configurations.
3. Boost type safety: Activate typedRoutes and generate type definitions in your CI.
4. Modernize linting: Ensure robust linting pipelines using standalone tools.
5. Upgrade-ready: Refactor deprecated patterns before Next.js 16 hits.

---

## TL;DR

Next.js 15.5 is a strategic, developer-first release: it dramatically improves build times, stabilizes full-server middleware, strengthens type safety, unifies linter strategy, and cues developers for Next.js 16. Upgrade now to build faster, code smarter, and stay future-ready.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ — even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team — I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
