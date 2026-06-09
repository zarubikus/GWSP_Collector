# Google Workspace Setup

1. Create or choose a Google Cloud project.
2. Enable the Admin SDK API and Google Workspace Alert Center API.
3. Create a service account and JSON key.
4. Enable domain-wide delegation on the service account.
5. In the Google Admin console, authorize the service account client ID with these OAuth scopes:

```text
https://www.googleapis.com/auth/admin.reports.audit.readonly,https://www.googleapis.com/auth/apps.alerts
```

6. Choose a delegated admin subject, for example `admin@example.com`.
7. Store the JSON key outside the repository and point `google.service_account_file` or `GWSP_SERVICE_ACCOUNT_FILE` to it.
8. Optionally store the delegated admin in `GWSP_DELEGATED_ADMIN` and set `google.delegated_admin_env`.

The service account key is sensitive. Limit filesystem access to the account running the collector.
