# Linkwarden Helm Chart (Unofficial) 🔗

This Helm chart deploys [Linkwarden](https://linkwarden.app/), a self-hosted, open-source collaborative bookmark manager to collect, organize and preserve webpages, on a Kubernetes cluster.


## Table of Contents 📚

- [Linkwarden Helm Chart (Unofficial) 🔗](#linkwarden-helm-chart-unofficial-)
  - [Table of Contents 📚](#table-of-contents-)
  - [Overview 🎯](#overview-)
  - [Features ✨](#features-)
  - [Security Considerations 🔒](#security-considerations-)
  - [Dependencies 📦](#dependencies-)
  - [Quick Start 🚀](#quick-start-)
  - [Configuration ⚙️](#configuration-️)
    - [`values.yaml` Deep Dive 🧐](#valuesyaml-deep-dive-)
    - [CNPG PostgreSQL Database 🐘](#cnpg-postgresql-database-)
      - [CNPG Operator Installation ⚙️](#cnpg-operator-installation-️)
    - [Storage Options 🗄️](#storage-options-️)
    - [Ingress Configuration 🌐](#ingress-configuration-)
    - [External Database Configuration 💽](#external-database-configuration-)
    - [SSO (Single Sign-On) Configuration 🔑](#sso-single-sign-on-configuration-)
  - [Setup Instructions 🛠️](#setup-instructions-️)
  - [Upgrading ⬆️](#upgrading-️)
  - [Troubleshooting 🐞](#troubleshooting-)
  - [Contributing 🤝](#contributing-)
  - [License 📜](#license-)
  - [Credits 🙏](#credits-)


## Overview 🎯

[Linkwarden](https://linkwarden.app/) is a self-hosted, open-source collaborative bookmark manager to collect, organize and archive webpages.

The objective is to organize useful webpages and articles you find across the web in one place, and since useful webpages can go away (see the inevitability of Link Rot), Linkwarden also saves a copy of each webpage as a Screenshot and PDF, ensuring accessibility even if the original content is no longer available.

Additionally, Linkwarden is designed with collaboration in mind, sharing links with the public and/or allowing multiple users to work together seamlessly. Linkwarden allows you to save, tag, and organize links, as well as automatically archive the content for future access.

This Helm chart simplifies the deployment of Linkwarden within a Kubernetes environment, providing flexibility in terms of storage, database, and network configuration.

If you find this chart helpful, please consider giving it a star on GitHub! ⭐ Your support is greatly appreciated!


## Features ✨

- **Simplified Deployment:** Easily deploy Linkwarden on Kubernetes using Helm.
- **Flexible Configuration:** Customize the deployment with a wide range of options via the `values.yaml` file.
- **Built-in PostgreSQL Database:** Optionally deploy Linkwarden with an integrated PostgreSQL database using the CNPG operator.
- **External Database Support:** Connect to an external PostgreSQL database.
- **Filesystem or S3 Storage:** Choose between local filesystem storage using a Persistent Volume Claim or S3-compatible object storage for data persistence.
- **Ingress Support:** Expose Linkwarden to the internet using Ingress rules.
- **SSO Integration:** Supports multiple SSO providers for user authentication.
- **Resource Management:** Configure CPU/memory resource requests and limits.
- **Autoscaling:** Automatically scale the number of Linkwarden replicas based on demand.
- **Security Contexts:** Configure pod and container security contexts for enhanced security.
- **Customizable Probes:** Configure liveness and readiness probes for health monitoring.
- **Easy Upgrades:** Easily upgrade the chart to newer versions with Helm.


## Security Considerations 🔒

- **Secrets Management:** Ensure that sensitive values (like `nextAuthSecret`, database passwords for external databases, and API keys) are properly handled, ideally using Kubernetes secrets and not directly in `values.yaml`.
- **Ingress Security:** When using ingress, configure TLS/SSL to ensure secure connections.
- **Resource Limits:** Set appropriate resource limits to prevent denial-of-service situations and ensure stability.
- **Pod Security Context:** Use the `podSecurityContext` and `containerSecurityContext` to apply principle of least privilege.
- **Network Policies:** (Optional) Consider implementing network policies to restrict network access between pods and services, especially for the database if you are not using the internal one.


## Dependencies 📦

- Kubernetes 1.25+
- Helm 3+
- (Optional) For Ingress: An Ingress Controller (e.g., Nginx, Traefik)
- (Optional) CloudNativePG (CNPG) Operator if using the built-in PostgreSQL database.


## Quick Start 🚀

1.  **Add the Helm repository (if necessary):**
    
    ```bash
    helm repo add tuxalex-linkwarden https://tuxalex.github.io/linkwarden-helm-chart/
    helm repo update
    ```

2.  **Install the chart:**

    ```bash
    helm install linkwarden tuxalex-linkwarden/linkwarden -f <values.yaml>
    ```

    *Replace `<values.yaml>` with the path to your values.yaml file or create one. See the [Configuration](#configuration) section for details.*

3.  **Access Linkwarden:**
    - If you have an ingress configured, access it through the URL specified in your ingress configuration.
    - If not, you may need to set up port forwarding. Please refer to your Kubernetes setup.


## Configuration ⚙️

This section details the configurable values in the `values.yaml` file and how they influence the deployment.

### `values.yaml` Deep Dive 🧐

Here's a breakdown of each section in your `values.yaml`:

| Key | Description | Default Value |
| --- | ----------- | ------------- |
| `nameOverride` | Override the chart name | `""` |
| `fullnameOverride` | Override the full release name | `""` |
| `replicaCount` | Number of pod replicas | `1` |
| `image.repository` | Container image repository | `ghcr.io/linkwarden/linkwarden` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `image.tag` | Image tag (defaults to chart `appVersion`) | `""` |
| `imagePullSecrets` | Secrets for pulling from private registries | `[]` |
| `serviceAccount.create` | Create a ServiceAccount | `true` |
| `serviceAccount.automount` | Auto‑mount ServiceAccount token | `true` |
| `serviceAccount.annotations` | Annotations for the ServiceAccount | `{}` |
| `serviceAccount.name` | Name of the ServiceAccount (generated if empty) | `""` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `3000` |
| `livenessProbe.httpGet.path` | Liveness probe HTTP path | `/` |
| `livenessProbe.httpGet.port` | Liveness probe port | `http` |
| `livenessProbe.initialDelaySeconds` | Initial delay before first check | `30` |
| `livenessProbe.periodSeconds` | Period between checks | `30` |
| `readinessProbe.httpGet.path` | Readiness probe HTTP path | `/` |
| `readinessProbe.httpGet.port` | Readiness probe port | `http` |
| `readinessProbe.initialDelaySeconds` | Initial delay | `30` |
| `readinessProbe.periodSeconds` | Period | `30` |
| `linkwarden.existingSecret` | Existing secret with Linkwarden env vars | `""` |
| `linkwarden.nextAuthUrl` | NextAuth URL | `"http://localhost:3000/api/v1/auth"` |
| `linkwarden.nextAuthSecret` | NextAuth secret | `""` |
| `linkwarden.disableRegistration` | Disable user registration | `false` |
| `linkwarden.credentialsEnabled` | Enable username/password login | `true` |
| `linkwarden.disableNewSSOUsers` | Prevent new SSO users from logging in | `false` |
| `linkwarden.paginationTakeCount` | Links fetched per page | `50` |
| `linkwarden.maxWorkers` | Max Playwright workers | `5` |
| `linkwarden.reArchiveLimit` | Re‑archive cooldown (minutes) | `5` |
| `linkwarden.autoScrollTimeout` | Auto‑scroll timeout (seconds) | `30` |
| `linkwarden.browserTimeout` | Browser timeout (ms) | `30000` |
| `linkwarden.ignoreUnauthorizedCA` | Skip TLS verification | `false` |
| `linkwarden.ignoreHttpsErrors` | Ignore HTTPS errors | `false` |
| `linkwarden.ignoreUrlSizeLimit` | Allow extremely long URLs | `false` |
| `linkwarden.disablePreservation` | Turn off archiving | `false` |
| `linkwarden.monolithMaxBuffer` | Monolith buffer size (MB) | `20` |
| `linkwarden.screenshotMaxBuffer` | Screenshot buffer size (MB) | `20` |
| `linkwarden.readabilityMaxBuffer` | Readability buffer size (MB) | `20` |
| `linkwarden.previewMaxBuffer` | Preview buffer size (MB) | `20` |
| `linkwarden.maxLinksPerUser` | Max links per user | `30000` |
| `linkwarden.maxFileBuffer` | Max file buffer (bytes) | `""` |
| `linkwarden.importLimit` | Max links per import | `""` |
| `linkwarden.rssSubscriptionLimitPerUser` | Max RSS feeds per user | `""` |
| `linkwarden.textContentLimit` | Max indexed characters | `100000` |
| `linkwarden.searchFilterLimit` | Max search filters | `100` |
| `linkwarden.archiveTakeCount` | Links archived per batch | `5` |
| `linkwarden.rssPollingIntervalMinutes` | RSS poll interval (minutes) | `60` |
| `linkwarden.monolithCustomOptions` | Custom CLI options for monolith | `""` |
| `linkwarden.indexTakeCount` | Items indexed per batch | `50` |
| `linkwarden.smtp.emailProvider` | Enable email | `""` |
| `linkwarden.smtp.emailFrom` | Email “from” address | `""` |
| `linkwarden.smtp.emailServer` | SMTP server URL | `""` |
| `linkwarden.browser.proxyUrl` | Proxy URL for Playwright | `""` |
| `linkwarden.browser.proxyUsername` | Proxy username | `""` |
| `linkwarden.browser.proxyPassword` | Proxy password | `""` |
| `linkwarden.browser.proxyBypassUrl` | URLs to bypass proxy (comma‑separated) | `""` |
| `linkwarden.meili.url` | MeiliSearch host URL | `""` |
| `linkwarden.meili.masterkey` | MeiliSearch master key | `""` |
| `linkwarden.meili.timeout` | MeiliSearch timeout (ms) | `30000` |
| `linkwarden.pdf.marginTop` | PDF top margin | `""` |
| `linkwarden.pdf.marginBottom` | PDF bottom margin | `""` |
| `linkwarden.pdf.maxBuffer` | PDF buffer size (MB) | `20` |
| `linkwarden.environment` | Additional env vars | `{}` |
| `linkwarden.storage.type` | Storage backend | `filesystem` |
| `linkwarden.storage.storageFolder` | Folder for filesystem storage | `/data` |
| `linkwarden.storage.pvc.enabled` | Enable PVC for filesystem | `true` |
| `linkwarden.storage.pvc.existingClaim` | Existing PVC name | `""` |
| `linkwarden.storage.pvc.storageClass` | StorageClass for PVC | `""` |
| `linkwarden.storage.pvc.accessModes` | Access modes | `["ReadWriteOnce"]` |
| `linkwarden.storage.pvc.size` | PVC size | `5Gi` |
| `linkwarden.storage.s3.accessKey` | S3 access key | `""` |
| `linkwarden.storage.s3.secretKey` | S3 secret key | `""` |
| `linkwarden.storage.s3.endpoint` | S3 endpoint URL | `""` |
| `linkwarden.storage.s3.bucketName` | S3 bucket name | `""` |
| `linkwarden.storage.s3.region` | S3 region | `""` |
| `linkwarden.storage.s3.forcePathStyle` | Force path‑style URLs | `false` |
| `linkwarden.ai.existingSecret` | Secret with AI env vars | `""` |
| `linkwarden.ai.providers.openai.enabled` | Enable OpenAI provider | `false` |
| `linkwarden.ai.providers.openai.model` | Model to use | `gpt-oss-120b` |
| `linkwarden.ai.providers.openai.apikey` | API key for OpenAI | `""` |
| `linkwarden.ai.providers.openai.secrets.CUSTOM_OPENAI_BASE_URL` | Custom base URL | `""` |
| `linkwarden.ai.providers.openai.secrets.CUSTOM_OPENAI_NAME` | Custom provider name | `""` |
| `linkwarden.externalDatabaseUrl` | External PostgreSQL connection URL | `""` |
| `linkwarden.sso.existingSecret` | Secret with SSO env vars | `""` |
| `linkwarden.sso.providers.google.enabled` | Enable Google SSO | `false` |
| `linkwarden.sso.providers.google.customName` | Display name for Google provider | `"Google"` |
| `linkwarden.sso.providers.google.secrets.GOOGLE_CLIENT_ID` | Google OAuth client ID | `""` |
| `linkwarden.sso.providers.google.secrets.GOOGLE_CLIENT_SECRET` | Google OAuth client secret | `""` |
| `postgresql.enabled` | Deploy integrated CNPG PostgreSQL | `true` |
| `postgresql.cluster.instances` | Number of PostgreSQL instances | `1` |
| `postgresql.cluster.storage.size` | PVC size for PostgreSQL | `2Gi` |
| `postgresql.cluster.storage.storageClass` | StorageClass for PostgreSQL PVC | `""` |
| `ingress.enabled` | Enable Ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts[].host` | Hostname for Ingress | `chart-example.local` |
| `ingress.hosts[].paths[].path` | Path for Ingress rule | `/` |
| `ingress.hosts[].paths[].pathType` | Path type | `ImplementationSpecific` |
| `ingress.tls[]` | TLS configuration | `[]` |
| `httpRoute.enabled` | Enable HTTPRoute | `false` |
| `httpRoute.redirectHttpToHttps` | Auto‑redirect HTTP to HTTPS | `true` |
| `httpRoute.pathPrefix` | Base path prefix | `/` |
| `httpRoute.parentRefsHttp[]` | Parent reference for HTTP traffic | `[{ name: "gateway" }]` |
| `httpRoute.parentRefsHttps[]` | Parent reference for HTTPS traffic | `[{ name: "gateway" }]` |
| `httpRoute.hostnames[]` | Hostnames for HTTPRoute | `["chart-example.local"]` |
| `resources` | Pod resource limits/requests | `{}` |
| `podAnnotations` | Annotations applied to the pod | `{}` |
| `podLabels` | Labels applied to the pod | `{}` |
| `autoscaling.enabled` | Enable Horizontal Pod Autoscaler | `false` |
| `autoscaling.minReplicas` | Minimum replica count | `1` |
| `autoscaling.maxReplicas` | Maximum replica count | `100` |
| `autoscaling.targetCPUUtilizationPercentage` | CPU target for scaling | `80` |
| `podSecurityContext` | Security context for the pod | `{}` |
| `containerSecurityContext` | Security context for the container | `{}` |
| `nodeSelector` | Node selector constraints | `{}` |
| `tolerations` | Tolerations for pod scheduling | `[]` |
| `affinity` | Affinity rules | `{}` |

### CNPG PostgreSQL Database 🐘

This chart includes an *optional* inbuilt PostgreSQL database using the CloudNativePG (CNPG) Operator. This is enabled by default.

- If `postgresql.enabled` is `true`, a PostgreSQL cluster will be deployed within your Kubernetes cluster.
- You can configure the number of instances, storage, and storage class in the `postgresql` section.
- If you use the internal PostgreSQL database, you do not need to set the `linkwarden.externalDatabaseUrl`.
- **Important**: If you enable the internal PostgreSQL database, you **MUST** have the [CloudNativePG Operator](https://cloudnative-pg.io) installed in your cluster.

#### CNPG Operator Installation ⚙️

1.  Add the CloudNativePG Helm repository:
    ```bash
    helm repo add cnpg https://cloudnative-pg.github.io/charts
    helm repo update
    ```

2.  Install the CNPG Operator:
    ```bash
    helm install cnpg cnpg/cloudnative-pg -n cnpg-system --create-namespace
    ```
    *You can choose a different namespace than `cnpg-system` if you want.*

3. Wait for the operator to be ready.
    ```bash
    kubectl -n cnpg-system get pods -l app.kubernetes.io/name=cloudnative-pg
    ```
    *Make sure the pod is in Running state.*


### Storage Options 🗄️

The chart supports two main storage options for Linkwarden data:

1.  **Filesystem Storage:**
    - Uses a Persistent Volume Claim (PVC) to store data in the specified folder: `/data/data`.
    - Enable by setting `linkwarden.storage.type` to `filesystem`.
    - Configure PVC settings under `linkwarden.storage.pvc`.
2. **S3 Storage:**
    - Uses s3 compatible storage to store data.
    - Enable by setting `linkwarden.storage.type` to `s3`.
    - Configure S3 settings under `linkwarden.storage.s3`.


### Ingress Configuration 🌐

- To expose Linkwarden outside the cluster, you can enable the ingress by setting `ingress.enabled` to `true`.
- You will need an Ingress controller (like Nginx or Traefik) installed in your cluster.
- Configure hostnames, paths, and TLS settings in the `ingress` section.


### External Database Configuration 💽

- To use an external PostgreSQL database, set `postgresql.enabled` to `false`, and provide the database URL through the `linkwarden.externalDatabaseUrl` value.
- **Note:** Ensure that the Linkwarden application can reach the database from within the Kubernetes cluster.


### SSO (Single Sign-On) Configuration 🔑

- The `sso` section allows you to configure various SSO providers.
- Refer to the NextAuth.js documentation for the specific settings per provider.
- If `linkwarden.sso.existingSecret` is set all environment variables required for SSO will be taken from that secret.
- Add your provider config to `linkwarden.sso.providers` to enable that provider.


## Setup Instructions 🛠️

1.  **Install Kubernetes:** Make sure you have a working Kubernetes cluster.
2.  **Install Helm:** Follow the official Helm installation guide.
3.  **Create a `values.yaml`:** Customize your `values.yaml` file. Start with the default file and customize based on the details in the [Configuration](#configuration) section of this file.
    - ⚠️ You **MUST** set a `nextAuthSecret`. Generate with `openssl rand -base64 32`
    - You might want to change `linkwarden.nextAuthUrl` if you are not accessing the service at `http://localhost:3000`.
    - If you enable the internal PostgreSQL database you do not need to change any database related settings.
    - If you use an external PostgreSQL database set `postgresql.enabled` to `false`, and provide the database URL through the `linkwarden.externalDatabaseUrl` value.
    - If you enable email you **MUST** specify `emailProvider`, `emailFrom`, and `emailServer`.
    - If you are using an s3 compatible object storage provider, ensure you have all the correct `linkwarden.storage.s3` settings.
4.  **Install the chart:** Use the Helm command in the [Quick Start](#quick-start) section.


## Upgrading ⬆️

To upgrade the chart to a newer version:

1.  Update your Helm repository:
    ```bash
    helm repo update
    ```
2.  Upgrade the release:
    ```bash
    helm upgrade linkwarden shivam-charts/linkwarden -f <values.yaml>
    ```


## Troubleshooting 🐞

- Check the logs of the Linkwarden pod for errors using `kubectl logs <pod-name>`.
- Verify that all necessary Kubernetes objects were created successfully.
- Ensure the database is reachable and properly configured.
- If you have any issues, please open an issue on [GitHub Issues](https://github.com/shivamkumar2002/linkwarden-helm-chart/issues).


## Contributing 🤝

Contributions are welcome! Please follow these steps:

1.  Fork the repository: [https://github.com/shivamkumar2002/linkwarden-helm-chart](https://github.com/shivamkumar2002/linkwarden-helm-chart)
2.  Create a new branch.
3.  Implement your changes.
4.  Submit a pull request.


## License 📜

This chart is licensed under the MIT License.


## Credits 🙏

- [Linkwarden](https://github.com/linkwarden/linkwarden)
- [CloudNativePG Operator](https://cloudnative-pg.io/)
- [Helm](https://helm.sh/)
