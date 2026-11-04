# Report Format

Use this structure for every workflow hardening review.

## 1. Summary Table (always first)

```
GitHub Actions Hardening — <workflow file(s) reviewed>

| Severity   | Count |
| ---------- | ----- |
| 🔴 CRITICAL | 1     |
| 🟠 HIGH     | 2     |
| 🟡 MEDIUM   | 1     |
| 🔵 LOW      | 1     |
| ⚪ INFO     | 0     |
```

If nothing was found: `No issues found. Checked: triggers, injection sinks, permissions, action
pinning, secret handling.`

## 2. Findings (grouped by issue type, not by file)

For each finding use a card:

```
### 🔴 CRITICAL — Script injection via PR title on a privileged trigger

File: .github/workflows/triage.yml  (line 14)
Trigger: pull_request_target

Offending code:
    - run: |
        echo "New PR: ${{ github.event.pull_request.title }}"

Risk: The interpolated title lets an attacker execute shell code in this job. `pull_request_target`
uses the base-repository context, but do not assume the job has write access or that secrets reach
the executed code: verify effective token permissions, exposed secrets, runner isolation or
persistence, and shared caches or other writable resources. State the concrete impact supported
by those checks and rate severity from that impact, not from the trigger alone.

Fix:
    - env:
        PR_TITLE: ${{ github.event.pull_request.title }}
      run: |
        printf '%s\n' "New PR: $PR_TITLE"

Confidence: High
```

## 3. Remediation Blocks

Every CRITICAL and HIGH finding includes a concrete before/after. Preserve the author's
indentation, step names, and surrounding structure — change only what fixes the issue, and add a
one-line comment explaining the change where it isn't obvious.

## 4. Closing Note

End with the explicit line:

> Review each change before committing. Nothing has been modified.

## Style Rules

* Quote the exact offending line and give its location.
* Explain risk in plain English — what an attacker actually does, not just the rule name.
* Per-finding confidence: High / Medium / Low.
* Do not rate untrusted-code execution as CRITICAL solely because the code is untrusted or the
  trigger is privileged. Verify effective token permissions, repository fork-PR settings, which
  credentials or secrets are actually exposed to executed code, runner isolation or persistence,
  and shared caches or other writable resources. Base severity on verified impact, and assess
  persistent/shared resource compromise independently of token permissions or trigger type.
