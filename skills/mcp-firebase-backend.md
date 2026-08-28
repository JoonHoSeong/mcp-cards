# Firebase MCP Server

## Overview
| Field | Value |
|-------|-------|
| **Name** | Firebase MCP |
| **Category** | Backend |
| **Official** | Community |
| **Source** | https://github.com/gannonh/firebase-mcp |
| **Transport** | stdio |
| **Install** | `npx firebase-mcp` |

## Description
The Firebase MCP server enables AI agents to interact with Firebase services including Firestore, Cloud Storage, Authentication, and Cloud Functions. It provides a unified backend interface for managing data, users, and serverless functions in mobile and web applications.

## Key Tools
| Tool | Description |
|------|-------------|
| `get_document` | Get a Firestore document by path |
| `set_document` | Create or update a Firestore document |
| `query_collection` | Query a Firestore collection with filters |
| `list_collections` | List available Firestore collections |
| `upload_storage` | Upload a file to Cloud Storage |
| `get_auth_user` | Get user authentication details |
| `list_auth_users` | List authenticated users |
| `call_function` | Invoke a Cloud Function |

## Configuration
```json
{
  "mcpServers": {
    "firebase": {
      "command": "npx",
      "args": ["firebase-mcp"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "${GOOGLE_APPLICATION_CREDENTIALS}"
      }
    }
  }
}
```

## Use Cases
1. AI-powered mobile app backend management
2. User authentication and access control automation
3. Real-time data synchronization and serverless function orchestration

## NVIDIA Skill Combinations
| Purpose | NVIDIA Skill | Reason |
|---------|-------------|--------|
| Mobile AI backend | aiq-deploy | Deploy AI inference endpoints backed by Firebase for mobile consumption |

## Prerequisites
- `GOOGLE_APPLICATION_CREDENTIALS` — Path to Firebase service account JSON file

## Security Notes
- Service account JSON grants admin access to Firebase project; never commit to version control
- Firestore rules should enforce access control independent of MCP-level permissions

## References
- https://github.com/gannonh/firebase-mcp
- https://firebase.google.com/docs
