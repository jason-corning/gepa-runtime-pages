# GEPA policy improvement runtime

[Explore the interactive diagram](https://jason-corning.github.io/gepa-runtime-pages/)

This repository hosts a point-in-time map of how Forethought generates and evaluates candidate policy improvements. 

## Start here

A **policy** is the set of instructions this process is trying to improve. A **candidate** is a proposed version of that policy. **GEPA** is the optimizer that creates and tests candidates using examples from earlier conversations. The diagram shows the runtime around that optimizer: how work starts, where examples come from, how candidates are evaluated, and where results go.

The overall path is:

**Select a workflow → queue a job → build examples → test candidates → save a run → optionally create a review.**

## Follow the diagram, step by step

### 1. Start a job

**What happens:** A weekly scheduler selects eligible organizations and workflows. Alternatively, an admin can request a run through an API, which checks eligibility before queueing it. These are two entry paths into the same core pipeline.

**What it produces:** A job waiting in Redis RQ's `self_improvement_queue`. A queued job is a request to do work; it is not a new policy.

### 2. Pick up the job and identify issues

**What happens:** A GEPA worker takes the job from the queue, identifies issues to work on, and starts the data and optimization pipeline. The queue lets the request wait until a worker can process it.

**What it produces:** An active run with a selected workflow and issues for the next stage to examine.

### 3. Build examples from past conversations

**What happens:** The dataset builder combines policy, review, and issue information from MongoDB with conversation transcripts from StarRocks. It forms positive and negative examples and separates examples for training and validation. Training examples guide the search; validation examples help check how a candidate performs.

**What it produces:** A dataset that the optimizer can use to propose and assess policy changes. These examples come from recorded data, not from changing a live conversation.

### 4. Propose and evaluate policy candidates

**What happens:** The DSPy GEPA optimizer proposes candidate policy changes and scores them. Autoflow replay runs candidate responses against saved conversation examples. Model serving supplies the judging and reflection calls used during evaluation.

**What it produces:** Evaluation results that let the optimizer compare candidates. A stronger score in this offline evaluation is evidence about the test examples, not proof of a better live customer outcome.

### 5. Save progress and results

**What happens:** S3 stores pinned data and optimizer checkpoints so a run can save progress and resume. The resulting experiment and production run records are saved in MongoDB.

**What it produces:** A record of the run and its candidate results. Here, a *production run record* means a stored result from the production path; saving it does not publish the candidate policy.

### 6. Create a review separately

**What happens:** An admin can make a separate request to create a PolicyDiff review from a run. The review step sits after the saved result in the diagram.

**What it produces:** A reviewable policy difference. The diagram does not show a saved run automatically changing the live policy.

## Terms in the diagram

| Term | Meaning here |
| --- | --- |
| **Redis RQ** | The job queue that holds requests until a worker picks them up. |
| **Worker** | The background process that runs the pipeline for a queued job. |
| **MongoDB inputs** | Stored policy, review, and issue information used to build examples. |
| **StarRocks** | The source of conversation transcripts used by the dataset builder. |
| **Autoflow replay** | A way to test candidate responses against saved conversations. |
| **Model serving** | The model calls used for judging and reflection during evaluation. |
| **S3 checkpoint** | Saved data and optimizer state that support resuming a run. |
| **PolicyDiff** | A separate review of a proposed policy change. |

## Use the diagram

Select a node to read its context and connections. Use **PATH** to trace the arrows from one component to another, **LENS** to inspect related components, and **Export** to save a copy. To follow the main flow, start at **Weekly GEPA selection** or **On-demand trigger** and trace toward **MongoDB GEPA runs**.

## Scope

The diagram reflects a local source snapshot reviewed on October 1, 2026. It is an explanatory view, not a live status monitor or a release plan. Candidate evaluation is offline; it does not guarantee changes in live deflection or CSAT. The downloaded source snapshot did not include Git metadata, so its exact upstream revision could not be verified.

## Publishing

GitHub Pages serves [`index.html`](./index.html) from the root of `main`. To update the diagram, replace that file and commit it to `main`, then check the Pages deployment in **Actions**.
