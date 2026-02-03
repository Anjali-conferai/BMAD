# Mattermost API Reference

This document provides a reference for the Mattermost REST API v4, WebSocket API, and integration patterns.

## API Overview

**Base URL:** `http://localhost:8065/api/v4`

**Authentication:** Bearer token in `Authorization` header

**Content-Type:** `application/json`

**API Specification:** OpenAPI v4 specs available in `api/v4/` directory

## Authentication

### Login

**Endpoint:** `POST /api/v4/users/login`

**Request:**
```json
{
  "login_id": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "id": "user_id",
  "username": "username",
  "email": "user@example.com",
  "roles": "system_user"
}
```

**Headers:**
- `Token`: Session token (use for subsequent requests)

### Using the Token

Include the token in the `Authorization` header:

```bash
curl -H "Authorization: Bearer <token>" \
  http://localhost:8065/api/v4/users/me
```

### Logout

**Endpoint:** `POST /api/v4/users/logout`

## Core API Endpoints

### Users

#### Get Current User

```
GET /api/v4/users/me
```

**Response:**
```json
{
  "id": "user_id",
  "create_at": 1234567890,
  "update_at": 1234567890,
  "delete_at": 0,
  "username": "john.doe",
  "first_name": "John",
  "last_name": "Doe",
  "email": "john.doe@example.com",
  "email_verified": true,
  "auth_service": "",
  "roles": "system_user",
  "locale": "en",
  "timezone": {
    "useAutomaticTimezone": true,
    "automaticTimezone": "America/New_York",
    "manualTimezone": ""
  }
}
```

#### Create User

```
POST /api/v4/users
```

**Request:**
```json
{
  "email": "newuser@example.com",
  "username": "newuser",
  "password": "Password1!",
  "first_name": "New",
  "last_name": "User"
}
```

#### Get User by ID

```
GET /api/v4/users/{user_id}
```

#### Update User

```
PUT /api/v4/users/{user_id}
```

**Request:**
```json
{
  "id": "user_id",
  "first_name": "Updated",
  "last_name": "Name"
}
```

#### Get Users

```
GET /api/v4/users?page=0&per_page=60
```

**Query Parameters:**
- `page`: Page number (default: 0)
- `per_page`: Users per page (default: 60, max: 200)
- `in_team`: Filter by team ID
- `not_in_team`: Exclude team ID
- `in_channel`: Filter by channel ID
- `not_in_channel`: Exclude channel ID
- `sort`: Sort by (default: username)

### Teams

#### Create Team

```
POST /api/v4/teams
```

**Request:**
```json
{
  "name": "team-name",
  "display_name": "Team Display Name",
  "type": "O"
}
```

**Types:**
- `O`: Open team
- `I`: Invite-only team

#### Get Team

```
GET /api/v4/teams/{team_id}
```

#### Get All Teams

```
GET /api/v4/teams
```

#### Add User to Team

```
POST /api/v4/teams/{team_id}/members
```

**Request:**
```json
{
  "team_id": "team_id",
  "user_id": "user_id"
}
```

#### Get Team Members

```
GET /api/v4/teams/{team_id}/members?page=0&per_page=60
```

### Channels

#### Create Channel

```
POST /api/v4/channels
```

**Request:**
```json
{
  "team_id": "team_id",
  "name": "channel-name",
  "display_name": "Channel Display Name",
  "type": "O",
  "purpose": "Channel purpose",
  "header": "Channel header"
}
```

**Types:**
- `O`: Public channel
- `P`: Private channel
- `D`: Direct message
- `G`: Group message

#### Get Channel

```
GET /api/v4/channels/{channel_id}
```

#### Get Channels for Team

```
GET /api/v4/teams/{team_id}/channels
```

#### Add User to Channel

```
POST /api/v4/channels/{channel_id}/members
```

**Request:**
```json
{
  "user_id": "user_id"
}
```

#### Get Channel Members

```
GET /api/v4/channels/{channel_id}/members?page=0&per_page=60
```

### Posts

#### Create Post

```
POST /api/v4/posts
```

**Request:**
```json
{
  "channel_id": "channel_id",
  "message": "Hello, world!",
  "props": {
    "attachments": []
  }
}
```

#### Get Post

```
GET /api/v4/posts/{post_id}
```

#### Get Posts for Channel

```
GET /api/v4/channels/{channel_id}/posts?page=0&per_page=60
```

**Response:**
```json
{
  "order": ["post_id_1", "post_id_2"],
  "posts": {
    "post_id_1": {
      "id": "post_id_1",
      "create_at": 1234567890,
      "update_at": 1234567890,
      "delete_at": 0,
      "user_id": "user_id",
      "channel_id": "channel_id",
      "message": "Post message",
      "type": "",
      "props": {}
    }
  }
}
```

#### Update Post

```
PUT /api/v4/posts/{post_id}
```

**Request:**
```json
{
  "id": "post_id",
  "message": "Updated message"
}
```

#### Delete Post

```
DELETE /api/v4/posts/{post_id}
```

#### Search Posts

```
POST /api/v4/teams/{team_id}/posts/search
```

**Request:**
```json
{
  "terms": "search terms",
  "is_or_search": false
}
```

### Files

#### Upload File

```
POST /api/v4/files
```

**Request:** `multipart/form-data`
- `files`: File data
- `channel_id`: Channel ID

**Response:**
```json
{
  "file_infos": [
    {
      "id": "file_id",
      "user_id": "user_id",
      "post_id": "",
      "create_at": 1234567890,
      "update_at": 1234567890,
      "delete_at": 0,
      "name": "filename.png",
      "extension": "png",
      "size": 12345,
      "mime_type": "image/png",
      "width": 800,
      "height": 600,
      "has_preview_image": true
    }
  ]
}
```

#### Get File

```
GET /api/v4/files/{file_id}
```

#### Get File Thumbnail

```
GET /api/v4/files/{file_id}/thumbnail
```

#### Get File Preview

```
GET /api/v4/files/{file_id}/preview
```

### Preferences

#### Get User Preferences

```
GET /api/v4/users/{user_id}/preferences
```

#### Save User Preferences

```
PUT /api/v4/users/{user_id}/preferences
```

**Request:**
```json
[
  {
    "user_id": "user_id",
    "category": "display_settings",
    "name": "use_military_time",
    "value": "true"
  }
]
```

### Roles and Permissions

#### Get Role

```
GET /api/v4/roles/{role_id}
```

#### Get Role by Name

```
GET /api/v4/roles/name/{role_name}
```

**Common Roles:**
- `system_admin`: System administrator
- `system_user`: Regular user
- `team_admin`: Team administrator
- `team_user`: Team member
- `channel_admin`: Channel administrator
- `channel_user`: Channel member

#### Patch Role

```
PUT /api/v4/roles/{role_id}/patch
```

**Request:**
```json
{
  "permissions": ["create_post", "edit_post", "delete_post"]
}
```

## WebSocket API

### Connection

**URL:** `ws://localhost:8065/api/v4/websocket`

**Authentication:** Include token in connection URL or send auth message

### Connect with Token

```javascript
const ws = new WebSocket('ws://localhost:8065/api/v4/websocket');

ws.onopen = () => {
  ws.send(JSON.stringify({
    seq: 1,
    action: 'authentication_challenge',
    data: {
      token: 'your_auth_token'
    }
  }));
};
```

### Event Types

#### Posted Event

Received when a new post is created:

```json
{
  "event": "posted",
  "data": {
    "channel_display_name": "Channel Name",
    "channel_name": "channel-name",
    "channel_type": "O",
    "post": "{\"id\":\"post_id\",\"message\":\"Hello\"}",
    "sender_name": "username",
    "team_id": "team_id"
  },
  "broadcast": {
    "omit_users": null,
    "user_id": "",
    "channel_id": "channel_id",
    "team_id": "team_id"
  },
  "seq": 2
}
```

#### Typing Event

Received when a user is typing:

```json
{
  "event": "typing",
  "data": {
    "parent_id": "",
    "user_id": "user_id"
  },
  "broadcast": {
    "channel_id": "channel_id"
  }
}
```

#### User Updated Event

```json
{
  "event": "user_updated",
  "data": {
    "user": "{\"id\":\"user_id\",\"username\":\"updated_name\"}"
  }
}
```

### Sending Events

#### Send Typing Event

```javascript
ws.send(JSON.stringify({
  action: 'user_typing',
  seq: 3,
  data: {
    channel_id: 'channel_id',
    parent_id: ''
  }
}));
```

## Error Handling

### Error Response Format

All API errors return `model.AppError`:

```json
{
  "id": "api.user.create_user.email_exists.app_error",
  "message": "An account with that email already exists.",
  "detailed_error": "",
  "request_id": "request_id",
  "status_code": 400
}
```

### Common Status Codes

- `200`: Success
- `201`: Created
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `500`: Internal Server Error

### Error IDs

Common error IDs:
- `api.context.session_expired.app_error`: Session expired
- `api.context.invalid_token.error`: Invalid token
- `api.context.permissions.app_error`: Insufficient permissions
- `store.sql_user.get.app_error`: User not found
- `store.sql_channel.get.app_error`: Channel not found

## Rate Limiting

**Default Limits:**
- 10 requests per second per user
- Configurable in System Console

**Rate Limit Headers:**
```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 9
X-RateLimit-Reset: 1234567890
```

## Pagination

Most list endpoints support pagination:

**Query Parameters:**
- `page`: Page number (0-indexed)
- `per_page`: Items per page (default: 60, max: 200)

**Example:**
```
GET /api/v4/users?page=0&per_page=100
```

## Integration Patterns

### Webhooks

#### Incoming Webhooks

**Create Incoming Webhook:**
1. Go to Integrations > Incoming Webhooks
2. Create webhook for a channel
3. Get webhook URL

**Send Message:**
```bash
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"text":"Hello from webhook!"}' \
  https://your-mattermost-url/hooks/webhook_id
```

**Advanced Payload:**
```json
{
  "text": "Message text",
  "username": "Custom Bot",
  "icon_url": "https://example.com/icon.png",
  "attachments": [
    {
      "fallback": "Fallback text",
      "color": "#FF8000",
      "pretext": "Optional pretext",
      "text": "Attachment text",
      "title": "Attachment Title",
      "title_link": "https://example.com",
      "fields": [
        {
          "short": false,
          "title": "Field Title",
          "value": "Field Value"
        }
      ]
    }
  ]
}
```

#### Outgoing Webhooks

**Create Outgoing Webhook:**
1. Go to Integrations > Outgoing Webhooks
2. Configure trigger words and callback URL
3. Mattermost POSTs to your URL when triggered

**Payload Received:**
```json
{
  "token": "webhook_token",
  "team_id": "team_id",
  "team_domain": "team-domain",
  "channel_id": "channel_id",
  "channel_name": "channel-name",
  "timestamp": 1234567890,
  "user_id": "user_id",
  "user_name": "username",
  "post_id": "post_id",
  "text": "trigger_word rest of message",
  "trigger_word": "trigger_word"
}
```

**Response Format:**
```json
{
  "text": "Response message",
  "username": "Bot Name",
  "icon_url": "https://example.com/icon.png"
}
```

### Slash Commands

**Create Slash Command:**
1. Go to Integrations > Slash Commands
2. Configure command trigger and callback URL
3. Mattermost POSTs to your URL when command is used

**Payload Received:**
```json
{
  "token": "command_token",
  "team_id": "team_id",
  "team_domain": "team-domain",
  "channel_id": "channel_id",
  "channel_name": "channel-name",
  "user_id": "user_id",
  "user_name": "username",
  "command": "/mycommand",
  "text": "command arguments"
}
```

**Response Format:**
```json
{
  "response_type": "in_channel",
  "text": "Command response",
  "username": "Bot Name",
  "icon_url": "https://example.com/icon.png"
}
```

**Response Types:**
- `in_channel`: Visible to everyone
- `ephemeral`: Visible only to user who ran command

### OAuth 2.0

**Register OAuth App:**
1. Go to Integrations > OAuth 2.0 Applications
2. Create new application
3. Get Client ID and Client Secret

**Authorization Flow:**

1. **Redirect to authorization URL:**
```
https://your-mattermost-url/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=REDIRECT_URI&
  state=STATE
```

2. **Exchange code for token:**
```bash
curl -X POST \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE" \
  -d "client_id=CLIENT_ID" \
  -d "client_secret=CLIENT_SECRET" \
  -d "redirect_uri=REDIRECT_URI" \
  https://your-mattermost-url/oauth/access_token
```

3. **Response:**
```json
{
  "access_token": "access_token",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "refresh_token"
}
```

## Client Libraries

### Official Clients

- **JavaScript/TypeScript**: `@mattermost/client`
- **Go**: `github.com/mattermost/mattermost/server/v8/model`
- **Python**: `mattermostdriver`

### JavaScript Example

```javascript
import {Client4} from '@mattermost/client';

const client = new Client4();
client.setUrl('http://localhost:8065');

// Login
const user = await client.login('user@example.com', 'password');
client.setToken(user.token);

// Get current user
const me = await client.getMe();

// Create post
const post = await client.createPost({
  channel_id: 'channel_id',
  message: 'Hello from API!'
});
```

### Go Example

```go
import (
    "github.com/mattermost/mattermost/server/v8/model"
)

client := model.NewAPIv4Client("http://localhost:8065")

// Login
user, _, err := client.Login("user@example.com", "password")
if err != nil {
    panic(err)
}

// Create post
post := &model.Post{
    ChannelId: "channel_id",
    Message:   "Hello from API!",
}

createdPost, _, err := client.CreatePost(post)
```

## Additional Resources

- [Full API Documentation](https://api.mattermost.com/)
- [WebSocket Events Reference](https://api.mattermost.com/#tag/WebSocket)
- [Integration Guide](https://developers.mattermost.com/integrate/getting-started/)
- [Plugin API Reference](https://developers.mattermost.com/integrate/plugins/server/reference/)
