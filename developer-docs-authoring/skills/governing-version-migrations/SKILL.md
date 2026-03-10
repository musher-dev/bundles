---
name: governing-version-migrations
version: 1.0.0
description: Govern API versioning documentation including changelog formats, deprecation notice standards, migration guide anatomy, version selector UX, multi-version documentation management, and version-pinned URL strategies. Use when managing versioned docs, writing changelogs, creating migration guides, or designing version selectors. Triggered by: versioning, changelog, deprecation notice, migration guide, version selector, multi-version docs, API versioning, breaking changes, version pinning, upgrade guide, version documentation, release notes, semver.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Governing Version Migrations

## Purpose

Define standards and processes for documenting API versions, communicating changes, guiding users through migrations, and managing multi-version documentation sites. As products evolve, versioning documentation becomes critical infrastructure — poorly managed version transitions drive churn, increase support load, and erode developer trust.

This skill handles version-related documentation governance. Page freshness lifecycle is handled by governing-docs-lifecycle. Individual page quality is handled by auditing-content-quality.

## Changelog Standards

### Format

Every product with an API or SDK must maintain a changelog following the Keep a Changelog convention:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [2.3.0] - 2026-03-01

### Added
- New `batch` endpoint for bulk operations ([#1234])
- Support for webhook signature verification via `X-Signature-256` header

### Changed
- `GET /users` now returns paginated results (default: 50 per page)
- Rate limit headers use `RateLimit-*` prefix (previously `X-RateLimit-*`)

### Deprecated
- `GET /users/all` — use `GET /users?limit=0` instead. Removal: v3.0.0

### Fixed
- `POST /webhooks` no longer returns 500 when `url` contains query parameters

## [2.2.1] - 2026-02-15

### Fixed
- Corrected `expires_at` timestamp format from Unix to ISO 8601
```

### Changelog Entry Rules

| Rule | Correct | Incorrect |
|------|---------|-----------|
| Start with affected resource | "`GET /users` now returns..." | "We updated the users endpoint..." |
| Include the version where change takes effect | "Removal: v3.0.0" | "Will be removed in a future version" |
| Link to PR or issue | "([#1234])" | No reference |
| Separate breaking from non-breaking | Use ### sections | Mixed in one list |
| One change per bullet | Single behavior change | "Updated auth and added pagination and fixed caching" |

### Changelog Categories

| Category | Definition | Example |
|----------|-----------|---------|
| **Added** | New functionality | New endpoint, new parameter, new SDK method |
| **Changed** | Non-breaking behavior changes | Default value change, response format update |
| **Deprecated** | Functionality marked for future removal | Endpoint slated for removal with timeline |
| **Removed** | Previously deprecated functionality removed | Endpoint no longer available |
| **Fixed** | Bug fixes | Corrected error response, fixed race condition |
| **Security** | Vulnerability patches | Fixed authentication bypass, updated dependencies |

### Cadence

| Release Type | Changelog Update | Notification |
|-------------|-----------------|--------------|
| Major (breaking) | Same day as release | Email + banner + migration guide |
| Minor (features) | Same day as release | Banner + changelog page |
| Patch (fixes) | Same day as release | Changelog page only |
| Security | Same day as patch | Email + banner + advisory |

## Deprecation Notice Standards

### Deprecation Timeline

Every deprecation must communicate three things: what's deprecated, what replaces it, and when it's removed.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  DEPRECATED  │────▶│  SUNSET      │────▶│  REMOVED     │
│  Announced   │     │  Period      │     │  Gone        │
│              │     │              │     │              │
│  Feature     │     │  Warnings in │     │  410 Gone    │
│  still works │     │  responses   │     │  response    │
│  Docs banner │     │  Docs banner │     │  Redirect to │
│              │     │  + countdown │     │  migration   │
└──────────────┘     └──────────────┘     └──────────────┘
     Day 0            Day 0 + 30           Removal Date
```

### Minimum Deprecation Periods

| Change Type | Minimum Notice | Recommended Notice |
|------------|---------------|-------------------|
| Endpoint removal | 6 months | 12 months |
| Parameter removal | 3 months | 6 months |
| Behavior change (breaking) | 3 months | 6 months |
| Authentication method change | 12 months | 18 months |
| SDK major version EOL | 6 months | 12 months |

### Deprecation Notice Template

```markdown
> **Deprecated**: `GET /users/all` is deprecated and will be removed on 2026-09-01.
>
> **Migration**: Use `GET /users?limit=0` instead. See the [migration guide](/migrations/v2-to-v3#users-all).
>
> **Timeline**:
> - 2026-03-01: Deprecated (this notice). Endpoint continues to work.
> - 2026-06-01: Sunset warning headers added to responses.
> - 2026-09-01: Endpoint removed. Returns `410 Gone`.
```

### Deprecation in API Responses

| Header | Value | Purpose |
|--------|-------|---------|
| `Deprecation` | `true` | RFC 8594 standard deprecation signal |
| `Sunset` | `Sat, 01 Sep 2026 00:00:00 GMT` | RFC 8594 removal date |
| `Link` | `</migrations/v2-to-v3>; rel="deprecation"` | Link to migration guide |

### Deprecation Banner Placement

| Location | When | Content |
|----------|------|---------|
| API Reference page | From deprecation date | Yellow banner with timeline and migration link |
| Changelog | On deprecation date | Entry in Deprecated section |
| SDK method docs | From deprecation date | `@deprecated` annotation + inline notice |
| Dashboard/Console | If applicable | Banner when user calls deprecated endpoint |

## Migration Guide Anatomy

### Structure

Every breaking change between major versions requires a dedicated migration guide:

```markdown
# Migrate from v2 to v3

Estimated time: 30-60 minutes

## Overview

v3 introduces [summary of changes]. This guide covers every breaking change
and provides step-by-step migration instructions.

### What's New in v3
- [Feature 1]: [one-sentence description]
- [Feature 2]: [one-sentence description]

### Breaking Changes Summary

| Change | Impact | Effort |
|--------|--------|--------|
| Pagination default changed | All list endpoints | Low — update client code |
| Auth header format changed | All authenticated requests | Medium — update auth middleware |
| `/users/all` removed | Users listing | Low — change endpoint |

## Step-by-Step Migration

### 1. Update Authentication

**Before (v2):**
```bash
curl -H "Authorization: Token abc123" https://api.example.com/v2/users
```

**After (v3):**
```bash
curl -H "Authorization: Bearer abc123" https://api.example.com/v3/users
```

**Why**: v3 adopts OAuth 2.0 Bearer token standard for consistency.

### 2. Update Pagination Handling

**Before (v2):**
```json
{ "users": [...all users...] }
```

**After (v3):**
```json
{
  "data": [...50 users...],
  "pagination": { "next": "/v3/users?cursor=abc", "has_more": true }
}
```

**Action**: Update client code to handle paginated responses.

## Verification

After completing all steps, verify your migration:

```bash
# Test authentication
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/v3/users

# Expected: 200 OK with paginated response
```

## Rollback

If you encounter issues, you can continue using v2 until [sunset date].
v2 and v3 run in parallel during the transition period.

## Getting Help

- [Migration FAQ](/migrations/v2-to-v3/faq)
- [Breaking changes reference](/changelog#v3.0.0)
- [Support channel](/support)
```

### Migration Guide Rules

| Rule | Requirement |
|------|------------|
| Time estimate | Must appear at the top |
| Breaking changes table | Summary table with impact and effort ratings |
| Before/After for every change | Show old code and new code side by side |
| Verification section | How to confirm migration succeeded |
| Rollback instructions | How to revert if issues arise |
| Support links | Where to get help during migration |

## Version Selector UX

### Design Patterns

| Pattern | Implementation | When to Use |
|---------|---------------|-------------|
| Dropdown selector | Persistent dropdown in header or sidebar | ≤5 active versions |
| URL path prefix | `/v2/docs/...` vs `/v3/docs/...` | All multi-version sites |
| Banner notice | "You're viewing docs for v2. [Switch to v3]" | When viewing non-latest version |
| Default behavior | Always land on latest stable version | Unless URL specifies version |

### Version Selector Requirements

| Requirement | Implementation |
|------------|---------------|
| Persist selection | User's version choice persists across page navigation |
| Show latest | Latest stable version is the default and visually prominent |
| Show status | Label each version: "latest", "stable", "deprecated", "EOL" |
| Cross-link | Same page in different version accessible via selector |
| Missing page | If page doesn't exist in selected version, show notice with alternatives |
| URL reflects version | URL always includes version: `/v3/guides/auth` |

### Version Status Labels

| Label | Meaning | Visual Treatment |
|-------|---------|-----------------|
| **Latest** | Current recommended version | Green badge, default selection |
| **Stable** | Supported but not latest | No badge |
| **Deprecated** | End-of-life announced, sunset date set | Yellow badge with date |
| **EOL** | No longer supported | Red badge, warning banner |

## Multi-Version Documentation Management

### Version-Pinned URL Strategy

| Strategy | URL Pattern | Trade-off |
|----------|------------|-----------|
| Path prefix | `/v3/guides/auth` | Clean, bookmarkable; requires routing config |
| Subdomain | `v3.docs.example.com` | Full isolation; more infrastructure |
| Query parameter | `/guides/auth?version=3` | Easy to implement; poor SEO |

**Recommended:** Path prefix. It balances SEO, bookmarkability, and infrastructure simplicity.

### URL Rules

| Rule | Implementation |
|------|---------------|
| Canonical URL | Always points to latest version |
| Version-less URL | Redirects to latest version |
| Old version URLs | Continue to work (no 404s) |
| Removed pages | Return 410 Gone with migration link |
| Redirects | `301` from old version paths to equivalents in latest |

### Content Management

| Strategy | Description | When to Use |
|----------|------------|-------------|
| Full copy | Complete docs copy per version | Major breaking changes, different architectures |
| Conditional content | Single source with version conditionals | Minor differences between versions |
| Shared + overrides | Shared content with version-specific overrides | Most content identical, few changes |

**Conditional Content Example:**

```markdown
## Authentication

{{#if version >= 3}}
Use Bearer token authentication:
```bash
curl -H "Authorization: Bearer $TOKEN" ...
```
{{else}}
Use Token authentication:
```bash
curl -H "Authorization: Token $TOKEN" ...
```
{{/if}}
```

### Version Lifecycle in Docs

| Phase | Docs Action | Duration |
|-------|------------|----------|
| Pre-release | Docs in draft, accessible via `/next/` or `/beta/` | Until GA |
| GA Launch | Docs become latest; previous moves to `/v{N-1}/` | Ongoing |
| Deprecation | Warning banner on old version; migration guide promoted | Per deprecation timeline |
| EOL | Old version docs read-only with prominent EOL banner | Indefinite (don't delete) |

## Audit Checklist

### Changelog
- [ ] Changelog follows Keep a Changelog format
- [ ] Each entry starts with affected resource, not "we"
- [ ] Breaking changes are in separate section or clearly labeled
- [ ] Each entry links to PR or issue number
- [ ] One change per bullet point
- [ ] Changelog updated same day as release

### Deprecation
- [ ] Every deprecated feature has a removal date
- [ ] Every deprecated feature has a migration path
- [ ] Deprecation notice includes timeline (deprecated, sunset, removed)
- [ ] API responses include `Deprecation` and `Sunset` headers
- [ ] Deprecated pages/endpoints have visible banners
- [ ] Minimum notice periods are met

### Migration Guides
- [ ] Migration guide exists for every major version bump
- [ ] Guide includes time estimate at the top
- [ ] Breaking changes summary table with impact and effort
- [ ] Before/After code examples for every breaking change
- [ ] Verification section to confirm successful migration
- [ ] Rollback instructions included
- [ ] Support links provided

### Version Selector
- [ ] Version selector is visible and persistent
- [ ] Latest version is the default
- [ ] Each version shows status label (latest/stable/deprecated/EOL)
- [ ] Version persists across page navigation
- [ ] URL reflects selected version
- [ ] Non-latest versions show banner to switch

### Multi-Version Management
- [ ] URLs are version-pinned via path prefix
- [ ] Canonical URLs point to latest version
- [ ] Old version URLs don't 404
- [ ] Removed pages return 410 with migration link
- [ ] Pre-release docs accessible but clearly labeled

## Output Format

### Version Documentation Audit: `{product or scope}`

**Active Versions**: {list with status labels}

**Changelog Health**:
| Criterion | Status | Notes |
|-----------|--------|-------|
| Format compliance | Pass/Fail | {note} |
| Entry quality | Pass/Fail | {note} |
| Update cadence | Pass/Fail | {note} |

**Deprecation Compliance**:
| Deprecated Feature | Removal Date | Migration Path | Notice Period | Status |
|-------------------|-------------|----------------|---------------|--------|
| {feature} | {date} | {link} | {months} | Compliant/Violation |

**Migration Guide Coverage**:
| Version Transition | Guide Exists | Complete | Time Estimate |
|-------------------|-------------|----------|---------------|
| v{N-1} → v{N} | Yes/No | {%} | {present/missing} |

**Priority Fixes**:
1. {fix}
2. {fix}
3. {fix}

## Guidelines

- **DO**: Set explicit removal dates for all deprecations — "future version" is not a date
- **DO**: Provide Before/After code for every breaking change — developers copy-paste, not read
- **DO**: Keep old version docs accessible — deleting them breaks bookmarks and trust
- **DO**: Default to the latest version — don't make developers guess which version to use
- **DON'T**: Deprecate without a migration path — every removal needs an alternative
- **DON'T**: Use query parameters for version selection — path prefixes are better for SEO and bookmarks
- **DON'T**: Skip the rollback section in migration guides — developers need a safety net
- **DON'T**: Remove old version docs — archive them with an EOL banner instead

## Related Components

- **governing-docs-lifecycle skill**: Page lifecycle states (published/stale/deprecated/archived) complement version lifecycle phases
- **auditing-content-quality skill**: Quality rubric applies to changelogs and migration guides as documentation pages
- **designing-navigation-systems skill**: Version selector integrates with navigation system design
- **scaffolding-page-archetypes skill**: Migration guides follow a distinct page anatomy defined here
- **docs-designer agent**: Ensures migration guides and changelogs meet documentation quality standards
