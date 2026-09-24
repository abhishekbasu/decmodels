# Laya Use-Case Suggestions

Based on `example.py` and the installed Laya implementation, think of Laya as **a semantic decision layer—not a chatbot**.

You give it context and define the decisions you need:

- **`choice`** → select a category or route.
- **`score`** → rate something against ordered criteria.
- **`noul`** → estimate the probability of a yes/no answer.

Its sweet spot is **turning short, messy text into structured signals that drive a workflow**.

## 1. Support triage and escalation

**The most natural extension of `example.py`.**

Input: a customer message, optionally with relevant account context.

Decisions:
- Which team should handle it?
- What does the customer want?
- Is there a deadline or blocking issue?
- Is a human needed?
- Does the customer threaten to cancel?

Your application then assigns the queue, prioritizes the ticket, or alerts an account manager.

**Important distinction:** the example’s `churn_risk` question detects a *threat to cancel*. Its output is **not the probability that this customer will actually churn**. Predicting churn requires historical behavior and outcome validation.

## 2. Routing requests between tools, agents, and LLMs

**Probably the most interesting infrastructure use case.**

Input: a user request and a compact description of the current task.

Decisions:
- Is this coding, research, writing, or account support?
- Does it require private data or external search?
- Is it simple enough for a smaller model?
- Does it have financial or other sensitive consequences?

Example workflow:

```text
User request
    ↓
Laya: domain + difficulty + needs_tools + sensitivity
    ↓
Deterministic routing policy
    ├── FAQ retrieval
    ├── Small LLM
    ├── Larger reasoning model
    └── Human review
```

This could reduce expensive LLM calls, **if measured routing quality and latency justify the extra step**. Estimated difficulty alone is not a guarantee that a smaller model can solve the task.

Laya’s `Router` in the example selects a **Laya checkpoint**. Routing to your own tools or LLMs would be application logic built around its answers.

## 3. Agent observability and review queues

Input: a short agent action or trace segment, including the request, tool call, and result.

Decisions:
- Did the action appear successful?
- Is the agent stuck or repeating itself?
- Did it attempt a risky operation?
- Does the result need review?

Useful for identifying:
- Repeated tool failures.
- Destructive actions needing approval.
- Unsupported success claims.
- Conversations that should be escalated.

**Use it to prioritize review, not to grant permissions.** Actual authorization and tool restrictions should remain deterministic.

## 4. Customer feedback and product intelligence

Input: reviews, survey responses, cancellation notes, or support snippets.

Decisions:
- Bug report, feature request, usability issue, or pricing complaint?
- Which product area?
- How severe is the frustration?
- Is a competitor mentioned?
- Is the problem blocking adoption?

The output feeds dashboards and review queues without requiring free-form summaries for every item.

This is a particularly good initial project because **errors are usually reversible and you can evaluate the labels directly**.

## 5. Inbox and sales-lead triage

Input: inbound emails, contact forms, or demo requests.

Decisions:
- Sales, billing, recruiting, support, or spam?
- Does it need a reply?
- Is there explicit buying intent?
- Is there a concrete deadline?
- Which team should follow up?

Example: prioritize “We need a demo before Friday’s procurement meeting” above a generic pricing inquiry.

Keep the questions grounded in visible evidence. “Does the sender request a demo?” is easier to validate than “Will this deal close?”

## 6. Security and content-review assistance

Input: a suspicious email, reported post, or compact alert summary.

Decisions:
- Does it resemble phishing?
- Does it contain threats or harassment?
- Is credential compromise suggested?
- How urgent is review?

Useful as **another detection signal or queue-ranking mechanism**, not a sole security boundary. Adversarial inputs and false negatives make autonomous blocking or dismissal risky without additional controls.

## 7. Document exception routing

Input: **already extracted** document fields plus relevant context.

Examples:
- Invoice disputes.
- Expense explanations.
- Procurement requests.
- Claims intake.

Laya can classify the issue and suggest a review route. Use code or database queries for exact arithmetic, duplicate detection, and purchase-order matching; use Laya for interpreting narrative ambiguity.

## Where not to use it

- Writing replies, summaries, or code.
- Extracting arbitrary names, dates, or amounts through this decision API.
- Reading entire contracts or long conversation histories in one call.
- Verifying facts that are absent from the supplied context.
- Autonomously approving payments, refunds, or other consequential actions.

The installed router documents relatively short context windows: **512 tokens for the English checkpoint and 1,024 for the other two**, with question text also consuming input space. Keep context focused and watch for truncation.

## What to build first

**For a product feature:** support triage or feedback classification.

**For AI infrastructure:** an LLM/tool router with a human-review fallback.

Start with a few hundred representative labeled examples, compare against simple rules and your current approach, and measure **per-category errors, latency, and downstream cost**. Treat probability outputs as model confidence—not calibrated real-world probabilities until tested.

The installed package already includes presets in `laya.presets` for support triage, email, moderation, guardrails, and model routing. Those provide starting schemas, **not evidence that each workflow is production-ready**.
