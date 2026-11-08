# NexusAsset Cloud — Frontend Container Image Description

**Original artifact:** `Dockerfile` (converted to English description; executable/config source removed per approved scope 1).

## Purpose
Describes the container build for `nxa-frontend`.

## Inputs / Parameters
Build context, frontend source, manifests, safe public config.

## Outputs / Behavior
Container image for frontend/BFF runtime.

## Detailed Description
Based on Node.js 20 Alpine, working directory `/app`, copies package metadata, installs dependencies, copies source, exposes port 3000, and starts the current skeleton command. Production-ready build should use deterministic installs, compiled build output, non-root runtime, and production start behavior.

## NexusAsset Cloud Constraints
- Never bake secrets into images.
- Use optimized production build/start and health checks.
