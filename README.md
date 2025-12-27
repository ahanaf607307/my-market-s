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


