# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

Excalidraw is a **monorepo** using Yarn workspaces with clear separation between the core library and the application:

- **`packages/excalidraw/`** - Main React component library published to npm as `@excalidraw/excalidraw`
- **`excalidraw-app/`** - Full-featured web application (excalidraw.com) that uses the library
- **`packages/common/`** - Shared utilities and constants
- **`packages/element/`** - Element manipulation, rendering, and scene management
- **`packages/math/`** - Math utilities and geometric types
- **`packages/utils/`** - General utility functions
- **`examples/`** - Integration examples (NextJS, browser script)

## Development Commands

```bash
# Development
yarn start                # Start the app (excalidraw-app)
yarn build                # Build the app
yarn build:packages       # Build all packages (common → math → element → excalidraw)

# Testing
yarn test:app             # Run tests with Vitest
yarn test:update          # Run tests and update snapshots (REQUIRED before committing)
yarn test:typecheck       # TypeScript type checking
yarn test:coverage        # Generate coverage report
yarn test:ui              # Run tests with Vitest UI

# Linting and Formatting
yarn fix                  # Auto-fix formatting and linting issues
yarn fix:code             # Fix code with ESLint
yarn fix:other            # Fix with Prettier
yarn test:code            # Run ESLint
yarn test:other           # Check Prettier formatting

# Package Building (individual)
yarn build:common         # Build @excalidraw/common
yarn build:math           # Build @excalidraw/math
yarn build:element        # Build @excalidraw/element
yarn build:excalidraw     # Build @excalidraw/excalidraw
```

## Architecture

### Monorepo Structure

- Uses **Yarn workspaces** for dependency management
- Internal package imports use path aliases defined in `tsconfig.json` and `vitest.config.mts`
- TypeScript strict mode throughout
- Build system: **esbuild** for packages, **Vite** for the app

### Path Aliases

All packages can be imported using `@excalidraw/*` aliases:
```typescript
import { KEYS } from "@excalidraw/common";
import { GlobalPoint } from "@excalidraw/math";
import { getNonDeletedElements } from "@excalidraw/element";
```

### Element System

- **`packages/element/`** contains core element logic
- Elements are immutable - use `mutateElement()` for updates
- Scene management through `Scene.ts` in `@excalidraw/element`
- Element types: rectangle, ellipse, diamond, arrow, line, freedraw, text, image, frame, embeddable
- Key files:
  - `newElement.ts` - Element creation
  - `mutateElement.ts` - Element mutation
  - `renderElement.ts` - Element rendering
  - `Scene.ts` - Scene graph management
  - `store.ts` - Element store with observer pattern

### Actions System

- Actions are defined in `packages/excalidraw/actions/`
- Each action file registers keyboard shortcuts and UI elements
- Use `register()` to define actions
- Actions follow a predicate pattern to determine when they're available

### State Management

- Uses **Jotai** for global state (see `editor-jotai.ts`, `app-jotai.ts`)
- `AppState` is the primary application state type
- State is immutable - always create new objects for updates

### Components

- Located in `packages/excalidraw/components/`
- Use functional components with hooks
- CSS modules for component styling (`.scss` files)
- Main app component: `App.tsx` (~365KB - handles canvas, events, rendering)

## Coding Standards

### TypeScript

- Use TypeScript for all new code
- Prefer implementations without allocation where possible
- Opt for performant solutions, trading RAM for CPU cycles
- Prefer immutable data (`const`, `readonly`)
- Use optional chaining (`?.`) and nullish coalescing (`??`)

### React

- Use functional components with hooks
- Follow React hooks rules (no conditional hooks)
- Keep components small and focused
- Use CSS modules for component styling

### Naming Conventions

- **PascalCase** for component names, interfaces, and type aliases
- **camelCase** for variables, functions, and methods
- **ALL_CAPS** for constants

### Math Types

- **IMPORTANT**: Always use the `Point` type from `packages/math/src/types.ts` instead of `{ x, y }` objects
- Use `GlobalPoint` for world/canvas coordinates
- Use `LocalPoint` for element-local coordinates
- Other typed primitives: `Radians`, `Degrees`, `Vector`, `Line`, `Polygon`, `Curve`, `Ellipse`

### Error Handling

- Use try/catch blocks for async operations
- Implement proper error boundaries in React components
- Always log errors with contextual information

## Testing Workflow

1. **Always run `yarn test:app` after modifications**
2. **Always attempt to fix test failures**
3. Run `yarn test:update` to update snapshots before committing
4. Use `yarn test:typecheck` to verify TypeScript

## Communication Style

Per `.github/copilot-instructions.md`:
- Be succinct - avoid expansive answers
- Avoid explanations unless asked
- Stop apologizing if corrected
- Prefer code unless asked for explanation
- Stop summarizing changes after modifications unless asked

## Build System

- Packages build sequentially: `common` → `math` → `element` → `excalidraw`
- Build script: `scripts/buildPackage.js`
- Uses esbuild with SASS plugin for CSS
- Generates TypeScript declarations in `dist/types/`

## Key Technical Details

- Canvas rendering with manual transform handling
- Hand-drawn style using roughjs
- Supports PWA (service workers)
- Real-time collaboration via socket.io (app-level)
- End-to-end encryption for collaboration (app-level)
- Local-first architecture with IndexedDB (app-level)
