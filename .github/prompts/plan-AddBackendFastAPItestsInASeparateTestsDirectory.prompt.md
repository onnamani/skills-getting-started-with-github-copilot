## Plan: Add backend FastAPI tests in a separate tests directory

TL;DR: Add a dedicated `tests/` directory, introduce `pytest`, and write FastAPI `TestClient`-based tests that exercise the current in-memory activity endpoints. The goal is to cover the API’s key behaviors—listing activities, successful signup, duplicate-signup rejection, participant removal, and not-found handling—while keeping the test suite cleanly separated from the frontend.

### Steps
1. Update `requirements.txt` to include `pytest`, and confirm the existing `pytest.ini` settings support discovery from a root `tests/` directory.
   - This is a required dependency for the backend test suite.
   - The plan should explicitly call out that `pytest` must be added to `requirements.txt` before running tests.
2. Create a new backend test file in a separate `tests/` directory, likely `tests/test_app.py`, using `fastapi.testclient.TestClient` against the app in `src/app.py`.
3. Write tests using the Arrange–Act–Assert pattern for the behaviors that matter most, with each test explicitly separating setup, request execution, and verification:
   - `GET /activities` returns the seeded activities
   - `POST /activities/{activity_name}/signup` adds a participant
   - duplicate signup returns `400`
   - `DELETE /activities/{activity_name}/participants/{email}` removes a participant
   - invalid activity names return `404`

   Suggested structure for each test:
   - Arrange: prepare the test state, such as selecting an activity and/or resetting the in-memory data store.
   - Act: call the FastAPI endpoint via `TestClient`.
   - Assert: verify the HTTP status code, response payload, and the resulting in-memory state for the relevant activity.
4. Add a reusable fixture or reset pattern so the global in-memory `activities` store starts from a known state for each test.
5. Run `pytest` from the workspace root, fix any failures, and confirm the new backend suite passes before moving on.

### Relevant files
- `src/app.py` — the FastAPI endpoints and current in-memory data model that will be tested
- `requirements.txt` — add the `pytest` dependency
- `pytest.ini` — confirm test discovery settings
- `tests/test_app.py` — new backend test file
- `tests/conftest.py` — optional shared fixture to reset state between tests

### Verification
1. Run `pytest` from the workspace root after adding the new test file and dependency.
2. Confirm the test suite passes with green output.
3. If any test reveals a contract mismatch, update the tests or app behavior intentionally, then rerun the suite.

### Decisions
- Use `TestClient` for backend API verification rather than browser testing.
- Keep tests in a separate `tests/` directory, as requested.
- Reset the in-memory activity store between tests because the app currently stores state in a global dictionary.

### Further considerations
1. Decide whether to add edge-case coverage such as max-participant limits or malformed email input.
2. If state resets become brittle, centralize them in a test fixture to avoid cross-test leakage.
