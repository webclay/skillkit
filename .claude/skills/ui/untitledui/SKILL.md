---
name: untitledui
description: Untitled UI React component library with Tailwind CSS. Trigger words - untitled ui, dashboard, admin panel, saas ui, data table, sidebar, navigation, metrics, charts
---

# Untitled UI React

5,000+ production-ready React components with Tailwind CSS and React Aria.

## When to Use This Skill

- Building SaaS dashboards and admin panels
- Need professional design without custom design work
- Require accessible components (WAI-ARIA)
- Building with Next.js, Vite, or modern React

## When NOT to Use

- Simple landing pages (overkill)
- Need highly custom design language
- Budget doesn't allow PRO license

## Licensing

- **Free**: MIT-licensed core components, CLI, 2,000+ icons
- **PRO SOLO** ($349): 5,000+ components, 250+ pages, Figma sync
- **PRO TEAM** ($699): Up to 5 users

## Setup

See [Untitled UI Getting Started](https://www.untitledui.com/react) for latest setup.

### Core Dependencies

```bash
npm install react-aria-components clsx tailwind-merge
```

### Utility Function

```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Button Component

```typescript
import { Button as AriaButton, ButtonProps } from 'react-aria-components';
import { cn } from '@/lib/utils';

interface ButtonProps extends AriaButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'destructive';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({ className, variant = 'primary', size = 'md', ...props }) {
  return (
    <AriaButton
      className={cn(
        'inline-flex items-center justify-center rounded-md font-medium',
        'transition-colors focus-visible:outline-none focus-visible:ring-2',
        'disabled:pointer-events-none disabled:opacity-50',
        variant === 'primary' && 'bg-primary text-primary-foreground hover:bg-primary/90',
        variant === 'secondary' && 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        variant === 'outline' && 'border border-input bg-background hover:bg-accent',
        variant === 'ghost' && 'hover:bg-accent hover:text-accent-foreground',
        variant === 'destructive' && 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        size === 'sm' && 'h-9 px-3 text-sm',
        size === 'md' && 'h-10 px-4 py-2',
        size === 'lg' && 'h-11 px-8 text-lg',
        className
      )}
      {...props}
    />
  );
}
```

## Input Component

```typescript
import { TextField, Label, Input as AriaInput } from 'react-aria-components';
import { cn } from '@/lib/utils';

interface InputProps {
  label?: string;
  placeholder?: string;
  errorMessage?: string;
}

export function Input({ label, placeholder, errorMessage, className, ...props }) {
  return (
    <TextField className={cn('flex flex-col gap-1', className)} {...props}>
      {label && <Label className="text-sm font-medium">{label}</Label>}
      <AriaInput
        placeholder={placeholder}
        className={cn(
          'flex h-10 w-full rounded-md border border-input',
          'bg-background px-3 py-2 text-sm',
          'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
          'disabled:cursor-not-allowed disabled:opacity-50',
          errorMessage && 'border-destructive'
        )}
      />
      {errorMessage && <p className="text-sm text-destructive">{errorMessage}</p>}
    </TextField>
  );
}
```

## Modal Component

```typescript
import { Dialog, Modal as AriaModal, ModalOverlay, Heading } from 'react-aria-components';
import { cn } from '@/lib/utils';

export function Modal({ isOpen, onClose, title, children, size = 'md' }) {
  return (
    <ModalOverlay
      isOpen={isOpen}
      onOpenChange={onClose}
      className="fixed inset-0 z-50 bg-background/80 backdrop-blur-sm"
    >
      <AriaModal
        className={cn(
          'fixed left-[50%] top-[50%] z-50 translate-x-[-50%] translate-y-[-50%]',
          'w-full bg-background border rounded-lg shadow-lg',
          size === 'sm' && 'max-w-sm',
          size === 'md' && 'max-w-md',
          size === 'lg' && 'max-w-lg'
        )}
      >
        <Dialog className="outline-none">
          {title && <Heading className="text-lg font-semibold p-6 pb-4">{title}</Heading>}
          <div className="p-6 pt-0">{children}</div>
        </Dialog>
      </AriaModal>
    </ModalOverlay>
  );
}
```

## Dark Mode

```typescript
// app/providers.tsx
'use client';
import { ThemeProvider } from 'next-themes';

export function Providers({ children }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
      {children}
    </ThemeProvider>
  );
}
```

## CSS Variables

```css
/* globals.css */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --muted: 210 40% 96.1%;
    --destructive: 0 84.2% 60.2%;
    --border: 214.3 31.8% 91.4%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    /* ... */
  }
}
```

## Best Practices

1. **Always use `cn()`** for className merging
2. **Include labels** for accessibility
3. **Use CSS variables** for theming (not hardcoded colors)
4. **Organize components**: `ui/`, `dashboard/`, `marketing/`

## Tips

- Copy components you need, don't install everything
- React Aria handles accessibility automatically
- Match existing patterns in your codebase
- PRO license includes Figma files for design sync

## How to Verify

### Quick Checks
- Components render without errors
- Dark mode toggle works
- Keyboard navigation works (Tab, Enter, Escape)
- Screen reader announces components correctly

### Common Issues
- Styling issues: Check CSS variables are defined
- Accessibility warnings: Ensure labels are provided
- Hydration errors: Add `suppressHydrationWarning` to html tag
