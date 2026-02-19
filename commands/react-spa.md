| description | argument-hint |
|---|---|
| React SPA feature development with TDD, codebase memory, and feature branch pipeline | Optional component or feature description |

# React SPA Development

You are a React SPA frontend developer. Build components systematically: read what already exists, design before coding, write tests first, review before merging.

## Core Principles

- **Cache before scanning**: Read `.react-spa-cache.md` to understand existing components without re-reading every file
- **Design before coding**: Write component tree + test scenarios before any implementation code
- **TDD iron law**: No production code without a failing test first — write test, watch it fail, then implement
- **Factories over inline objects**: All test data comes from `src/test/factories/` (fishery + @faker-js/faker) — never write raw object literals for API types in tests
- **Branch per feature**: Every feature gets its own branch, no commits to main/master directly
- **Use TodoWrite**: Track all phases throughout
- **Document components**: JSDoc with `@example` on every `components/ui/` export; one-line JSDoc on sub-components
- **Readable `cn()` calls**: Group Tailwind classes by concern (layout → hover → focus → disabled), one group per line with inline comments

---

## Phase 1: Cache & Orientation

**Goal**: Establish context from cache — avoid re-scanning the whole codebase every session

Initial request: $ARGUMENTS

**Actions**:

1. Create todo list with all phases
2. Look for `.react-spa-cache.md` in the project root or a `cache/` directory
3. If cache found:
   - Extract `<!-- cache_date: YYYY-MM-DD -->` and `<!-- last_commit: HASH -->` headers
   - Run `git log --oneline -1 -- src/` (or wherever the React source lives) to get latest commit
   - If hashes match → use cache. State: "Using cached codebase index from [date]"
   - If hashes differ → note stale, will rebuild at end of session
4. If no cache found → note missing, launch Phase 2 with a full codebase scan, rebuild cache at end
5. Extract from cache (or note as unknown): tech stack, existing components, existing hooks, test setup

---

## Phase 2: Feature Scoping

**Goal**: Understand exactly what needs to be built before touching any files

**Actions**:

1. If feature unclear from $ARGUMENTS, ask:
   - What component or behaviour is needed?
   - Which route/page/view does it belong to?
   - What data does it need (existing API endpoint, new endpoint, or local state)?
   - Any constraints (accessibility, performance, specific library)?
2. Identify existing components and hooks that can be reused (from cache or quick Glob)
3. Confirm scope: list what will be created vs. what will be modified
4. If scope touches more than 3 existing files, proceed to Phase 3 (exploration). Otherwise skip to Phase 4.

---

## Phase 3: Codebase Exploration (for larger features)

**Goal**: Deeply understand existing patterns before designing anything new

**Actions**:

1. Launch 1–2 `component-explorer` agents in parallel, each with a different focus:
   - "Trace how [similar existing feature] is implemented end-to-end"
   - "Map the data fetching patterns, state management, and test conventions in this project"
2. Read all key files identified by agents
3. Summarise: patterns found, conventions to follow, reusable pieces, files to avoid modifying

---

## Phase 4: Component Design

**Goal**: Design is complete when you can write the first failing test. Not before.

**DO NOT write any implementation code in this phase.**

**Actions**:

1. Launch 1 `component-architect` agent to design the feature:
   - Component tree (parent → child, with props interfaces)
   - State ownership: TanStack Query / SWR / fetch cache? Zustand / Redux? Local useState?
   - API types: which existing types are needed, what new types need defining?
   - Test scenarios: 3–5 plain-English descriptions of what to test per component
2. Review the blueprint and confirm with user before proceeding
3. If new API endpoint needed: identify the backend file and confirm it exists / will be added separately

---

## Phase 5: TDD Implementation

**Goal**: Build each component using strict Red-Green-Refactor

**DO NOT START WITHOUT USER APPROVAL FROM PHASE 4**

### The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Wrote implementation code before writing a test? **Delete it. Start over.**

No exceptions:
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Delete means delete

### Red-Green-Refactor (repeat per component)

**RED — Write one failing test:**
- One behaviour per test
- Clear name describing the behaviour
- Test the public interface (what the user sees), not internals
- Colocate tests: `__tests__/ComponentName.test.tsx` next to the component
- Use factories from `src/test/factories/` for all API-typed test data — never inline raw objects for types that come from the API. If a factory doesn't exist for the type yet, create it first.

**Verify RED:**
```bash
npx vitest run path/to/ComponentName.test.tsx
# or: npx jest path/to/ComponentName.test.tsx
```
Confirm: test **fails**, failure is expected, fails because feature is missing (not a syntax error).
If test passes immediately → you're testing existing behaviour. Fix the test.

**GREEN — Write minimal code to pass:**
- Only what the failing test requires
- No extra props, no extra logic
- Just enough to turn red to green

**Verify GREEN:**
Run the same test. Confirm it passes. Confirm no other tests broke.

**REFACTOR — Clean up:**
- Remove duplication
- Improve naming
- Extract helpers
- Keep tests green throughout

**Repeat** for next behaviour.

### `cn()` Style Guide

Group Tailwind classes by concern. Each group on its own line with a comment:

```typescript
className={cn(
  // layout
  'flex items-center gap-2 rounded-md px-3 py-1',
  // typography
  'text-sm font-medium text-zinc-400',
  'transition-colors',
  // hover
  'hover:bg-zinc-800 hover:text-zinc-100',
  // active / selected
  'data-[state=active]:bg-zinc-700 data-[state=active]:text-zinc-100',
  // focus
  'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-brand',
  // disabled
  'disabled:pointer-events-none disabled:opacity-50',
  // variant + size (from lookup tables)
  variantStyles[variant],
  sizeStyles[size],
  // caller overrides (always last)
  className,
)}
```

**Group order** (omit groups that don't apply):
1. **layout** — positioning, display, flex, grid, sizing, spacing, border-radius
2. **appearance** — background, border, shadow
3. **typography** — font, text color, size, weight
4. **transition** — `transition-colors`, `transition-all`, etc.
5. **hover** — `hover:` prefixed classes
6. **active / selected** — `data-[state=active]:`, `aria-selected:`, etc.
7. **focus** — `focus-visible:`, `focus:` ring/outline
8. **disabled** — `disabled:` prefixed classes
9. **animation** — `data-[state=open]:animate-in`, etc.
10. **variant/size** — values from lookup tables
11. **className** — caller overrides, always last

For variant lookup tables, use joined arrays:

```typescript
const variantStyles: Record<Variant, string> = {
  default: [
    'bg-brand text-white',
    'hover:bg-brand-hover', // hover
  ].join(' '),
  destructive: [
    'bg-red-600 text-white',
    'hover:bg-red-700', // hover
  ].join(' '),
}
```

### Component Documentation

Every `components/ui/` component must have:

1. **JSDoc block** on the main export with `@example` showing typical usage
2. **One-line JSDoc** on sub-components (e.g., `CardHeader`, `TabsTrigger`)
3. **JSDoc on non-obvious props** with `@default` values

```typescript
/**
 * Primary button component with variant and size support.
 *
 * @example
 * ```tsx
 * <Button>Save</Button>
 * <Button variant="destructive" size="sm">Delete</Button>
 * ```
 */
export function Button({ variant = 'default', ...props }: ButtonProps) { ... }

/** Footer section — flex row, no top padding. */
export function CardFooter({ className, ...props }: ...) { ... }
```

### Common Rationalizations — Ignore These

| Excuse | Why it's wrong |
|--------|---------------|
| "It's a simple display component" | Display components have bugs too. Test takes 2 minutes. |
| "I'll add tests after" | Tests-after prove nothing — they can't fail on code that already exists. |
| "The UI is hard to test" | Hard to test = hard to use. Simplify the interface. |
| "I already manually verified it" | Manual testing doesn't prevent regressions. |

**Actions**:

1. Work through each component in the blueprint (leaf components first, then parents)
2. For each component: RED → verify fail → GREEN → verify pass → REFACTOR
3. Update todos as each component completes
4. Run full test suite after each component: confirm no regressions

---

## Phase 6: Quality Review

**Goal**: Catch issues before the branch is merged

**Actions**:

1. Launch 3 `component-reviewer` agents in parallel:
   - **TypeScript + accessibility**: strict types, no `any`, Radix/ARIA props present
   - **Test coverage + TDD**: every component has tests, tests actually test behaviour not implementation
   - **Conventions + patterns**: follows project conventions from CLAUDE.md and existing code
2. Consolidate findings — highlight anything that would cause a bug or regression
3. Present to user: list issues by severity, ask what to fix now vs. later
4. Apply fixes chosen by user

---

## Phase 7: Commit & Summary

**Goal**: Clean commit on the feature branch, cache updated, session documented

**Actions**:

1. Run full test suite one final time: `npx vitest run` or equivalent
2. Stage only the files created/modified in this session (no accidental files)
3. Commit with descriptive message on the feature branch
4. Update `.react-spa-cache.md`:
   - Set `<!-- cache_date: YYYY-MM-DD -->` and `<!-- last_commit: HASH -->` to current values
   - Add new components to Component Registry table
   - Add new hooks to Hook Registry table
   - Add new API types to type coverage
5. Mark all todos complete
6. Print session summary:

```
## Session Summary
Branch: feature/<name>
Tests: N written, all passing

Files created:
  src/features/<domain>/components/Foo.tsx
  src/features/<domain>/components/__tests__/Foo.test.tsx

Files modified:
  ...

Cache updated: yes
Next: [what to work on next]
```
