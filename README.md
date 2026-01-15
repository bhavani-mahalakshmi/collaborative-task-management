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

## Checkpoint 2: Figuring out what actually needs to be built

Functional Requirements:

- Creating and viewing multiple projects
- Adding, updating, and deleting tasks within a project
- Updating task fields like title, status, assignees, and configuration data
- Basic task dependencies and simple status transition checks
- Adding comments to tasks and viewing comment threads
- Seeing task and comment updates from other clients in (near) real-time

This means tasks and comments are the ones that would need real time updates, project level metadata will mostly be static

Non-functional requirements:
- Avoid resending full project payloads on every change
- Keep updates small and incremental as project data grows
- Maintain a consistent view of state across connected clients
- Support multiple clients interacting with the same project at the same time
- Keep API routes stateless and treat the database as the source of truth so the backend can scale beyond a single instance

These will drive my design decisions.


