# Architect Playbook

> A practical operating system for architects to manage ownership, flow, and system clarity.

## 0. Role Anchor

> **I design systems, not execute tasks.**  
> **I create clarity, ownership, and flow.**

**Decision question:**
> Does this improve the system or just solve it once?

## 1. Core Principles

### 1.1 Ownership > Effort
- No owner → no progress
- Always establish ownership

### 1.2 Process > Heroics
- Avoid one-off help
- No "quick fixes" that bypass system

### 1.3 Enable > Do
- Guide ✔
- Clarify ✔
- Do it yourself ❌

### 1.4 Clarity > Speed
- Especially under pressure
- Especially from leadership

### 1.5 System > Local Optimization
- Solve root causes
- Avoid patching symptoms

## 2. Decision Patterns

### Status request
→ Redirect to owner

### Incident / failure
→ Engage only if you are the owner

### "Create ticket / ask DevOps"
→ Route to process

### Blocked work (review, dependency)
→ Highlight blocker + assign owner

### VP / stakeholder pressure
→ Acknowledge urgency → explain risk → propose sustainable path

### "They don’t know how"
→ Enable, don’t execute

### No-show / ignored meetings
→ Add impact → question priority

## 3. Communication Patterns

### Ownership redirect
> This is owned by `<Team X>`, they can provide the latest update.

### Boundary setting
> I’m not involved in `<area>`, but happy to support on architecture.

### Process enforcement
> This should go through the standard process via `<link>`.

### Risk framing (for leadership)
> If we approach it this way, it may create `<risk>`.  
> The better approach is `<X>`.

### Enablement
> I can help clarify the process so the team can handle it directly.

### Blocker escalation
> This is currently blocking `<impact>`. We need an owner to move it forward.

### Priority alignment
> If this is not a priority, we can adjust timelines accordingly.

## 4. Anti-Patterns

If you see yourself doing this — stop:

- Creating tickets for others
- Chasing teams for updates
- Tracking delivery status
- Debugging runtime issues
- Owning meetings without an owner

> This is delivery management, not architecture.

## 5. Operating Model

### Ownership
- Every system has a DRI (Directly Responsible Individual)
- No owner = problem

### Flow
- Where is the blocker?
- Who removes it?

### Architecture
- Boundaries (DDD)
- Contracts between systems

### Governance
- Rules
- Processes
- ADRs

### Interaction Model
- Teams communicate directly
- No routing through architect

## 6. Mental Models

- **If I do it once — I own it forever**
- **Short-term speed vs long-term system**
- **No owner = no progress**
- **People optimize for least resistance**

> If you are the easiest path — you become the system.

## 7. Escalation Ladder

1. Redirect
2. Add impact
3. Question priority
4. Escalate to owner / leadership
5. Formalize process

## 8. Scaling This

To reduce chaos long-term:

- Introduce DRI / RACI explicitly
- Document processes (DevOps, APIM, support)
- Remove yourself as dependency
- Enforce: **"no owner = no meeting"**

## 9. Signals You're Doing It Right

- You are pulled less into operational noise
- Teams use processes instead of you
- Conversations shift from status → decisions
- You have time for architecture and strategy

## Final Formula

> **Architect = Clarity + Ownership + Boundaries**

## Final Insight

You are not the person who solves problems.

You are the person who:
> **builds a system where problems are solved without you**
