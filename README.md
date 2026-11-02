# Taiga Link

A small integration project for working with [Taiga](https://taiga.io/) through its REST API. It keeps the API connection settings in an environment file and provides API guidance that can be used by an AI assistant or other client.

## Contents

- `taiga-api.md` — API reference and operating instructions for an AI assistant.
- `taiga-get-token.bat` — Windows helper that signs in to Taiga and saves the authentication tokens to `.env`.
- `.env` — Local configuration and credentials. Create this file locally; **do not commit it**.

## Requirements

- A Taiga account.
- Windows, if you want to use `taiga-get-token.bat`.
- PowerShell (included with supported Windows versions).
- An application, script, or AI client that can make HTTP requests to the Taiga REST API.

## 1. Configure the project

Create a `.env` file in the project root:

```dotenv
TAIGA_BASE_URL=https://api.taiga.io/api/v1
TAIGA_AUTH_TOKEN=
TAIGA_REFRESH_TOKEN=
```

For a self-hosted Taiga installation, replace `TAIGA_BASE_URL` with that instance's API base URL.

Make sure `.env` is ignored by Git. Add this to `.gitignore`:

```gitignore
.env
```

If `.env` was already committed, remove it from Git's index (this does not delete your local file):

```bash
git rm --cached .env
```

You may commit an `.env.example` file containing the variable names and non-secret defaults, but never put real tokens or passwords in it.

## 2. Get your Taiga authentication tokens

On Windows:

1. Run `taiga-get-token.bat` from the project directory.
2. Enter your Taiga username or email address.
3. Enter your password when prompted. The password prompt is hidden.
4. If authentication succeeds, the script writes the API base URL and the returned tokens to `.env`.

The script preserves an existing, non-empty `TAIGA_BASE_URL`. If that setting is missing or empty, it uses the Taiga Cloud API URL.

After successful authentication, `.env` should look like this:

```dotenv
TAIGA_BASE_URL=https://api.taiga.io/api/v1
TAIGA_AUTH_TOKEN=your_access_token
TAIGA_REFRESH_TOKEN=your_refresh_token
```

The example values above are placeholders. Do not copy them as real credentials.

If authentication fails, check your credentials, network connection, and API base URL. The script should not replace `.env` when the authentication request fails.

## 3. Use the Taiga API

Use the values from `.env` when making API requests:

- Base URL: `TAIGA_BASE_URL`
- Access token: `TAIGA_AUTH_TOKEN`
- Refresh token: `TAIGA_REFRESH_TOKEN`

Authenticated requests use this HTTP header:

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
```

For example, a request to an API endpoint has this general form:

```http
GET https://api.taiga.io/api/v1/ENDPOINT/
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Replace `ENDPOINT` with the endpoint documented by Taiga. Refer to [`taiga-api.md`](./taiga-api.md) for the project's API instructions and endpoint guidance.

### Token refresh

Access tokens expire. When an API request fails because the access token has expired:

1. Send a `POST` request to `{TAIGA_BASE_URL}/auth/refresh`.
2. Send the current refresh token in the JSON body:

   ```json
   {
     "refresh": "YOUR_REFRESH_TOKEN"
   }
   ```

3. If Taiga returns new tokens, update **both** `TAIGA_AUTH_TOKEN` and `TAIGA_REFRESH_TOKEN` in `.env` when both are supplied.
4. If refresh fails or the refresh token is no longer valid, authenticate again using the login helper.

Do not repeatedly retry an unauthorized request with the same expired access token.

## 4. Working with an AI assistant

If you use an AI assistant to operate Taiga:

1. Provide the assistant with the instructions in [`taiga-api.md`](./taiga-api.md).
2. Make the required `.env` variables available to the integration at runtime through a secure mechanism.
3. Ask the assistant to perform a specific Taiga operation, such as finding a project, listing user stories, or updating a task.
4. Require it to verify the result by retrieving the affected resource after a write operation.

Example request:

> Find the project named “My Project”, list its open user stories, and show their assigned members.

Another example:

> Update task 123 to the required status, then retrieve the task again to verify the change.

The examples are illustrative. The available operations depend on the endpoints and permissions supported by your Taiga instance and the integration client.

### Instructions for AI Assistants

```
You are an assistant that helps the user interact with their Taiga instance through the Taiga REST API.

General Instructions:

- Read `taiga-api.md` first. It contains the API reference and instructions for interacting with Taiga.
- Read `.env` to obtain the configured API base URL and authentication tokens. Treat all values in `.env` as secrets.
- Never reveal, print, or repeat passwords or tokens.
- Use the API endpoints and request formats documented in `taiga-api.md`.
- Verify changes after creating, updating, or deleting resources.
- If the access token has expired, try refreshing it according to `taiga-api.md`.
- If token refresh fails or the refresh token is expired or invalid, stop and tell the user to run `taiga-get-token.bat` to authenticate again.
- Never ask the user to paste passwords or tokens into the chat.
- Never claim an operation succeeded unless it has been verified.

Activity Logging:

- You MUST record every action you perform through the Taiga API, including READ operations and failed requests.
- Store activity logs in the `logs/` directory.
- Use one log file per day, named `YYYY-MM-DD.log`.
- Create the `logs/` directory if it does not exist.
- Record each action immediately after the API request completes.
- Include:
  - Timestamp
  - Action (CREATE, READ, UPDATE, DELETE)
  - HTTP method and API endpoint
  - Resource type (Project, User Story, Task, Issue, etc.)
  - Resource ID, if available
  - Description of what was requested or changed
  - Result (SUCCESS or FAILED)
  - Verification status for write operations
  - Error details, if applicable
- For updates, record the relevant fields changed, including their previous and new values when available.
- Never write passwords, access tokens, refresh tokens, or other secrets into log files.
- Never claim an action succeeded if the API request failed.
- If an action cannot be logged, inform the user.

Example log entries:

[2026-11-02 14:30:15] UPDATE | PATCH /tasks/123 | Task | ID: 123 | Status: Open -> In Progress | SUCCESS | VERIFIED
[2026-11-02 14:32:10] CREATE | POST /userstories | User Story | Project: 5 | ID: 456 | SUCCESS | VERIFIED
[2026-11-02 14:35:20] DELETE | DELETE /issues/789 | Issue | ID: 789 | FAILED | Error: Permission denied
[2026-11-02 14:40:00] READ | GET /projects | Project | Listed available projects | SUCCESS
```

## Security

- Never commit `.env`, access tokens, refresh tokens, or passwords.
- Never paste credentials into source code, public issues, logs, or screenshots.
- Do not expose tokens to browser-side code or untrusted users.
- Restrict access to the machine, process, and files that contain the tokens.
- If a token is exposed, revoke or rotate it through the appropriate Taiga account or instance procedure, then authenticate again.
- Use HTTPS for Taiga API requests.

## Troubleshooting

| Problem                          | What to check                                                                                |
| -------------------------------- | -------------------------------------------------------------------------------------------- |
| Login fails                      | Check the username/email, password, network connection, and `TAIGA_BASE_URL`.                |
| API returns `401 Unauthorized`   | Refresh the access token. If refresh fails, sign in again.                                   |
| API returns `403 Forbidden`      | Check the Taiga account's permissions for the requested project or resource.                 |
| API returns `404 Not Found`      | Check the endpoint path, resource ID, and whether the resource is accessible to the account. |
| `.env` is missing tokens         | Run `taiga-get-token.bat` again and confirm that authentication succeeds.                    |
| Using a self-hosted Taiga server | Set `TAIGA_BASE_URL` to that server's API base URL.                                          |

## Notes

This README documents the configuration and authentication workflow currently provided by the project. Add the actual application-specific installation and run commands here once the app's runtime/framework and entry point are defined.
