# Calibrating Cognitive Gap

These scenarios illustrate response depth and scope. Paths, symbols, and outputs below are fictional; substitute evidence from the actual repository. They are examples for the agent, not executed tests.

## A focused concurrency fix

**Request:** “Use Cognitive Gap to fix this cache bug: cancelling one request also fails another request waiting for the same value. Preserve the cache API and keep the explanation brief.”

Trace how callers join an in-flight load and how cancellation reaches that shared work. If one caller's cancellation cancels the shared task, separate the lifetime of each waiter's await from the lifetime of the load. Keep a strong reference to a load that must survive after all waiters leave or invalidation detaches it from lookup. Keep result publication and pending-entry cleanup owned by the shared task, so a cancelled waiter cannot remove work another caller still needs. Follow the cache's existing invalidation and expiration contracts.

Verify a concrete interleaving: two callers join one load, one cancels, and the other still receives the value. Also check that a failed load is cleaned up and a later request can retry. Explain the ownership boundary that the repair establishes.

An appropriate handoff might be:

> Cancelling one request no longer cancels the load shared by other requests. `AsyncCache.get` in `src/cache.py` shields each caller's await; the cache retains unfinished loads, and each load owns result publication and cleanup. The cancellation and failed-load retry regressions pass.

Use that mechanism only if it matches the implementation, and report passing regressions only if they were actually run. A few grounded sentences can explain a consequential fix; add a diagram when the interleaving needs one, and match the user's request for brevity.

## A feature from intent through implementation

**Request:** “Use Cognitive Gap to implement team invitations. Help me understand the decisions as we build it so I can maintain it afterward.”

Inspect the existing account model, authorization, persistence, and invitation or notification behavior. Before editing, explain the intended flow and the material assumptions, such as who can invite someone and when an invitation expires. Follow established product behavior; ask only when an unresolved choice materially affects the feature.

Complete the implementation across the components the feature needs. Explain decisions when they matter: which component owns invitation state, how acceptance and revocation interact, and what happens if notification delivery fails. Update the explanation when tests or code inspection change the approach. Use actual symbols and a small flow or state diagram when that makes the behavior easier to follow.

Verify the relevant behavior and finish with maintenance entry points, such as where to change expiration or inspect an authorization failure. Keep understanding connected throughout the task, across the files and commits involved. Match the user's requested level of participation; exercises belong to an explicit learning request.

## A change across boundaries

**Request:** “Add retries to this API client, and help me understand what could go wrong.”

Inspect callers, error handling, request semantics, and any retry behavior already provided by dependencies. Establish which operations are safe to retry and whether timeouts or partial failures can leave the remote outcome unknown. Do not assume that every failed request had no side effect.

Explain the retry boundary through a concrete case: the server processes a request, but its response is lost. A second attempt could repeat the operation. Let the repository's contracts and the user's intent determine whether the solution retries only safe operations, uses an existing idempotency mechanism, or needs a decision before implementing a broader behavior.

Make the retry limit, delay, and failure outcome understandable. Verify the relevant success and failure behavior available in the test environment. A useful handoff connects a caller to the retry loop and final error, identifies the invariant preventing duplicate effects, and names any behavior that could not be tested.

Do not turn a retry request into an unrequested queueing system or a general reliability audit.

## Understanding an existing diff

**Request:** “Explain this cache PR. Why did they add a version to the cache key? Don't edit anything.”

Read the relevant diff and trace both cache writes and reads. If the implementation supports it, show how the same entity can have separate entries under two key versions and explain the effect on old data and cache misses. Inspect expiration or cleanup before claiming that old entries disappear.

Point to the key builder and its callers. Explain the compatibility boundary, the verification you inspected, and any uncertainty about rollout order. For example, coexistence of old and new application versions may matter, but do not claim there is a rollout defect without evidence.

Keep the working tree untouched. An explanation request is complete when the user has a grounded account of the change and any relevant findings; it does not require implementing your preferred alternative.

## A product decision hidden in a bug report

**Request:** “Fix our report. It drops rows with missing dates, but I need those rows.”

Inspect where rows are filtered and what the report promises. Preserve the explicit requirement to include undated rows. If existing conventions establish their placement, follow them and state the assumption briefly. If ordering has meaningful business consequences and no convention resolves it, ask where undated rows should appear. Continue independent investigation while waiting.

Explain how inclusion and ordering differ, implement the agreed behavior, and verify a mix of dated and undated rows. Avoid silently assigning today's date, which would invent data rather than include missing values.

## A guided learning request

**Request:** “Walk me through this race condition one step at a time, then let me try the fix.”

Respect the requested learning pace. Trace two concrete interleavings through the actual shared state and identify the assumption that fails. Give the user a focused next step and wait for their attempt because they asked to do the fix themselves. Do not implement the entire solution while they are working.

## A request outside the skill's scope

**Request:** “Change the footer text to © 2026. Just make the edit.”

Ordinary coding behavior is sufficient. This request supplies no need for a learning workflow. If the skill has been explicitly invoked anyway, honor the requested brevity: make the edit and provide a short confirmation, without introducing checkpoints or a tutorial.

## A review with an unstaged draft

**Request:** “Review the staged patch. I have a newer local fix in the same file; don't include it in the decision.”

Treat the index and working tree as separate artifacts. Read staged content and its unchanged callers before making claims about the candidate. Running the working-tree suite can be useful, but it does not verify a staged implementation that differs. For a reproduction, load the intended version into an isolated environment and say which version you executed. Preserve both the staged patch and the draft. Anchor findings to the version under review so a code pointer into the local draft does not obscure the defect.

## A green suite at a deployment boundary

**Request:** “The new service tests pass. Can old workers keep running during this rollout?”

Trace the producer, stored representation, and consumers across the versions that can coexist. A new writer and new reader may agree while an old reader fails, or queued old data may outlive the deployment. Explain the compatibility invariant using an actual value or event and distinguish local reproductions from observations of a deployed system. Name what the passing tests exercise and which boundary requires separate evidence. Offer a practical development direction within the requested scope; an assessment of readiness does not authorize deployment.

## A pull request summary for human reviewers

**Request:** “Use Cognitive Gap to draft a PR summary for this branch. Keep it focused on what reviewers need to own the change.”

Establish the branch's intended base and inspect the diff with its relevant callers. Connect the PR's intent to its observable mechanism: describe the primary state or data transition, identify the important invariant or contract, and explain the failure boundary reviewers should scrutinize. Scale the description to the change; a small fix can need only a few sentences.

Report verification supported by the branch and available execution evidence. A passing working-tree check describes that working-tree version; establish whether it matches the proposed PR. Point to the useful entry point and name a specific follow-up check when relevant. Return a title and description ready to paste into the PR.

## A security check at a multi-tenant boundary

**Request:** “Use Cognitive Gap to fix this invoice retrieval endpoint; users report 404s when loading valid invoice IDs. Explain how the fix preserves authorization.”

Trace the authenticated identity, authorization policy, lookup, and error mapping. Establish whether the caller is entitled to the invoice: an existing invoice can correctly return 404 to a caller who cannot access it. Identify where access is enforced, such as middleware, the query, or database row-level security, before selecting a fix.

If query scoping is the cause, derive tenant scope from the trusted authorization context and preserve the intended role and sharing rules. If access is already enforced elsewhere, preserve that policy while fixing the demonstrated defect. Explain the invariant through a concrete case: an authorized caller can retrieve the invoice, while a caller from another tenant without access is denied.

Verify both permitted and denied access using the endpoint's actual policy, including its intended error response. A passing retrieval test covers the success path; cross-tenant checks exercise the isolation boundary. Connect the handoff to the enforcement point and report the checks actually run.
