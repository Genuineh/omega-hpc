---
name: frontend-state-management
description: Zustand State Management Guide. Covers store organization, persistence, and selector optimization.
version: 1.0.0
---

# Zustand State Management Guide

Guide for using Zustand for state management.

---

## 1. Store Organization

### Atomic Stores

```typescript
// stores/userStore.ts
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  isLoading: boolean;
  setUser: (user: User | null) => void;
  setLoading: (loading: boolean) => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  isLoading: false,
  setUser: (user) => set({ user }),
  setLoading: (isLoading) => set({ isLoading }),
}));
```

### Feature-Based Stores

```
stores/
├── authStore.ts      # Authentication
├── cartStore.ts      # Shopping cart
├── uiStore.ts        # UI state (modals, theme)
└── userStore.ts      # User data
```

---

## 2. Persistence

### Basic Persistence

```typescript
import { persist, createJSONStorage } from 'zustand/middleware';
import { persist } from 'zustand/middleware';

interface SettingsState {
  theme: 'light' | 'dark';
  language: string;
  setTheme: (theme: 'light' | 'dark') => void;
}

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'settings-storage',
      storage: createJSONStorage(() => localStorage),
    }
  )
);
```

### Tauri Persistence

```typescript
import { persist, createJSONStorage } from 'zustand/middleware';
import { Storage } from '@tauri-apps/api/store';

const tauriStorage = {
  getItem: async (name: string) => {
    const store = await Storage.load({ dir: 'appData' });
    return await store.get(name);
  },
  setItem: async (name: string, value: string) => {
    const store = await Storage.load({ dir: 'appData' });
    await store.set(name, value);
    await store.save();
  },
  removeItem: async (name: string) => {
    const store = await Storage.load({ dir: 'appData' });
    await store.delete(name);
    await store.save();
  },
};

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({ ... }),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => tauriStorage),
    }
  )
);
```

---

## 3. Selectors

### Basic Usage

```typescript
// Component subscribes to specific slice
function UserName() {
  const name = useUserStore((state) => state.user?.name);
  return <span>{name}</span>;
}
```

### Optimized Selectors

```typescript
// Avoid: Recreates on every render
const name = useUserStore((state) => state.user?.name);

// Better: Use selector function
const user = useUserStore((state) => state.user);
const name = user?.name;

// Best: Use shallow for arrays
import { shallow } from 'zustand/shallow';
const items = useStore((state) => state.items, shallow);
```

### Selector Pattern

```typescript
// stores/userStore.ts
export const useUserName = () => useUserStore((s) => s.user?.name);
export const useUserLoading = () => useUserStore((s) => s.isLoading);
export const useSetUser = () => useUserStore((s) => s.setUser);
```

---

## 4. Best Practices

- **Single Responsibility**: One store per feature
- **Immutability**: Always return new objects
- **Computed Values**: Calculate in components or derive
- **Persistence**: Persist only what's needed

---

## Checklist

- [ ] Stores organized by feature
- [ ] Persistence configured for settings
- [ ] Selectors used properly
- [ ] State immutability maintained
- [ ] Types defined
