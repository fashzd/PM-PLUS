# PM-PLUS API

Agent orchestration API with Band.ai integration for multi-agent project management system.

## Architecture Overview

This API serves as the **Data & Orchestrator** (Backbone) component, responsible for:
- State management and storage
- Band.ai API integration and proxy
- Human-in-the-Loop approval workflows
- Real-time updates via Server-Sent Events (SSE)

## Prerequisites

- Node.js 18+
- npm or yarn
- Band.ai API key

## Installation

```bash
npm install
```

## Configuration

Create a `.env` file in the root directory:

```env
BAND_API_KEY=your_band_api_key_here
PORT=3000
```

## Running the Server

### Development mode (with auto-reload)
```bash
npm run dev
```

### Production mode
```bash
npm run build
npm start
```

The server will start on `http://localhost:3000` (or your configured PORT).

## 📚 Interactive API Documentation

Once the server is running, you can access the interactive Swagger UI documentation:

**Swagger UI**: [http://localhost:3000/api-docs](http://localhost:3000/api-docs)

The Swagger UI provides:
- **Interactive API explorer** - Test all endpoints directly from your browser
- **Automatic API key authentication** - Enter your Band.ai API key once and it persists across requests
- **Request/response examples** - See example payloads for all endpoints
- **Schema documentation** - Detailed information about all data models

### Using the Swagger UI

1. Open [http://localhost:3000/api-docs](http://localhost:3000/api-docs) in your browser
2. Click the **"Authorize"** button at the top right
3. Enter your Band.ai API key in the `X-API-Key` field
4. Click **"Authorize"** and then **"Close"**
5. All subsequent requests will automatically include your API key

You can also download the OpenAPI specification as JSON:
**OpenAPI Spec**: [http://localhost:3000/api-docs.json](http://localhost:3000/api-docs.json)

## API Endpoints

### Agent API (Band.ai Proxy)

#### Get Current Agent Profile
```bash
GET /agent/me
Headers: X-API-Key: <apiKey>
```

Fetches the current agent's profile from Band.ai.

**Example:**
```bash
curl -X GET http://localhost:3000/agent/me \
  -H "X-API-Key: your_api_key"
```

---

#### Get Agent Context
```bash
GET /agent/chats/:chatId/context?limit=50&page=1&page_size=50
Headers: X-API-Key: <apiKey>
```

Retrieves agent context for rehydration from a specific chat.

**Parameters:**
- `chatId` (path) - Chat room ID
- `limit` (query) - Number of items to return (default: 50)
- `page` (query) - Page number (default: 1)
- `page_size` (query) - Items per page (default: 50)

**Example:**
```bash
curl -X GET "http://localhost:3000/agent/chats/chat_123/context?limit=50" \
  -H "X-API-Key: your_api_key"
```

---

#### Report Agent Activity
```bash
POST /agent/chats/:chatId/activity
Headers: X-API-Key: <apiKey>
Content-Type: application/json
Body: {"working": true}
```

Send a keep-alive signal to indicate the agent is working.

**Example:**
```bash
curl -X POST http://localhost:3000/agent/chats/chat_123/activity \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{"working": true}'
```

---

### Human API (Band.ai Proxy)

#### Send Message as Human
```bash
POST /me/chats/:chatId/messages
Headers: X-API-Key: <apiKey>
Content-Type: application/json
Body: {
  "message": {
    "content": "@Agent please analyze this",
    "mentions": [{}]
  }
}
```

Send a message as the human user to a chat room.

**Example:**
```bash
curl -X POST http://localhost:3000/me/chats/chat_123/messages \
  -H "X-API-Key: your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "message": {
      "content": "@DataAnalyst please analyze the Q4 sales data",
      "mentions": [{}]
    }
  }'
```

---

#### Get Chat Messages
```bash
GET /me/chats/:chatId/messages?limit=50&page=1
Headers: X-API-Key: <apiKey>
```

List messages in a chat room.

**Example:**
```bash
curl -X GET "http://localhost:3000/me/chats/chat_123/messages?limit=50" \
  -H "X-API-Key: your_api_key"
```

---

### Human-in-the-Loop Endpoints

#### Request Approval
```bash
POST /human/approval-request
Content-Type: application/json
Body: {
  "sessionId": "session_123",
  "agentId": "agent_456",
  "action": "deploy_to_production",
  "context": {
    "description": "Deploy version 2.0",
    "impact": "high"
  }
}
```

Create a new approval request requiring human intervention.

**Example:**
```bash
curl -X POST http://localhost:3000/human/approval-request \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "session_123",
    "agentId": "agent_456",
    "action": "deploy_to_production",
    "context": {"description": "Deploy v2.0"}
  }'
```

---

#### Respond to Approval
```bash
POST /human/approval-response
Content-Type: application/json
Body: {
  "requestId": "req_789",
  "approved": true,
  "reason": "Approved after review"
}
```

Respond to a pending approval request.

**Example:**
```bash
curl -X POST http://localhost:3000/human/approval-response \
  -H "Content-Type: application/json" \
  -d '{
    "requestId": "req_789",
    "approved": true,
    "reason": "Looks good"
  }'
```

---

### State Management

#### Get Session State
```bash
GET /state?sessionId=session_123
```

Retrieve the current state of a session.

**Example:**
```bash
curl -X GET "http://localhost:3000/state?sessionId=session_123"
```

---

#### Record State Event
```bash
POST /state/event
Content-Type: application/json
Body: {
  "sessionId": "session_123",
  "agentId": "agent_456",
  "eventType": "task_completed",
  "payload": {
    "taskId": "task_001",
    "result": "success"
  }
}
```

Record a new event in the session state.

**Example:**
```bash
curl -X POST http://localhost:3000/state/event \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "session_123",
    "agentId": "agent_456",
    "eventType": "task_completed",
    "payload": {"taskId": "task_001"}
  }'
```

---

### Real-Time Updates

#### Subscribe to Updates (SSE)
```bash
GET /updates?sessionId=session_123
```

Server-Sent Events endpoint for real-time updates about a session.

**Example:**
```javascript
const eventSource = new EventSource('http://localhost:3000/updates?sessionId=session_123');

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Update:', data);
};
```

**Event Types:**
- `connected` - Initial connection established
- `state_change` - Session state updated
- `approval_request` - New approval request created
- `agent_message` - New agent event recorded
- `error` - Error occurred

---

### Legacy/Demo Endpoints

#### Start Check-in (Legacy)
```bash
GET /demo/start-checkin
Headers: X-API-Key: <apiKey>
```

Legacy endpoint that proxies to `/agent/me`.

---

### Health Check

#### Health Status
```bash
GET /health
```

Returns server health status.

**Example:**
```bash
curl -X GET http://localhost:3000/health
```

**Response:**
```json
{
  "status": "ok",
  "timestamp": "2026-06-17T12:00:00.000Z"
}
```

---

## Data Models

### SessionState
```typescript
{
  sessionId: string;
  status: 'active' | 'paused' | 'completed' | 'failed';
  agents: Array<{
    agentId: string;
    status: string;
    lastActivity: string;
  }>;
  events: StateEvent[];
  pendingApprovals: ApprovalRequest[];
  createdAt: string;
  updatedAt: string;
}
```

### ApprovalRequest
```typescript
{
  requestId: string;
  sessionId: string;
  agentId: string;
  action: string;
  context: Record<string, any>;
  requestedAt: string;
}
```

### StateEvent
```typescript
{
  eventId: string;
  sessionId: string;
  agentId: string;
  eventType: string;
  payload: Record<string, any>;
  timestamp: string;
}
```

---

## Authentication

All Band.ai proxy endpoints require authentication via either:
1. **X-API-Key header** (preferred for client requests)
2. **BAND_API_KEY environment variable** (fallback)

The API key is obtained from your Band.ai account.

---

## Error Handling

All endpoints return standardized error responses:

```json
{
  "error": "Error description"
}
```

**HTTP Status Codes:**
- `200` - Success
- `201` - Created
- `400` - Bad Request (missing/invalid parameters)
- `401` - Unauthorized (missing/invalid API key)
- `404` - Not Found (resource doesn't exist)
- `500` - Internal Server Error

---

## Project Structure

```
PM-PLUS/
├── src/
│   ├── index.ts           # Main server entry point
│   ├── types.ts           # TypeScript type definitions
│   ├── store.ts           # In-memory state store
│   └── routes/
│       ├── agent.ts       # Agent API endpoints
│       ├── messages.ts    # Human API message endpoints
│       ├── human.ts       # Human-in-the-Loop endpoints
│       ├── state.ts       # State management endpoints
│       ├── updates.ts     # SSE updates endpoint
│       └── demo.ts        # Legacy demo endpoints
├── .env                   # Environment variables
├── package.json
├── tsconfig.json
└── README.md
```

---

## Development

### Scripts
- `npm run dev` - Start development server with auto-reload
- `npm run build` - Compile TypeScript to JavaScript
- `npm start` - Run compiled production build

### Adding New Endpoints

1. Create a new route file in `src/routes/`
2. Import and register it in `src/index.ts`
3. Update this README with documentation

---

## Integration with Band.ai

This API acts as a proxy and extension layer for Band.ai's Agent and Human APIs:

- **Agent API** - Used by autonomous agents to communicate and report activity
- **Human API** - Used by human users (PMs) to interact with agents
- **Custom Extensions** - State management, approval workflows, and real-time updates

Refer to [Band.ai documentation](https://docs.band.ai) for more details on their API capabilities.

---

## License

MIT
