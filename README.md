# NetBox

[NetBox](https://netbox.dev) is an IP address management (IPAM) and
data center infrastructure management (DCIM) tool.

## TL;DR

```shell
helm install netbox oci://ghcr.io/netbox-community/netbox-chart/netbox
```

> [!tip]
> Please see [Production Usage](#production-usage) below before using this chart for real.

## Prerequisites

- Kubernetes [1.25+](https://kubernetes.io/releases/)
- Helm [3.10+](https://helm.sh/docs/topics/version_skew/)

## Installing the Chart

To install the chart with the release name `my-release` and default configuration:

```shell
helm install netbox ./ --namespace netbox --create-namespace
```

## Superuser

Once the deployment is successful, create a superuser to test out the deployment. Shell into the main pod and run the following.

```shell
python3 manage.py createsuperuser
```

### Production Usage

We recommend using separate external PostgreSQL and Redis instances. This
de-couples those services from the chart's bundled versions which may have
complex upgrade requirements. A clustered PostgreSQL server (e.g. using Zalando's
[Postgres Operator](https://github.com/zalando/postgres-operator)) and Redis
with Sentinel (e.g. using [Aaron Layfield](https://github.com/DandyDeveloper)'s
[redis-ha chart](https://github.com/DandyDeveloper/charts/tree/master/charts/redis-ha)).

Set `persistence.enabled` to `false` and use the S3 `storageBackend` and `storageConfig`
for object storage. This works well with Minio or Ceph RGW as well as Amazon S3.
See [Persistent storage pitfalls](#persistent-storage-pitfalls), below.

Run multiple replicas of the NetBox web frontend to avoid interruptions during
upgrades or at other times when the pods need to be restarted. There's no need
to have multiple workers (`worker.replicaCount`) for better availability. Set
up `affinity.podAntiAffinity` to avoid multiple NetBox pods being colocated on
the same node, for example:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app.kubernetes.io/instance: netbox
            app.kubernetes.io/name: netbox
            app.kubernetes.io/component: netbox
        topologyKey: kubernetes.io/hostname
```

## Configuration

The following table lists the configurable parameters for this chart and their default values.

<!-- prettier-ignore-start -->

| Parameter                                       | Description                                                         | Default                                      |
| ------------------------------------------------|---------------------------------------------------------------------|----------------------------------------------|
| `global.imageRegistry`                          | Netbox and subchart image registry for pulling container images     | `""`                                         |
| `global.imagePullSecrets`                       | Netbox and subchart registry secret names as an array               | `[]`                                         |
| `global.storageClass`                           | Netbox and subchart default StorageClass for Persistent Volume(s)   | `""`                                         |
| `replicaCount`                                  | The desired number of NetBox pods                                   | `1`                                          |
| `image.registry`                                | NetBox container image registry                                     | `ghcr.io`                                    |
| `image.repository`                              | NetBox container image repository                                   | `netboxcommunity/netbox`                     |
| `image.tag`                                     | NetBox container image tag                                          | `""`                                         |
| `image.pullPolicy`                              | NetBox container image pull policy                                  | `IfNotPresent`                               |
| `superuser.name`                                | Initial super-user account to create                                | `admin`                                      |
| `superuser.email`                               | Email address for the initial super-user account                    | `admin@example.com`                          |
| `superuser.password`                            | Password for the initial super-user account                         | `admin`                                      |
| `superuser.apiToken`                            | API token created for the initial super-user account                | `0123456789abcdef0123456789abcdef01234567`   |
| `superuser.existingSecret`                      | Use an existing Kubernetes `Secret` for secret values               | `""`                                         |
| `allowedHosts`                                  | List of valid FQDNs for this NetBox instance                        | `["*"]`                                      |
| `admins`                                        | List of admins to email about critical errors                       | `[]`                                         |
| `allowTokenRetrieval`                           | Permit the retrieval of API tokens after their creation             | `false`                                      |
| `authPasswordValidators`                        | Configure validation of local user account passwords                | `[]`                                         |
| `allowedUrlSchemes`                             | URL schemes that are allowed within links in NetBox                 | *see `values.yaml`*                          |
| `banner.top`                                    | Banner text to display at the top of every page                     | `""`                                         |
| `banner.bottom`                                 | Banner text to display at the bottom of every page                  | `""`                                         |
| `banner.login`                                  | Banner text to display on the login page                            | `""`                                         |
| `basePath`                                      | Base URL path if accessing NetBox within a directory                | `""`                                         |
| `changelogRetention`                            | Maximum number of days to retain logged changes (0 = forever)       | `90`                                         |
| `customValidators`                              | Custom validators for NetBox field values                           | `{}`                                         |
| `defaultUserPreferences`                        | Default preferences for newly created user accounts                 | `{}`                                         |
| `cors.originAllowAll`                           | [CORS]: allow all origins                                           | `false`                                      |
| `cors.originWhitelist`                          | [CORS]: list of origins authorised to make cross-site HTTP requests | `[]`                                         |
| `cors.originRegexWhitelist`                     | [CORS]: list of regular expression matching authorised origins      | `[]`                                         |
| `csrf.cookieName`                               | Name of the CSRF authentication cookie                              | `csrftoken`                                  |
| `csrf.trustedOrigins`                           | A list of trusted origins for unsafe (e.g. POST) requests           | `[]`                                         |
| `dataUploadMaxMemorySize`                       | The maximum size (in bytes) of an incoming HTTP request             | `2621440`                                    |
| `debug`                                         | Enable NetBox debugging (NOT for production use)                    | `false`                                      |
| `defaultLanguage`                               | Set the default preferred language/locale                           | `en-us`                                      |
| `dbWaitDebug`                                   | Show details of errors that occur when applying migrations          | `false`                                      |
| `email.server`                                  | SMTP server to use to send emails                                   | `localhost`                                  |
| `email.port`                                    | TCP port to connect to the SMTP server on                           | `25`                                         |
| `email.username`                                | Optional username for SMTP authentication                           | `""`                                         |
| `email.password`                                | Password for SMTP authentication (see also `existingSecret`)        | `""`                                         |
| `email.useSSL`                                  | Use SSL when connecting to the server                               | `false`                                      |
| `email.useTLS`                                  | Use TLS when connecting to the server                               | `false`                                      |
| `email.sslCertFile`                             | SMTP SSL certificate file path (e.g. in a mounted volume)           | `""`                                         |
| `email.sslKeyFile`                              | SMTP SSL key file path (e.g. in a mounted volume)                   | `""`                                         |
| `email.timeout`                                 | Timeout for SMTP connections, in seconds                            | `10`                                         |
| `email.from`                                    | Sender address for emails sent by NetBox                            | `""`                                         |
| `enforceGlobalUnique`                           | Enforce unique IP space in the global table (not in a VRF)          | `true`                                      |
| `exemptViewPermissions`                         | A list of models to exempt from the enforcement of view permissions | `[]`                                         |
| `fieldChoices`                                  | Configure custom choices for certain built-in fields                | `{}`                                         |
| `fileUploadMaxMemorySize`                       | The maximum amount (in bytes) of uploaded data that will be held in memory before being written to the filesystem  | `2621440` |
| `graphQlEnabled`                                | Enable the GraphQL API                                              | `true`                                       |
| `httpProxies`                                   | HTTP proxies NetBox should use when sending outbound HTTP requests  | `null`                                       |
| `internalIPs`                                   | IP addresses recognized as internal to the system                   | `['127.0.0.1', '::1']`                       |
| `jobRetention`                                  | The number of days to retain job results (scripts and reports)      | `90`                                         |
| `logging`                                       | Custom Django logging configuration                                 | `{}`                                         |
| `loginPersistence`                              | Enables users to remain authenticated to NetBox indefinitely        | `false`                                      |
| `loginRequired`                                 | Permit only logged-in users to access NetBox                        | `false` (unauthenticated read-only access)   |
| `loginTimeout`                                  | How often to re-authenticate users                                  | `1209600` (14 days)                          |
| `logoutRedirectUrl`                             | View name or URL to which users are redirected after logging out    | `home`                                       |
| `maintenanceMode`                               | Display a "maintenance mode" banner on every page                   | `false`                                      |
| `mapsUrl`                                       | The URL to use when mapping physical addresses or GPS coordinates   | `https://maps.google.com/?q=`                |
| `maxPageSize`                                   | Maximum number of objects that can be returned by a single API call | `1000`                                       |
| `storageBackend`                                | Django-storages backend class name                                  | `null`                                       |
| `storageConfig`                                 | Django-storages backend configuration                               | `{}`                                         |
| `paginateCount`                                 | The default number of objects to display per page in the web UI     | `50`                                         |
| `plugins`                                       | Additional plugins to load into NetBox                              | `[]`                                         |
| `pluginsConfig`                                 | Configuration for the additional plugins                            | `{}`                                         |
| `powerFeedDefaultAmperage`                      | Default amperage value for new power feeds                          | `15`                                         |
| `powerFeedMaxUtilisation`                       | Default maximum utilisation percentage for new power feeds          | `80`                                         |
| `powerFeedDefaultVoltage`                       | Default voltage value for new power feeds                           | `120`                                        |
| `preferIPv4`                                    | Prefer devices' IPv4 address when determining their primary address | `false`                                      |
| `rackElevationDefaultUnitHeight`                | Rack elevation default height in pixels                             | `22`                                         |
| `rackElevationDefaultUnitWidth`                 | Rack elevation default width in pixels                              | `220`                                        |
| `remoteAuth.enabled`                            | Enable remote authentication support                                | `false`                                      |
| `remoteAuth.backends`                           | Remote authentication backend classes                               | `[netbox.authentication.RemoteUserBackend]`  |
| `remoteAuth.header`                             | The name of the HTTP header which conveys the username              | `HTTP_REMOTE_USER`                           |
| `remoteAuth.userFirstName`                      | HTTP header which contains the user's first name                    | `HTTP_REMOTE_USER_FIRST_NAME`                |
| `remoteAuth.userLastName`                       | HTTP header which contains the user's last name                     | `HTTP_REMOTE_USER_LAST_NAME`                 |
| `remoteAuth.userEmail`                          | HTTP header which contains the user's email address                 | `HTTP_REMOTE_USER_EMAIL`                     |
| `remoteAuth.autoCreateUser`                     | Enables the automatic creation of new users                         | `false`                                      |
| `remoteAuth.autoCreateGroups`                   | Enables the automatic creation of new groups                        | `false`                                      |
| `remoteAuth.defaultGroups`                      | A list of groups to assign to newly created users                   | `[]`                                         |
| `remoteAuth.defaultPermissions`                 | A list of permissions to assign newly created users                 | `{}`                                         |
| `remoteAuth.groupSyncEnabled`                   | Sync remote user groups from an HTTP header set by a reverse proxy  | `false`                                      |
| `remoteAuth.groupHeader`                        | The name of the HTTP header which conveys the groups to which the user belongs | `HTTP_REMOTE_USER_GROUP`          |
| `remoteAuth.superuserGroups`                    | The list of groups that promote an remote User to Superuser on login| `[]`                                         |
| `remoteAuth.superusers`                         | The list of users that get promoted to Superuser on login           | `[]`                                         |
| `remoteAuth.staffGroups`                        | The list of groups that promote an remote User to Staff on login    | `[]`                                         |
| `remoteAuth.staffUsers`                         | The list of users that get promoted to Staff on login               | `[]`                                         |
| `remoteAuth.groupSeparator`                     | The Seperator upon which `remoteAuth.groupHeader` gets split into individual groups | `\|`                        |
| `remoteAuth.ldap.serverUri`                     | see [django-auth-ldap](https://django-auth-ldap.readthedocs.io)     | `""`                                         |
| `remoteAuth.ldap.startTls`                      | if StarTLS should be used                                           | *see values.yaml*                            |
| `remoteAuth.ldap.ignoreCertErrors`              | if Certificate errors should be ignored                             | *see values.yaml*                            |
| `remoteAuth.ldap.caCertDir`                     | CA certificate directory                                            | *see auth.md*                                |
| `remoteAuth.ldap.caCertData`                    | CA certificate data                                                 | *see auth.md*                                |
| `remoteAuth.ldap.bindDn`                        | Distinguished Name to bind with                                     | `""`                                         |
| `remoteAuth.ldap.bindPassword`                  | Password for bind DN                                                | `""`                                         |
| `remoteAuth.ldap.userDnTemplate`                | see [AUTH_LDAP_USER_DN_TEMPLATE](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-user-dn-template) | *see values.yaml* |
| `remoteAuth.ldap.userSearchBaseDn`              | see base_dn of [django_auth_ldap.config.LDAPSearch](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#django_auth_ldap.config.LDAPSearch) | *see values.yaml* |
| `remoteAuth.ldap.userSearchAttr`                | User attribute name for user search                                 | `sAMAccountName`                             |
| `remoteAuth.ldap.groupSearchBaseDn`             | base DN for group search                                            | *see values.yaml*                            |
| `remoteAuth.ldap.groupSearchClass`              | [django-auth-ldap](https://django-auth-ldap.readthedocs.io) for group search | `group`                             |
| `remoteAuth.ldap.groupType`                     | see [AUTH_LDAP_GROUP_TYPE](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-group-type) | `GroupOfNamesType` |
| `remoteAuth.ldap.requireGroupDn`                | DN of a group that is required for login                            | `null`                                       |
| `remoteAuth.ldap.findGroupPerms`                | see [AUTH_LDAP_FIND_GROUP_PERMS](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-find-group-perms) | true |
| `remoteAuth.ldap.mirrorGroups`                  | see [AUTH_LDAP_MIRROR_GROUPS](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-mirror-groups) | `null` |
| `remoteAuth.ldap.cacheTimeout`                  | see [AUTH_LDAP_MIRROR_GROUPS_EXCEPT](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-mirror-groups-except) | `null` |
| `remoteAuth.ldap.isAdminDn`                     | required DN to be able to login in Admin-Backend, "is_staff"-Attribute of [AUTH_LDAP_USER_FLAGS_BY_GROUP](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-user-flags-by-group) | *see values.yaml* |
| `remoteAuth.ldap.isSuperUserDn`                 | required DN to receive SuperUser privileges, "is_superuser"-Attribute of [AUTH_LDAP_USER_FLAGS_BY_GROUP](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-user-flags-by-group) | *see values.yaml* |
| `remoteAuth.ldap.attrFirstName`                 | first name attribute of users, "first_name"-Attribute of [AUTH_LDAP_USER_ATTR_MAP](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-user-attr-map) | `givenName` |
| `remoteAuth.ldap.attrLastName`                  | last name attribute of users, "last_name"-Attribute of [AUTH_LDAP_USER_ATTR_MAP](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-user-attr-map) | `sn` |
| `remoteAuth.ldap.attrMail`                      | mail attribute of users, "email_name"-Attribute of [AUTH_LDAP_USER_ATTR_MAP](https://django-auth-ldap.readthedocs.io/en/latest/reference.html#auth-ldap-user-attr-map) | `mail` |
| `releaseCheck.url`                              | Release check URL (GitHub API URL; see `values.yaml`)               | `null` (disabled by default)                 |
| `rqDefaultTimeout`                              | Maximum execution time for background tasks, in seconds             | `300` (5 minutes)                            |
| `sessionCookieName`                             | The name to use for the session cookie                              | `"sessionid"`                                |
| `enableLocalization`                            | Localization                                                        | `false`                                      |
| `timeZone`                                      | The time zone NetBox will use when dealing with dates and times     | `UTC`                                        |
| `dateFormat`                                    | Django date format for long-form date strings                       | `"N j, Y"`                                   |
| `shortDateFormat`                               | Django date format for short-form date strings                      | `"Y-m-d"`                                    |
| `timeFormat`                                    | Django date format for long-form time strings                       | `"g:i a"`                                    |
| `metrics.enabled`                               | Expose Prometheus metrics at the `/metrics` HTTP endpoint           | `false`                                      |
| `metrics.serviceMonitor.enabled`                | Whether to enable a [ServiceMonitor](https://prometheus-operator.dev/docs/operator/design/#servicemonitor) for Netbox | `false`                                      |
| `metrics.serviceMonitor.additionalLabels`       | Additonal labels to apply to the ServiceMonitor                     | `{}`                                         |
| `metrics.serviceMonitor.honorLabels`            | honorLabels chooses the metric's labels on collisions               | `false`                                      |
| `metrics.serviceMonitor.interval`               | Interval at which metrics should be scraped                         | `""`                                         |
| `metrics.serviceMonitor.scrapeTimeout`          | Timeout duration for scraping metrics                               | `""`                                         |
| `metrics.serviceMonitor.metricRelabelings`      | Specify additional relabeling of metrics                            | `[]`                                         |
| `metrics.serviceMonitor.relabelings`            | Specify general relabeling                                          | `[]`                                         |
| `metrics.serviceMonitor.selector`               | Prometheus instance selector labels                                 | `{}`                                         |
| `shortTimeFormat`                               | Django date format for short-form time strings                      | `"H:i:s"`                                    |
| `dateTimeFormat`                                | Django date format for long-form date and time strings              | `"N j, Y g:i a"`                             |
| `shortDateTimeFormat`                           | Django date format for short-form date and time strongs             | `"Y-m-d H:i"`                                |
| `extraConfig`                                   | Additional NetBox configuration (see `values.yaml`)                 | `[]`                                         |
| `secretKey`                                     | Django secret key used for sessions and password reset tokens       | `""` (generated)                             |
| `existingSecret`                                | Use an existing Kubernetes `Secret` for secret values (see below)   | `""` (use individual chart values)           |
| `overrideUnitConfig`                            | Override the NGINX Unit application server configuration            | `{}` (*see values.yaml*)                     |
| `postgresql.enabled`                            | Deploy PostgreSQL using bundled Bitnami PostgreSQL chart            | `true`                                       |
| `postgresql.auth.username`                      | Username to create for NetBox in bundled PostgreSQL instance        | `netbox`                                     |
| `postgresql.auth.database`                      | Database to create for NetBox in bundled PostgreSQL instance        | `netbox`                                     |
| `postgresql.*`                                  | Values under this key are passed to the bundled PostgreSQL chart    | n/a                                          |
| `externalDatabase.host`                         | PostgreSQL host to use when `postgresql.enabled` is `false`         | `localhost`                                  |
| `externalDatabase.port`                         | Port number for external PostgreSQL                                 | `5432`                                       |
| `externalDatabase.database`                     | Database name for external PostgreSQL                               | `netbox`                                     |
| `externalDatabase.username`                     | Username for external PostgreSQL                                    | `netbox`                                     |
| `externalDatabase.password`                     | Password for external PostgreSQL (see also `existingSecret`)        | `""`                                         |
| `externalDatabase.existingSecretName`           | Fetch password for external PostgreSQL from a different `Secret`    | `""`                                         |
| `externalDatabase.existingSecretKey`            | Key to fetch the password in the above `Secret`                     | `postgresql-password`                        |
| `externalDatabase.connMaxAge`                   | The lifetime of a database connection, as an integer of seconds     | `300`                                        |
| `externalDatabase.disableServerSideCursors`     | Disable the use of server-side cursors transaction pooling          | `false`                                      |
| `externalDatabase.options`                      | Additional PostgreSQL client parameters             | `{}`                                            |
| `redis.enabled`                                 | Deploy Redis using bundled Bitnami Redis chart                      | `true`                                       |
| `redis.*`                                       | Values under this key are passed to the bundled Redis chart         | n/a                                          |
| `tasksRedis.database`                           | Redis database number used for NetBox task queue                    | `0`                                          |
| `tasksRedis.ssl`                                | Enable SSL when connecting to Redis                                 | `false`                                      |
| `tasksRedis.insecureSkipTlsVerify`              | Skip TLS certificate verification when connecting to Redis          | `false`                                      |
| `tasksRedis.caCertPath`                         | Path to CA certificates bundle for Redis (needs mounting manually)  | `""`                                         |
| `tasksRedis.host`                               | Redis host to use when `redis.enabled` is `false`                   | `"netbox-redis"`                             |
| `tasksRedis.port`                               | Port number for external Redis                                      | `6379`                                       |
| `tasksRedis.sentinels`                          | List of sentinels in `host:port` form (`host` and `port` not used)  | `[]`                                         |
| `tasksRedis.sentinelService`                    | Sentinel master service name                                        | `"netbox-redis"`                             |
| `tasksRedis.sentinelTimeout`                    | Sentinel connection timeout, in seconds                             | `300` (5 minutes)                            |
| `tasksRedis.username`                           | Username for external Redis                                         | `""`                                         |
| `tasksRedis.password`                           | Password for external Redis (see also `existingSecret`)             | `""`                                         |
| `tasksRedis.existingSecretName`                 | Fetch password for external Redis from a different `Secret`         | `""`                                         |
| `tasksRedis.existingSecretKey`                  | Key to fetch the password in the above `Secret`                     | `redis-password`                             |
| `cachingRedis.database`                         | Redis database number used for caching views                        | `1`                                          |
| `cachingRedis.ssl`                              | Enable SSL when connecting to Redis                                 | `false`                                      |
| `cachingRedis.insecureSkipTlsVerify`            | Skip TLS certificate verification when connecting to Redis          | `false`                                      |
| `cachingRedis.caCertPath`                       | Path to CA certificates bundle for Redis (needs mounting manually)  | `""`                                         |
| `cachingRedis.host`                             | Redis host to use when `redis.enabled` is `false`                   | `"netbox-redis"`                             |
| `cachingRedis.port`                             | Port number for external Redis                                      | `6379`                                       |
| `cachingRedis.sentinels`                        | List of sentinels in `host:port` form (`host` and `port` not used)  | `[]`                                         |
| `cachingRedis.sentinelService`                  | Sentinel master service name                                        | `"netbox-redis"`                             |
| `cachingRedis.sentinelTimeout`                  | Sentinel connection timeout, in seconds                             | `300` (5 minutes)                            |
| `cachingRedis.username`                         | Username for external Redis                                         | `""`                                         |
| `cachingRedis.password`                         | Password for external Redis (see also `existingSecret`)             | `""`                                         |
| `cachingRedis.existingSecretName`               | Fetch password for external Redis from a different `Secret`         | `""`                                         |
| `cachingRedis.existingSecretKey`                | Key to fetch the password in the above `Secret`                     | `redis-password`                             |
| `imagePullSecrets`                              | List of `Secret` names containing private registry credentials      | `[]`                                         |
| `nameOverride`                                  | Override the application name (`netbox`) used throughout the chart  | `""`                                         |
| `fullnameOverride`                              | Override the full name of resources created as part of the release  | `""`                                         |
| `serviceAccount.create`                         | Create a ServiceAccount for NetBox                                  | `true`                                       |
| `serviceAccount.annotations`                    | Annotations to add to the service account                           | `{}`                                         |
| `serviceAccount.name`                           | The name of the service account to use                              | `""` (use the fullname)                      |
| `serviceAccount.imagePullSecrets`               | Add an imagePullSecrets attribute to the serviceAccount             | `""`                                         |
| `serviceAccount.automountServiceAccountToken`   | Whether to automatically mount the token in the containers using this serviceAccount or not | `false`              |
| `persistence.enabled`                           | Enable storage persistence for uploaded media (images)              | `true`                                       |
| `persistence.existingClaim`                     | Use an existing `PersistentVolumeClaim` instead of creating one     | `""`                                         |
| `persistence.subPath`                           | Mount a sub-path of the volume into the container, not the root     | `""`                                         |
| `persistence.storageClass`                      | Set the storage class of the PVC (use `-` to disable provisioning)  | `""`                                         |
| `persistence.selector`                          | Set the selector for PVs, if desired                                | `{}`                                         |
| `persistence.accessMode`                        | Access mode for the volume                                          | `ReadWriteOnce`                              |
| `persistence.size`                              | Size of persistent volume to request                                | `1Gi`                                        |
| `reportsPersistence.enabled`                    | Enable storage persistence for NetBox reports                       | `false`                                      |
| `reportsPersistence.existingClaim`              | Use an existing `PersistentVolumeClaim` instead of creating one     | `""`                                         |
| `reportsPersistence.subPath`                    | Mount a sub-path of the volume into the container, not the root     | `""`                                         |
| `reportsPersistence.storageClass`               | Set the storage class of the PVC (use `-` to disable provisioning)  | `""`                                         |
| `reportsPersistence.selector`                   | Set the selector for PVs, if desired                                | `{}`                                         |
| `reportsPersistence.accessMode`                 | Access mode for the volume                                          | `ReadWriteOnce`                              |
| `reportsPersistence.size`                       | Size of persistent volume to request                                | `1Gi`                                        |
| `scriptsPersistence.enabled`                    | Enable storage persistence for NetBox reports                       | `false`                                      |
| `scriptsPersistence.existingClaim`              | Use an existing `PersistentVolumeClaim` instead of creating one     | `""`                                         |
| `scriptsPersistence.subPath`                    | Mount a sub-path of the volume into the container, not the root     | `""`                                         |
| `scriptsPersistence.storageClass`               | Set the storage class of the PVC (use `-` to disable provisioning)  | `""`                                         |
| `scriptsPersistence.selector`                   | Set the selector for PVs, if desired                                | `{}`                                         |
| `scriptsPersistence.accessMode`                 | Access mode for the volume                                          | `ReadWriteOnce`                              |
| `scriptsPersistence.size`                       | Size of persistent volume to request                                | `1Gi`                                        |
| `podAnnotations`                                | Additional annotations for NetBox pods                              | `{}`                                         |
| `podLabels`                                     | Additional labels for NetBox pods                                   | `{}`                                         |
| `podSecurityContext`                            | Security context for NetBox pods                                    | *see `values.yaml`*                          |
| `securityContext`                               | Security context for NetBox containers                              | *see `values.yaml`*                          |
| `service.type`                                  | Type of `Service` resource to create                                | `ClusterIP`                                  |
| `service.port`                                  | Port number for the service                                         | `80`                                         |
| `service.nodePort`                              | The port used on the node when `service.type` is NodePort           | `""`                                         |
| `service.clusterIP`                             | The cluster IP address assigned to the service                      | `""`                                         |
| `service.clusterIPs`                            | A list of cluster IP addresses assigned to the service              | `[]`                                         |
| `service.externalIPs`                           | A list of external IP addresses aliased to this service             | `[]`                                         |
| `service.externalTrafficPolicy`                 | Policy for routing external traffic                                 | `Cluster`                                    |
| `service.ipFamilyPolicy`                        | Represents the dual-stack-ness of the service                       | `""`                                         |
| `service.loadBalancerIP`                        | Request a specific IP address when `service.type` is `LoadBalancer` | `""`                                         |
| `service.loadBalancerSourceRanges`              | A list of allowed IP ranges when `service.type` is `LoadBalancer`   | `[]`                                         |
| `service.loadBalancerClass`                     | Load Balancer class if `service.type` is `LoadBalancer`             | `""`                                         |
| `service.sessionAffinity`                       | Control where client requests go, to the same pod or round-robin    | `None`                                       |
| `service.sessionAffinityConfig`                 | Additional settings for the sessionAffinity                         | `{}`                                         |
| `ingress.enabled`                               | Create an `Ingress` resource for accessing NetBox                   | `false`                                      |
| `ingress.className`                             | Use a named IngressClass                                            | `""`                                         |
| `ingress.annotations`                           | Extra annotations to apply to the `Ingress` resource                | `{}`                                         |
| `ingress.hosts`                                 | List of hosts and paths to map to the service (see `values.yaml`)   | `[{host:"chart-example.local",paths:["/"]}]` |
| `ingress.tls`                                   | TLS settings for the `Ingress` resource                             | `[]`                                         |
| `resources`                                     | Configure resource requests or limits for NetBox                    | `{}`                                         |
| `automountServiceAccountToken`                  | Whether to automatically mount the serviceAccount token in the main container or not | `false`                     |
| `priorityClassName`                             | Pods' priorityClassName                                             | `""`                                         |
| `schedulerName`                                 | Name of the k8s scheduler (other than default) for pods             | `""`                                         |
| `terminationGracePeriodSeconds`                 | Seconds pods need to terminate gracefully                           | `""`                                         |
| `topologySpreadConstraints`                     | Configure Pod Topology Spread Constraints for NetBox                | `[]`                                         |
| `livenessProbe.enabled`                         | Enable Kubernetes livenessProbe, see [liveness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-command) | *see `values.yaml`* |
| `livenessProbe.initialDelaySeconds`             | Number of seconds                                                   |  *see `values.yaml`*                         |
| `livenessProbe.timeoutSeconds`                  | Number of seconds                                                   |  *see `values.yaml`*                         |
| `livenessProbe.periodSeconds`                   | Number of seconds                                                   |  *see `values.yaml`*                         |
| `livenessProbe.successThreshold`                | Number of seconds                                                   |  *see `values.yaml`*                         |
| `readinessProbe.enabled`                        | Enable Kubernetes readinessProbe, see [readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes) | *see `values.yaml`* |
| `readinessProbe.initialDelaySeconds`            | Number of seconds                                                   |  *see `values.yaml`*                         |
| `readinessProbe.timeoutSeconds`                 | Number of seconds                                                   |  *see `values.yaml`*                         |
| `readinessProbe.periodSeconds`                  | Number of seconds                                                   |  *see `values.yaml`*                         |
| `readinessProbe.successThreshold`               | Number of seconds                                                   |  *see `values.yaml`*                         |
| `lifecycleHooks`                                | Automate configuration before or after container startup            | `{}`                                         |
| `startupProbe.enabled`                          | Enable Kubernetes startupProbe, see [startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes) | *see `values.yaml`* |
| `startupProbe.initialDelaySeconds`              | Number of seconds                                                   |  *see `values.yaml`*                         |
| `startupProbe.timeoutSeconds`                   | Number of seconds                                                   |  *see `values.yaml`*                         |
| `startupProbe.periodSeconds`                    | Number of seconds                                                   |  *see `values.yaml`*                         |
| `startupProbe.successThreshold`                 | Number of seconds                                                   |  *see `values.yaml`*                         |
| `init.image.registry`                           | Init container image registry                                       | `ghcr.io`                                    |
| `init.image.repository`                         | Init container image repository                                     | `busybox`                                    |
| `init.image.tag`                                | Init container image tag                                            | `1.37.0`                                     |
| `init.image.pullPolicy`                         | Init container image pull policy                                    | `IfNotPresent`                               |
| `init.resourcesPreset`                          | Configure a preset for requests or limits for init container        | `nano`                                       |
| `init.resources`                                | Configure resource requests or limits for init container            | `{}`                                         |
| `init.securityContext`                          | Security context for init container                                 | *see `values.yaml`*                          |
| `test.image.registry`                           | Test container image registry                                       | `ghcr.io`                                    |
| `test.image.repository`                         | Test container image repository                                     | `busybox`                                    |
| `test.image.tag`                                | Test container image tag                                            | `1.37.0`                                     |
| `test.image.pullPolicy`                         | Test container image pull policy                                    | `IfNotPresent`                               |
| `test.resourcesPreset`                          | Configure a preset for requests or limits for test container        | `nano`                                       |
| `test.resources`                                | Configure resource requests or limits for test container            | `{}`                                         |
| `test.securityContext`                          | Security context for test container                                 | *see `values.yaml`*                          |
| `autoscaling.enabled`                           | Whether to enable the HorizontalPodAutoscaler                       | `false`                                      |
| `autoscaling.minReplicas`                       | Minimum number of replicas when autoscaling is enabled              | `1`                                          |
| `autoscaling.maxReplicas`                       | Maximum number of replicas when autoscaling is enabled              | `100`                                        |
| `autoscaling.targetCPUUtilizationPercentage`    | Target CPU utilisation percentage for autoscaling                   | `80`                                         |
| `autoscaling.targetMemoryUtilizationPercentage` | Target memory utilisation percentage for autoscaling                | `null`                                       |
| `nodeSelector`                                  | Node labels for pod assignment                                      | `{}`                                         |
| `tolerations`                                   | Toleration labels for pod assignment                                | `[]`                                         |
| `updateStrategy`                                | Configure deployment update strategy                                | `{}` (defaults to `RollingUpdate`)           |
| `affinity`                                      | Affinity settings for pod assignment                                | `{}`                                         |
| `extraEnvs`                                     | Additional environment variables to set in the NetBox container     | `[]`                                         |
| `extraVolumeMounts`                             | Additional volumes to mount in the NetBox container                 | `[]`                                         |
| `extraVolumes`                                  | Additional volumes to reference in pods                             | `[]`                                         |
| `sidecars`                                      | Additional sidecar containers to be added to pods                   | `[]`                                         |
| `initContainers`                                | Additional init containers to run before starting main containers   | `[]`                                         |
| `command`                                       | NetBox container custom command/entrypoint                          | `[]`                                         |
| `args`                                          | NetBox container custom args                                        | `[]`                                         |
| `worker`                                        | Worker specific variables. Most global variables also apply here.   | *see `values.yaml`*                          |
| `housekeeping.enabled`                          | Whether the [Housekeeping][housekeeping] `CronJob` should be active | `true`                                       |
| `housekeeping.concurrencyPolicy`                | ConcurrencyPolicy for the Housekeeping CronJob.                     | `Forbid`                                     |
| `housekeeping.failedJobsHistoryLimit`           | Number of failed jobs to keep in history                            | `5`                                          |
| `housekeeping.command`                          | The shell command to execute in the housekeeping job.               | `[/opt/netbox/venv/bin/python, /opt/netbox/netbox/manage.py, housekeeping]`|
| `housekeeping.args`                             | NetBox housekeeping container custom args                           | `[]`                                         |
| `housekeeping.restartPolicy`                    | Restart Policy for the Housekeeping CronJob.                        | `OnFailure`                                  |
| `housekeeping.schedule`                         | Schedule for the CronJob in [Cron syntax][cron syntax].             | `0 0 * * *` (Midnight daily)                 |
| `housekeeping.successfulJobsHistoryLimit`       | Number of successful jobs to keep in history                        | `5`                                          |
| `housekeeping.suspend`                          | Whether to suspend the CronJob                                      | `false`                                      |
| `housekeeping.podAnnotations`                   | Additional annotations for housekeeping CronJob pods                | `{}`                                         |
| `housekeeping.podLabels`                        | Additional labels for housekeeping CronJob pods                     | `{}`                                         |
| `housekeeping.podSecurityContext`               | Security context for housekeeping CronJob pods                      | *see `values.yaml`*                          |
| `housekeeping.securityContext`                  | Security context for housekeeping CronJob containers                | *see `values.yaml`*                          |
| `housekeeping.automountServiceAccountToken`     | Whether to automatically mount the serviceAccount token in the housekeeping container or not | `false`             |
| `housekeeping.resources`                        | Configure resource requests or limits for housekeeping CronJob      | `{}`                                         |
| `housekeeping.nodeSelector`                     | Node labels for housekeeping CronJob pod assignment                 | `{}`                                         |
| `housekeeping.tolerations`                      | Toleration labels for housekeeping CronJob pod assignment           | `[]`                                         |
| `housekeeping.affinity`                         | Affinity settings for housekeeping CronJob pod assignment           | `{}`                                         |
| `housekeeping.extraEnvs`                        | Additional environment variables to set in housekeeping CronJob     | `[]`                                         |
| `housekeeping.extraVolumeMounts`                | Additional volumes to mount in the housekeeping CronJob             | `[]`                                         |
| `housekeeping.extraVolumes`                     | Additional volumes to reference in housekeeping CronJob pods        | `[]`                                         |
| `housekeeping.sidecars`                         | Additional sidecar containers to be added to housekeeping CronJob   | `[]`                                         |
| `housekeeping.initContainers`                   | Additional init containers for housekeeping CronJob pods            | `[]`                                         |

<!-- prettier-ignore-end -->

[CORS]: https://github.com/ottoyiu/django-cors-headers
[housekeeping]: https://netboxlabs.com/docs/netbox/en/stable/administration/housekeeping/
[cron syntax]: https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-schedule-syntax

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install` or provide a YAML file containing the values for the above parameters:

```shell
helm install my-release --devel --values values.yaml \
  oci://ghcr.io/netbox-community/netbox-chart/netbox
```

## Persistent storage pitfalls

Persistent storage for media is enabled by default, but unless you take special
care you will run into issues. The most common issue is that one of the NetBox
pods gets stuck in the `ContainerCreating` state. There are several ways around
this problem:

<!-- prettier-ignore-start -->

1. Use the recommended S3 `storageBackend` and **disable** persistent storage
    by setting `persistence.enabled` to `false`. This can
    be used with any S3-compatible storage provider including Amazon S3, Minio,
    Ceph RGW, and many others. See further down for an example of this.
2. Use a `ReadWriteMany` volume that can be mounted by several pods across
    nodes simultaneously.
3. Configure pod affinity settings to keep all the pods on the same node. This
    allows a `ReadWriteOnce` volume to be mounted in several pods at the same
    time.
4. Disable persistent storage of media altogether and just manage without. The
    storage functionality is only needed to store uploaded image attachments.

<!-- prettier-ignore-end -->

To configure the pod affinity to allow using a `ReadWriteOnce` volume you can
use the following example configuration:

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app.kubernetes.io/name: netbox
        topologyKey: kubernetes.io/hostname

housekeeping:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app.kubernetes.io/name: netbox
          topologyKey: kubernetes.io/hostname

worker:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app.kubernetes.io/name: netbox
          topologyKey: kubernetes.io/hostname
```

## Using an Existing Secret

Rather than specifying passwords and secrets as part of the Helm release values,
you may pass these to NetBox using pre-existing `Secret` resources. When using
this, the respective `Secret`s must contain the following keys.

### Superuser secret (`superuser.existingSecret`)

Type: `kubernetes.io/basic-auth`

| Key         | Description                                          | Required? |
| ----------- | ---------------------------------------------------- | --------- |
| `username`  | Username for the initial super-user account          | Yes       |
| `password`  | Password for the initial super-user account          | Yes       |
| `email`     | Email address for the initial super-user account     | Yes       |
| `api_token` | API token created for the initial super-user account | Yes       |

### Config secret (`existingSecret`)

| Key                  | Description                                                   | Required?                                                                                         |
| -------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `ldap_bind_password` | Password for LDAP bind DN                                     | If `remoteAuth.enabled` is `true` and `remoteAuth.backend` is `netbox.authentication.LDAPBackend` |
| `secret_key`         | Django secret key used for sessions and password reset tokens | Yes                                                                                               |

### Email secret (`email.existingSecretName`)

| Key                  | Description                                                   | Required?                                                                                         |
| -------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `email-password`     | SMTP user password                                            | Yes, but the value may be left blank if not required                                              |

### PostgreSQL secret (`externalDatabase.existingSecretName`)

| Key                   | Description                                       | Required?                          |
| --------------------- | ------------------------------------------------- | ---------------------------------- |
| `postgresql-password` | The password for the external PostgreSQL database | If `postgresql.enabled` is `false` |

### Redis secrets (`tasksRedis.existingSecretName` & `cachingRedis.existingSecretName`)

| Key              | Description                                                   | Required?                     |
| ---------------- | ------------------------------------------------------------- | ----------------------------- |
| `redis-password` | Password for the external Redis database (tasks and/or cache) | If `redis.enabled` is `false` |

## Authentication

- [Single Sign On](/docs/auth.md#configuring-sso)
- [LDAP Authentication](/docs/auth.md#using-ldap-authentication)

## License

This project is licensed under [Apache License, Version 2.0](LICENSE).
