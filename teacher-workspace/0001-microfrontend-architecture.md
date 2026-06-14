## Author(s)

- Ameera @ameeracadam
- Eileen @kmye
- Fei Dong @Dng132FEI
- Victor Loh @vlwk
- Yi Ming @YimingIsCOLD

## Discussion & Voting Timeline

- Discussion open until 2026-06-03
- Voting starts when the author calls for it
- Voting ends on 2026-06-03 or when consensus is reached

## Summary

This RFC proposes adopting a Micro-Frontend (MFE) architecture built on Module Federation 2.0 (MFv2) using native ESM build-time plugins (@module-federation/vite and @module-federation/rsbuild-plugin).

Our core objectives are complete team autonomy via isolated Git repositories, independent deployment pipelines using static storage, automated compiler-driven contracts, and native cryptographic subresource verification.

## Motivation

As our platform expands—particularly with complex, standalone domains like the Teacher Workspace—our monolithic frontend structure has introduced significant operational friction. Multiple engineering squads are shipping to a single codebase, resulting in tight deployment coupling, continuous integration bottlenecks, and high regression risks.

Our goal is to transition to an ecosystem that provides:

- True Team Autonomy: Squads can develop, test, and release features (e.g., teaching tools, attendance tracking) without waiting for or risking the stability of the Host container.
- Instantaneous Feedback Loops: Transitioning local development from Webpack to Vite/RSBuild yields near-instant Hot Module Replacement (HMR), keeping developer efficiency high.
- Resilient Infrastructure: Isolating domain-specific failures ensures that if a single workspace module degrades, the primary host application remains fully functional.

## Proposal

### 1. Runtime Linking & Host Architecture

Upon booting, the Host Shell asynchronously resolves remote boundaries by reading automated manifest files directly from static CDN storage targets via MFv2's native orchestration runtime. This keeps the compilation pipelines decoupled, though manifest target origins remain statically registered in the application bootstrap entry.

```typescript
// host/src/main.ts
import { init } from '@module-federation/enhanced/runtime';

async function bootstrap() {
  try {
    // Initialize federation boundaries natively using remote manifest targets
    init({
      name: 'host-shell',
      remotes: [
        {
          name: 'teacher_workspace',
          entry: 'https://cdn.domain.com/modules/teacher-workspace/mf-manifest.json',
          type: 'esm',
        },
        {
          name: 'attendance_tracker',
          entry: 'https://cdn.domain.com/modules/attendance/mf-manifest.json',
          type: 'esm',
        },
      ],
    });
  } catch (error) {
    console.error('Critical: Failed to orchestrate Module Federation 2.0 runtime', error);
  }

  // Mount the actual Host Shell Bundle
  import('./bootstrap');
}

bootstrap();
```

### 2. Configuration Profiles

The Host leverages RSBuild's first-party Module Federation plugin for robust tree-shaking, fast compilation, and optimized chunk splits. Remote micro-frontends can continue utilizing Vite via the official Vite module federation plugin.

Host Shell Configuration (rsbuild.config.ts)

```typescript
import { defineConfig } from '@rsbuild/core';
import { pluginReact } from '@rsbuild/plugin-react';
import { pluginModuleFederation } from '@module-federation/rsbuild-plugin';

export default defineConfig({
  plugins: [
    pluginReact(),
    pluginModuleFederation({
      name: 'host-shell',
      remotes: {}, // Manifest URLs are statically registered but resolved asynchronously at runtime via init()
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        'react-router': { singleton: true, requiredVersion: '^7.0.0' },
      },
    }),
  ],
  output: {
    target: 'web',
  },
});
```

Remote App Configuration (vite.config.ts)

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { federation } from '@module-federation/vite';

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'teacher_workspace',
      filename: 'remoteEntry.js',
      manifest: true, // Automates production mf-manifest.json generation
      dts: true, // Compiles and zips type definitions (.d.ts) for consumer use
      exposes: {
        './WorkspaceModule': './src/pages/WorkspaceRoot.tsx',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        'react-router': { singleton: true, requiredVersion: '^7.0.0' },
      },
    }),
  ],
  build: {
    target: 'chrome89',
  },
});
```

### 3. State & Context Propagation (Data/Auth Exchange)

To link component updates between the Host and Remotes, we reject global window-polluting state strategies. Data exchange is divided into two distinct patterns:

Auth & User Context (Global, Static/Read-Only)
Authentication tokens and user profiles are managed by the Host Shell. This data is propagated to remotes down the React component tree via standard React Context Providers exposed by the host or through a lightweight internal reactive event layer. Remotes access this context transparently at runtime without making duplicate network requests for session validation.

Domain Data Sync (Cross-Component Communication)
For standard data updates or notifications between adjacent micro-frontends (e.g., updating a global navigation counter based on a workspace action), components will utilize standard browser Custom Events (window.dispatchEvent) with strictly versioned event payloads. This keeps the boundary between modules loosely coupled.

### 4. Bidirectional Type Safety & Contracts

To remove human error from cross-repository dependencies, the compilation layer natively enforces a two-way contract schema:Remote $\rightarrow$ Host SafetyBecause remotes build with dts: true, they automatically compress and publish an @mf-types.zip file to S3 alongside their bundle assets. When running the Host Shell locally, the build plugin streams these remote types on the fly, providing native IDE IntelliSense and compile-time validation for exposed modules:

```typescript
// Fully typed and verified at compile-time across separate repositories
import WorkspaceModule from 'teacher_workspace/WorkspaceModule';
```

Host $\rightarrow$ Remote Safety (Shared Typings)To type arguments passed from the Host down to Remotes (such as User/Auth schemas or custom event payloads), we will maintain a lightweight internal npm package. Both the Host and individual Remote repositories will install this package as a devDependency to guarantee structural contract agreement.

### 5. Routing Architecture & Sub-Path Delegation

The platform utilizes a distributed, nested routing model powered natively by React Router v7. We explicitly favor v7 Declarative Mode (using JSX <Routes> and <Route> elements) over Data Mode (createBrowserRouter), as Data Mode forces a centralized, static route tree that contradicts remote autonomy.

Top-Level Route Orchestration
The Host App instantiates the application's singular, declarative wrapper. It maps top-level domain paths to their respective remotes using wildcard paths (/\*). The trailing wildcard hands over sub-path routing control directly to the remote app once the base URL match occurs.

```typescript
// Inside Host App routing matrix (React Router v7 Declarative Mode)
import { Routes, Route } from 'react-router';

<Routes>
  <Route path="/workspace/*" element={<RemoteTeacherWorkspace />} />
</Routes>
```

Sub-Routing Autonomy
Remote MFEs seamlessly inherit the router context provided by the Host. Internal remote views are configured entirely with relative paths instead of absolute paths. This ensures the remote application remains agnostic of its parent route mount point.

```typescript
// Inside Remote MFE (Teacher Workspace)
import { Routes, Route } from 'react-router';

<Routes>
  <Route path="/" element={<WorkspaceDashboard />} />   {/* Resolves to /workspace */}
  <Route path="roster" element={<ClassRoster />} />    {/* Resolves to /workspace/roster */}
</Routes>
```

### 6. Subresource Integrity (SRI)

In standard web applications, Subresource Integrity (SRI) is straightforward because the HTML is static: build tools calculate script hashes and inject them directly into the index HTML. With MFv2, remotes load asynchronously at runtime, bypassing the initial HTML layer and breaking traditional SRI.

To solve this, we will leverage a Manifest-Driven SRI Interceptor built natively and build a plug that can be used to fetch the integrity hash into the runtime lifecycle:

1. Remote Generation: The remote compilation step computes the cryptographic hashes of chunks/assets and appends them directly into mf-manifest.json.
2. Host Ingestion: When the Host boots up, its init lifecycle fetches the remote manifests, creating an in-memory map of remote paths and their verified cryptographic hashes.
3. Runtime Injection: When a user navigates to a federated route, the MFv2 runtime engine intercepts the outgoing module script request, looks up the associated hash from the manifest map, and injects the integrity attribute to the element before DOM insertion.

### 7. Deployment Strategy (GitOps)

Each Micro-Frontend operates inside its own isolated Git repository. Deployments are executed independently by pushing build outputs straight to unique subfolders within a cloud storage environment backed by a content delivery network (CDN).

### 8. Observability & Blast Radius Isolation

Structural Error Boundaries: All federated micro-frontends are processed asynchronously via dynamic lazy-loading wrappers. If a network disruption or critical error crashes a remote component, an enveloping React Error Boundary isolates the failure, maintaining the execution of the rest of the application interface.

Distributed Event Telemetry: We hook into the runtime lifecycle engines (errorLoadRemote) to trace load errors. The system automatically tags runtime exceptions with specific micro-frontend names, build version tags, and asset URLs before routing logs directly to tools like Sentry or Datadog RUM.

### 9. AI Agent Optimization

By utilizing standardized JSON schemas and native build plugins rather than proprietary orchestrators, AI software tools (such as Cursor or GitHub Copilot) parse context windows with ease.

Boilerplate Reduction: Eliminating complex, custom lifecycle states (bootstrap, mount, unmount) allows agents to reason about code accuracy with zero framework hallucinations.

Auto-Correction Capabilities: When compilation disruptions occur, LLM-driven workflows can seamlessly read standard manifest structures to auto-heal missing TypeScript definition linkages.

## Alternatives Considered

Option A: Single-SPA Orchestration

Why it was rejected: It introduces significant boilerplate, lacks modern out-of-the-box build tools, and forces structural platform lock-in. Managing it requires maintaining custom helper systems (like an Import Map Deployer) which adds unnecessary infrastructure overhead.

Option B: Webpack Module Federation

Why it was rejected: Webpack introduces slower local build times and compromises the near-instantaneous Hot Module Replacement (HMR) speeds that our engineers expect from modern Vite workflows.

Option C: Traditional @originjs/vite-plugin-federation

Why it was rejected: This plugin relies on a community emulation of Webpack's protocol. It lacks an automated manifest design, native type compilation across isolated repositories, and first-party runtime injection lifecycles.

## Drawbacks

- Asynchronous Script Latency: Loading mf-manifest.json at startup introduces a minor network dependency. If asset delivery environments experience lag, user Time to Interactive (TTI) indicators will degrade.
- Shared Dependency Version Mismatches: If shared singleton libraries (like react) drift past accepted semantic version compatibility bounds across independent repositories, the runtime engine will pull down duplicate library bundles, increasing browser payload sizes.
- Rigid Origin Cache Controls: If your storage infrastructure misconfigures cache headers for files like mf-manifest.json, clients will get locked out of new software deployments until browser local caches expire.

## Impact

Development Workflow: Teams gain absolute autonomy over their respective codebases. Local development speeds will drastically increase since engineers only need to run their local Vite dev server and can stream other dependencies from a remote environment.

CI/CD Pipelines: Releasing code shifts from complex, coordinated operations to a simple, standard file upload to a static CDN bucket.

Operational Training: Frontend engineers will need to follow strict governance regarding shared dependencies and error handling practices (Error Boundaries) to maintain a highly resilient web ecosystem.

# Notes

Technical Spike and findings: https://github.com/String-sg/teacher-workspace/issues/194
