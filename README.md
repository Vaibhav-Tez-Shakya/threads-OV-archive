# Threads-OV Archive

This repository stores conversation threads created by Threads-OV.

## Storage Format

Each conversation is stored as a separate Markdown file:

    threads/<thread-id>.md

The Markdown file contains:

- Thread metadata
- Complete user messages
- Complete assistant responses
- Message timestamps

## Architecture

Claude
  |
  v
Threads-OV MCP
  |
  v
GitHub API
  |
  v
threads-OV-archive
  |
  v
threads/<thread-id>.md

## Rules

- One Claude chat corresponds to one thread.
- A thread is created once at the beginning of a chat.
- All subsequent messages are saved to the same Markdown file.
- User messages are stored completely.
- Assistant responses are stored completely.
- Topic changes do not create a new thread.
- Tool calls and intermediate processing are not stored as separate messages.
- PostgreSQL is not used by this archive repository.
