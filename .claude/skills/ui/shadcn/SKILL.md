---
name: shadcn
description: Use this skill when building UI with shadcn/ui components like buttons, forms, dialogs, tables, or cards. Activate when the user mentions shadcn, Radix UI, Base UI primitives, Tailwind-based components, presets, monorepo UI setup, Magic UI, Aceternity, or ChatCN component patterns.
---

# Shadcn UI (CLI v4)

Copy-paste component library built on Radix UI or Base UI primitives and Tailwind CSS. Full control over component code. CLI v4 adds skills, presets, monorepo support, dry-run, and more.

## When to Use This Skill

- User asks to add UI components
- `components.json` exists in project root
- User mentions Shadcn, Radix UI, Base UI, or component library
- User needs forms, dialogs, tables, or common UI patterns
- Working with Tailwind CSS components
- User wants to scaffold a project with shadcn templates
- User mentions presets, monorepo setup, or switching design systems
- User wants to add components from other libraries (Magic UI, Aceternity, ChatCN Studio, etc.)

## CLI v4 Overview

### Installing the Official shadcn/skills

shadcn now has official AI skills that give coding agents project-aware context. Install them in any project:

```bash
[runner] skills add shadcn/ui
```

Once installed, agents can use `shadcn info --json` for project configuration, `shadcn docs [component]` for component docs, and `shadcn search` to find registry components.

### Project Scaffolding with Templates

`shadcn init` can scaffold full project templates with dark mode included:

```bash
# Basic init
[runner] shadcn@latest init

# With a specific template
[runner] shadcn@latest init -t next        # Next.js
[runner] shadcn@latest init -t vite        # Vite
[runner] shadcn@latest init -t start       # TanStack Start
[runner] shadcn@latest init -t react-router # React Router
[runner] shadcn@latest init -t astro       # Astro
[runner] shadcn@latest init -t laravel     # Laravel

# Monorepo (Turborepo)
[runner] shadcn@latest init --monorepo

# Choose primitive library
[runner] shadcn@latest init --base radix   # Radix UI (default)
[runner] shadcn@latest init --base base    # Base UI
```

### Presets

Presets pack a design config (colors, fonts, radius) into a short code. Use them to scaffold or switch designs:

```bash
# Scaffold with a preset
[runner] shadcn@latest init --preset adtk27v

# Switch preset on existing project (reconfigures components, theme, deps)
[runner] shadcn@latest init --preset aKqgMa9

# Works with templates
[runner] shadcn@latest init -t next --preset adtk27v
```

Build custom presets at https://ui.shadcn.com/create - preview how colors, fonts, and radius apply to real components.

### Adding Components

```bash
# Add single component
[runner] shadcn@latest add button

# Add multiple
[runner] shadcn@latest add button card dialog form input

# Add all components
[runner] shadcn@latest add --all

# Add the demo component (previews your entire design system)
[runner] shadcn@latest add demo

# Add/change fonts as first-class registry items
[runner] shadcn@latest add font merriweather
[runner] shadcn@latest add font jetbrains-mono

# From namespaced registries (trusted registries like react-bits, tailark, etc.)
[runner] shadcn@latest add @acme/auth @v0/dashboard
```

### Preview Before Installing (Dry-Run, Diff, View)

These flags are essential for security when adding components from third-party registries. Always use `--view` to inspect code before it enters your project.

```bash
# Preview what will be added without writing files (shows which files are new vs identical vs overwritten)
[runner] shadcn@latest add login-01 --dry-run

# Check for updates and see exact diff against local changes (e.g., spot unwanted "use client" additions)
[runner] shadcn@latest add login-01 --diff

# View full file contents from registry before installing (security: inspect third-party code)
[runner] shadcn@latest add login-01 --view

# View items without installing
[runner] shadcn@latest view button card dialog
```

### Search Registries

```bash
# Search a registry
[runner] shadcn@latest search @shadcn -q "button"

# Search multiple registries
[runner] shadcn@latest search @shadcn @v0 @acme

# List all items in a registry
[runner] shadcn@latest list @acme
```

### Project Info and Docs

```bash
# Full project info (framework, version, CSS vars, installed components)
[runner] shadcn@latest info
[runner] shadcn@latest info --json    # Machine-readable for agents

# Component documentation
[runner] shadcn@latest docs button
[runner] shadcn@latest docs dialog --base radix
```

### Migrations

```bash
# Migrate to unified radix-ui package
[runner] shadcn@latest migrate radix

# Add RTL support
[runner] shadcn@latest migrate rtl

# Migrate icon library
[runner] shadcn@latest migrate icons
```

### Registry Building

```bash
# Build registry from registry.json
[runner] shadcn@latest build
[runner] shadcn@latest build --output ./public/registry
```

New registry types: `registry:base` distributes an entire design system as a single payload (components, deps, CSS vars). `registry:font` makes fonts a first-class registry type.

## Monorepo Structure

When using `--monorepo`, shadcn creates a Turborepo workspace:

```
apps/
  web/                    # Your app
    components/           # App-specific components (blocks, pages)
    components.json       # App config (aliases point to @workspace/ui)
packages/
  ui/                     # Shared UI components
    src/
      components/         # shadcn base components (button, card, etc.)
      hooks/
      lib/utils.ts
      styles/globals.css
    components.json       # UI package config
turbo.json
```

Import from the shared package:
```tsx
import { Button } from "@workspace/ui/components/button"
import { cn } from "@workspace/ui/lib/utils"
```

Both `components.json` files must have matching `style`, `iconLibrary`, and `baseColor`.

## Component Library Organization

**IMPORTANT:** Organize components by source library in subfolders to avoid conflicts.

### Folder Structure

Base shadcn/ui primitives live in `components/ui/`. Third-party library components each get their own folder under `components/`:

```
src/components/
  ui/                   # Base shadcn/ui primitives (button, card, dialog, etc.)
  magic-ui/             # Magic UI animated components
  aceternity/           # Aceternity UI 3D/effect components
  kibo-ui/              # Kibo UI components
  shadcn-studio/        # Shadcn Studio blocks
```

| Source | Folder | Import Path |
|--------|--------|-------------|
| shadcn/ui (base) | `components/ui/` | `@/components/ui/card` |
| Magic UI | `components/magic-ui/` | `@/components/magic-ui/shimmer-button` |
| Aceternity | `components/aceternity/` | `@/components/aceternity/spotlight` |
| Kibo UI | `components/kibo-ui/` | `@/components/kibo-ui/kanban` |

### CLI Install Safety

When running any component CLI:
1. **NEVER say yes to overwriting** existing files like `card.tsx`, `utils.ts`, etc.
2. If the CLI places files in the wrong location, move them to the correct library subfolder
3. Update imports inside the moved component if needed

### When User Provides Component Code

1. **Always ask which library/site** it's from (unless already mentioned)
2. **Create the subfolder** if it doesn't exist
3. **Save with a clear name** (e.g., `shimmer-button.tsx` not `button.tsx`)
4. **Install dependencies** (with approval) if the component needs them
5. **Update imports** to use the correct path

## Examples

**Button:**
```tsx
import { Button } from "@/components/ui/button";

<Button variant="default">Default</Button>
<Button variant="destructive">Delete</Button>
<Button variant="outline">Outline</Button>
<Button variant="ghost">Ghost</Button>
<Button size="sm">Small</Button>
<Button size="lg">Large</Button>
```

**Card:**
```tsx
import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter } from "@/components/ui/card";

<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>Content here</CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

**Dialog:**
```tsx
import { Dialog, DialogTrigger, DialogContent, DialogHeader, DialogTitle, DialogDescription, DialogFooter } from "@/components/ui/dialog";

<Dialog>
  <DialogTrigger asChild>
    <Button>Open</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Title</DialogTitle>
      <DialogDescription>Description</DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <Button>Save</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

**Form with React Hook Form:**
```tsx
"use client";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import * as z from "zod";
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage } from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";

const schema = z.object({
  email: z.string().email(),
});

export function MyForm() {
  const form = useForm({ resolver: zodResolver(schema) });

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(console.log)}>
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Submit</Button>
      </form>
    </Form>
  );
}
```

**Loading Button:**
```tsx
import { Loader2 } from "lucide-react";

<Button disabled={loading}>
  {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
  {loading ? "Saving..." : "Save"}
</Button>
```

## Reference Files

- [forms.md](forms.md) - Form patterns with validation
- [dark-mode.md](dark-mode.md) - Theme setup

## Dark Mode Setup

```bash
pnpm add next-themes
```

```tsx
// app/layout.tsx
import { ThemeProvider } from "next-themes";

<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
  {children}
</ThemeProvider>
```

## Agent Workflow Tips

When working as a coding agent with shadcn/ui:

1. **Use `shadcn info --json`** to get the project's full config (framework, version, CSS vars, preset, installed components)
2. **Use `shadcn docs [component]`** to check component APIs before using them
3. **Use `--dry-run`** before adding components to see what will be new, identical, or overwritten
4. **Use `--diff`** to check for component updates and merge with local changes
5. **Use `--view`** to inspect third-party registry code before installing (security)
6. **Use presets** when the user wants to try different designs - one command switches everything (colors, fonts, radius, even base library)
7. **Use `shadcn add demo`** to give the user a visual preview of their current design system
8. **Use `shadcn add font <name>`** to switch fonts instantly
9. **Check `components.json`** for framework, aliases, base library, and icon library
10. **Trusted registries** - The skills file includes registries like react-bits, tailark that can be used directly

## Tips

- Components are in `components/ui/` - modify directly
- Use `cn()` utility for conditional classes
- All components are accessible (Radix UI or Base UI primitives)
- Check https://ui.shadcn.com for component docs
- **Always use subfolders** for components from different libraries
- **Never override** existing components without checking usage first
- Use `--base radix` (default) or `--base base` when initializing to choose primitives
- Use `shadcn migrate radix` to migrate from individual `@radix-ui/react-*` packages to unified `radix-ui`

## Popular Component Libraries

| Library | Website | Notes |
|---------|---------|-------|
| shadcn/ui | ui.shadcn.com | Base components, use as foundation |
| ChatCN Studio | chatcn.studio | Modern, clean components (has CLI) |
| Magic UI | magicui.design | Animated components |
| Aceternity UI | ui.aceternity.com | 3D and effect components |
| Cult UI | cult-ui.com | Additional variants |

## How to Verify

### Quick Checks
- `bun run dev` - App starts without component import errors
- Components appear in correct subfolder after adding
- No TypeScript errors on component imports
- Different libraries don't conflict

### Manual Verification
- View the page with new component - renders correctly
- Click buttons, open dialogs - interactions work
- Check mobile view - components are responsive
- Verify dark mode works (if ThemeProvider configured)

### Common Issues
- "Module not found": Component not added or wrong import path
- Styles missing: Check `tailwind.config` includes component paths
- Form not submitting: Ensure `handleSubmit` is called, check console for errors
- Dialog not opening: Verify DialogTrigger wraps the trigger element
- Component conflict: Two libraries have same component name - use subfolders
- Override warning from CLI: Don't override - install to subfolder instead
- Radix import errors after migration: Run `shadcn migrate radix` to unify imports
