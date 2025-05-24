
# Vertex AI Services Authentication

This is a short compilation of the official documentation:

-   [Vertex AI Authentication](https://cloud.google.com/vertex-ai/docs/authentication)
-   [Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials)


[Audio version](https://github.com/svetamorag/VertexAI_Authentication/blob/main/VertexAI_Auth_Eng.mp3)

[Hebrew Version](https://github.com/svetamorag/VertexAI_Authentication/blob/main/VertexAI_Auth_Heb.md)
[Hebrew audio version](https://github.com/svetamorag/VertexAI_Authentication/blob/main/VertexAI_Auth_Heb.mp3)

Authentication is the process of verifying *who* is making a request, establishing the identity of a user or an application. This is a prerequisite for authorization, which determines *what resources* the authenticated identity can access and *what level of access* they have. Understanding the distinction between authentication and authorization is crucial for diagnosing access issues. Vertex AI uses Identity and Access Management (IAM) for authorization after successful authentication.

Depending on where your code is running and the tools you are using, you can authenticate to Vertex AI using various interfaces and methods.

## 1. Using Application Default Credentials (ADC)

-   **How it works:** ADC is a strategy used by Google Cloud authentication libraries to find credentials based on the application's environment automatically. This allows your code to run in different environments (local development, production) without changing how it authenticates. Client libraries are designed to discover and use credentials via ADC automatically. REST requests can also use ADC.
-   **ADC** searches for credentials in the following order:
    1.  The `GOOGLE_APPLICATION_CREDENTIALS` environment variable can point to a credential JSON file (Workforce Identity Federation config, Workload Identity Federation config, or a service account key).
    2.  A credential file created by using the `gcloud auth application-default login` command.
    3.  The attached service account, returned by the metadata server.
    4.  Sample usage (terminal): `$env:GOOGLE_APPLICATION_CREDENTIALS="C:\\Users\\sveta\\Documents\\DevProjects\\VertexAI Auth\\key.json"`
-   **Limitations/Considerations:** ADC itself is a strategy; limitations depend on the underlying credential source it finds.

## 2. Using User Credentials via `gcloud auth application-default login` (Recommended for Local Development)

-   **How it works:** This method is primarily for setting up credentials in a local development environment. Running the `gcloud auth application-default login` command authenticates your user account and stores credentials in a local JSON file that ADC uses.
-   **Limitations/Considerations:** This method is not suitable for production environments. By default, the access tokens generated include the broad `https://www.googleapis.com/auth/cloud-platform` scope, which provides extensive access and potentially more privileges than strictly needed for an application. You can use the `--scopes` flag to limit permissions.
-   **Ideal Use Case:** Rapid prototyping, personal development, and local testing on a workstation.

## 3. Using a Service Account Key File (Alternative for Local Development or non-GCP CI/CD)

-   **How it works:** A service account key is a JSON file containing credentials for a service account. You can provide this key file to ADC by setting the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to its absolute path. This allows testing an application with the exact permissions of a specific service account.
-   **Limitations/Considerations:** Service account keys pose a significant security risk if compromised. Unlike other credential types, a stolen key can provide direct access without additional authentication. Service account keys are strongly discouraged for production environments and must never be committed to version control. They require meticulous management and secure storage. Keys are static and do not automatically rotate.
-   **Ideal Use Case:** Testing specific service account permissions locally in non-GCP hosted CI/CD pipelines where attached service accounts are not an option.

## 4. Using an Attached Service Account (Recommended for Production on Google Cloud)

-   **How it works:** When your application runs on a Google Cloud compute resource (like Cloud Run, Compute Engine, GKE), you can attach a service account to that resource. ADC uses the metadata server to obtain credentials for this attached service account. Google Cloud automatically handles the secure acquisition of short-lived access tokens.
-   **Limitations/Considerations:** Proper IAM role configuration is required for the service account. For most services (including Cloud Run), the service account must be attached when the resource is created or deployed and can generally not be changed later without redeploying.
-   **Ideal Use Case:** This is the preferred and most secure method for production deployments on Cloud Run, Compute Engine, GKE, and other GCP-managed services. It eliminates the need to manage key files within your code or container.

## 5. Using `gcloud` CLI Credentials

-   **How it works:** When you use the `gcloud` CLI, you log in with a user account, which provides the credentials for executing `gcloud` commands. You can also use service account impersonation with the `gcloud` CLI. These credentials are distinct from the ADC credentials created by `gcloud auth application-default login`.
-   **Limitations/Considerations:** Primarily used for interacting with Google Cloud via the command line. If organizational security policies prevent user accounts from having necessary permissions, service account impersonation might be required.
-   **Ideal Use Case:** Managing Vertex AI and other Google Cloud resources from the command line.

## 6. Using REST

-   **How it works:** You can authenticate REST requests to the Vertex AI API using your `gcloud` CLI credentials (e.g., by including an access token obtained via `gcloud auth print-access-token` in the request header) or by using Application Default Credentials (ADC).
-   **Limitations/Considerations:** Requires including authentication information (like an access token) as part of the request.
-   **Ideal Use Case:** Direct programmatic interaction with the Vertex AI API or using command-line tools like `curl`.

## 7. Using Service Account Impersonation

-   **How it works:** Allows one principal (like a user or another service account) to obtain credentials for a service account, effectively acting as that service account. This requires the principal to have the `iam.serviceAccounts.getAccessToken` permission, which is included in the `roles/iam.serviceAccountTokenCreator` role. You can configure the `gcloud` CLI to use impersonation or create a local ADC file with impersonation for specific client libraries.
-   **Limitations/Considerations:** This function requires the invoking principal to have the `roles/iam.serviceAccountTokenCreator` role. Creating a local ADC file with impersonation is only supported for the Go, Java, Node.js, and Python client libraries.
-   **Ideal Use Case:** Testing the permissions assigned to a specific service account. When organizational security policies prevent the use of user credentials with the necessary permissions.

## 8. Using Workload Identity Federation or Workforce Identity Federation (On-premises or another cloud)

-   **How it works:** These methods allow entities authenticated by an external identity provider (IdP) to access Google Cloud resources without needing service account keys. This is the preferred method for authenticating from outside of Google Cloud. The `GOOGLE_APPLICATION_CREDENTIALS` environment variable can point to a credential configuration file.
-   **Limitations/Considerations:** This involves a more complex setup than the methods used within Google Cloud.
-   **Ideal Use Case:** Authenticating applications or users outside Google Cloud (on-premises, other cloud providers) and integrating with existing enterprise identity systems.

Regardless of the authentication method used, proper authorization via IAM roles is essential to ensure the authenticated identity has the necessary permissions to interact with Vertex AI resources. Adhering to the Principle of Least Privilege – granting only the minimum necessary permissions – is a critical security best practice. Avoid assigning overly broad roles like "Project Editor" or "Project Owner" to service accounts.
