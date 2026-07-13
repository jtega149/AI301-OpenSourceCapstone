# Contribution [2]: [Issue Title]

**Contribution Number:** [2]  
**Student:** [John Ortega]  
**Issue:** [Implement deregistration for providers with Cloud API](https://github.com/the-momentum/open-wearables/issues/682)  
**Status:** [Phase II] [In Progress]

---

## Why I Chose This Issue

This issue interests me because it revolves around backend infrastructure integrated with cloud services. Cloud computing is a passion of mine and integrating backends to cloud services plays a huge role in this field.

---

## Understanding the Issue

### Problem Description

What is missing is the ability to deregister other accounts that aren't just Garmin accounts. So far only Garmin has the ability to deregister, and its just through one specific method called "open-wearables". Other providers like Strava, Polar, etc.., also provide the ability to degister accounts via their API's, but the app isn't using that. 

### Expected Behavior

Garmin accounts, along with other accounts connected with OAuth, should all support deregistration.

### Current Behavior

There is no capability to deregister accounts connected via OAuth that aren't Garmin accounts

### Affected Components

The codebase isn't negatively impacted from this, but it's a helpful feature to add upon the app.

---

## Reproduction Process

### Environment Setup

The set up is as easy as cloning or forking the repo, and then setting environment variables.
Note:
    - Theres also the option to create your own admin account for the app itself on your local machine

### Steps to Reproduce

1. **Clone the repository:**
   ```bash
   git clone https://github.com/the-momentum/open-wearables.git
   cd open-wearables
   ```

2. **Configure environment variables:**
   
   **Backend configuration:**
   ```bash
   cp ./backend/config/.env.example ./backend/config/.env
   ```
   
   **Frontend configuration:**
   ```bash
   cp ./frontend/.env.example ./frontend/.env
   ```

3. **Start the application**
   
   **Using Docker (Recommended):**
   
   The easiest way to get started is with Docker Compose:
   ```bash
   docker compose up -d
   ```
   
   For local development setup without Docker take a look at [docs](https://openwearables.io/docs/quickstart#local-development-setup)

4. **Log in to the developer portal:**

   An admin account is automatically created on startup using the `ADMIN_EMAIL` and `ADMIN_PASSWORD` environment variables (defaults: `admin@admin.com` / `your-secure-password`).

   Open http://localhost:3000 to access the developer portal and create API keys.

5. **Seed sample data** (optional):
   If you want test users and sample activity data:
   ```bash
   make seed
   ```

   This will create:
   - Test users
   - Sample activity data for test users


6. **View API documentation:**

   Open http://localhost:8000/docs in your browser to explore the interactive Swagger UI.

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/jtega149/open-wearables]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The Cloud API currently only supports deregistering a user's Garmin connection, and even that was only added as a workaround via "open-wearables" (#638). Other providers (Polar, Strava, and possibly others) have deregistration support on their end, but the codebase has no abstracted way to call it. The fix is to add a `deregister_user` abstract method to the base OAuth class (`base_oauth.py`) so every provider is forced to implement its own version, then implement that method concretely for each provider that supports it.

**Match:** Look at how other abstract/shared OAuth methods are already structured in `base_oauth.py` (e.g. however `authenticate`, `refresh_token`, or similar lifecycle methods are defined and overridden per-provider). The Garmin implementation from #638 is the closest existing precedent, even though it was done through open-wearables rather than natively — so it's worth checking that PR/code to see what a deregistration call looks like end-to-end (API call, error handling, cleanup of local user records).

**Plan:**
1. Add an abstract method `deregister_user(self, user_id)` (or similar signature) to the base provider class in `base_oauth.py`.
2. Implement `deregister_user` for each concrete provider class that supports it (starting with Polar and Strava, since those were confirmed in the issue), each calling that provider's respective "revoke/deregister" API endpoint.
3. Wire up any calling code (e.g. an API route or service layer) that currently only handles Garmin deregistration so it works generically across providers via the new abstract method.
4. Handle providers that don't support deregistration gracefully (e.g. raise a clear `NotImplementedError` or return an informative response) rather than silently failing.
5. Add/update tests: unit tests for each provider's `deregister_user` implementation, and an integration test hitting the deregistration flow end-to-end.
6. Update any relevant docs or API reference for the deregistration endpoint.

**Implement:**
- [First Commit](https://github.com/jtega149/open-wearables)

**Review:**
- [ ] Follows the project's existing code style and naming conventions in `base_oauth.py`
- [ ] Abstract method is properly documented (docstring explaining expected behavior/return)
- [ ] No provider-specific logic leaked into the base class
- [ ] Tests added for each newly implemented provider
- [ ] Existing Garmin deregistration flow still works (no regression)
- [ ] PR description references issue #682 and #638 for context

**Evaluate:**

Run the existing/new test suite locally to confirm each provider's `deregister_user` works in isolation (mocked API calls). Manually test against a real (sandbox/test) account for at least one non-Garmin provider like Polar or Strava to confirm the actual API call succeeds and the user's local record is updated correctly. Confirm unsupported providers fail gracefully rather than throwing unhandled errors.

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
