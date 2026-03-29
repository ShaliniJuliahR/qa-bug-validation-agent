# QA Bug Validation Agent

## Setup

1. Clone this repo and open it in Cursor.
2. Run `npm install` to install dependencies.
3. Create a `.env` file at the project root and update the following variables:

```
JIRA_URL=<your-jira-url>
JIRA_PERSONAL_TOKEN=<your-jira-pat>

ZEPHYR_PROJECT_KEY=<your-project-id>
ZEPHYR_BASE_URL=<your-jira-url>
ZEPHYR_API_KEY=<your-jira-pat>

POSTMAN_WORKSPACE_ID=<your-workspace-id>
POSTMAN_API_KEY=<your-postman-api-key>

TRIMBLE_QA_UI_EMAIL=<your-test-email>
TRIMBLE_QA_UI_PASSWORD=<your-test-password>
```

4. Update the Postman API key in `.cursor/mcp.json` under `postman_mcp_server > headers > Authorization`.
5. Open a new Cursor chat, attach `.cursor/rules/setup/initial-setup.mdc`, and type **hi** to run the guided setup.
