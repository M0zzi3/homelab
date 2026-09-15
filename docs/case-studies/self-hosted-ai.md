# Self-hosted AI

## Problem

The AI server has limited GPU memory, system memory and storage. It must support persistent memory workloads, local embeddings and Immich machine learning without allowing one model to consume all available capacity.

## Design

LM Studio provides local model serving. Honcho uses a small Qwen model for memory derivation, summaries and routine reasoning, together with Nomic embeddings. More demanding tasks are routed through JARVIS to stronger external providers.

Immich machine learning shares the AI host, so model file size alone is not a capacity plan. Context length, KV cache, concurrency, GPU memory, system RAM and stored model variants all matter.

## Operating approach

1. Keep the persistent local workload small and predictable.
2. Load models explicitly and verify their API identifiers.
3. Test configuration changes through the consuming application.
4. Monitor GPU, RAM and disk pressure.
5. Preserve headroom for Immich and future experiments.
6. Route tasks elsewhere when they exceed the local model's reliable capability.

## Failure lesson

A model can answer normal chat requests while failing when an application expects tool use or structured behaviour. Successful loading therefore does not prove compatibility. The useful test is an end-to-end request through Honcho or the relevant workflow.

## Evidence to add

- sanitised VM and model allocation;
- GPU observations under normal and competing workloads;
- a real Honcho search and reasoning check;
- a small latency and reliability comparison;
- local-inference failure and fallback behaviour.

## What I learned

- the best local model is the one that remains reliable beside other workloads;
- routing is often better than forcing one model to handle everything;
- capacity headroom is operational value rather than wasted hardware;
- application-level verification matters more than a successful model load.

## Professional improvements

A professional platform would add central metrics, request tracing, workload-specific evaluations, privacy classification, automated fallback and explicit inference lifecycle management.

## Skills demonstrated

Local model serving, embeddings, AI infrastructure, GPU capacity planning, model routing, API verification and reliability testing.
