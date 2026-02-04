---
name: dev-server
description: Manage development servers - start, stop, restart, and troubleshoot. Trigger words - kill server, stop server, restart server, port conflict, port in use, start dev, npm run dev, dev server
---

# Dev Server Manager

Helps you manage your development server - stop all running servers for a clean restart, fix "port already in use" errors, and start your dev server.

## When to Use This Skill

- User says "kill servers", "stop servers", or "terminate processes"
- User says "restart dev server" or "clean restart"
- User is having server issues (port in use, stuck process)
- User asks to start or run the development server
- User mentions ports 3000, 5173, 8080, or dev server issues

## Instructions

### Stop All Servers

**Triggers:** "kill servers", "stop everything", "terminate", "clean restart"

1. Kill all Node.js processes: `pkill -f node 2>/dev/null || true`
2. Clear common dev ports if needed:
   ```bash
   lsof -ti:3000 | xargs kill -9 2>/dev/null || true
   lsof -ti:5173 | xargs kill -9 2>/dev/null || true
   ```
3. Confirm to user:

```
SERVERS STOPPED

Killed:
- Node processes: [count]
- Port 3000: cleared
- Port 5173: not in use

Ready to run: npm run dev
```

### Fix Port Issues

**Triggers:** "port in use", "address in use", "EADDRINUSE"

1. Find what's using the port: `lsof -i:[port]`
2. Kill that process: `kill -9 [PID]`
3. Confirm:

```
PORT CLEARED

Port 3000 was in use by: node (PID 12345)
Process terminated.

Ready to run: npm run dev
```

### Start Dev Server

**Triggers:** "start server", "run dev", "start the app"

1. Detect package manager (check for pnpm-lock.yaml, yarn.lock, or package-lock.json)
2. Run appropriate command: `pnpm dev` or `npm run dev`
3. Watch for startup errors
4. Confirm:

```
DEV SERVER STARTED

Running: pnpm dev
URL: http://localhost:3000

Open in browser or continue working.
```

## Examples

**Clean Restart:**
```
User: "Kill all servers"

Claude: "SERVERS STOPPED

Killed:
- Node processes: 3
- Port 3000: cleared
- Port 5173: not in use

Ready to run: npm run dev"
```

---

**Port Error:**
```
User: "I'm getting 'port already in use' error"

Claude: "PORT CLEARED

Port 3000 was in use by: node (PID 12345)
Process terminated.

Ready to run: npm run dev"
```

---

**Starting Server:**
```
User: "Start the dev server"

Claude: "DEV SERVER STARTED

Running: pnpm dev
URL: http://localhost:3000"
```

## Common Ports Reference

| Port | Usually Used By |
|------|-----------------|
| 3000 | Next.js, React |
| 3001 | Alternate Next.js |
| 5173 | Vite |
| 8080 | Various servers |
| 5432 | PostgreSQL |
| 6379 | Redis |
