---
name: sealed-secret-credentials
description: Use when creating, updating, rotating, or verifying real credentials such as OAuth2 client credentials or API keys as SealedSecrets in a Kubernetes GitOps repository. Covers reading the repository's existing conventions before acting, handling values without leaking them, verifying through the actual consumer, approval boundaries, rollback, and safe reporting. Do not use for plain Kubernetes Secret work that involves no sealing, for other secret management methods such as SOPS, for issuing or requesting the credentials themselves, or for changing configuration on the external service side.
---

# Handling real credentials as SealedSecrets

Two risks dominate this work: leaking the value, and reporting a rollout as
done when nothing has actually been verified. The order below exists to
prevent both.

## Before you start

Do not fill gaps by guessing. Seal nothing and commit nothing until you have
confirmed:

- The repository's `AGENTS.md` and any equivalent operating conventions.
- Where existing SealedSecrets live, how they are named, and how they are generated.
- The Secret contract for the target: namespace, name, key names, value format.
- The consumers that read this Secret (Deployment, Operator CR, monitoring config).
- The GitOps delivery path: which controller reconciles which path into which cluster.

When judgment is required, the repository's existing pattern wins. This skill
supplies decision criteria only, so commands, namespaces, and Secret names come
from the repository, not from here.

## When the credential is not in hand

Do not encrypt or commit an empty value, a dummy, or a placeholder as if it
were the real Secret. Not even with the intent to swap it later: a placeholder
decrypts and syncs cleanly, so nothing surfaces as broken until the consumer
fails.

Instead, state the following and hold until the real credential arrives:

- The Secret contract that is needed (namespace, name, keys, value format).
- The steps that will run once it is received.
- What is currently unfinished.

## Handling values

Keep credentials, OAuth tokens, authentication response bodies, and resource
identifier values out of all of these:

- Stdout and logs.
- Arguments passed in a way that lands in shell history.
- Plaintext manifests and temporary files that could be committed.
- Commit messages, PR descriptions, and issues.

An encrypted SealedSecret is a legitimate GitOps artifact. Anything holding a
decryptable value is not.

Never write a plaintext Secret to disk. If temporary input is unavoidable,
create it with owner-only permissions and delete it reliably when the work ends.

## Sealing

Follow the repository's existing method. If the Sealed Secrets public key for
the same cluster, or the cluster access needed to obtain it, cannot be
confirmed, stop rather than reaching for a different key or a different
mechanism. The wrong key means the controller cannot decrypt, and it makes the
failure much harder to diagnose.

Match the scope (strict, namespace-wide, cluster-wide) and the encryption
granularity of the existing SealedSecrets. Do not loosen scope to save
verification effort.

## Optional pre-check against the authentication endpoint

Check the credential against an external endpoint only where you have
permission to do so. When you do, use the exact input you will seal, and keep
the number of attempts minimal.

- Never store or display the token, the response body, or the value.
- Report only success, or a safe classification of the failure: OAuth error
  type, HTTP authorization error, DNS, TLS, connection, timeout.
- Do not conclude that the credential is wrong. The same symptoms come from
  permissions, scope, network path, and clock skew.

If permission is absent or unclear, skip the pre-check and let the verification
stage below carry the load.

## Verification

`Synced=True` on a `SealedSecret` confirms that the controller decrypted it and
produced a Secret. It is not evidence that the credential works. Verify in
stages:

1. Delivery: the commit has reached the cluster.
2. Decrypt and sync: the Secret exists with the expected keys. Check for key
   presence only, never print values.
3. Consumer behavior: whatever reads the Secret is working as expected.

Report the rollout as complete only after stage 3. What stage 3 looks like
depends on the consumer: an authenticated endpoint responds, a monitoring
scrape target goes up, a Pod runs without entering a restart loop. Pick
something observable from outside.

Existing Pods can still be holding the old Secret, so confirm the consumer
actually reloaded the value.

## Rotation

Keep the order:

1. Receive the new credential, already issued.
2. Seal it and let it reconcile.
3. Confirm the consumer works with the new credential.
4. Only then revoke the old credential.

Never revoke the old credential first. If the mechanism cannot have both
credentials valid at once, agree on the cutover downtime before starting.

## Approval boundaries

Do the following only within an explicitly approved scope:

- Updating a live Secret.
- Restarting Pods or forcing reconciliation.
- Revoking a credential.
- Any change to an external system.

When the request is limited to inspection, design, or committing, make no
external change. Propose the operation instead of running it.

## Rollback

Follow the repository's existing GitOps pattern. Identify the affected
resources first, then revert the smallest range that fixes the problem. Broad
deletion and credential reissue are not rollback mechanisms.

## Reporting

Report only:

- The files changed and the commit.
- Which verification stages ran and what they showed (delivery, decrypt and
  sync, consumer behavior).
- What is unfinished, and why.
- On failure, the cause only as far as a safe classification supports.

Keep credentials, tokens, authentication response bodies, resource identifier
values, and any other decryptable value out of the report. Do not stop at "the
Secret was created" — say how far verification got. If consumer behavior was
not confirmed, say that it was not.
