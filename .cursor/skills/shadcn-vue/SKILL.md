---
name: shadcn-vue
description: >-
  Add shadcn-vue UI components to the Nuxt app in this Bun monorepo.
  Use when the user asks to add a UI component (button, dialog, dropdown, etc.),
  mentions shadcn, or wants to use shadcn-vue components.
---

# shadcn-vue Component Workflow

## Adding components

Run from `apps/website/`:

```bash
bunx --bun shadcn-vue@latest add <component> -y
```

Components land in `app/components/ui/<name>/` and are auto-imported by Nuxt.
`shadcn-nuxt` also re-exports `Ui*` prefixed names (e.g. `UiButton`).

Examples:

```bash
bunx --bun shadcn-vue@latest add button -y
bunx --bun shadcn-vue@latest add dialog -y
bunx --bun shadcn-vue@latest add dropdown-menu -y
```

The `-y` flag skips the confirmation prompt.

## Using components in templates

Since Nuxt auto-imports from `components/`, use importless PascalCase tags:

```vue
<template>
  <Button variant="outline">Click me</Button>
  <DropdownMenu>
    <DropdownMenuTrigger>Open</DropdownMenuTrigger>
    <DropdownMenuContent>
      <DropdownMenuItem>Profile</DropdownMenuItem>
    </DropdownMenuContent>
  </DropdownMenu>
</template>
```

For the `cn()` class merge utility, import from `@/lib/utils`:

```ts
import { cn } from "@/lib/utils"
```

## Key facts

- **App directory**: `apps/website/` — always run shadcn CLI from here
- **Component output**: `app/components/ui/`
- **Tailwind**: Tailwind v4 via `@tailwindcss/vite` (not `@nuxtjs/tailwindcss`)
- **Theming**: CSS variables in `app/assets/css/tailwind.css` (Neutral base, Nova style)
- **Icon library**: Lucide (`@lucide/vue`)
- **Base components**: Reka UI (`reka-ui`)

## Gotchas

- **Never** use `@nuxtjs/tailwindcss` — it conflicts. Only `@tailwindcss/vite`.
- If a build fails with `Failed to resolve extends base type` on `PrimitiveProps`, ensure `typescript@^5` is installed, **not** `typescript@7`.
- If Nuxt complains about missing component dir during `prepare`, it's harmless — resolves after the first component add.
- Use `bunx --bun` (not plain `bunx`) in Bun environments.
- Run all shadcn commands from `apps/website/`, not the monorepo root.
