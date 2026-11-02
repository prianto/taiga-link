# Taiga REST API — AI Integration Guide

> Purpose: This document provides instructions and API references for an AI assistant that needs to read, create, update, and manage Taiga project management data.

## 1. API Configuration

### Cloud instance

```env
TAIGA_BASE_URL=https://api.taiga.io/api/v1
```

### Self-hosted instance

Replace the base URL with your own Taiga server:

```env
TAIGA_BASE_URL=https://taiga.example.com/api/v1
```

All API endpoints in this document are relative to `TAIGA_BASE_URL`.

### Default headers

All authenticated requests must include:

```http
Authorization: Bearer YOUR_AUTH_TOKEN
Content-Type: application/json
```

Use the `Authorization` header for all authenticated API requests.

---

## 2. Authentication

Taiga uses token-based authentication.

### 2.1. Obtain an authentication token

**Endpoint**

```http
POST /auth
```

**Request body**

```json
{
  "type": "normal",
  "username": "your-email@example.com",
  "password": "YOUR_PASSWORD"
}
```

The `username` field accepts either a Taiga username or email address.

**Example using cURL**

```bash
curl -X POST \
  "${TAIGA_BASE_URL}/auth" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "normal",
    "username": "your-email@example.com",
    "password": "YOUR_PASSWORD"
  }'
```

**Successful response**

HTTP `200 OK`.

The response contains authentication information, including:

- `auth_token`: Access token for API requests.
- `refresh`: Refresh token used to obtain a new access token.
- `id`: Authenticated user ID.
- `username`: Authenticated username.

Example:

```json
{
  "auth_token": "ACCESS_TOKEN",
  "refresh": "REFRESH_TOKEN",
  "id": 123,
  "username": "andi"
}
```

Use the `auth_token` value in subsequent API requests.

### 2.2. Refresh an expired authentication token

Access tokens expire. Use the refresh token to obtain a new authentication token.

**Endpoint**

```http
POST /auth/refresh
```

**Request body**

```json
{
  "refresh": "YOUR_REFRESH_TOKEN"
}
```

**Example**

```bash
curl -X POST \
  "${TAIGA_BASE_URL}/auth/refresh" \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "YOUR_REFRESH_TOKEN"
  }'
```

Use the new authentication details returned by the API. Update the stored access token and refresh token when new values are provided.

### 2.3. Authentication rules for AI

1. Never ask the user to provide their password in an ordinary conversation.
2. Never expose authentication tokens in responses.
3. Store credentials and tokens in secure environment variables or a secret manager.
4. Never hardcode credentials in source code or commit them to Git.
5. Reuse the existing access token while it remains valid.
6. If authentication fails because the access token expired, attempt token refresh.
7. If refreshing fails, request that the user re-authenticate through a secure process.
8. Never repeatedly retry invalid credentials.

---

## 3. API Endpoint Reference

### 3.1. Authentication

| Method | Endpoint         | Description                  |
| ------ | ---------------- | ---------------------------- |
| POST   | `/auth`          | Authenticate user            |
| POST   | `/auth/refresh`  | Refresh authentication token |
| POST   | `/auth/register` | Register user                |

### 3.2. Projects

| Method | Endpoint                        | Description              |
| ------ | ------------------------------- | ------------------------ |
| GET    | `/projects`                     | List projects            |
| POST   | `/projects`                     | Create project           |
| GET    | `/projects/{projectId}`         | Get project details      |
| GET    | `/projects/by_slug?slug={slug}` | Get project by slug      |
| PUT    | `/projects/{projectId}`         | Replace project data     |
| PATCH  | `/projects/{projectId}`         | Partially update project |
| DELETE | `/projects/{projectId}`         | Delete project           |

Example: List projects

```http
GET /projects
```

Example: Get project by slug

```http
GET /projects/by_slug?slug=my-project
```

### 3.3. User Stories

| Method | Endpoint                                            | Description                 |
| ------ | --------------------------------------------------- | --------------------------- |
| GET    | `/userstories`                                      | List user stories           |
| POST   | `/userstories`                                      | Create user story           |
| GET    | `/userstories/{userStoryId}`                        | Get user story              |
| GET    | `/userstories/by_ref?ref={ref}&project={projectId}` | Get by reference            |
| PUT    | `/userstories/{userStoryId}`                        | Replace user story          |
| PATCH  | `/userstories/{userStoryId}`                        | Partially update user story |
| DELETE | `/userstories/{userStoryId}`                        | Delete user story           |

Useful query parameters for listing:

- `project`
- `milestone`
- `status`
- `assigned_to`
- `epic`
- `tags`
- `status__is_closed`

Example:

```http
GET /userstories?project=1
```

### 3.4. Tasks

| Method | Endpoint                                      | Description           |
| ------ | --------------------------------------------- | --------------------- |
| GET    | `/tasks`                                      | List tasks            |
| POST   | `/tasks`                                      | Create task           |
| GET    | `/tasks/{taskId}`                             | Get task              |
| GET    | `/tasks/by_ref?ref={ref}&project={projectId}` | Get task by reference |
| PUT    | `/tasks/{taskId}`                             | Replace task data     |
| PATCH  | `/tasks/{taskId}`                             | Partially update task |
| DELETE | `/tasks/{taskId}`                             | Delete task           |
| POST   | `/tasks/bulk_create`                          | Create multiple tasks |

Useful query parameters for listing:

- `project`
- `status`
- `user_story`
- `assigned_to`
- `owner`
- `milestone`
- `tags`
- `status__is_closed`

Example:

```http
GET /tasks?project=1
```

### 3.5. Task Statuses

| Method | Endpoint                    | Description             |
| ------ | --------------------------- | ----------------------- |
| GET    | `/task-statuses`            | List task statuses      |
| POST   | `/task-statuses`            | Create task status      |
| GET    | `/task-statuses/{statusId}` | Get status              |
| PUT    | `/task-statuses/{statusId}` | Update status           |
| PATCH  | `/task-statuses/{statusId}` | Partially update status |
| DELETE | `/task-statuses/{statusId}` | Delete status           |

Always retrieve the available task statuses before attempting to change a task's status.

Do not assume status IDs or status names are identical across projects.

### 3.6. User Story Statuses

| Method | Endpoint                         | Description              |
| ------ | -------------------------------- | ------------------------ |
| GET    | `/userstory-statuses`            | List user story statuses |
| POST   | `/userstory-statuses`            | Create status            |
| GET    | `/userstory-statuses/{statusId}` | Get status               |
| PATCH  | `/userstory-statuses/{statusId}` | Update status            |
| DELETE | `/userstory-statuses/{statusId}` | Delete status            |

### 3.7. Issues

| Method | Endpoint                                       | Description            |
| ------ | ---------------------------------------------- | ---------------------- |
| GET    | `/issues`                                      | List issues            |
| POST   | `/issues`                                      | Create issue           |
| GET    | `/issues/{issueId}`                            | Get issue              |
| GET    | `/issues/by_ref?ref={ref}&project={projectId}` | Get issue by reference |
| PUT    | `/issues/{issueId}`                            | Replace issue          |
| PATCH  | `/issues/{issueId}`                            | Partially update issue |
| DELETE | `/issues/{issueId}`                            | Delete issue           |

### 3.8. Issue Statuses

| Method | Endpoint                     | Description         |
| ------ | ---------------------------- | ------------------- |
| GET    | `/issue-statuses`            | List issue statuses |
| POST   | `/issue-statuses`            | Create issue status |
| GET    | `/issue-statuses/{statusId}` | Get issue status    |
| PATCH  | `/issue-statuses/{statusId}` | Update issue status |
| DELETE | `/issue-statuses/{statusId}` | Delete issue status |

### 3.9. Milestones / Sprints

| Method | Endpoint                    | Description      |
| ------ | --------------------------- | ---------------- |
| GET    | `/milestones`               | List milestones  |
| POST   | `/milestones`               | Create milestone |
| GET    | `/milestones/{milestoneId}` | Get milestone    |
| PATCH  | `/milestones/{milestoneId}` | Update milestone |
| DELETE | `/milestones/{milestoneId}` | Delete milestone |

### 3.10. Users

| Method | Endpoint          | Description                        |
| ------ | ----------------- | ---------------------------------- |
| GET    | `/users/me`       | Get authenticated user information |
| GET    | `/users`          | List users                         |
| GET    | `/users/{userId}` | Get user details                   |

Refer to the official documentation for additional user operations and permissions.

---

## 4. Common API Operations

### 4.1. List projects

```http
GET /projects
```

Retrieve available projects and identify the correct `projectId`.

### 4.2. List tasks in a project

```http
GET /tasks?project=1
```

Replace `1` with the actual project ID.

### 4.3. Create a task

```http
POST /tasks
```

Example request body:

```json
{
  "project": 1,
  "subject": "Implement vessel tracking",
  "description": "Implement vessel tracking functionality using AIS data.",
  "status": 1,
  "assigned_to": null,
  "user_story": null,
  "is_blocked": false,
  "is_closed": false,
  "tags": ["AIS", "vessel-tracking"]
}
```

Required fields:

- `project`
- `subject`

Other fields are optional depending on the operation and project configuration.

Use valid IDs obtained from the API. Do not assume that `status: 1` is the correct status for a project.

### 4.4. Update a task

```http
PATCH /tasks/{taskId}
```

Example request body:

```json
{
  "subject": "Implement vessel tracking module",
  "description": "Implement vessel tracking using AIS position reports."
}
```

Only send the fields that need to change.

### 4.5. Change task status

```http
PATCH /tasks/{taskId}
```

Example request body:

```json
{
  "status": 2,
  "is_closed": true
}
```

Important:

- Retrieve task statuses first.
- Identify the correct status ID.
- Do not assume that `is_closed` alone changes the workflow status.
- Verify the task after updating it.

### 4.6. Create a user story

```http
POST /userstories
```

Example:

```json
{
  "project": 1,
  "subject": "Vessel Monitoring",
  "description": "Provide real-time vessel monitoring through AIS integration."
}
```

### 4.7. Delete a task

```http
DELETE /tasks/{taskId}
```

Deletion is destructive. Always request explicit user confirmation before executing it.

---

## 5. AI Operating Instructions

When interacting with Taiga, follow these rules.

### 5.1. Identify the correct project

When a user mentions a project by name:

1. Retrieve the project list.
2. Match the requested project name or slug.
3. If multiple projects match, ask the user which one they mean.
4. Store the resolved project ID for subsequent operations in the current workflow.

Never guess project IDs.

### 5.2. Identify existing records

When a user requests an update:

1. Search the relevant project.
2. Identify the requested task, user story, or issue.
3. Check its ID, reference number, and subject.
4. If multiple records match, ask for clarification.
5. Retrieve the full record when necessary.
6. Update only the requested fields.

Never create a duplicate record when the user intends to update an existing one.

### 5.3. Resolve status IDs

Before changing statuses:

1. Retrieve the appropriate status list.
2. Identify the status that matches the user's requested status.
3. Use its actual ID in the update request.
4. Confirm that the status belongs to the correct project or configuration.

### 5.4. Create records

When creating tasks or user stories:

- Use the correct project ID.
- Use a concise and meaningful subject.
- Write a clear description.
- Associate a user story or milestone only when appropriate.
- Assign users only when the correct user ID is known.
- Do not invent IDs or configuration values.

### 5.5. Verify every modification

After creating or updating a record:

1. Check the HTTP response.
2. Retrieve the affected record using its ID.
3. Confirm that the requested fields were saved.
4. Report the result to the user.

Never claim an operation succeeded if the API returned an error or the result could not be verified.

---

## 6. Safety and Permission Rules

| Operation                    | Confirmation                   |
| ---------------------------- | ------------------------------ |
| List projects/tasks          | Not required                   |
| Read task details            | Not required                   |
| Create task                  | Follow user instruction        |
| Update task description      | Follow user instruction        |
| Change task status           | Follow user instruction        |
| Reassign task                | Follow user instruction        |
| Delete task                  | Explicit confirmation required |
| Delete project               | Explicit confirmation required |
| Delete user story            | Explicit confirmation required |
| Change project configuration | Explicit confirmation required |

Additional rules:

- Do not modify unrelated records.
- Do not silently change multiple records when the user requested one.
- Do not expose private user information.
- Do not change project memberships or permissions without explicit authorization.
- If an API operation fails, explain the error instead of pretending it succeeded.

---

## 7. API Error Handling

Common HTTP response codes:

| HTTP Code | Meaning           | AI Action                                  |
| --------- | ----------------- | ------------------------------------------ |
| 200       | Success           | Process response                           |
| 201       | Created           | Record creation succeeded                  |
| 204       | No content        | Operation succeeded without response body  |
| 400       | Bad request       | Inspect request body and validation errors |
| 401       | Unauthorized      | Refresh token or re-authenticate           |
| 403       | Forbidden         | Check user permissions                     |
| 404       | Not found         | Verify record ID and endpoint              |
| 409       | Conflict          | Inspect conflict and retry only when safe  |
| 429       | Too many requests | Respect rate limits and retry later        |
| 500       | Server error      | Report failure and retry cautiously        |

Taiga also documents optimistic concurrency control. If an update encounters a version conflict, retrieve the latest record before deciding how to proceed.

---

## 8. Recommended AI Workflow

Example user request:

> Move the AIS Integration task to Done in the Marine Management project.

Expected workflow:

1. Authenticate with Taiga.
2. Retrieve the project list.
3. Resolve the Marine Management project ID.
4. Retrieve tasks belonging to that project.
5. Find the task named AIS Integration.
6. Resolve the correct Done status ID.
7. Send a PATCH request to update the task.
8. Retrieve the task again.
9. Verify its status.
10. Report the result.

Expected response:

```text
Successfully updated task.

Project: Marine Management
Task: AIS Integration
Previous status: In Progress
New status: Done
```

The response must reflect actual API results, not assumptions.

---

## 9. Official Documentation and References

Use these sources as the primary reference for API behavior, endpoint parameters, request schemas, and response structures.

**Official Taiga REST API documentation**

https://docs.taiga.io/api.html

**Official Taiga documentation**

https://docs.taiga.io/

**Official Taiga API documentation source repository**

https://github.com/taigaio/taiga-doc

### Important

This document is a practical reference for common Taiga operations, not an exhaustive API specification.

When an endpoint, parameter, request body, or response field is unclear, consult the official REST API documentation before making the request.

Do not invent undocumented endpoints or request parameters.

---

## 10. Final Instructions to AI

You are authorized to interact with Taiga through its REST API to help the user manage projects, tasks, user stories, issues, and milestones.

Your priorities are:

1. Understand the user's intention.
2. Resolve the correct project and record.
3. Use the official Taiga REST API.
4. Validate IDs and status values.
5. Modify only the requested data.
6. Verify changes after every write operation.
7. Protect authentication credentials.
8. Report accurate results in clear language.

Treat Taiga as the source of truth for all project management data.
