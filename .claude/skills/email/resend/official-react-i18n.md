# Internationalization (i18n) Guide

Complete guide for implementing multi-language email support with React Email.

React Email officially supports three popular i18n libraries: next-intl, react-i18next, and react-intl.

## next-intl (Recommended for Next.js)

### Installation

```bash
npm install next-intl
```

### Setup

1. Create message files (`messages/en.json`, `messages/es.json`, etc.)
2. Use `createTranslator` in email template
3. Pass `locale` prop when sending

```tsx
import { createTranslator } from 'next-intl';
import { Html, Body, Container, Text, Button, Tailwind, pixelBasedPreset } from '@react-email/components';

interface EmailProps {
  name: string;
  locale: string;
}

export default async function WelcomeEmail({ name, locale }: EmailProps) {
  const t = createTranslator({
    messages: await import(`../messages/${locale}.json`),
    namespace: 'welcome-email',
    locale
  });

  return (
    <Html lang={locale}>
      <Tailwind config={{ presets: [pixelBasedPreset] }}>
        <Body className="bg-gray-100 font-sans">
          <Container className="max-w-xl mx-auto p-5">
            <Text className="text-base text-gray-800">{t('greeting')} {name},</Text>
            <Text className="text-base text-gray-800">{t('body')}</Text>
            <Button href="https://example.com" className="bg-blue-600 text-white px-5 py-3 rounded">
              {t('cta')}
            </Button>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  );
}
```

## react-intl (FormatJS)

Good choice for complex formatting needs (plurals, dates, numbers). Use `createIntl` from 'react-intl'.

## react-i18next

Best for non-Next.js applications. Requires `react-i18next`, `i18next`, and `i18next-resources-to-backend`.

## Best Practices

1. **Always pass locale** - Make locale a required prop
2. **Set HTML lang attribute** - `<Html lang={locale}>`
3. **Support RTL languages** - Check for Arabic, Hebrew, etc. and set `dir="rtl"`
4. **Provide fallback translations** - Handle missing translations gracefully
5. **Test all locales** - Verify rendering for each supported language
6. **Keep keys consistent** - Same keys across all locale files
7. **Translate subject lines** - Don't forget email subjects
8. **Format consistency** - Use `Intl` APIs for locale-specific date/number formatting
