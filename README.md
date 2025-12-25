# AI Configuration System: Technical Implementation

This document explains the architecture and data flow for saving and managing AI task configurations in the Discurso API.

## 1. Database Connectivity (The Fix)
The system uses **Firebase Admin SDK** to interact with Google Firestore.

### Service Account Initialization
The connection is established in `src/v1/db.js`. I implemented a fix to ensure the `firebase-admin-config.json` file is correctly detected. The initialization logic now searches for credentials in this order:
1. `FIREBASE_ADMIN_CONF` environment variable (JSON string).
2. `firebase-admin-config.json` in the project root.
3. `serviceAccountKey.json` in the project root.
4. `datastore-key.json` in the project root.
5. Application Default Credentials (GCP environment).

### Multi-Database Support
The system supports specific Firestore databases via the `FIRESTORE_DB_NAME` environment variable (currently set to `dev-db3`).

## 2. Configuration Saving Flow (UI to DB)

When a user saves a configuration in the UI, the following flow occurs:

1.  **UI Component (`page.jsx`)**: The `handleSaveConfig` function is triggered.
2.  **API Call (`apiFetch`)**: A `POST` request is sent to `/v1/configs` on the backend.
    *   **Authentication**: The request includes the `x-api-key` header (`aItDCI7ee8q3sjU4myuwKvA3aDEPfN`).
    *   **Payload**: The configuration object (model settings, system prompt, user prompt, etc.) is sent as JSON.
3.  **Backend Route (`server.mjs`)**: The `/v1/configs` endpoint receives the request and calls `updateConfig`.
4.  **Logic Layer (`configs.js`)**: 
    *   `updateConfig` strips existing IDs and timestamps to create a *new* version.
    *   It adds new metadata: `active: 0` (inactive by default), `author: userId`, and `time: Date.now()`.
5.  **Persistence Layer (`db.js`)**: The `saveDoc` function is called. It uses `db.collection("aiConfigs").doc().set()` to generate a unique ID and save the document to Firestore.

## 3. Configuration Retrieval Flow (DB to UI)

1.  **UI Mounting**: The `loadTaskConfigs` function is called via `useEffect`.
2.  **API Call**: A `GET` request is sent to `/v1/configs/by-task/${taskId}`.
3.  **Backend Logic**:
    *   **Retrieval**: `getConfigsByTask` calls `retrieveAllConfigs`.
    *   **Database First**: The system tries to fetch from the `aiConfigs` collection in Firestore.
    *   **Fallback**: If the database is empty for that task, it reads from `src/v1/defaultConfigs.json` and automatically migrates those defaults into the database.
    *   **Filtering**: The backend filters configurations by the requested `taskId` and sorts them by time (newest first).
4.  **UI Display**: The results (an array of configuration versions) are stored in the `taskConfigs` state and rendered as cards.

## 4. Setting Active Configuration

A separate `PATCH` request is sent to `/v1/configs/:id/active`. This updates the `active` field in the database with a timestamp (or `0` to deactivate). The UI and backend logic treat the config with the highest `active` timestamp as the "currently live" version for that task.

## Key Files Reference
- `api-ui/app/(inner)/ai-config/task/page.jsx`: Main UI logic.
- `src/v1/db.js`: Firestore initialization and CRUD helpers.
- `src/v1/configs.js`: Business logic for managing versions and defaults.
- `server.mjs`: API route definitions.
- `firebase-admin-config.json`: Database credentials.
