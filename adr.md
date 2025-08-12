# ADR: Use CamelCase for TypeScript Types & Interfaces (no `T`/`I` prefixes, no `Dto` suffix)

**Status:** Approved  
**Date:** 2025‑08‑12  
**Decision Type:** Convention / Code Style  
**Deciders:** Architecture Guild, Frontend Chapter Leads, DX Working Group  
**Consulted:** Backend Chapter Leads, QA, DevRel  
**Informed:** All Engineering, Docs Team, Delivery Managers

---

## Context

Our TypeScript codebase mixes naming styles for type aliases and interfaces. Some modules use `IUser`, others `User`; some introduce `TUser`, and some append `Dto` (e.g., `UserDto`) to indicate transport shapes. This inconsistency hurts readability, increases onboarding time, and complicates automated linting/refactoring.

We need a single, simple convention that:
- Maximizes clarity in IDEs and code reviews,
- Minimizes cognitive load and boilerplate,
- Plays nicely with TypeScript ecosystem norms and tooling,
- Scales to domain-driven designs without prefix/suffix sprawl.

---

## Decision

Adopt **PascalCase (a.k.a. CamelCase with leading capital) names** for all TypeScript **types** and **interfaces**, with **no leading `I` or `T` prefixes** and **no `Dto` suffix**.

**Examples (✅ correct):**
```ts
interface User {
  id: UserId;
  email: EmailAddress;
  profile: Profile;
}

type UserId = string & { __brand: 'UserId' };
type EmailAddress = string;
interface Profile { displayName: string; }

type CreateUserRequest = {
  email: EmailAddress;
  displayName: string;
};

type CreateUserResponse = {
  user: User;
};
```

**Non‑examples (🚫 avoid):**
```ts
interface IUser { /* ... */ }         // no `I` prefix
type TUser = { /* ... */ };           // no `T` prefix
type UserDto = { /* ... */ };         // no `Dto` suffix
```

**When transport vs. domain shapes differ:** use explicit, descriptive names rather than `Dto`, e.g., `UserApiModel`, `UserHttpPayload`, or `PublicUserView`. Favor the **context** (API, HTTP, View, Persistence) over a generic “DTO” tag.

---

## Rationale

1. **Aligns with TS ecosystem norms.** The TypeScript handbook, DefinitelyTyped, and most community packages use PascalCase without `I`/`T` prefixes for types and interfaces. This reduces surprise and friction for new hires and external collaborators.  
2. **Improves readability.** Prefixes/suffixes add noise that IDEs already disambiguate (hover info, go‑to‑definition). Clean names keep focus on the domain.  
3. **Easier refactors.** Moving from `interface` to `type` (or vice‑versa) shouldn’t force a rename; prefixes couple the name to the construct.  
4. **Clearer domain language.** Names like `User`, `Order`, `Price` read as ubiquitous language. Technical tags (`I`, `T`, `Dto`) leak implementation concerns into the domain vocabulary.  
5. **Linting consistency.** One style is simpler to enforce (ESLint rules, codemods) and document.

---

## Considered Options

1. **Keep mixed styles (status quo).**  
   *Pros:* No churn.  
   *Cons:* Ongoing confusion, inconsistent reviews, harder tooling rules.

2. **Prefixes (`IUser`, `TUser`) and `Dto` suffix.**  
   *Pros:* Visual cue of construct/transport shape.  
   *Cons:* Verbose, couples names to constructs, diverges from common TS practice, invites overuse of `Dto` where context-specific names are better.

3. **Adopt PascalCase without prefixes/suffixes (chosen).**  
   *Pros:* Clean, idiomatic TS, domain-first names; minimal noise; easy refactors.  
   *Cons:* Requires one-time rename pass and guardrails to distinguish contexts.

---

## Consequences

### Positive
- **Standardized code reviews** and easier onboarding.
- **Less churn during refactors** between `interface` and `type`.
- **Domain‑centric naming** improves readability and docs.

### Negative / Mitigations
- **One-time migration effort.**  
  *Mitigate:* Provide codemods and ESLint autofix; batch changes per package to limit PR size.
- **Potential ambiguity between domain and transport shapes.**  
  *Mitigate:* Use contextual suffixes only when needed: `UserApiModel`, `UserDbRecord`, `PublicUserView`. Document these in the style guide.

---

## Implementation

1. **ESLint rules**
   - Enable/adjust:  
     - `@typescript-eslint/naming-convention` to enforce PascalCase for `typeLike`.  
     - Custom rule to **ban `^I[A-Z]` and `^T[A-Z]`** for interface/type names.  
     - Lint failure for `/Dto$/` on type/interface names.
2. **Codemods**
   - Scripted rename: `I([A-Z]\w+) -> $1`, `T([A-Z]\w+) -> $1`, `(\w+)Dto -> $1`.
   - Run per package to minimize conflicts; update imports automatically.
3. **Docs**
   - Update engineering handbook “Type & Interface Naming” section with examples above.
4. **CI Gate**
   - Block merges on naming violations after a grace period (two sprints).

---

## Scope & Timeline

- **Scope:** All TypeScript code in web, services, shared libraries, tooling.  
- **Start:** Next sprint.  
- **Completion target:** Two sprint window for migration; enforcement enabled at end.

---

## Measures of Success

- Lint pass rate > 98% for naming rules within two sprints.  
- New PRs show < 1% violations after enforcement.  
- Dev survey indicates improved clarity (baseline vs. +2 sprints).

---

## Risks & Assumptions

- **Risk:** Third‑party types may conflict.  
  *Plan:* Do not rewrite external declarations; wrap or alias locally if needed.  
- **Assumption:** IDE and tooling widely support symbol renames (true for VS Code/WebStorm/tsserver).

---

## Related Decisions

- ADR: File naming conventions for TS (kebab‑case files, PascalCase types).  
- ADR: API boundary modeling (explicit `*ApiModel`/`*HttpPayload` when necessary).

---

## References

- Decision record template by GIG Cymru NHS Wales (structure inspiration).

---

## Appendix: Quick Reference

- **Always PascalCase:** `User`, `OrderItem`, `CreateUserRequest`.  
- **Never use:** `IUser`, `TUser`, `UserDto`.  
- **Use context when needed:** `UserApiModel`, `UserDbRecord`, `PublicUserView`.  
- **Prefer domain language** over technical tags.
