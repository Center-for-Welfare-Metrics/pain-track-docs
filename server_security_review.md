# Security Review

Date: 2026-04-18

This document summarizes the vulnerabilities identified during a static security review of the repository.

## Scope

- Static source review only
- No live exploitation was performed
- No dependency CVE audit was performed
- Findings below are confirmed or high-confidence from code inspection

## Summary

The most serious problems are authorization failures that allow cross-tenant access to other users' data, missing protection on some guest mutation endpoints, non-expiring JWTs, plaintext storage of password-reset secrets, and missing abuse controls on public and auth endpoints.

## Fix Ease Scale

- `1/5`: Hard fix, likely needs broader redesign, data migration, or cross-cutting changes
- `2/5`: Moderately hard fix, touches multiple flows and needs careful regression testing
- `3/5`: Moderate fix, localized but still affects several components or deployment behavior
- `4/5`: Relatively easy fix, mostly localized code changes with limited side effects
- `5/5`: Very easy fix, small and low-risk change

## Findings

### 1. Critical: Broken object-level authorization on episodes

Fix ease: `4/5`

Affected files:

- `src/utils/episode/validate.ts`
- `src/routes/services/episode.ts`

Evidence:

- The ownership check is commented out in `EpisodePermissionValidate`.
- Authenticated episode routes rely on that validator.

Relevant code:

```ts
// if (episode.creator_id.toString() !== user_id) {
// throw new Error(NO_PERMISSION_ERROR);
// }
```

Impact:

- Any authenticated user who knows an `episode_id` can read, update, delete, and export another user's episode.

Affected endpoints:

- `GET /episode/:episode_id`
- `PATCH /episode/:episode_id`
- `DELETE /episode/:episode_id`
- `GET /episode/export/:episode_id`

Recommended fix:

- Restore the ownership check in `EpisodePermissionValidate`.
- Enforce authorization again inside the use case or database layer, not only in validators.

### 2. Critical: Broken object-level authorization on patient reads

Fix ease: `5/5`

Affected files:

- `src/useCases/patientUseCases/getPatientByIdUseCase/getPatientByIdUseCase.ts`
- `src/useCases/patientUseCases/getPatientByIdUseCase/getPatientByIdController.ts`

Evidence:

- The use case accepts `user_id` but returns the patient without checking `creator_id`.

Relevant code:

```ts
if (patient) {
  // if (patient.creator_id.toString() === user_id) {
  return patient;
  // }
}
```

Impact:

- Any authenticated user can fetch another user's patient record by id.

Affected endpoints:

- `GET /patient/:id`

Recommended fix:

- Enforce `patient.creator_id === user_id` before returning the record.
- Prefer querying by both `_id` and `creator_id` together.

### 3. Critical: Broken object-level authorization on episode listing by patient

Fix ease: `5/5`

Affected file:

- `src/useCases/episodeUseCases/listEpisodesUseCase/listEpisodesController.ts`

Evidence:

- The validator checks that the patient exists but the ownership check is commented out.

Relevant code:

```ts
// if (patient.creator_id.toString() !== user_id) {
//   throw new Error("Patient not found");
// }
```

Impact:

- Any authenticated user can enumerate all episodes for any patient id they know.

Affected endpoints:

- `GET /episode?patient_id=...`

Recommended fix:

- Restore the patient ownership check.
- Consider moving authorization to the episode listing use case or DB query.

### 4. Critical: Discussion APIs lack authorization checks

Fix ease: `2/5`

Affected files:

- `src/routes/services/discussion.ts`
- `src/useCases/discussionUseCases/listDiscussionUseCase/listDiscussionController.ts`
- `src/useCases/discussionUseCases/getDiscussionByIdUseCase/getDiscussionByIdController.ts`
- `src/useCases/discussionUseCases/updateDiscussionTextUseCase/updateDiscussionTextController.ts`
- `src/useCases/discussionUseCases/deleteDiscussionUseCase/deleteDiscussionController.ts`
- `src/implementations/mongoose/discussion.ts`

Evidence:

- Routes require authentication, but the validators only ensure that ids are syntactically valid.
- The use cases and mongoose implementation perform direct reads and writes without confirming resource ownership.

Impact:

- Any authenticated user can list discussions for another user's records.
- Any authenticated user can read, edit, create under, or soft-delete other users' discussions if they know the relevant ids.

Affected endpoints:

- `GET /discussion`
- `POST /discussion`
- `GET /discussion/:discussion_id`
- `PATCH /discussion/:discussion_id`
- `DELETE /discussion/:discussion_id`

Recommended fix:

- Validate that the caller owns the referenced patient, episode, track, or segment before any discussion action.
- Enforce ownership again in the write path, not just request validation.

### 5. High: Guest track and segment mutation paths can affect authenticated users' data

Fix ease: `4/5`

Affected files:

- `src/utils/track/validate.ts`
- `src/useCases/trackUseCases/guestUseCases/updateTrackGuestUseCase/updateTrackGuestController.ts`
- `src/useCases/trackUseCases/guestUseCases/deleteTrackGuestUseCase/deleteTrackGuestController.ts`
- `src/useCases/segmentUseCases/guestUseCases/createSegmentGuestUseCase/createSegmentGuestController.ts`
- `src/useCases/segmentUseCases/guestUseCases/updateSegmentGuestUseCase/updateSegementGuestController.ts`
- `src/useCases/segmentUseCases/guestUseCases/deleteSegmentGuestUseCase/deleteSegmentGuestController.ts`

Evidence:

- The guest-only check that should reject owned episodes is commented out in `TrackGuestPermissionValidateAsync`.

Relevant code:

```ts
// if (!!episode.creator_id?.toString()) {
//   throw new Error("Episode not found");
// }
```

Impact:

- If an attacker gets a `track_id` or `segment_id` belonging to a normal user episode, some guest endpoints can mutate that data without authentication.

Recommended fix:

- Restore the guest-only validation.
- Re-check ownership at the use case or persistence layer.

### 6. High: JWTs never expire

Fix ease: `3/5`

Affected file:

- `src/utils/jwt.ts`

Evidence:

- JWTs are signed without `expiresIn`.

Relevant code:

```ts
jwt.sign(content, process.env.APP_SECRET);
```

Impact:

- A stolen token remains valid indefinitely until the signing secret is rotated.

Recommended fix:

- Add expiration with `expiresIn`.
- Add token rotation and revocation handling for sensitive flows.

### 7. High: Password reset and account setup secrets are stored in plaintext

Fix ease: `2/5`

Affected files:

- `src/models/recovery-password.ts`
- `src/models/set-password-code.ts`
- `src/useCases/accountUseCases/confirmSetPasswordCode/confirmSetPasswordCodeController.ts`

Evidence:

- Recovery tokens and set-password secret tokens are stored as indexed plaintext values.
- The set-password secret token is returned directly to the client after code confirmation.

Impact:

- Any database read exposure can be turned into account takeover.
- Indexed plaintext secrets are easy to enumerate and query.

Recommended fix:

- Store only hashed reset/setup tokens.
- Compare hashes when validating tokens.
- Minimize exposure of secret material in API responses.

### 8. Medium: Account enumeration through distinct error messages

Fix ease: `4/5`

Affected files:

- `src/utils/validate.ts`
- `src/useCases/auhUseCases/SignUp/signUpController.ts`
- `src/useCases/accountUseCases/recoveryPassword/recoveryPasswordController.ts`
- `src/useCases/accountUseCases/recoveryPassword/recoveryPasswordUseCase.ts`

Evidence:

- Signup reveals `Email already in use`.
- Recovery password reveals `User not found`.
- Validation middleware returns structured field errors directly.

Impact:

- Attackers can enumerate valid email addresses.

Recommended fix:

- Use generic messages for signup and recovery flows.
- Avoid exposing account existence in validation responses.

### 9. Medium: Missing rate limiting and abuse controls

Fix ease: `3/5`

Affected files:

- `app-config/middlewares.ts`
- `src/routes/services/auth.ts`
- `src/routes/services/public.ts`
- `src/useCases/publicUseCases/generateResponseUseCase/generateResponseUseCase.ts`
- `src/utils/email/sendContactEmail.ts`

Evidence:

- No rate limiting or brute-force protection middleware is installed.
- Public endpoints can trigger OpenAI requests and SES email sends.

Impact:

- Increased risk of brute-force attacks on auth endpoints.
- Increased risk of cost abuse and email abuse on public endpoints.

Recommended fix:

- Add per-IP and per-account rate limits.
- Add stricter abuse controls for public AI and email endpoints.
- Consider quotas, CAPTCHA thresholds, and circuit breakers.

### 10. Medium: Google OAuth flow missing `state` protection

Fix ease: `3/5`

Affected files:

- `src/useCases/auhUseCases/googleOAuthGetUrl/googleOAuthGetUrlUseCase.ts`
- `src/useCases/auhUseCases/googleOAuthAutenticate/googleOAuthAutenticateController.ts`

Evidence:

- The generated authorization URL has no `state` parameter.
- The callback flow validates only the `code`.

Impact:

- The OAuth flow is exposed to login CSRF or login-swapping style attacks.

Recommended fix:

- Generate a per-request `state` value.
- Bind it to the user session or a signed nonce.
- Verify it on callback before exchanging the code.

### 11. Low: Static serving allows dotfiles

Fix ease: `5/5`

Affected file:

- `bin/server.ts`

Evidence:

```ts
app.use(express.static(__dirname, { dotfiles: "allow" }));
```

Impact:

- If sensitive dotfiles are ever placed under the served directory, they will be exposed over HTTP.

Current deployment note:

- In the current deployment model, this is mitigated in practice because the application runs in Google Cloud Run inside a Docker container, and the deployed image/layout currently prevents sensitive dotfiles from being exposed from the served path.
- This should still be treated as a code-level defense-in-depth issue, because the risk can reappear if the image contents, build output, or static serving path changes later.

Recommended fix:

- Remove `dotfiles: "allow"` unless there is a hard requirement.
- Restrict static serving to a dedicated public asset directory.

## Notes

- A raw password comparison exists in `src/implementations/mongoose/auth.ts`, but it appears to be dead code. The active login path uses bcrypt comparison in the sign-in use case, so it was not counted as an active vulnerability.

## Recommended Remediation Order

1. Fix authorization on patient, episode, discussion, and guest mutation paths.
2. Add JWT expiry and OAuth `state` validation.
3. Hash reset/setup tokens at rest.
4. Add rate limiting and generic auth/recovery error responses.
5. Tighten static file serving and review public endpoint abuse controls.