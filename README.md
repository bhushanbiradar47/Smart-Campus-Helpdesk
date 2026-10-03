# Smart Campus Helpdesk

A student support ticket workflow built with **n8n**, **Google Sheets**, **Gmail**, **Docker**, **Caddy**, and a **Cloudflare Quick Tunnel**.

## What it does

Students submit a complaint through an n8n form. The workflow then:

1. Collects student name, email, category, and complaint description.
2. Generates a ticket ID.
3. Classifies the complaint into a department.
4. Assigns a priority.
5. Sets the ticket status.
6. Appends the ticket to Google Sheets.
7. Sends a confirmation email through Gmail.

## Workflow

```text
Student
   |
   v
n8n Form Trigger
   |
   v
Code in JavaScript
(ticket ID + department + priority + status)
   |
   +--------------------+
   |                    |
   v                    v
Google Sheets          Gmail
(ticket record)        (confirmation)
```

## Public-access architecture used for the demo

```text
Student Browser
      |
      | HTTPS
      v
Cloudflare Quick Tunnel
      |
      v
Caddy Gateway (:8080)
      |
      | only /form/*
      v
n8n (:5678)
      |
      +--> Google Sheets
      +--> Gmail
```

The Caddy gateway intentionally returns `404 Not Found` for paths outside `/form/*`, so the public tunnel does not directly expose the n8n editor.

## Technology stack

- **n8n** — workflow automation
- **Docker Desktop** — local container runtime
- **Google Sheets** — ticket storage
- **Gmail** — student confirmation emails
- **Caddy** — reverse proxy / public gateway
- **Cloudflare Quick Tunnel** — temporary HTTPS access for demonstration

## Repository structure

```text
Smart-Campus-Helpdesk/
├── README.md
├── .gitignore
├── workflow/
│   └── student-support-ticket-workflow.json
└── gateway/
    └── Caddyfile
```

## Import the workflow

The workflow file in this repository is a **sanitized copy** of the working n8n workflow.

It does not include the original Google credentials or the original Google Sheet ID. After importing it into your n8n instance:

1. Import `workflow/student-support-ticket-workflow.json`.
2. Create/connect your own Google Sheets credential.
3. Create/connect your own Gmail credential.
4. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` with your Google Sheet document ID.
5. Map the Google Sheets node to your own spreadsheet.
6. Activate/test the workflow.

Do not commit credential files, n8n data directories, or private environment files.

## Google Sheet columns

Use columns matching the workflow output:

```text
Ticket ID
Student Name
Student Email
Category
Description
Department
Priority
Status
Submitted At
```

## Complaint classification

The JavaScript Code node currently uses keyword-based rules to classify complaints into departments and priority levels. The categories represented by the form are:

- Classroom/Lab
- Wifi
- Maintenance
- Hostel
- Canteen

The workflow also produces a ticket status of `Open` when the required email and description are present; otherwise it marks the ticket `Invalid`.

## Running with Docker

The project was developed with n8n running in Docker and connected to a Docker network shared with the Caddy gateway.

A typical local setup is:

```text
n8n container  --->  n8n-public network  <---  Caddy container
```

The exact Docker volume and container names are local-environment choices and should not be committed as secrets.

## Caddy gateway

The included `gateway/Caddyfile` exposes only the n8n form path:

```text
/form/*
```

All other paths receive a 404 response.

## Cloudflare Quick Tunnel

For a temporary public demo, Cloudflare's Quick Tunnel can forward the Caddy port to an HTTPS URL.

Example:

```text
cloudflared tunnel --url http://localhost:8080
```

Quick Tunnel URLs are temporary and can change when the tunnel process stops or restarts. Therefore, a Quick Tunnel URL should not be treated as a permanent production endpoint.

## Security notes

- The public repository contains a sanitized workflow rather than the original export.
- Google credential objects were removed from the repository copy.
- The original Google Sheet document ID was replaced with a placeholder.
- Local n8n data and database files are excluded through `.gitignore`.
- The Caddy gateway limits public access to `/form/*`.
- The public demo uses a temporary Cloudflare Quick Tunnel.

## Testing

The working workflow was tested by submitting the student form and confirming that:

- a ticket row was added to Google Sheets, and
- a confirmation email was delivered through Gmail.

## Future enhancements

Possible extensions include:

- ticket-status lookup for students,
- administrator dashboard,
- department-specific notifications,
- escalation rules for critical tickets,
- duplicate-ticket detection,
- analytics and reporting,
- persistent production hosting with a permanent domain.
