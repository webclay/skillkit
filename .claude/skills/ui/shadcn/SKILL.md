---
name: shadcn
description: Shadcn UI component patterns and usage. Trigger words - shadcn, component, button, form, dialog, modal, dropdown, input, select, table, card, toast, alert, tabs, accordion, tailwind, radix, magic ui, aceternity, chatcn
---

# Shadcn UI & Component Libraries

Copy-paste component libraries built on Radix UI primitives and Tailwind CSS. Full control over component code.

## When to Use This Skill

- User asks to add UI components
- components.json exists in project root
- User mentions Shadcn, Radix UI, or component library
- User needs forms, dialogs, tables, or common UI patterns
- Working with Tailwind CSS components
- User wants to add components from other libraries (Magic UI, Aceternity, ChatCN Studio, etc.)

## Component Library Organization

**IMPORTANT:** Organize components by source library in subfolders to avoid conflicts and keep track of where components came from.

### Folder Structure

```
components/
└── ui/
    ├── shadcn/           # Base shadcn/ui components
    │   ├── button.tsx
    │   ├── card.tsx
    │   └── dialog.tsx
    ├── chatcn/           # ChatCN Studio components
    │   ├── animated-card.tsx
    │   └── gradient-button.tsx
    ├── magic-ui/         # Magic UI components
    │   ├── shimmer-button.tsx
    │   └── border-beam.tsx
    ├── aceternity/       # Aceternity UI components
    │   ├── spotlight.tsx
    │   └── 3d-card.tsx
    └── custom/           # Your custom components
        └── logo.tsx
```

### Adding Components from Different Libraries

**Rule: Never override existing components. Always use subfolders.**

| Source | Folder | Import Path |
|--------|--------|-------------|
| shadcn/ui | `components/ui/shadcn/` | `@/components/ui/shadcn/button` |
| ChatCN Studio | `components/ui/chatcn/` | `@/components/ui/chatcn/animated-card` |
| Magic UI | `components/ui/magic-ui/` | `@/components/ui/magic-ui/shimmer-button` |
| Aceternity | `components/ui/aceternity/` | `@/components/ui/aceternity/spotlight` |
| Custom | `components/ui/custom/` | `@/components/ui/custom/logo` |

### When User Provides Component Code

When a user pastes component code from any site:

1. **Always ask which library/site** it's from (unless they already mentioned it)
2. **Create the subfolder** if it doesn't exist
3. **Save with a clear name** (e.g., `shimmer-button.tsx` not `button.tsx`)
4. **Install dependencies** (with approval) if the component needs them
5. **Update imports** to use the correct path

**Example - User mentions the source:**
```
User: "Add this component from Magic UI"
[pastes code]

Claude: "I'll add this to components/ui/magic-ui/shimmer-button.tsx

This component needs framer-motion. Should I install it?"
```

**Example - User doesn't mention the source:**
```
User: "Add this component to my project"
[pastes code]

Claude: "Which site or library is this component from? I'll create a subfolder
to keep your components organized (e.g., components/ui/[library-name]/)."

User: "It's from Aceternity"

Claude: "I'll add this to components/ui/aceternity/spotlight-card.tsx"
```

**Why this matters:** Knowing the source helps organize components, avoid naming conflicts, and makes it easy to update or find components later.

### Handling CLI Installations

Some libraries have CLIs (like ChatCN Studio). When using them:

1. **Check if they support custom paths** - many do via config
2. **If CLI creates in wrong location** - move to correct subfolder after install
3. **Never let CLI override** existing components from other libraries

For ChatCN Studio specifically, it already creates its own subfolder which is the correct pattern.

## Instructions

### Step 1: Initialize Shadcn

```bash
npx shadcn@latest init
```

When prompted, configure the components path to use subfolders if supported.

### Step 2: Add Base Components

```bash
npx shadcn@latest add button
npx shadcn@latest add card dialog form input
```

Base shadcn components go to `components/ui/` (or `components/ui/shadcn/` if you prefer full separation).

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

**Toast:**
```tsx
import { useToast } from "@/components/ui/use-toast";

const { toast } = useToast();

toast({ title: "Success", description: "Changes saved" });
toast({ variant: "destructive", title: "Error", description: "Something went wrong" });
```

**Select:**
```tsx
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem } from "@/components/ui/select";

<Select>
  <SelectTrigger>
    <SelectValue placeholder="Choose..." />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="a">Option A</SelectItem>
    <SelectItem value="b">Option B</SelectItem>
  </SelectContent>
</Select>
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

## Common Components

```bash
npx shadcn@latest add button card dialog form input label select table tabs toast dropdown-menu sheet alert-dialog
```

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

## Tips

- Components are in `components/ui/` - modify directly
- Use `cn()` utility for conditional classes
- All components are accessible (Radix UI)
- Check https://ui.shadcn.com for component docs
- **Always use subfolders** for components from different libraries
- **Never override** existing components without checking usage first

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
