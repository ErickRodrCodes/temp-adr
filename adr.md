# ADR: Use PascalCase for TypeScript Types & Interfaces (no `T`/`I` prefixes, no `Dto` suffix)

**Status:** Approved  
**Date:** 2025-08-12  
**Review date:** 2026-02-12  
**Deciders:** Architecture Guild, Frontend Chapter Leads, DX Working Group  
**Consulted:** Backend Chapter Leads, QA, DevRel  
**Informed:** All Engineering, Docs Team, Delivery Managers

---

## Context and problem statement

Our TypeScript codebase uses mixed naming styles for type aliases and interfaces. Examples include `IUser`, `TUser`, and `UserDto`, alongside clean names such as `User`. This inconsistency creates noise in code reviews, increases onboarding time, and complicates lint rules and refactors between `interface` and `type`.

We need a single, simple convention that is idiomatic in TypeScript, reduces cognitive load, and scales across domains and boundaries (API, View, Persistence).

---

## Decision drivers

- Improve clarity and readability in IDEs and code reviews.
- Reduce renaming churn when switching between `interface` and `type`.
- Align with common TypeScript community practice.
- Keep names focused on domain language instead of implementation details.
- Enable straightforward linting and codemods.

---

## Considered options

1. **PascalCase names without prefixes/suffixes** (e.g., `User`, `CreateUserRequest`) — *no `I`/`T` prefixes, no `Dto` suffix.*
2. **Prefixes for construct and `Dto` suffix** (e.g., `IUser`, `TUser`, `UserDto`).
3. **Status quo** (keep mixed styles by team/package preference).

---

## Decision outcome

**Chosen option:** *PascalCase names without prefixes/suffixes* for all **types** and **interfaces**.

**Explanation:** This is the cleanest and most idiomatic approach in TypeScript. It minimizes visual noise, keeps names domain-centric, and avoids coupling the symbol name to a specific construct. It also makes refactors between `interface` and `type` mechanical and low-risk.

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

**Non-examples (🚫 avoid):**
```ts
interface IUser { /* ... */ }         // no `I` prefix
type TUser = { /* ... */ };           // no `T` prefix
type UserDto = { /* ... */ };         // no `Dto` suffix
```

**When shapes differ across boundaries:** prefer descriptive context over `Dto`, e.g., `UserApiModel`, `UserHttpPayload`, `UserDbRecord`, or `PublicUserView` (use only when necessary).

---

## Pros and cons of the options

### 1) PascalCase without prefixes/suffixes — *Chosen*
- **Pros:** idiomatic TypeScript; clean, domain-first names; easy refactors between `interface` and `type`; simpler lint rules.
- **Cons:** requires one-time rename and lint enforcement; needs contextual naming for boundary-specific shapes.

### 2) Prefixes (`IUser`, `TUser`) and `Dto` suffix
- **Pros:** quick visual cue for construct or transport shape; mirrors some legacy codebases.
- **Cons:** verbose; couples names to constructs; diverges from common TS practice; overuses generic `Dto` where specific context is better.

### 3) Status quo (mixed styles)
- **Pros:** no immediate churn.
- **Cons:** inconsistent reviews; harder onboarding; messy linting; ongoing confusion about meaning and intent.

---

## Consequences

### Positive
- Consistent, domain-centric naming across the codebase.
- Less churn and risk during refactors between `type` and `interface`.
- Simpler ESLint rules and easier automated enforcement.

### Negative (and mitigations)
- **Migration effort:** use codemods and ESLint autofixes; batch changes by package.
- **Boundary ambiguity:** allow contextual suffixes only when needed (`*ApiModel`, `*HttpPayload`, `*DbRecord`, `*View`) and document usage examples.

---

## Follow-up actions

- **Linting:** configure `@typescript-eslint/naming-convention` to enforce PascalCase for `typeLike`; add a rule to disallow `^I[A-Z]`, `^T[A-Z]`, and `/Dto$/` on type/interface names.
- **Codemods:** apply scripted renames (`I([A-Z]\w+) -> \1`, `T([A-Z]\w+) -> \1`, `(\w+)Dto -> \1`) and update imports automatically.
- **Docs:** add a “Type & Interface Naming” page with the examples above.
- **CI gate:** after two sprints, block merges on naming violations.

---

## References

- Internal engineering handbook (naming conventions).  
- TypeScript community conventions (handbook, DefinitelyTyped, OSS repos).
