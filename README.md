# Contribution [#]: [Issue Title]

**Contribution Number:** [1 / 2 / 3] 

**Student:** [John Ortega]

**Issue:** [https://github.com/InnerWarden/innerwarden/issues/514] 

**Status:** [Phase I] [In Progress]

---

## Why I Chose This Issue

I chose this issue because it covers some concepts related to Cloud Engineering, which aligns with my future career ambitions. In addition this issue dealt with a common issue found in many cloud infrastructures, which is the handling of cloud attacks that compromise ones credentials.  

---

## Understanding the Issue

### Problem Description

The issue here is that there is no sigma rule that matches outbound network connections to a specific IP Address from processes that arent on some allowlist (or whitelisted addresses). 

### Expected Behavior

The expected behavior is for outbound network connections to be matched to a specific IP Address if its from processes that aren't coming from some allowlist of addresses. 

### Current Behavior

A legitimate cloud-init or monitoring agent can access the AWS cloud instance metadata service (IMDS) and so can an attacker enumerating cloud credentials. 

### Affected Components

AWS cloud credentials can be comprised in an attack like this.

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. Fork main branch repo or install via ```curl -fsSL https://innerwarden.com/install | sudo bash```
2. Configure AI api keys in the /etc/innerwarden/agent.env
3. Created a didcated innerwarden service user, and downloaded sensor + agent + ctl binaries. And it wrote a config to /etc/innerwarden
4. Can also use the following [instructions](https://github.com/jtega149/innerwarden/blob/main/CONTRIBUTING.md), very helpful for contributors

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/jtega149/innerwarden/commit/d0be1abcb325adc5e0fdce86db24f814c3ff2e9d]
- **My findings:** When you run the project locally, you'll launch a web-based security monitoring dashboard accessible through your browser. The application is designed to observe and report system activity, displaying security-related events, alerts, and operational information in a centralized interface. On a fresh setup, the dashboard may contain minimal data until the agent begins collecting activity.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
