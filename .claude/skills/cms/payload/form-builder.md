# Form Builder Plugin

Reference for the Payload CMS Form Builder plugin - create forms in the admin, manage submissions, and render forms on the frontend.

## Installation

```bash
npm install @payloadcms/plugin-form-builder
```

## Configuration

```typescript
// payload.config.ts
import { formBuilderPlugin } from '@payloadcms/plugin-form-builder';

export default buildConfig({
  plugins: [
    formBuilderPlugin({
      fields: {
        // Use default fields (text, email, textarea, select, checkbox, etc.)
      },
      // Optional: customize form submissions collection
      formSubmissionOverrides: {
        slug: 'contact-submissions',
      },
    }),
  ],
});
```

This creates two collections automatically:
- **forms** - Where you create and manage forms
- **form-submissions** - Where form submissions are stored

## Creating Forms in Admin

1. Go to **Forms** collection in admin
2. Click **Create New**
3. Add a title (e.g., "Newsletter Signup")
4. Add fields (text, email, textarea, select, checkbox, etc.)
5. Configure submit button label
6. Add confirmation message (rich text)
7. Save

**Example form fields:**
- First Name (text, required)
- Last Name (text, required)
- Email (email, required)
- Submit button label: "Sign Up"
- Confirmation: "Thanks for signing up!"

## Using Forms in Blocks

**Form Block Definition:**

```typescript
// blocks/FormBlock.ts
import { Block } from 'payload';

export const FormBlock: Block = {
  slug: 'form',
  fields: [
    {
      name: 'heading',
      type: 'text',
    },
    {
      name: 'form',
      type: 'relationship',
      relationTo: 'forms',
      required: true,
    },
  ],
};
```

Add to your Pages collection's layout blocks, and editors can select which form to display.

## Rendering Forms on Frontend

```typescript
// components/FormBlock.tsx
'use client';

import { useState, FormEvent } from 'react';
import { RichText } from '@payloadcms/richtext-lexical/react';

interface FormBlockProps {
  block: {
    heading?: string;
    form: {
      id: string;
      title: string;
      fields: Array<{
        name: string;
        label: string;
        fieldType: string;
        required: boolean;
      }>;
      submitButtonLabel: string;
      confirmationMessage: any; // Rich text
    };
  };
}

export function FormBlock({ block }: FormBlockProps) {
  const { heading, form } = block;
  const [formState, setFormState] = useState({
    loading: false,
    error: null,
    success: false,
  });

  const handleSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setFormState({ loading: true, error: null, success: false });

    const formData = new FormData(e.currentTarget);
    const data = Object.fromEntries(formData.entries());

    try {
      // Submit to Payload's form submission endpoint
      const response = await fetch('/api/form-submissions', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          form: form.id,
          submissionData: Object.entries(data).map(([field, value]) => ({
            field,
            value: value as string,
          })),
        }),
      });

      if (!response.ok) throw new Error('Failed to submit form');

      setFormState({ loading: false, error: null, success: true });
      e.currentTarget.reset();

      // Clear success message after 5 seconds
      setTimeout(() => {
        setFormState({ loading: false, error: null, success: false });
      }, 5000);
    } catch (error) {
      setFormState({
        loading: false,
        error: 'Failed to submit form',
        success: false,
      });
    }
  };

  return (
    <section className="form-block">
      {heading && <h2>{heading}</h2>}

      <form onSubmit={handleSubmit}>
        {form.fields.map((field) => (
          <div key={field.name} className="form-field">
            <label htmlFor={field.name}>{field.label}</label>
            <input
              id={field.name}
              name={field.name}
              type={field.fieldType}
              required={field.required}
              placeholder={field.label}
            />
          </div>
        ))}

        {formState.error && <p className="error">{formState.error}</p>}

        {formState.success ? (
          <div className="success">
            <RichText data={form.confirmationMessage} />
          </div>
        ) : (
          <button type="submit" disabled={formState.loading}>
            {formState.loading ? 'Submitting...' : form.submitButtonLabel}
          </button>
        )}
      </form>
    </section>
  );
}
```

## Viewing Form Submissions

All submissions appear in the **Form Submissions** collection in the admin panel, organized by form. Each submission shows:
- Which form it came from
- Submitted data
- Submission timestamp
- IP address (optional)
