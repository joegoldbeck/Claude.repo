# Frontend CLAUDE.md

This file provides specific guidance to Claude Code when working with the frontend code in this directory.

## Frontend Stack Overview

- **React 18** with TypeScript
- **Chakra UI** for component library
- **TanStack Router** for file-based routing
- **TanStack Query** for server state management
- **OpenAPI TypeScript Client** auto-generated from backend

## Key Commands

```bash
# Use correct Node version (required)
fnm use  # or nvm use

# Install dependencies
npm install

# Start development server
npm run dev

# Generate TypeScript client from backend API
npm run generate-client

# Run linting
npm run lint

# Build for production
npm run build

# Preview production build
npm run preview
```

## Project Structure

```
frontend/
├── src/
│   ├── client/           # Auto-generated API client (DO NOT EDIT)
│   ├── components/       # Reusable UI components
│   │   ├── Common/      # Shared components (Layout, Footer, etc.)
│   │   ├── Items/       # Item-specific components
│   │   └── UserSettings/ # User settings components
│   ├── hooks/           # Custom React hooks
│   ├── routes/          # File-based routing (TanStack Router)
│   └── theme.tsx        # Chakra UI theme configuration
├── tests/               # E2E tests (Playwright)
└── index.html          # Entry HTML file
```

## Important Development Patterns

### 1. API Client Usage

The TypeScript client is auto-generated. Always use it for API calls:

```typescript
import { ItemsService, UsersService } from "@/client"

// Example: Fetch items
const items = await ItemsService.readItems({ skip: 0, limit: 10 })

// Example: Create item
const newItem = await ItemsService.createItem({ 
  requestBody: { title: "New Item", description: "Description" }
})
```

### 2. Authentication Pattern

Use the `useAuth` hook for authentication:

```typescript
import useAuth from "@/hooks/useAuth"

function MyComponent() {
  const { user, login, logout, isLoading } = useAuth()
  
  if (!user) {
    // Redirect to login or show login prompt
  }
}
```

### 3. Data Fetching with TanStack Query

```typescript
import { useQuery, useMutation } from "@tanstack/react-query"
import { ItemsService } from "@/client"

// Query example
const { data, isLoading } = useQuery({
  queryKey: ["items"],
  queryFn: () => ItemsService.readItems({ skip: 0, limit: 10 }),
})

// Mutation example
const mutation = useMutation({
  mutationFn: (data: ItemCreate) => ItemsService.createItem({ requestBody: data }),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["items"] })
  },
})
```

### 4. Protected Routes

Routes requiring authentication use the route guard pattern:

```typescript
// In route file (e.g., src/routes/_layout/items.tsx)
export const Route = createFileRoute("/_layout/items")({
  beforeLoad: async () => {
    // This is handled by the parent _layout route
  },
})
```

### 5. Form Handling

Use Chakra UI form components with controlled inputs:

```typescript
import { useForm } from "react-hook-form"

function ItemForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<ItemCreate>()
  
  const onSubmit = (data: ItemCreate) => {
    // Handle form submission
  }
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <FormControl isInvalid={!!errors.title}>
        <FormLabel>Title</FormLabel>
        <Input {...register("title", { required: "Title is required" })} />
        <FormErrorMessage>{errors.title?.message}</FormErrorMessage>
      </FormControl>
    </form>
  )
}
```

## Common Tasks

### Adding a New Page

1. Create route file in `src/routes/` following the naming pattern
2. Use `createFileRoute` from TanStack Router
3. Add to navigation if needed in `src/components/Common/Sidebar.tsx`

### Working with API Endpoints

1. Ensure backend endpoint exists and is documented
2. Run `npm run generate-client` to update TypeScript client
3. Use the generated service methods in your components

### Styling Components

- Use Chakra UI components and props for styling
- Avoid custom CSS unless absolutely necessary
- Use the theme tokens for consistency (colors, spacing, etc.)

### State Management

- Server state: TanStack Query (for API data)
- Local state: React useState/useReducer
- Global state: React Context (sparingly, only for truly global state like auth)

## Testing Approach

- E2E tests use Playwright and are in the `tests/` directory
- Focus on user flows rather than implementation details
- Run with `npm run test:e2e`

## Performance Considerations

- Use React.lazy() for code splitting on route level
- Implement pagination for large lists
- Use TanStack Query's caching effectively
- Optimize images with proper sizing and formats

## Security Best Practices

- Never store sensitive data in localStorage
- Always validate user input
- Use the generated API client for type-safe requests
- Let the backend handle authorization logic

## Debugging Tips

- Check browser console for errors
- Use React DevTools for component inspection
- Use Network tab to debug API calls
- TanStack Query DevTools for query state inspection

## Common Pitfalls to Avoid

1. **Don't edit the generated client**: Changes will be overwritten
2. **Don't bypass TypeScript**: Use proper types, avoid `any`
3. **Don't create custom auth logic**: Use the existing `useAuth` hook
4. **Don't ignore Chakra UI**: Use it for consistent styling
5. **Don't forget to regenerate client**: After backend API changes