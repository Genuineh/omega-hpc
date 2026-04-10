---
name: frontend-tauri-native
description: Tauri 2.0 Desktop Development Guide. Covers Tauri 2.0 setup, IPC, native capabilities, window management, plugins, security, and best practices for production apps.
version: 1.0.0
---

# Tauri 2.0 Development Guide

Guide for Tauri 2.0 desktop application development.

## Tauri 2.0 vs 1.x Key Differences

| Feature | Tauri 1.x | Tauri 2.0 |
|---------|-----------|-----------|
| Plugin System | Manual setup | Built-in plugin system |
| Capabilities | Manual in Rust | JSON-based capability files |
| Window API | `window` | `WebviewWindow` |
| App Handle | `App` | `AppHandle` |
| Frontend APIs | `@tauri-apps/api` | `@tauri-apps/api` + `@tauri-apps/plugin-*` |
| Mobile | Limited | First-class support |

---

## 1. Project Setup

### Initialize Tauri 2.0 Project

```bash
# Create with npm create tauri-app
npm create tauri-app@latest

# Or with pnpm
pnpm create tauri-app

# Or with yarn
yarn create tauri-app
```

### Select Options

```
Project name: my-app
Identifier: com.myapp.desktop
Frontend language: TypeScript / JavaScript
Package manager: npm / pnpm / yarn
```

### Manual Setup (Existing Project)

```bash
# Install Tauri CLI
npm install -D @tauri-apps/cli@latest

# Initialize Tauri
npx tauri init

# Install frontend API
npm install @tauri-apps/api@latest

# Install plugins as needed
npm install @tauri-apps/plugin-fs
npm install @tauri-apps/plugin-dialog
npm install @tauri-apps/plugin-shell
```

### Cargo Dependencies

```toml
# src-tauri/Cargo.toml
[lib]
name = "my_app_lib"
crate-type = ["staticlib", "cdylib", "rlib"]

[dependencies]
tauri = { version = "2", features = [] }
tauri-plugin-fs = "2"
tauri-plugin-dialog = "2"
tauri-plugin-shell = "2"
tauri-plugin-opener = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tokio = { version = "1", features = ["full"] }
```

---

## 2. Tauri 2.0 Configuration

### tauri.conf.json

```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "My App",
  "version": "1.0.0",
  "identifier": "com.myapp.desktop",
  "build": {
    "beforeDevCommand": "npm run dev",
    "devUrl": "http://localhost:5173",
    "beforeBuildCommand": "npm run build",
    "frontendDist": "../dist",
    "devtools": true
  },
  "app": {
    "withGlobalTauri": true,
    "windows": [
      {
        "title": "My App",
        "width": 1200,
        "height": 800,
        "minWidth": 800,
        "minHeight": 600,
        "resizable": true,
        "fullscreen": false,
        "center": true
      }
    ],
    "security": {
      "csp": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: asset: https://asset.localhost"
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": ""
    }
  }
}
```

---

## 3. Capabilities System

Tauri 2.0 introduces a new capability-based permission system.

### Create Capability File

```json
// src-tauri/capabilities/main.json
{
  "$schema": "https://schema.tauri.app/config/2/capability",
  "identifier": "main-capability",
  "description": "Main window capabilities",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:window:allow-close",
    "core:window:allow-minimize",
    "core:window:allow-toggle-maximize",
    "core:window:allow-set-size",
    "core:window:allow-set-position",
    "core:window:allow-center",
    "fs:default",
    "fs:allow-read-text-file",
    "fs:allow-write-text-file",
    "fs:allow-exists",
    "fs:allow-create",
    "fs:allow-mkdir",
    "dialog:default",
    "dialog:allow-open",
    "dialog:allow-save",
    "dialog:allow-message",
    "shell:default",
    "shell:allow-open"
  ]
}
```

### Permission Granularity

```json
{
  "permissions": [
    // Allow specific operations
    "fs:allow-read-text-file",
    "fs:allow-write-text-file",

    // Deny specific operations
    {
      "identifier": "fs:allow-write-text-file",
      "allow": [{ "path": "$APPDATA/**" }]
    },
    {
      "identifier": "fs:allow-write-text-file",
      "deny": [{ "path": "$HOME/**" }]
    }
  ]
}
```

### Scopes

```json
{
  "permissions": [
    {
      "identifier": "fs:scope",
      "allow": [
        { "path": "$APPDATA/**" },
        { "path": "$DOCUMENT/**" },
        { "path": "$DOWNLOAD/**" }
      ]
    }
  ]
}
```

---

## 4. IPC Commands

### Rust Commands (Tauri 2.0)

```rust
// src-tauri/src/lib.rs
use tauri::Manager;
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct User {
    pub id: u32,
    pub name: String,
    pub email: String,
}

#[tauri::command]
pub fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[tauri::command]
pub async fn get_user(id: u32) -> Result<User, String> {
    // Simulate async operation
    Ok(User {
        id,
        name: "John Doe".to_string(),
        email: "john@example.com".to_string(),
    })
}

#[tauri::command]
pub async fn process_batch(items: Vec<String>) -> Result<Vec<String>, String> {
    // Process items
    let results: Vec<String> = items
        .iter()
        .map(|item| format!("processed: {}", item))
        .collect();
    Ok(results)
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_shell::init())
        .invoke_handler(tauri::generate_handler![
            greet,
            get_user,
            process_batch,
        ])
        .setup(|app| {
            // App setup code
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Frontend Invocation

```typescript
// TypeScript/JavaScript
import { invoke } from '@tauri-apps/api/core';

// Basic call
const greeting = await invoke<string>('greet', { name: 'World' });
console.log(greeting); // "Hello, World!"

// With typed response
interface User {
  id: number;
  name: string;
  email: string;
}

const user = await invoke<User>('get_user', { id: 1 });
console.log(user.name); // "John Doe"

// Batch processing
const items = ['item1', 'item2', 'item3'];
const results = await invoke<string[]>('process_batch', { items });
console.log(results); // ["processed: item1", "processed: item2", "processed: item3"]
```

---

## 5. Plugin System

Tauri 2.0 uses a new plugin system. Install from npm and enable in Rust.

### Common Plugins

```bash
# Install plugins
npm install @tauri-apps/plugin-fs
npm install @tauri-apps/plugin-dialog
npm install @tauri-apps/plugin-shell
npm install @tauri-apps/plugin-http
npm install @tauri-apps/plugin-store
npm install @tauri-apps/plugin-log
npm install @tauri-apps/plugin-notification
npm install @tauri-apps/plugin-clipboard-manager
```

### Enable Plugins in Rust

```rust
// src-tauri/src/lib.rs
use tauri::Builder;

fn main() {
    Builder::default()
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_dialog::init())
        .plugin(tauri_plugin_shell::init())
        .plugin(tauri_plugin_http::init())
        .plugin(tauri_plugin_store::Builder::new().build())
        .plugin(tauri_plugin_log::Builder::new().build())
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Using Plugins

```typescript
// File System
import { readTextFile, writeTextFile } from '@tauri-apps/plugin-fs';

await writeTextFile('note.txt', 'Hello World');
const content = await readTextFile('note.txt');

// Dialog
import { open, save, message } from '@tauri-apps/plugin-dialog';

const file = await open({
  multiple: false,
  filters: [{ name: 'Text', extensions: ['txt'] }]
});

await message('Operation complete!', { title: 'Success' });

// HTTP
import { fetch } from '@tauri-apps/plugin-http';

const response = await fetch('https://api.example.com/data');
const data = await response.json();

// Store
import { Store } from '@tauri-apps/plugin-store';

const store = await Store.load('settings.json');
await store.set('theme', 'dark');
await store.save();
```

---

## 6. Window Management

### WebviewWindow (Tauri 2.0)

```typescript
import { WebviewWindow } from '@tauri-apps/api/webviewWindow';

// Create new window
const webview = new WebviewWindow('my-window', {
  url: 'https://example.com',
  title: 'New Window',
  width: 800,
  height: 600,
  center: true,
});

webview.once('tauri://created', () => {
  console.log('Window created');
});

webview.once('tauri://error', (e) => {
  console.error('Error creating window:', e);
});

// Get existing window
import { getCurrentWindow } from '@tauri-apps/api/window';

const win = getCurrentWindow();

// Minimize
await win.minimize();

// Toggle maximize
await win.toggleMaximize();

// Close
await win.close();

// Set size
await win.setSize({ width: 1024, height: 768 });

// Set position
await win.setPosition({ x: 100, y: 100 });

// Center
await win.center();

// Set title
await win.setTitle('New Title');

// Focus
await win.setFocus();
```

### Window Events

```typescript
import { getCurrentWindow } from '@tauri-apps/api/window';

const win = getCurrentWindow();

win.onCloseRequested(async (event) => {
  // Prevent close
  // event.preventDefault();

  // Or show confirmation
  const confirmed = await confirm('Are you sure you want to close?');
  if (!confirmed) {
    event.preventDefault();
  }
});

win.onResized(({ payload }) => {
  console.log('Resized:', payload.size);
});

win.onMoved(({ payload }) => {
  console.log('Moved:', payload.position);
});
```

---

## 7. System Tray

### Setup System Tray

```rust
// src-tauri/src/lib.rs
use tauri::{
    menu::{Menu, MenuItem},
    tray::{TrayIconBuilder, TrayIconEvent, MouseButton, MouseButtonState},
    Manager, Emitter,
};

fn setup_tray(app: &tauri::App) -> Result<(), Box<dyn std::error::Error>> {
    let quit = MenuItem::with_id(app, "quit", "Quit", true, None::<&str>)?;
    let show = MenuItem::with_id(app, "show", "Show Window", true, None::<&str>)?;
    let hide = MenuItem::with_id(app, "hide", "Hide Window", true, None::<&str>)?;

    let menu = Menu::with_items(app, &[&show, &hide, &quit])?;

    let _tray = TrayIconBuilder::new()
        .icon(app.default_window_icon().unwrap().clone())
        .menu(&menu)
        .menu_on_left_click(false)
        .on_menu_event(|app, event| {
            match event.id.as_ref() {
                "quit" => {
                    app.exit(0);
                }
                "show" => {
                    if let Some(window) = app.get_webview_window("main") {
                        let _ = window.show();
                        let _ = window.set_focus();
                    }
                }
                "hide" => {
                    if let Some(window) = app.get_webview_window("main") {
                        let _ = window.hide();
                    }
                }
                _ => {}
            }
        })
        .on_tray_icon_event(|tray, event| {
            if let TrayIconEvent::Click {
                button: MouseButton::Left,
                button_state: MouseButtonState::Up,
                ..
            } = event {
                let app = tray.app_handle();
                if let Some(window) = app.get_webview_window("main") {
                    let _ = window.show();
                    let _ = window.set_focus();
                }
            }
        })
        .build(app)?;

    Ok(())
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            setup_tray(app)?;
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## 8. Best Practices

### IPC Optimization

```typescript
// Bad: Multiple calls in loop
for (const item of items) {
  await invoke('process_item', { item });
}

// Good: Batch processing
await invoke('process_items', { items });
```

### Large Data Transfer

```typescript
// Use binary for large data
const buffer = await invoke<number[]>('get_large_data');

// For files, use file paths instead of content
await invoke('process_file', { path: '/path/to/file' });
```

### Error Handling

```typescript
try {
  const result = await invoke<string>('my_command', { data });
} catch (error) {
  console.error('Command failed:', error);
}
```

### Type Safety

```typescript
// Use TypeScript for type safety
interface CommandResult {
  success: boolean;
  data?: unknown;
  error?: string;
}

const result = await invoke<CommandResult>('my_command');
```

### Lazy Loading

```typescript
// Only load heavy modules when needed
async function handleHeavyOperation() {
  const { heavyModule } = await import('./heavyModule');
  heavyModule.doSomething();
}
```

---

## 9. Testing

### Unit Testing

Use `test-frontend-unit` skill with Tauri API mocking:

```typescript
// Mock Tauri invoke
vi.mock('@tauri-apps/api/core', () => ({
  invoke: vi.fn(),
}));
```

### Integration Testing

```rust
// src-tauri/src/lib.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_greet() {
        assert_eq!(greet("World"), "Hello, World!");
    }
}
```

### E2E Testing

Use `test-e2e` skill:

```typescript
// Test with Playwright
import { test, expect } from '@playwright/test';

test('app launches correctly', async ({ page }) => {
  // Launch app or navigate to dev server
  await page.goto('http://localhost:5173');

  // Test your app
  await expect(page.locator('h1')).toContainText('My App');
});
```

---

## 10. Build & Release

### Build Commands

```bash
# Development
npm run tauri dev

# Build for production
npm run tauri build

# Build specific target
npm run tauri build -- --target x86_64-pc-windows-msvc
npm run tauri build -- --target aarch64-apple-darwin
```

### Bundle Configuration

```json
// tauri.conf.json
{
  "bundle": {
    "active": true,
    "targets": ["msi", "nsis", "dmg", "app", "deb", "rpm", "appimage"],
    "category": "Utility",
    "shortDescription": "My App Description",
    "longDescription": "Detailed description of my application",
    "publisher": "My Company",
    "copyright": "Copyright 2024 My Company",
    "windows": {
      "certificateThumbprint": null,
      "digestAlgorithm": "sha256",
      "timestampUrl": ""
    }
  }
}
```

---

## 11. Integration with Other Skills

When working with Tauri, combine with:

- `test-frontend-unit` - Frontend unit testing with Vitest + Tauri mocks
- `test-e2e` - E2E testing with Playwright
- `project-rust` - Rust project management and Cargo
- `frontend-framework-nextjs` - If using Next.js as frontend
- `frontend-styling-twind` - Styling with Tailwind

---

## Checklist

- [ ] Project initialized with Tauri 2.0
- [ ] Capabilities configured properly
- [ ] IPC commands typed correctly
- [ ] Plugins installed and configured
- [ ] Error handling implemented
- [ ] Window management works
- [ ] Production build successful
- [ ] Tests written
