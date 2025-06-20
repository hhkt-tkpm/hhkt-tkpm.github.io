---
sidebar_position: 7
---

# Registering New Routes

## Overview

This document provides a comprehensive guide for adding new routes to the Student Management System frontend using Next.js 14 App Router. The system follows a feature-based routing structure with consistent patterns across all modules.

## App Router Structure

### Current Route Organization

```
app/
├── course-management/           # Course features
│   ├── page.tsx                # Course list page
│   └── [id]/                   # Dynamic course routes
│       └── page.tsx            # Course detail page
├── faculty-management/          # Faculty features
│   ├── page.tsx                # Faculty list page
│   └── [id]/                   # Dynamic faculty routes
│       └── page.tsx            # Faculty detail page
├── student-management/          # Student features
│   ├── page.tsx                # Student list page
│   └── [id]/                   # Dynamic student routes
│       └── page.tsx            # Student detail page
├── class-registration/          # Registration features
│   └── page.tsx                # Registration page
├── score-management/            # Grade management
│   └── page.tsx                # Score management page
├── layout.tsx                  # Root application layout
├── page.tsx                    # Dashboard/Home page
└── globals.css                 # Global styles
```

## Route Registration Patterns

### Basic Feature Route

When adding a new feature (e.g., `exam-management`), follow this structure:

```
app/exam-management/
├── page.tsx                    # Main listing page
├── [id]/                      # Dynamic routes
│   ├── page.tsx               # Detail/edit page
│   └── results/               # Nested feature
│       └── page.tsx           # Exam results page
├── create/                    # Static nested route
│   └── page.tsx               # Creation form page
└── components/                # Feature-specific components
    ├── ExamTable.tsx
    ├── ExamModal.tsx
    └── ExamForm.tsx
```

### Route Component Structure

Each route component follows this pattern:

#### Main Feature Page (`page.tsx`)
- **Purpose**: Primary listing/management interface
- **Components**: Data table, filters, action buttons
- **Features**: CRUD operations, search, export
- **Layout**: Uses shared layout with feature-specific content

#### Dynamic Detail Page (`[id]/page.tsx`)
- **Purpose**: Individual entity management
- **Parameters**: Dynamic route parameters
- **Features**: View/edit specific entity
- **Navigation**: Breadcrumb navigation

#### Nested Feature Pages
- **Purpose**: Related functionality
- **Structure**: Follows parent-child relationship
- **Features**: Context-aware operations

## Adding New Routes

### Step 1: Create Route Directory

Create the directory structure under `app/`:

```
app/new-feature/
├── page.tsx
├── [id]/
│   └── page.tsx
└── components/
```

### Step 2: Implement Page Components

Main feature page template:

```typescript
// app/new-feature/page.tsx
export default function NewFeaturePage() {
  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-semibold">
          {t('new-feature.title')}
        </h1>
        <Button type="primary" onClick={handleCreate}>
          {t('new-feature.add-new')}
        </Button>
      </div>
      
      <NewFeatureTable />
      <NewFeatureModal />
    </div>
  );
}
```

### Step 3: Add Navigation Integration

Update the navigation structure in the main layout to include the new route:

- Add navigation menu item in the sidebar
- Include proper icons and labels
- Set active state detection
- Add role-based access if needed

### Step 4: Configure Internationalization

Add translation keys to message files:

```json
// messages/en.json
{
  "new-feature": {
    "title": "New Feature Management",
    "add-new": "Add New Item",
    "edit": "Edit Item",
    "delete": "Delete Item"
  }
}
```

## Dynamic Route Patterns

### Single Parameter Routes

For entity-specific pages:

```
[id]/page.tsx           # /new-feature/123
[slug]/page.tsx         # /new-feature/item-name
[...params]/page.tsx    # /new-feature/any/nested/path
```

### Multiple Parameter Routes

For complex nested structures:

```
[category]/[id]/page.tsx        # /products/electronics/123
[faculty]/[program]/page.tsx    # /faculties/engineering/cs
```

### Optional Parameters

For flexible routing:

```
[[...slug]]/page.tsx    # Matches /new-feature and /new-feature/any/path
```

## Route Groups and Layouts

### Feature-Specific Layouts

Create shared layouts for related routes:

```
app/(management)/
├── layout.tsx              # Shared management layout
├── student-management/
├── course-management/
└── faculty-management/
```

### Authentication-Based Groups

Organize routes by access level:

```
app/(protected)/
├── layout.tsx              # Protected routes layout
├── admin/
├── teacher/
└── student/
```

### Layout Hierarchy

```
┌─────────────────────────────────────────┐
│            Root Layout                   │
├─────────────────────────────────────────┤
│        Feature Group Layout             │
├─────────────────────────────────────────┤
│         Feature Layout                  │
├─────────────────────────────────────────┤
│            Page Component               │
└─────────────────────────────────────────┘
```

## Route Configuration

### Page Structure Template

Every new route should follow this structure:

```typescript
// Standard imports
import React from 'react';
import { Button } from 'antd';
import { useTranslations } from 'next-intl';

// Feature-specific components
import FeatureTable from './components/FeatureTable';
import FeatureModal from './components/FeatureModal';

// Custom hooks
import { useFeatureData } from '@/libs/hooks/useFeatureData';

export default function FeaturePage() {
  const t = useTranslations('feature-name');
  const { data, isLoading } = useFeatureData();

  return (
    <div className="p-6 space-y-6">
      {/* Page Header */}
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-semibold text-gray-900">
          {t('title')}
        </h1>
        <Button type="primary">
          {t('add-new')}
        </Button>
      </div>

      {/* Main Content */}
      <FeatureTable data={data} loading={isLoading} />
      
      {/* Modals and Forms */}
      <FeatureModal />
    </div>
  );
}
```

## Existing Route Patterns Analysis

### Student Management Routes

```
app/student-management/
├── page.tsx                    # Student listing with search/filter
├── [id]/                      # Dynamic student routes
│   └── page.tsx               # Student detail/edit
└── components/                # Student-specific components
    ├── StudentTable.tsx       # Data table component
    ├── StudentModal.tsx       # Add/edit modal
    └── StudentForm.tsx        # Form component
```

### Course Management Routes

```
app/course-management/
├── page.tsx                   # Course catalog
├── [id]/                     # Course details
│   └── page.tsx              # Course information
└── components/               # Course components
    ├── CourseTable.tsx
    ├── CourseModal.tsx
    └── CourseForm.tsx
```

### Class Registration Routes

```
app/class-registration/
├── page.tsx                  # Registration interface
└── components/              # Registration components
    ├── RegistrationTable.tsx
    ├── RegistrationModal.tsx
    └── StudentSelector.tsx
```

## Route Metadata and SEO

### Page Metadata

Configure metadata for each route:

```typescript
export const metadata = {
  title: 'New Feature - Student Management System',
  description: 'Manage new feature items and configurations',
  keywords: ['management', 'feature', 'admin']
};
```

### Dynamic Metadata

For dynamic routes with entity-specific information:

```typescript
export async function generateMetadata({ params }) {
  const item = await fetchItem(params.id);
  return {
    title: `${item.name} - New Feature`,
    description: item.description
  };
}
```

## Loading and Error States

### Loading States

Implement loading UI for each route:

```typescript
// loading.tsx
export default function Loading() {
  return (
    <div className="space-y-4">
      <Skeleton.Input style={{ width: 200 }} active />
      <Skeleton active />
    </div>
  );
}
```

### Error Boundaries

Handle route-specific errors:

```typescript
// error.tsx
export default function Error({ error, reset }) {
  return (
    <div className="text-center py-8">
      <h2>Something went wrong!</h2>
      <Button onClick={reset}>Try again</Button>
    </div>
  );
}
```

### Not Found Pages

Custom 404 pages for specific routes:

```typescript
// not-found.tsx
export default function NotFound() {
  return (
    <div className="text-center py-8">
      <h2>Item Not Found</h2>
      <Link href="/new-feature">Back to List</Link>
    </div>
  );
}
```

## Route Integration Checklist

### Required Files
- [ ] Create `page.tsx` for main route
- [ ] Add `[id]/page.tsx` for dynamic routes (if needed)
- [ ] Create feature-specific components directory
- [ ] Add loading states (`loading.tsx`)
- [ ] Add error handling (`error.tsx`)

### Navigation Integration
- [ ] Update main navigation menu
- [ ] Add proper route icons
- [ ] Configure active state detection
- [ ] Test navigation flow

### Internationalization
- [ ] Add translation keys to message files
- [ ] Implement translation in components
- [ ] Test language switching
- [ ] Configure dynamic translation if needed

### API Integration
- [ ] Create API client functions
- [ ] Implement custom hooks
- [ ] Add React Query integration
- [ ] Test data fetching and mutations

### Component Development
- [ ] Create table component
- [ ] Implement modal/form components
- [ ] Add search and filter functionality
- [ ] Implement CRUD operations