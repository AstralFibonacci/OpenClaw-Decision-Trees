---
id: error-fallback
trigger: "After any API call returns a non-200 response"
---

## Inputs

- **error_code** — HTTP status code returned
- **error_message** — error body or message string
- **attempt_count** — how many times this specific call has been attempted
- **max_retries** — maximum retries allowed for this call (default: 3)

## Tree

START → evaluate_error_type

evaluate_error_type:
  error_code == 429 → handle_rate_limit
  error_code == 401 → EXIT: abort_auth_error
  error_code == 403 → EXIT: abort_permission_error
  error_code == 404 → EXIT: abort_not_found
  error_code >= 500 → handle_server_error
  error_code >= 400 → EXIT: abort_client_error
  default → EXIT: abort_unknown_error

handle_rate_limit:
  attempt_count < max_retries → EXIT: retry_with_backoff
  attempt_count >= max_retries → EXIT: abort_rate_limit_exceeded

handle_server_error:
  attempt_count < max_retries → EXIT: retry_immediate
  attempt_count >= max_retries → EXIT: abort_server_error

## Exits

**retry_with_backoff**
- Action: Wait (30 × attempt_count) seconds, increment attempt_count, retry the call
- Report: "Rate limited (429). Waiting Xs before retry (attempt {attempt_count}/{max_retries})"
- Next: Re-enter this tree after retry completes

**retry_immediate**
- Action: Retry the call immediately, increment attempt_count
- Report: "Server error ({error_code}). Retrying immediately (attempt {attempt_count}/{max_retries})"
- Next: Re-enter this tree after retry completes

**abort_rate_limit_exceeded**
- Action: Stop execution, surface to user
- Report: "Rate limit exceeded after {max_retries} attempts. Manual intervention required."
- Next: Halt skill execution

**abort_server_error**
- Action: Stop execution, surface to user
- Report: "Server error ({error_code}) persists after {max_retries} attempts: {error_message}"
- Next: Halt skill execution

**abort_auth_error**
- Action: Stop execution, surface to user
- Report: "Authentication failed (401). Check API credentials."
- Next: Halt skill execution

**abort_permission_error**
- Action: Stop execution, surface to user
- Report: "Permission denied (403). Check API key scopes or account access."
- Next: Halt skill execution

**abort_not_found**
- Action: Stop execution, surface to user
- Report: "Resource not found (404): {error_message}"
- Next: Halt skill execution, note the missing resource for user review

**abort_client_error**
- Action: Stop execution, surface to user
- Report: "Client error ({error_code}): {error_message}. Check request parameters."
- Next: Halt skill execution

**abort_unknown_error**
- Action: Stop execution, surface to user
- Report: "Unexpected error ({error_code}): {error_message}"
- Next: Halt skill execution
