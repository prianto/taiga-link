# Taiga Link

A small integration project for working with [Taiga](https://taiga.io/) through its REST API. It keeps the API connection settings in an environment file and provides API guidance that can be used by an AI assistant or other client.

## Contents

- `AGENTS.md` — Operating instructions for AI assistants (rules, token handling, activity logging).
- `taiga-api.md` — API reference used by AI assistants.
- `taiga-get-token.bat` — Windows helper that signs in to Taiga and saves the authentication tokens to `.env`.
- `.env` — Local configuration and credentials. Create this file locally; **do not commit it**.
- `logs/` — Daily activity logs written by the AI assistant (`YYYY-MM-DD.log`). Created automatically.

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

Replace `ENDPOINT` with the endpoint documented by Taiga. Refer to [`taiga-api.md`](./taiga-api.md) for endpoint guidance.

### Token expiry

Access tokens expire. An AI assistant will try to refresh the token automatically (see [`AGENTS.md`](./AGENTS.md)). If the refresh token is also expired or invalid, run `taiga-get-token.bat` again to sign in.

## 4. Working with an AI assistant

1. Point your AI assistant to [`AGENTS.md`](./AGENTS.md) and [`taiga-api.md`](./taiga-api.md). Many tools read `AGENTS.md` automatically.
2. Make the required `.env` variables available to the integration at runtime through a secure mechanism.
3. Ask the assistant to perform a specific Taiga operation, such as finding a project, listing user stories, or updating a task.

Example requests:

> Find the project named “My Project”, list its open user stories, and show their assigned members.

> Update task 123 to the required status, then retrieve the task again to verify the change.

The examples are illustrative. The available operations depend on the endpoints and permissions supported by your Taiga instance and the integration client.

### What to expect

- The assistant verifies every create, update, and delete by re-reading the affected resource.
- Every API action (including reads and failures) is recorded in `logs/YYYY-MM-DD.log`. Tokens and passwords are never written to logs.
- If authentication cannot be refreshed, the assistant stops and asks you to run `taiga-get-token.bat`. Never paste passwords or tokens into the chat.

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
