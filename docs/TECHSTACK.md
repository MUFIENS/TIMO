# Tech Stack — Quick Reference for Agents

| Layer | Approved direction | Notes |
|---|---|---|
| Desktop | Tauri 2 | Windows desktop shell |
| Native core | Rust | Tracking and system integration |
| OS APIs | Windows API via `windows` crate | Foreground, idle, session, power events |
| Frontend | React + TypeScript | UI only; no direct OS tracking |
| Build tool | Vite | Frontend bundling |
| Styling | Tailwind CSS | Design tokens via CSS variables |
| UI primitives | shadcn/ui + Radix | Customized to Timo visual identity |
| Motion | Motion for React | Purposeful UI transitions |
| Charts | Recharts | Analytics |
| Timeline | Custom CSS Grid/SVG component | More control than generic chart library |
| 3D | Three.js + React Three Fiber + Drei | Optimized GLB mascot |
| State | Zustand (selectively) | UI state only |
| Tables | TanStack Table/Virtual (as needed) | Application catalog |
| Validation | Zod + Rust typed validation | Boundary validation |
| Database | SQLite via Rust (`rusqlite` candidate) | No direct frontend DB access |
| Icons | Lucide React | One icon family |
| MCP required | Context7, Playwright | Figma MCP explicitly excluded |
| MCP optional | GitHub | Repository workflow |
| Installer | NSIS | Windows setup EXE |

## Agent rules

1. Verify package compatibility and current stable versions before installation.
2. Use installed Antigravity skills as required by `SKILL.md`.
3. Do not add a library without a concrete use case and maintenance/license check.
4. Do not store SQLite access logic in React components.
5. Keep OS-specific logic behind Rust platform modules.
6. Treat memory targets as measured acceptance goals, not framework marketing claims.
