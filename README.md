# Contribution 1: Add Sigma Detection for Unauthorized Cloud Metadata Service Access

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

Or Use:
```bash
# 1. Fork + clone
gh repo fork InnerWarden/innerwarden --clone --remote
cd innerwarden

# 2. Toolchain (rustup picks the pinned version from rust-toolchain.toml if present)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup component add rustfmt clippy

# 3. Build + test
make test       # all unit + integration tests
make build      # debug binaries for sensor / agent / ctl

# 4. Try the dashboard locally
cargo run -p innerwarden-agent -- --data-dir ./data --dashboard
# open http://127.0.0.1:7378 (default admin password: innerwarden)
```

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/jtega149/innerwarden/commit/d0be1abcb325adc5e0fdce86db24f814c3ff2e9d]
- **My findings:** When you run the project locally, you'll launch a web-based security monitoring dashboard accessible through your browser. The application is designed to observe and report system activity, displaying security-related events, alerts, and operational information in a centralized interface. On a fresh setup, the dashboard may contain minimal data until the agent begins collecting activity.

---

## Solution Approach

### Analysis

The issue is that the system is prone to attack via scraping of the cloud instance metadata service (IMDS), because any process that hits this endpoint is either a valid cloud init / monitoring agent (small allowlist), or an attacker enumerating cloud credentials. The issue here is that there is no Sigma rule that matches outbound network connections to `169.254.169.254` from processes that aren't on a small allowlist of legit metadata clients.

### Proposed Solution

1. Review current Sigma rule structure and detection patterns.
2. Create a rule that matches `169.254.169.254` access from non-allowlisted processes.
3. Verify alerts fire for unauthorized processes and are suppressed for approved clients.
4. Confirm the rule loads correctly and includes the proper severity and MITRE mapping.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:**

Add a new Sigma rule that detects processes accessing the cloud instance metadata service (169.254.169.254) unless the process is part of an approved allowlist of legitimate metadata clients.

**Match:** 

Use existing Linux Sigma rules in rules/sigma/ as references for rule structure, detection syntax, severity levels, and MITRE ATT&CK mappings.

**Plan:**

1. Review existing Sigma rules and confirm the supported field names (destination_ip, process_comm).
2. Create rules/sigma/network/lnx_imds_access_from_non_metadata_client.yml with detection logic for IMDS access from non-allowlisted processes.
3. Validate that the rule is loaded by the Sigma rule loader and test against sample events.

**Implement:** [Link to your branch/commits as you work]

Link: https://github.com/jtega149/innerwarden

**Review:**

- [ ] Rule follows existing project conventions and Sigma format.
- [ ] Severity is set to Medium.
- [ ] MITRE ATT&CK mapping is T1552.005.
- [ ] Allowlist contains all specified metadata clients.
- [ ] Rule file is placed in the correct directory and loads successfully.

**Evaluate:** Run the test cases provided on the app itself, and then try to access the endpoint from a process that isn't on the allowlist

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: Test if any processes that don't come from allowlist are redirected
- [ ] Test case 2: Test to see if `169.254.169.254` recieves any process in allowlist 
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]
- Ran ```make test``` command provided by app, 84 unit tests fail

---

## Implementation Notes

### Week [1] Progress

- Created the YML file within the rules/sigma/network
- Drafted out code for the YML file
- Some unit tests aren't working currently, will need to see if I configured project correctly

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
    - rules/sigma/network/lnx_imds_access_from_non_metadata_client.yml
    - rules/sigma/network/Instructions.md
- **Key commits:** [Links to important commits]
    - [Commit 1](https://github.com/jtega149/innerwarden/commit/5035b2e5b6045e3a5fe13b6d8188a893d5999463)
    - [Commit 2](https://github.com/jtega149/innerwarden/commit/a469af185449a2125527a95ef89063b70041da05)
- **Approach decisions:** [Why you chose certain approaches]
    - Rule must live here in order to correctly operate as other rules
    - Instructions MD so claude and agents have context of problem I am working on
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
