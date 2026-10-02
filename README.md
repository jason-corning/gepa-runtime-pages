# GEPA policy improvement runtime

[Explore the interactive diagram](https://jason-corning.github.io/gepa-runtime-pages/)

This repository hosts a point-in-time map of the backend flow for generating and evaluating candidate policy improvements.

## Pipeline at a glance

1. Weekly selection or an admin-initiated request queues an optimization job in Redis RQ. Both paths use the same core pipeline.
2. The worker builds training and validation examples from MongoDB policy and feedback data and StarRocks conversation transcripts.
3. DSPy GEPA evaluates candidates through Autoflow replay and model-based judging and reflection. S3 stores checkpoints so a run can resume.
4. Results are saved in MongoDB. Creating a PolicyDiff review is a separate admin action; saving a run does not publish a policy.

## Use the diagram

Select a node to read its context and connections. Use **PATH** to trace a directed route, **LENS** to inspect related nodes, and **Export** to save a copy.

## Scope

The diagram reflects a local source snapshot reviewed on October 1, 2026. It is an explanatory view, not a live status monitor or a release plan. Candidate evaluation is offline; it does not guarantee changes in live deflection or CSAT.

## Publishing

GitHub Pages serves [`index.html`](./index.html) from the root of `main`. To update the diagram, replace that file and commit it to `main`, then check the Pages deployment in **Actions**.
