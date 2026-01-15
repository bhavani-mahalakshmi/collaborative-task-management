Problem Statement:
A collaborative task management system where multiple clients see changes in near real-time, without using a managed real-time database, and where the data model can grow large and still sync efficiently.

## Checkpoint 1: Understanding the problem

At a surface level, this is a task management app with projects, tasks, and comments. But the real challenge is collaboration and data sync. The prompt explicitly calls out large project payloads (2MB+) and asks for near real-time updates without using a managed real-time database.

