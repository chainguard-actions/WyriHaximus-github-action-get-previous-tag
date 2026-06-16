<!-- markdownlint-disable -->

# Hardening Report: WyriHaximus--github-action-get-previous-tag/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **WyriHaximus--github-action-get-previous-tag/v1.2.2** was hardened automatically. 0 finding(s) were identified and resolved across 1 iteration(s).

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in main.js: introduced `safeFallback` variable that strips all carriage return and newline characters from `process.env.INPUT_FALLBACK` using `.replace(/[\r\n]/g, '')` before writing to GITHUB_OUTPUT. This prevents an attacker from injecting arbitrary key=value pairs into the output context via the `fallback` input. Also updated the console.log statement to use the sanitized value.

