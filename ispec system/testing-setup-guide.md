# Testing Setup Guide
## How to configure testing for an .ispec-generated Python project

*This guide covers concrete setup steps for Python projects generated
from .ispec specifications. It uses pytest, SQLite, and bandit as the
example stack. Other choices are valid — the principles apply regardless.*

---

## Before You Start

You need:
- A working Python project generated from an .ispec file
- Python 3.10 or later
- pip or a virtual environment manager

You do not need prior testing experience. This guide starts from zero.

---

## Step 1 — Install the Testing Stack

```bash
pip install pytest
pip install pytest-cov
pip install pytest-asyncio
pip install httpx
pip install bandit
pip install safety
```

Or add to your `requirements-dev.txt`:

```
pytest>=7.0
pytest-cov>=4.0
pytest-asyncio>=0.21
httpx>=0.24
bandit>=1.7
safety>=2.3
```

Then install:
```bash
pip install -r requirements-dev.txt
```

---

## Step 2 — Project Structure

After the compiler generates your code, your project should look like this:

```
your-project/
  specs/
    client-tracker.ispec       ← spec files
  generated/
    python/
      client_tracker.py        ← compiler output — DO NOT EDIT
  tests/
    unit/
      test_operations.py       ← one file per domain
      test_queries.py
    integration/
      test_events.py
      test_cross_domain.py
    contract/
      test_contracts.py        ← spec contract verification
    e2e/
      test_workflows.py        ← full user journey tests
    conftest.py                ← shared fixtures
  pytest.ini                   ← pytest configuration
  .bandit                      ← security scan configuration
```

Create the test directories:

```bash
mkdir -p tests/unit tests/integration tests/contract tests/e2e
touch tests/conftest.py
touch tests/unit/test_operations.py
touch tests/unit/test_queries.py
touch tests/integration/test_events.py
touch tests/contract/test_contracts.py
touch tests/e2e/test_workflows.py
```

---

## Step 3 — Configure pytest

Create `pytest.ini` in your project root:

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
asyncio_mode = auto

# Coverage settings
addopts =
    --cov=generated
    --cov-report=term-missing
    --cov-report=html:coverage_report
    --cov-fail-under=80

# Test markers
markers =
    unit: unit tests — fast, isolated
    integration: integration tests — require database
    contract: contract tests — verify spec compliance
    e2e: end-to-end tests — full workflow
    security: security-related tests
    slow: tests that take more than 1 second
```

---

## Step 4 — Configure the Test Database

Create `tests/conftest.py`:

```python
import pytest
import sqlite3
import os
from datetime import datetime, timezone

# ── Database setup ────────────────────────────────────────────────────────────

@pytest.fixture(scope="session")
def db_path(tmp_path_factory):
    """
    Creates a temporary SQLite database for the test session.
    Each test session gets a fresh database.
    """
    db_dir = tmp_path_factory.mktemp("data")
    return str(db_dir / "test.db")


@pytest.fixture(scope="function")
def db(db_path):
    """
    Provides a clean database connection for each test.
    Rolls back all changes after the test completes.
    This ensures tests are isolated from each other.
    """
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    conn.execute("PRAGMA foreign_keys = ON")

    # Run schema setup
    from generated.python.client_tracker import create_schema
    create_schema(conn)

    yield conn

    # Rollback after every test — clean slate
    conn.rollback()
    conn.close()


# ── Common fixtures ───────────────────────────────────────────────────────────

@pytest.fixture
def now():
    """Current UTC timestamp for use in tests."""
    return datetime.now(timezone.utc)


@pytest.fixture
def test_user(db):
    """
    A basic user fixture. Most operations require an actor.
    Adjust fields to match your user-account domain.
    """
    return {
        "id": "user-001",
        "email": "test@example.com",
        "display_name": "Test User",
        "status": "Active"
    }


@pytest.fixture
def test_client(db, test_user):
    """A basic client fixture for client-tracker tests."""
    from generated.python.client_tracker import add_client
    return add_client(
        db=db,
        name="Acme Corp",
        email="contact@acme.com",
        created_by=test_user["id"]
    )


@pytest.fixture
def test_project(db, test_client, test_user):
    """A basic project fixture for client-tracker tests."""
    from generated.python.client_tracker import add_project
    return add_project(
        db=db,
        client_id=test_client["id"],
        name="Website Rebuild",
        created_by=test_user["id"]
    )
```

---

## Step 5 — Ask the AI to Generate Your Initial Test Suite

This is the most important step. Do not write tests by hand from scratch.
Give the AI the .ispec file and ask it to generate the test suite.

Use this prompt in Claude Code:

```
Read ispec-language-spec.md for the language rules.
Read specs/client-tracker.ispec for the specification.
Read tests/conftest.py for the available fixtures.
Read testing-setup-guide.md for the testing conventions.

Generate a complete test suite for the client-tracker domain.

For every operation in the [spec] section, generate:
  1. A test that verifies the happy path (valid inputs, correct output)
  2. A test for every requires condition (verify it enforces the precondition)
  3. A test for every failure listed in failures (verify the correct
     exception is raised under the stated condition)
  4. A contract test that verifies ensures conditions are met

For every query, generate:
  1. A test with matching records (verify correct results returned)
  2. A test with no matching records (verify empty result or NotFoundError)
  3. A test verifying sort order is correct

Use the fixtures in conftest.py where available.
Output to:
  tests/unit/test_operations.py
  tests/unit/test_queries.py
  tests/contract/test_contracts.py
```

---

## Step 6 — Understand the Generated Tests

The AI will generate tests that look like this. Read them and understand
what each one is doing.

**Example unit test — happy path:**

```python
import pytest
from generated.python.client_tracker import add_client, ClientStatus
from generated.python.exceptions import NameEmpty, EmailEmpty


class TestAddClient:

    @pytest.mark.unit
    def test_add_client_creates_record(self, db, test_user):
        """
        Spec: operation AddClient
        Verifies: Client created with correct fields (ensures clause)
        """
        client = add_client(
            db=db,
            name="New Corp",
            email="contact@newcorp.com",
            created_by=test_user["id"]
        )

        assert client["name"] == "New Corp"
        assert client["email"] == "contact@newcorp.com"
        assert client["status"] == ClientStatus.PROSPECT  # default
        assert client["created_by"] == test_user["id"]
        assert client["id"] is not None
        assert client["created_at"] is not None

    @pytest.mark.unit
    def test_add_client_default_status_is_prospect(self, db, test_user):
        """
        Spec: operation AddClient
        Verifies: default: Prospect on status field
        """
        client = add_client(
            db=db,
            name="New Corp",
            email="contact@newcorp.com",
            created_by=test_user["id"]
        )
        assert client["status"] == ClientStatus.PROSPECT
```

**Example unit test — failure conditions:**

```python
    @pytest.mark.unit
    def test_add_client_raises_name_empty_when_name_blank(self, db, test_user):
        """
        Spec: failures: NameEmpty: ValidationError
              condition: name is empty
        """
        with pytest.raises(NameEmpty):
            add_client(
                db=db,
                name="",
                email="contact@newcorp.com",
                created_by=test_user["id"]
            )

    @pytest.mark.unit
    def test_add_client_raises_name_empty_when_name_whitespace(self, db, test_user):
        """
        Spec: failures: NameEmpty: ValidationError
        Boundary: whitespace-only name should also be rejected
        """
        with pytest.raises(NameEmpty):
            add_client(
                db=db,
                name="   ",
                email="contact@newcorp.com",
                created_by=test_user["id"]
            )
```

**Example contract test — immutable history:**

```python
class TestClientContractImmutableHistory:

    @pytest.mark.contract
    def test_updating_client_preserves_history(self, db, test_client, test_user):
        """
        Spec: entity Client — history: immutable (default)
        Contract: updating status creates a new record, old record preserved
        """
        from generated.python.client_tracker import update_client_status

        original_id = test_client["id"]
        original_status = test_client["status"]

        update_client_status(
            db=db,
            client=test_client,
            status="Active",
            actor=test_user["id"]
        )

        # Current state should reflect the update
        from generated.python.client_tracker import get_client_by_id
        current = get_client_by_id(db=db, id=original_id)
        assert current["status"] == "Active"

        # History should contain the original record
        from generated.python.client_tracker import get_client_history
        history = get_client_history(db=db, client_id=original_id)
        assert len(history) >= 2
        assert any(h["status"] == original_status for h in history)
```

---

## Step 7 — Configure Security Scanning

Create `.bandit` in your project root:

```ini
[bandit]
targets = generated
exclude = tests, .venv
skips =
# Remove any skips — start with nothing excluded.
# Only add skips with documented justification.

[bandit.assert_used]
skips = tests/*
```

Run the security scan:

```bash
bandit -r generated/ -c .bandit
```

Check for dependency vulnerabilities:

```bash
safety check
```

**Interpreting bandit output:**

```
Severity: HIGH   — fix immediately, do not release with these
Severity: MEDIUM — fix before release, document if deferring
Severity: LOW    — review, fix where practical
```

Do not add items to the skip list to make the scan pass.
Fix the underlying issue.

---

## Step 8 — Run the .ispec Security Review

Before compiling any .ispec, run the security review.
Use this prompt in Claude Code:

```
Read ispec-language-spec.md for the language rules.
Read testing-strategy.md for the security review checklist.
Read specs/client-tracker.ispec for the specification.

Run the .ispec security review from testing-strategy.md against
this spec file.

For every operation, check:
  - Is there an authorization check in requires?
  - Are all inputs validated?
  - Does returns expose data the caller should not have?
  - Are failure conditions exhaustive?

For every query, check:
  - Can a user query another user's data?
  - Are where conditions necessary and sufficient?
  - Is pagination enforced?

For every event, check:
  - Is the source authenticated?
  - Does it carry sensitive data inappropriately?

Report every gap found. For each gap, state:
  - What is missing
  - Why it is a security concern
  - What change to the .ispec would fix it
```

Fix the spec before compiling. Fixing a spec costs nothing.
Fixing a deployed vulnerability costs everything.

---

## Step 9 — Set Up the Test Runner Script

Create `run_tests.sh` in your project root:

```bash
#!/bin/bash
set -e

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "Running .ispec project test suite"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

echo ""
echo "▸ Static security analysis..."
bandit -r generated/ -c .bandit -q
echo "  ✓ bandit passed"

echo ""
echo "▸ Dependency vulnerability check..."
safety check -q
echo "  ✓ safety passed"

echo ""
echo "▸ Contract tests..."
pytest tests/contract/ -v --no-header -m contract
echo "  ✓ contracts satisfied"

echo ""
echo "▸ Unit tests..."
pytest tests/unit/ -v --no-header -m unit
echo "  ✓ unit tests passed"

echo ""
echo "▸ Integration tests..."
pytest tests/integration/ -v --no-header -m integration
echo "  ✓ integration tests passed"

echo ""
echo "▸ Coverage report..."
pytest tests/ --cov=generated --cov-report=term-missing -q
echo ""

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "All gates passed."
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

Make it executable:

```bash
chmod +x run_tests.sh
```

Run it:

```bash
./run_tests.sh
```

---

## Step 10 — Connect Tests to Phase Gates

Open `phases.md`. For each phase, the test gate section should reference
specific test commands. Example:

```
Phase 1 — Core client management
  Test gate:
    ./run_tests.sh
    All contract tests pass
    All unit tests for AddClient, UpdateClientStatus,
    AddProject, UpdateProjectStatus pass
    Coverage >= 80% on generated/python/client_tracker.py
    bandit reports zero HIGH severity findings
```

The gate is specific. Not "tests pass" — which tests, what threshold,
what security level. Specific and measurable.

---

## Common Problems and Fixes

**Tests fail because the database schema is wrong**
The compiler may have generated a schema that does not match the test
fixtures. Check that the entity fields in the spec match the schema
the compiler generated. Fix the spec or the compiler prompt, recompile,
rerun.

**Contract test fails — history not preserved**
The compiler generated update-in-place code for an entity that should
be `history: immutable`. Add a compiler rule: entities without
`history: transient` must generate append-only storage. Recompile.

**bandit reports SQL injection risk**
The compiler generated string-formatted SQL queries. This is always
wrong. The compiler must generate parameterized queries. Add this as
a compiler rule. Recompile. Do not skip the bandit finding.

**Coverage is below threshold**
The AI-generated tests may not cover all branches. Ask the AI:
```
Read tests/unit/test_operations.py and the coverage report.
Identify which branches are not covered.
Generate additional tests to cover them.
```

**Tests pass locally but fail in CI**
Usually a timezone issue (use UTC everywhere), a missing environment
variable, or a path issue. Fix the environment, not the tests.

---

## What to Do When the AI Gets a Test Wrong

The AI will occasionally generate tests that are themselves incorrect —
testing the wrong thing, using wrong assertions, or missing edge cases.

Review every generated test and ask:
- Does this test actually verify what the spec says?
- Would this test fail if the code was wrong?
- Does this test cover the boundary conditions?

A test that always passes regardless of the code is worse than no test.
It creates false confidence.

If a generated test is wrong, fix it and note the pattern.
Feed the correction back to the compiler prompt so future tests
are generated correctly.

---

*Testing Setup Guide — Version 0.1 — March 2026*
*Part of the .ispec language framework*
*Example stack: pytest, SQLite, bandit*
*Copyright 2026 Christopher Rehm. All rights reserved.*
