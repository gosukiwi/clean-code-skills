---
name: clean-react-file-organization
description: Use when creating, moving, or splitting React files, owner folders, component folders, hooks, presenters, utils, CSS modules, common folders, or shared/private dependencies.
---

# Clean File Organization

Organize files by dependency ownership. Keep private dependencies inside the owner that uses them, and move shared dependencies only as high as their nearest shared owner.

## R11: Owner Folders

Give a React owner a folder when it has private dependencies.

```text
A/
  A.tsx
  hooks.ts
  B/
    B.tsx
    C.tsx
```

In this structure:

- `C.tsx` is private to `B.tsx`.
- `B.tsx` is private to `A.tsx`.
- `hooks.ts` is owned by `A/`, so `A.tsx`, `B.tsx`, and `C.tsx` may use it.

Use the same rule for hooks, presenters, utils, and similar helpers. If only one owner uses it, keep it with that owner.

## Shared Dependencies

If multiple nearby owners use the same dependency, move it to their nearest `common/` folder.

```text
common/
  hooks.ts
A/
  A.tsx
  B.tsx
C/
  C.tsx
```

Inside `common/`, create layer folders only when they are useful for that area: `components/`, `hooks/`, `presenters/`, or `utils/`. For single files, prefer `MyHelperComponent.tsx`, `hooks.ts`, `presenters.ts`, or `utils.ts`.

## CSS Modules

CSS modules are owner-scoped by default. Prefer one owner folder per CSS module, such as:

```text
BattleStage/
  BattleStage.tsx
  BattleStage.module.css
```

Do not share a CSS module across unrelated features. If styling must be shared, keep it inside the nearest shared owner or `common/`, and prefer a shared component over shared CSS when possible.

## Naming

For React owner folders, prefer PascalCase for the owner folder, owner component file, and owner CSS module:

```text
BattleStage/
  BattleStage.tsx
  BattleStage.module.css
```

## Outside React

Apply the same ownership principle outside React. Non-UI modules do not need React-style layer folders unless they help. Do not create folders preemptively for one standalone file; add a folder when the owner gains private dependencies or enough internal structure to justify it.

## Common Mistakes

- Creating broad `components/`, `hooks/`, or `utils/` folders before ownership is clear.
- Moving a helper to global `common/` when only one owner uses it.
- Sharing CSS modules instead of extracting a shared component.
- Creating folders preemptively for one standalone file.
