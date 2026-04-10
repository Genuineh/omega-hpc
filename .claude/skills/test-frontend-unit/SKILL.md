---
name: test-frontend-unit
description: Frontend Unit Testing Guide. Covers unit testing strategies, testing libraries, and implementation for frontend components and logic. Use when setting up or writing unit tests for frontend code.
version: 1.0.0
---

# Frontend Unit Testing Skill

Skill for creating and maintaining unit tests for frontend applications.

## Purpose

Frontend unit tests verify that individual components and functions work correctly:
- Test component rendering
- Test user interactions
- Test state changes
- Test utility functions
- Ensure code quality

## When to Use

- Setting up unit testing for a frontend project
- Writing tests for React/Vue components
- Testing utility functions and hooks
- Improving test coverage
- Debugging failing tests

---

## Test Levels

| Level | Scope | Speed | Purpose |
|-------|-------|-------|---------|
| **Unit** | Single function/component | Fast | Verify isolated logic |
| **Integration** | Multiple components | Medium | Verify component interactions |
| **E2E** | Full application | Slow | Verify user flows |

### When to Use Unit Tests

- Testing pure functions
- Testing component logic
- Testing hooks
- Testing utility functions
- Testing state management logic
- Test-driven development (TDD)

---

## Tool Selection

### Popular Testing Libraries

| Library | Framework | Key Features |
|---------|-----------|---------------|
| **Vitest** | All (Vite) | Fast, Vite integration, Jest-compatible |
| **Jest** | All | Mature, snapshot testing, mocking |
| **Mocha** | All | Flexible, browser support |
| **React Testing Library** | React | Accessible, user-centric |
| **Vue Test Utils** | Vue | Vue-specific, composition API |
| **Testing Library** | All | Framework-agnostic, accessible |

### Recommendation

- **Vite projects**: Vitest (recommended)
- **React projects**: Vitest + React Testing Library
- **Vue projects**: Vitest + Vue Test Utils
- **Legacy projects**: Jest

---

## Vitest Guide (Recommended)

### Setup

```bash
# Install Vitest
npm install -D vitest

# With React
npm install -D @vitejs/plugin-react vitest

# Add to vite.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
  },
});
```

### Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      include: ['src/**/*.{ts,tsx}'],
      exclude: ['src/test/**/*'],
    },
    include: ['src/**/*.{test,spec}.{js,ts}'],
    exclude: ['node_modules', 'dist'],
  },
});
```

### Setup File

```typescript
// src/test/setup.ts
import { expect, afterEach, vi } from 'vitest';
import { cleanup } from '@testing-library/react';
import '@testing-library/jest-dom';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});
```

---

## React Testing Library

### Basic Component Test

```tsx
// Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Testing Form Components

```tsx
// LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('shows validation errors for empty fields', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: /submit/i }));

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
    expect(await screen.findByText(/password is required/i)).toBeInTheDocument();
  });

  it('submits form with valid data', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /submit/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });
});
```

### Testing Async Operations

```tsx
// UserList.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { UserList } from './UserList';

// Mock API
vi.mock('./api', () => ({
  fetchUsers: vi.fn(),
}));

import { fetchUsers } from './api';

describe('UserList', () => {
  it('shows loading state', () => {
    vi.mocked(fetchUsers).mockImplementation(
      () => new Promise(() => {}) // Never resolves
    );

    render(<UserList />);
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('displays users after loading', async () => {
    vi.mocked(fetchUsers).mockResolvedValue([
      { id: 1, name: 'John Doe' },
      { id: 2, name: 'Jane Doe' },
    ]);

    render(<UserList />);

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('Jane Doe')).toBeInTheDocument();
    });
  });

  it('shows error state on failure', async () => {
    vi.mocked(fetchUsers).mockRejectedValue(new Error('Failed to fetch'));

    render(<UserList />);

    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

---

## Hooks Testing

### Testing Custom Hooks

```tsx
// useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });
});
```

### Testing Hooks with Dependencies

```tsx
// useLocalStorage.test.ts
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { useLocalStorage } from './useLocalStorage';

// Mock localStorage
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn(),
  clear: vi.fn(),
};

beforeEach(() => {
  vi.stubGlobal('localStorage', localStorageMock);
  localStorageMock.getItem.mockReturnValue(null);
  localStorageMock.setItem.mockReturnValue(undefined);
});

describe('useLocalStorage', () => {
  it('reads from localStorage', () => {
    localStorageMock.getItem.mockReturnValue('"initial value"');

    const { result } = renderHook(() => useLocalStorage('key', 'default'));
    expect(result.current[0]).toBe('initial value');
  });

  it('writes to localStorage on change', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'default'));

    act(() => {
      result.current[1]('new value');
    });

    expect(localStorageMock.setItem).toHaveBeenCalledWith(
      'key',
      '"new value"'
    );
  });
});
```

---

## Mocking

### Mocking Modules

```typescript
// Mock a module
vi.mock('./api', () => ({
  fetchData: vi.fn().mockResolvedValue({ data: 'mocked' }),
  postData: vi.fn().mockRejectedValue(new Error('Error')),
}));

// Mock with implementation
vi.mock('./utils', async () => {
  const actual = await vi.importActual('./utils');
  return {
    ...actual,
    formatDate: vi.fn((date) => '2024-01-01'),
  };
});
```

### Mocking Functions

```typescript
// Mock a single function
const mockFn = vi.fn();
mockFn.mockReturnValue('mocked return');
mockFn.mockResolvedValue('async mocked');
mockFn.mockImplementation((arg) => `processed: ${arg}`);
```

### Spying

```typescript
// Spy on object methods
const obj = { method: vi.fn() };
vi.spyOn(obj, 'method');

// Spy on module exports
import * as apiModule from './api';
vi.spyOn(apiModule, 'fetchData');
```

---

## Vitest + Vue

### Setup

```bash
npm install -D @vue/test-utils vitest
```

### Component Test

```vue
// Button.spec.ts
import { describe, it, expect } from 'vitest';
import { mount } from '@vue/test-utils';
import { Button } from './Button.vue';

describe('Button', () => {
  it('renders properly', () => {
    const wrapper = mount(Button, {
      slots: { default: 'Click me' },
    });

    expect(wrapper.text()).toContain('Click me');
  });

  it('emits click event', async () => {
    const wrapper = mount(Button);

    await wrapper.trigger('click');

    expect(wrapper.emitted('click')).toBeTruthy();
  });

  it('applies variant classes', () => {
    const wrapper = mount(Button, {
      props: { variant: 'primary' },
    });

    expect(wrapper.classes()).toContain('btn-primary');
  });
});
```

---

## Tauri Testing

Tauri applications have two distinct test targets:
1. **Frontend** (TypeScript/JavaScript) - Vitest
2. **Rust Core** - Cargo test

### Frontend Testing with Vitest

Same as standard frontend testing, but requires mocking Tauri APIs.

#### Setup

```bash
# Install Vitest for Tauri
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

#### Configuration

```typescript
// vite.config.ts (for Tauri)
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{js,ts,jsx,tsx}'],
    // Mock Tauri APIs
    globals: {
      __TAURI_INTERNALS__: {},
    },
  },
});
```

### Mocking Tauri APIs

Tauri provides APIs like `invoke`, `window`, `path`, etc. that need mocking in tests.

#### Mocking invoke

```typescript
// src/test/tauri-mocks.ts
import { vi } from 'vitest';

// Mock @tauri-apps/api/core
vi.mock('@tauri-apps/api/core', () => ({
  invoke: vi.fn(),
}));

// Mock @tauri-apps/api/window
vi.mock('@tauri-apps/api/window', () => ({
  getCurrentWindow: vi.fn(() => ({
    close: vi.fn(),
    minimize: vi.fn(),
    maximize: vi.fn(),
    unmaximize: vi.fn(),
    isMaximized: vi.fn().mockResolvedValue(false),
  })),
  Window: vi.fn(),
}));

// Mock @tauri-apps/api/path
vi.mock('@tauri-apps/api/path', () => ({
  appDataDir: vi.fn().mockResolvedValue('/mock/data/dir'),
  appConfigDir: vi.fn().mockResolvedValue('/mock/config/dir'),
}));
```

#### Mocking in Setup

```typescript
// src/test/setup.ts
import { expect, vi, beforeAll, afterEach, afterAll } from 'vitest';
import { cleanup } from '@testing-library/react';
import '@testing-library/jest-dom';

// Mock console.error for unhandled rejections
const originalError = console.error;
beforeAll(() => {
  console.error = (...args: unknown[]) => {
    if (
      typeof args[0] === 'string' &&
      args[0].includes('Warning: React')
    ) {
      return;
    }
    originalError.call(console, ...args);
  };
});

afterEach(() => {
  cleanup();
});

afterAll(() => {
  console.error = originalError;
});
```

#### Testing Tauri Components

```tsx
// TauriButton.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { TauriButton } from './TauriButton';

// Import mocked invoke
import { invoke } from '@tauri-apps/api/core';

vi.mock('@tauri-apps/api/core');

describe('TauriButton', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('calls invoke on click', async () => {
    vi.mocked(invoke).mockResolvedValue({ success: true });

    render(<TauriButton command="open-file">Open File</TauriButton>);

    await fireEvent.click(screen.getByRole('button'));

    expect(invoke).toHaveBeenCalledWith('open-file');
  });

  it('shows loading state while waiting', async () => {
    vi.mocked(invoke).mockImplementation(
      () => new Promise(() => {}) // Never resolves
    );

    render(<TauriButton command="slow-command">Click</TauriButton>);

    await fireEvent.click(screen.getByRole('button'));

    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  it('handles error from invoke', async () => {
    vi.mocked(invoke).mockRejectedValue(new Error('Command failed'));

    render(<TauriButton command="failing-command">Click</TauriButton>);

    await fireEvent.click(screen.getByRole('button'));

    expect(screen.getByText(/error/i)).toBeInTheDocument();
  });
});
```

#### Testing File Operations

```tsx
// FileSelector.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { invoke } from '@tauri-apps/api/core';

vi.mock('@tauri-apps/api/core');

describe('FileSelector', () => {
  it('opens file dialog and returns path', async () => {
    vi.mocked(invoke).mockResolvedValue(['/selected/file.txt']);

    render(<FileSelector />);

    await screen.findByRole('button', { name: /select file/i });
    // ... test implementation
  });

  it('handles empty selection', async () => {
    vi.mocked(invoke).mockResolvedValue([]);

    render={<FileSelector />};
    // ... test implementation
  });
});
```

---

## Rust Core Testing (Cargo Test)

For Tauri applications, the Rust core requires its own testing approach.

### Test Organization

```
src-tauri/
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── commands/
│       ├── mod.rs
│       └── user.rs
├── tests/
│   └── integration_test.rs
└── Cargo.toml
```

### Unit Tests in Rust

```rust
// src/commands/user.rs

/// Validates user input
fn validate_username(username: &str) -> Result<(), String> {
    if username.is_empty() {
        return Err("Username cannot be empty".to_string());
    }
    if username.len() < 3 {
        return Err("Username must be at least 3 characters".to_string());
    }
    Ok(())
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_valid_username() {
        assert!(validate_username("john").is_ok());
    }

    #[test]
    fn test_empty_username() {
        let result = validate_username("");
        assert!(result.is_err());
        assert_eq!(result.unwrap_err(), "Username cannot be empty");
    }

    #[test]
    fn test_short_username() {
        let result = validate_username("ab");
        assert!(result.is_err());
    }
}
```

### Testing Commands

```rust
// src/commands/user.rs
use tauri::command;

#[command]
pub fn get_user(id: u32) -> Result<User, String> {
    if id == 0 {
        return Err("Invalid ID".to_string());
    }
    Ok(User { id, name: "John".to_string() })
}

#[cfg(test)]
mod command_tests {
    use super::*;

    #[test]
    fn test_get_user_valid_id() {
        let result = get_user(1);
        assert!(result.is_ok());
        assert_eq!(result.unwrap().id, 1);
    }

    #[test]
    fn test_get_user_invalid_id() {
        let result = get_user(0);
        assert!(result.is_err());
    }
}
```

### Integration Tests

```rust
// tests/integration_test.rs
use tauri::test::mock::TestWindow;
use tauri::Manager;

#[test]
fn test_app_launch() {
    // Test that the app can launch successfully
    // This requires the test feature in Cargo.toml
}

#[tauri::test]
async fn test_command_invoke() {
    // Use Tauri's test helper
    // This runs a test instance of the Tauri app
}
```

### Cargo.toml Test Configuration

```toml
[features]
default = ["custom-protocol"]
custom-protocol = ["tauri/custom-protocol"]

[dependencies]
tauri = { version = "1.6", features = ["test"] }

[dev-dependencies]
# For integration testing
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full", "test-util"] }
```

### Running Tests

```bash
# Run all tests
cargo test

# Run tests for specific module
cargo test commands

# Run tests with output
cargo test -- --nocapture

# Run doc tests
cargo test --doc

# Run with coverage
cargo tarpaulin --out Html

# Run specific test
cargo test test_get_user_valid_id
```

### Test Patterns

```rust
// Test with setup/teardown
#[cfg(test)]
mod tests {
    use super::*;

    // Setup before all tests
    setup!(my_fixture);

    #[test]
    fn test_one() {
        // Test code
    }

    #[test]
    fn test_two() {
        // Test code
    }
}

// Or use a simple approach
fn setup_test_db() -> DbConnection {
    // Create test database
}

fn teardown_test_db(conn: DbConnection) {
    // Cleanup
}

#[test]
fn test_with_db() {
    let conn = setup_test_db();
    // Test code
    teardown_test_db(conn);
}
```

### Mocking in Rust

```rust
// Using mockall for Rust
use mockall::mock;
use mockall::predicate::*;

// Mock a trait
mock! {
    pub UserRepository {
        fn find_by_id(&self, id: u32) -> Option<User>;
        fn save(&self, user: User) -> Result<(), Error>;
    }
}

#[test]
fn test_user_service() {
    let mut mock_repo = MockUserRepository::new();
    mock_repo.expect_find_by_id()
        .with(eq(1))
        .return_const(Some(User { id: 1, name: "John".to_string() }));

    let service = UserService::new(mock_repo);
    let user = service.get_user(1).unwrap();
    assert_eq!(user.name, "John");
}
```

### Best Practices for Rust Tests

```rust
// 1. Test public API, not implementation
#[test]
fn test_public_interface() {
    // Test the command function
    let result = my_command_function(input);
    assert!(result.is_ok());
}

// 2. Test error cases
#[test]
fn test_error_handling() {
    let result = my_function(invalid_input);
    assert!(result.is_err());
}

// 3. Use descriptive test names
#[test]
fn test_command_returns_error_when_user_not_found() {
    // Test implementation
}

// 4. Keep tests independent
#[test]
fn test_each_independent() {
    // Each test should setup its own data
}
```

---

## Code Coverage

### Coverage Configuration

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      include: [
        'src/**/*.{ts,tsx}',
        'src/**/*.vue',
      ],
      exclude: [
        'src/**/*.d.ts',
        'src/**/index.ts',
        'src/test/**',
        'src/**/*.stories.tsx',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },
    },
  },
});
```

### Running Coverage

```bash
# Run coverage
npx vitest run --coverage

# Open HTML report
npx vitest --coverage --open
```

---

## Best Practices

### Test Structure

```typescript
describe('ComponentName', () => {
  // Setup for all tests in this describe block
  beforeEach(() => {
    // Setup code
  });

  describe('when condition A', () => {
    // Tests for specific condition
    it('should do X', () => { });
  });

  describe('when condition B', () => {
    it('should do Y', () => { });
  });
});
```

### Naming Conventions

```typescript
// Good: Descriptive names
describe('calculateTotal', () => {
  it('returns sum of all items', () => { });
  it('returns 0 for empty array', () => { });
  it('throws error for negative prices', () => { });
});

// Bad: Vague names
describe('calculateTotal', () => {
  it('works', () => { });
  it('test1', () => { });
});
```

### AAA Pattern

```typescript
it('should add item to cart', () => {
  // Arrange
  const cart = new Cart();
  const item = { id: 1, name: 'Test', price: 10 };

  // Act
  cart.add(item);

  // Assert
  expect(cart.items).toHaveLength(1);
  expect(cart.total).toBe(10);
});
```

### What to Test

| Test | Don't Test |
|------|------------|
| Public API | Implementation details |
| User interactions | Internal state |
| Rendered output | Private methods |
| Edge cases | Third-party code |
| Error states | Styles (visual regression) |

---

## CI/CD Integration

### GitHub Actions

```yaml
name: Unit Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:coverage
```

### Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "test:watch": "vitest"
  }
}
```

---

## Integration with Other Skills

When writing unit tests, combine with:

- `frontend-framework-nextjs` - For Next.js specific testing
- `frontend-tauri-native` - For Tauri desktop app testing
- `frontend-state-management` - For testing Zustand/Redux
- `frontend-api-integration` - For testing API calls
- `project-rust` - For Rust core testing with Cargo
- `test-e2e` - For E2E test coverage

---

## Self-Check / Validation

### Validation Commands

```bash
# Run tests
npm run test:run

# Run with coverage
npm run test:coverage

# Watch mode
npm run test:watch

# UI mode
npm run test:ui

# Run specific file
npx vitest run src/utils/format.test.ts
```

### Validation Checklist

- [ ] Tests follow AAA pattern (Arrange, Act, Assert)
- [ ] Tests are independent (no shared state)
- [ ] Tests use user-centric queries (getByRole, getByLabelText)
- [ ] Tests cover happy path and edge cases
- [ ] Tests are descriptive and readable
- [ ] Mocks are properly isolated
- [ ] Coverage meets thresholds

### Test Quality Guidelines

| Check | Standard |
|-------|----------|
| Query priority | Accessible queries (role > label > text) |
| Naming | Describes behavior, not implementation |
| Isolation | Each test can run independently |
| Assertions | Specific, meaningful |
| Coverage | Critical paths covered |
| Mocking | Mock external dependencies only |
