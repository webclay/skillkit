# Rich Text Customization and Custom React Components

Reference for Payload's Lexical rich text editor customization and injecting custom React components into the admin UI.

## Rich Text Customization

Payload uses Lexical and supports custom blocks within rich text.

### Custom Blocks (Full-Width)

```typescript
// Define the block
const CarHighlight = {
  slug: 'carHighlight',
  fields: [
    {
      name: 'car',
      type: 'relationship',
      relationTo: 'cars',
      required: true,
    },
    {
      name: 'highlightType',
      type: 'select',
      options: ['image', 'gallery', 'specs'],
      defaultValue: 'image',
    }
  ],
};

// Add to rich text field
{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      ...defaultFeatures,
      BlocksFeature({
        blocks: [CarHighlight],
      }),
    ],
  }),
}
```

### Inline Blocks

```typescript
// Inline block for dynamic references
const CarReference = {
  slug: 'carReference',
  fields: [
    {
      name: 'car',
      type: 'relationship',
      relationTo: 'cars',
      required: true,
    },
    {
      name: 'displayField',
      type: 'select',
      options: ['title', 'price', 'year'],
    }
  ],
};

// Add to rich text as inline
{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      ...defaultFeatures,
      InlineBlocksFeature({
        inlineBlocks: [CarReference],
      }),
    ],
  }),
}
```

Editors can insert these blocks directly in the text flow, and values populate dynamically on the frontend.

## Custom React Components

Inject custom React components anywhere in the admin UI.

```typescript
// Custom label component
const CustomLabel: React.FC = () => {
  const { value: carId } = useField<string>({ path: 'car' });
  const [carTitle, setCarTitle] = useState('');

  useEffect(() => {
    if (carId) {
      fetch(`/api/cars/${carId}`)
        .then(r => r.json())
        .then(data => setCarTitle(data.title));
    }
  }, [carId]);

  return <span>{carTitle || 'Select a car'}</span>;
};

// Use in field definition
{
  name: 'car',
  type: 'relationship',
  relationTo: 'cars',
  admin: {
    components: {
      Label: CustomLabel,
    }
  }
}
```

You can customize:
- `Label` - Field label
- `Field` - Entire field component
- `Error` - Error message display
- `Description` - Field description
