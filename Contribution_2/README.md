# Contribution [2]: [Add deregistration for providers with Cloud API]

**Contribution Number:** [2]  
**Student:** [John Ortega]  
**Issue:** [Implement deregistration for providers with Cloud API](https://github.com/the-momentum/open-wearables/issues/682)  
**Status:** [Phase III] [In Progress]

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

The one problem I faced was setting up the backend because I was missing some environment variables. To overcome this problem I ended up looking at the [documentation](https://openwearables.io/docs/quickstart#test-the-api)

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
- **My findings:**
   - Dashboard with the ability to add users and connect them with providers via OAuth

---

## Solution Approach

### Analysis

The root cause of this issue is that the OAuth classes of most providers, doesn't override the deregister_user method of the base class BaseOAuth template. However majority of these providers do in fact have a valid endpoint for revoking access tokens.

### Proposed Solution

I made a step by step plan for my solution:
- Look at all the providers that the app uses, and investigate which has a valid deregistering endpoint
- Override the ```deregister_user``` method in the classes within the ```oauth.py``` file of the providers that have valid deregistering endpoints
- Add unit tests for Polar provider's oauth testing as this uses an extra parameter and must be validated against

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
- [Adding unit tests for Polar deregistration](https://github.com/the-momentum/open-wearables/commit/ae026ca95f115d58fbc5cfa4990654f24e91370f)
- [Overriding `deregister_user` methods](https://github.com/the-momentum/open-wearables/commit/42818f2bc3df50eb6f916161b4b7a4991fe8050c)
- [Adding a proper commit message](https://github.com/the-momentum/open-wearables/commit/32b37b4b323648edc83a3ec5f89229c5b72c4269)
- [Final Commit, switching out legacy Strava deregistration endpoint](https://github.com/the-momentum/open-wearables/commit/1b3568998a55370f94da304bb6738b6e290d41c0)

**Review:**
- [x] Follows the project's existing code style and naming conventions in `base_oauth.py`
- [x] Abstract method is properly documented (docstring explaining expected behavior/return)
- [x] No provider-specific logic leaked into the base class
- [x] Tests added for each newly implemented provider
- [x] Existing Garmin deregistration flow still works (no regression)
- [x] PR description references issue #682 and #638 for context

**Evaluate:**

Run the existing/new test suite locally to confirm each provider's `deregister_user` works in isolation (mocked API calls). Manually test against a real (sandbox/test) account for at least one non-Garmin provider like Polar or Strava to confirm the actual API call succeeds and the user's local record is updated correctly. Confirm unsupported providers fail gracefully rather than throwing unhandled errors.

---

## Testing Strategy

### Unit Tests

- [x] Test case 1: [Test calling polar deregistration endpoint]
- [x] Test case 2: [Test HTTP error on polar deregistration being handled by caller]

### Integration Tests

- [x] Integration scenario 1: [Run `uv run pre-commit run --all-files` to ensure other integration tests pass]

### Manual Testing

- I made accounts for some of these other providers, then registered them to a user and deregistered them at the same time to test the new deregistration.

---

## Implementation Notes

### Week [1] Progress

- Update deregister_user signature to include user_id as a optional parameter
- Override `deregister_user` in Polar's oauth class, utilizing `provider_user_id` as required for polar deregistration

### Week [2] Progress

- Override `deregister_user` in the following remaining providers: FitBit, Oura, UltraHuman, Strava, Whoop.

### Week [3] Progress

- Update any calls to `deregister_user` throughout the codebase, so that it correctly passes the `provider_user_id`
- Add unit tests for deregistering a user within Polars OAuth test file

### Code Changes

- **Files modified:**
   1. backend/app/services/providers/fitbit/oauth.py
   2. backend/app/services/providers/garmin/oauth.py
   3. backend/app/services/providers/oura/oauth.py
   4. backend/app/services/providers/polar/oauth.py
   5. backend/app/services/providers/strava/oauth.py
   6. backend/app/services/providers/templates/base_oauth.py
   7. backend/app/services/providers/ultrahuman/oauth.py
   8. backend/app/services/providers/whoop/oauth.py
   9. backend/app/services/user_connection_service.py
   10. backend/app/services/user_service.py
   11. backend/tests/providers/polar/test_polar_oauth.py
- **Key commits:**
   - [Adding unit tests for Polar deregistration](https://github.com/the-momentum/open-wearables/commit/ae026ca95f115d58fbc5cfa4990654f24e91370f)
   - [Overriding `deregister_user` methods](https://github.com/the-momentum/open-wearables/commit/42818f2bc3df50eb6f916161b4b7a4991fe8050c)
   - [Adding a proper commit message](https://github.com/the-momentum/open-wearables/commit/32b37b4b323648edc83a3ec5f89229c5b72c4269)
   - [Final Commit, switching out legacy Strava deregistration endpoint](https://github.com/the-momentum/open-wearables/commit/1b3568998a55370f94da304bb6738b6e290d41c0)
- **Approach decisions:**
   - The most important approach I took was adding the `provider_user_id` parameter to the deregistration method, and I made this descision because Polar was one of the providers that required it for deregistering.

---

## Pull Request

**PR Link:** [GitHub PR URL](https://github.com/the-momentum/open-wearables/pull/1320)

**PR Description:**

Adds provider-side OAuth deregistration for Fitbit, Oura, Polar, Ultrahuman, and Whoop when a user disconnects or is deleted.

Previously only Garmin (and Strava) implemented `deregister_user`; other providers fell through to the base no-op. This wires real revoke/deregister API calls for the providers that support them, extends `deregister_user` with an optional `provider_user_id` (required by Polar’s `DELETE /v3/users/{user-id}`), and passes that id from the disconnect and user-delete call sites. Polar deregistration is covered with unit tests. Suunto remains on the base no-op pending a confirmed deauthorize API.

Closes #682

**Checklist**

**General**

- [x] My code follows the project's code style
- [x] I have performed a self-review of my code
- [x] I have added tests that prove my fix/feature works (if applicable)
- [x] New and existing tests pass locally
- [x] I have updated relevant documentation in `docs/` (or no docs update needed)

**Backend Changes**

<!-- If your PR includes backend changes, please verify: -->
You have to be in `backend` directory to make it work:
- [x] `uv run pre-commit run --all-files` passes

**Testing Instructions**

<!-- Describe how reviewers can test your changes -->

**Steps to test:**
1. From `backend/`, run: `uv run pytest tests/providers/polar/test_polar_oauth.py::TestPolarUserDeregistration -v`
2. (Optional) Connect a provider that implements deregistration (e.g. Fitbit/Whoop/Oura), then disconnect it from the user’s **Connected Providers** card (... -> Disconnect).
3. Confirm backend logs show a successful deregister, or a captured error if the provider call fails (local disconnect should still complete).

**Expected behavior:**
- Polar unit tests pass (success path, HTTP error propagation, missing `provider_user_id` raises `ValueError`).
- On disconnect/user delete, providers with `deregister_user` overrides call their revoke/deregister APIs with the access token (and Polar with `provider_user_id`).
- Deregistration failures are best-effort and do not block local disconnect/delete.

**Maintainer Feedback:**
- [07/01/2026]: Maintainer's initially wanted Polar deregistration to be implemented, but it was done by another contributor. However they followed up and let me know that I can work on it because the previous contributor broke the codebase because his implementation was vibe coded.
- [07/18/2026]: From my initial PR, the bot had told me the current endpoint I used for Strava deregistration was a legacy endpoint, and there was a new version I should use instead, which I ended up updating. [Commit Reference](https://github.com/the-momentum/open-wearables/pull/1320/changes/1b3568998a55370f94da304bb6738b6e290d41c0)
- [07/20/2026]: From my initial PR, the bot had told me I updated formatting of a markdown file, I reverted it to its original form just to not cause any confusion amongst the contributors. [Commit Reference](https://github.com/the-momentum/open-wearables/pull/1320/changes/fe3502416a55fe659bff2c11f7d7b719b773f66a)
- [07/20/2026]: Maintainer simply thanked me for the contribution, no problems occured

<!-- Statuses: Awaiting review / Iterating / Approved / Merged-->
**Status:** [Merged]

---

## Learnings & Reflections

### Technical Skills Gained

I learned the importance of Object Oriented Programming in massive applications, many of my personal project's have similar features to this app, however this app takes heavy use of OOP principles when compared to the apps I typically do. It made me appreciate OOP more and made me more aware of where I can integrate it in my codebase.

### Challenges Overcome

One of the providers didn't have any documentation on any existing endpoints for revoking access tokens, therefore I simply let the base class handle the providers specific deregistering method.

### What I'd Do Differently Next Time

I'd defintely communicate more with the collaborators of this project, as they had a discord that I was unaware of. This discord was mean't for contributors as well, and it would've served as a great help for me along this process.

---

## Resources Used

- [Deregistering on Garmin](https://github.com/the-momentum/open-wearables/pull/638)
- [Oura OAuth Documentation](https://cloud.ouraring.com/v2/docs)
- [Polar OAuth Documentation](https://polar.sh/docs/integrate/oauth2/connect)
