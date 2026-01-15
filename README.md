Problem Statement:
A collaborative task management system where multiple clients see changes in near real-time, without using a managed real-time database, and where the data model can grow large and still sync efficiently.

## Checkpoint 1: Understanding the problem

At a surface level, this is a task management app with projects, tasks, and comments. But the real challenge is collaboration and data sync. The prompt explicitly calls out large project payloads (2MB+) and asks for near real-time updates without using a managed real-time database.

### Constraints I took seriously

- Project data can grow large, so resending the full project on every change is not acceptable
- Multiple clients can be connected to the same project at the same time
- Changes made by one client should show up for others quickly
- No Firebase / Supabase or similar managed real-time solutions

### To keep the scope reasonable, I'm intentionally leaving a few things out:

- Authentication and authorization
- Support for offline users
- Complex conflict resolution for simultaneous edits

These are all interesting problems, but they would distract from the core goal.

### Early takeaway

The heart of the project will be how state changes are represented, stored, and propagated to other clients in a way that stays efficient as data grows.

