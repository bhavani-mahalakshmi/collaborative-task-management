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


## Checkpoint 3: Architecture and data flow

### High level architecture
The system is split into three main pieces:

- A Next.js frontend that manages UI state and optimistic updates
- API routes for commands like creating or updating tasks
- A database that acts as the source of truth
 
### Source of truth
Clients don’t talk to each other directly. All changes flow through the backend.

### Events
- We dont need to send full project data around. We can do events. Each event represents a single action, like updating a task status or adding a comment.

- These events are stored in the database and applied in order. Clients rebuild their local state by applying events rather than replacing entire objects.

### How events are passed around between clients
WebSockets are the obvious choice here. They support two-way communication, which makes it easy for clients to both send updates and receive changes

### Ordering
- The server will take care of validating commands and ordering events.

- Clients may update the UI optimistically, but they still reconcile their state based on the events they receive from the server.


## Checkpoint 4: Data model and event design

### Core Entities (as mentioned)
- Project
- Task
- Comment

Tasks belong to a project. Comments belong to a task.

These are stored in regular tables.

### Events
An event represents a single change, for example:
- Task created
- Task status updated
- Comment added

Each event has the following attached to it:
- The project it belongs to
- The type of change
- The minimal data needed to apply that change
- A timestamp/version

Events are append-only. Once written, they are never updated or deleted.

Storing changes as events:
- Keeps updates small
- Broadcasts only what changed
- Applies changes in the right order

### Versions
Events are ordered using a project-level version.

Each new event increments the version. Clients apply events in version order.


## Checkpoint 5: Backend API and WebSocket flow

### Flow of the app

Clients don’t write events directly. They call backend APIs to send commands.

The backend validates the command, applies the change, and records an event.

Examples of commands:
- Create or update a task
- Add a comment

API routes are stateless. The database is the source of truth.

### WebSockets for real-time updates

When a client opens a project, it connects to a WebSocket and subscribes to that project.

Each project maps to a logical room. Clients connected to the same project join the same room.

### Event flow

When a command succeeds:
- The backend writes a new event with the next project version
- The event is broadcasted to all clients connected to that project

Clients receive the event and apply it to their local state in version order.

### Optimistic updates

Clients update the UI optimistically when sending a command.

If the server rejects a command or sends back a conflicting version, the client reconciles based on the events it receives.

### Failure handling

If a WebSocket connection drops, the client asks for events after its last known project version and applies them after reconnection.

Any missed events should be fetched from the backend before applying new ones.
