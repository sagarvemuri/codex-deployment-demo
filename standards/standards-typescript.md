---
description: TypeScript standards — Zod, Drizzle, TRPC, package boundaries. Apply when writing or reviewing TypeScript code.
globs:
  - "**/*.ts"
  - "**/*.tsx"
---

# TypeScript Standards v1.0.0

These rules are mandatory for all TypeScript code. See the code standards for universal principles. Rules 2–4 apply to React UI code.

---

## 1. No Type Escape Hatches

```ts
// NEVER — unsafe casts and suppression
any, as unknown as X, @ts-nocheck, @ts-ignore

// ALWAYS — explicit interfaces + runtime validation
const UserSchema = z.object({
    id: z.string(),
    name: z.string(),
    email: z.string().email(),
});
const user = UserSchema.parse(rawData);
```

Validate external data at system edges with Zod. After `parse`, trust the type.

**Permitted uses of `as`:** `as const` for literal types, and narrowing after a runtime check where TypeScript cannot infer the type (e.g., after a Zod parse or type guard). The test: does the `as` assert something the runtime has already verified? If yes, it's safe. If no, use Zod instead.

## 2. Memoize Only When Necessary

Don't memoize by default. Measure first, optimize when needed.

**Use `useCallback` when**: function passed to memoized child, function used as dependency in `useEffect`/`useMemo`, function creates closures over frequently changing values.

**Use `useMemo` when**: array operations on large lists where profiling shows avoidable work, complex calculations that are measurably slow, object/array creation passed to memoized children.

**Don't memoize**: event handlers for non-memoized components, simple string operations, functions only used within the component.

## 3. useEffect — Escape Hatch Only

Derive state during render; use `useMemo` only when Rule 2 applies. Event handlers for events. `useEffect` ONLY for external system synchronization.

## 4. Declarative UI Composition

Prefer compound components, children-based slots, and props-driven state over boolean configuration, render props, and imperative refs.

```ts
// NEVER
<DataTable showHeader showFooter showPagination showSearch />
<Modal ref={modalRef} />  // imperative open/close

// ALWAYS
<DataTable>
    <DataTable.Header><SearchBar /></DataTable.Header>
    <DataTable.Body>{data.map(...)}</DataTable.Body>
</DataTable>
<Modal isOpen={isOpen} onClose={() => setIsOpen(false)} />
```

## 5. TRPC — Always Spread queryOptions

```ts
// NEVER
useQuery({ queryKey: ['...'], queryFn: () => trpc.x.y.query(...) });

// ALWAYS
useQuery({ ...trpc.x.y.queryOptions({...}), enabled: !!id });
```

## 6. Drizzle ORM — No Ad-Hoc Queries

Use the Drizzle query builder. `sql` template fragments within the builder are fine; ad-hoc `db.execute(sql\`...\`)` is not.

```ts
// NEVER
db.execute(sql`SELECT ...`)

// ALWAYS
db.select({ count: sql<number>`count(*)` }).from(table).where(eq(table.id, id))
```

## 7. Select Only What You Need

```ts
// NEVER: downloads entire record to check existence
const config = await db.query.table.findFirst({ where: eq(table.id, id) });

// ALWAYS: downloads 10 bytes
const config = await db.query.table.findFirst({
    where: eq(table.id, id),
    columns: { id: true },
});
```

## 8. Thin Data Boundaries

Routers validate input, call business logic, handle errors, return results. Extract business logic to a dedicated `lib/` directory (e.g., `packages/trpc/src/lib/`). DB queries select data and handle relations/joins — no transformations, calculations, or business logic. Filter at the TRPC level, not in the frontend. Handle soft-deletions server-side.

## 9. Use Color Palette

```ts
// NEVER: hardcoded hex values
// ALWAYS: var(--c-text-primary), var(--c-surface-mid)
```

## 10. Respect Package Boundaries

Define a clear dependency graph between packages and enforce it:

```
apps/ (UI) → packages/trpc → packages/db → packages/schemas → packages/shared
```

Never reverse this flow. Shared types belong in a dedicated schemas package.

---

## Checklist

- [ ] No `any`, no unsafe `as` casts, no `@ts-ignore`; `as const` and post-guard narrowing OK; Zod at system edges
- [ ] No premature memoization — measured before optimized
- [ ] `useEffect` only for external system synchronization
- [ ] Declarative composition: compound components, children slots, props-driven state
- [ ] TRPC with spread `queryOptions`
- [ ] Drizzle query builder — no ad-hoc `db.execute`
- [ ] Explicit column selection — select only what you need
- [ ] Data filtered upstream; routers thin; DB queries data-access only
- [ ] Colors from palette
- [ ] Package boundaries respected
