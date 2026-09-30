# Branching Strategy

This document defines the branch naming and release flow for MedicBD.

## 1. Default branch

- Main branch: main
- All release-ready code must merge into main through pull requests.

## 2. API-only release line

- Release branch pattern: release/v<semver>-api-only
- Example: release/v1.0.0-api-only
- Release tag pattern: v<semver>-api-only
- Example: v1.0.0-api-only

Recommended API-only flow:
1. Create release branch from main.
2. Apply only stabilization fixes (no new features).
3. Run CI and smoke tests.
4. Tag the release branch commit.
5. Merge back to main if patch changes were made.

## 3. Fullstack transition line

- Foundation branch (optional short-lived): epic/fullstack-foundation
- Long-running integration branch: feat/fullstack-mvp
- Feature branch pattern: feat/fullstack-<feature>
- Release branch pattern: release/fullstack-m<milestone> or release/fullstack-mvp

Recommended fullstack flow:
1. Create feature branch from feat/fullstack-mvp.
2. Merge small vertical slices (route + controller + view + test).
3. Keep API behavior under /api/v1 backward-compatible.
4. Cut milestone release branches from feat/fullstack-mvp.
5. Merge milestone branches to main after verification.

## 4. Hotfix branches

- Hotfix branch pattern: hotfix/<scope>-<short-description>
- Example: hotfix/api-auth-token-expiry

Hotfix flow:
1. Branch from main.
2. Apply minimal fix and tests.
3. Merge to main.
4. Cherry-pick to active release branch if needed.

## 5. Naming rules

- Use lowercase letters, numbers, and hyphens.
- Keep names short but descriptive.
- Prefer semantic versioning in release branch and tag names.
- Keep tag and release branch version aligned.

## 6. Examples

- release/v1.0.0-api-only
- v1.0.0-api-only
- feat/fullstack-mvp
- feat/fullstack-doctor-index
- release/fullstack-m1
- hotfix/api-login-rate-limit

