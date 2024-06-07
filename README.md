# Helm Chart for DokuWiki Backup

This Helm chart sets up a simple backup strategy for a [Bitnami DokuWiki](https://github.com/bitnami/charts/tree/main/bitnami/dokuwiki/#installing-the-chart) deployment. DokuWiki does not use a database, and instead stores information using a file-based storage system. This chart sets up a backup strategy that integrates closely with the [BackupTool for Dokuwiki Plugin](https://www.dokuwiki.org/plugin:backup) to create and store backups in a similar manner. The chart includes a backup container, scheduled cron jobs for daily, weekly, and monthly backups, and configurations for persistent storage and service accounts.

After setting up the chart, you can view the latest backups in the DokuWiki admin panel under the "Backup" tab. Additional backups are stored in the specified backup directory on the backup container.

## Configuration

### Backup Container

- **backupcontainer.name**: Name of the backup container. Default is `backup-box`.
- **backupcontainer.image**: Docker image to use for the backup container. Default is `busybox:latest`.
- **backupcontainer.command**: Command to run within the container. Default is `["sleep", "infinity"]`.
- **backupcontainer.resources**: Resource requests and limits for the container.

```yaml
backupcontainer:
  name: backup-box
  image: busybox:latest
  command: ["sleep", "infinity"]
  resources:
    requests:
      memory: "64Mi"
      cpu: "50m"
    limits:
      memory: "128Mi"
      cpu: "100m"
```

### Shared Configuration for Cron Jobs

- **sharedCronjobConfig.image**: Docker image to use for the cron job. Default is `bitnami/dokuwiki`.
- **sharedCronjobConfig.command**: Command to run in the cron job container. Default is `["/bin/bash", "-c"]`.
- **sharedCronjobConfig.destinationBackupDirectory**: Base directory for storing backups. Default is `/destination/backups`.
- **sharedCronjobConfig.backupName**: Base name for the backup files. Default is `dw-backup`.
- **sharedCronjobConfig.backupDirectory**: Directory containing the files to be backed up. Default is `/bitnami/dokuwiki/data/media/wiki/backup`.

```yaml
sharedCronjobConfig:
  image: bitnami/dokuwiki
  command: ["/bin/bash", "-c"]
  destinationBackupDirectory: /destination/backups
  backupName: dw-backup
  backupDirectory: /bitnami/dokuwiki/data/media/wiki/backup
```

### Test Job Configuration

- **testJob.enabled**: Enable the test backup job. Default is `true`.
- **testJob.retentionTime**: Time (in seconds) to retain the test job. Default is `86400` (one day).

```yaml
testJob:
  enabled: true
  retentionTime: 86400
```

### Cron Jobs for Scheduling Backups

Add named configurations to generate additional cron jobs.

- **cronjobs.\*.enabled**: Enable the backup cron job.
- **cronjobs.\*.schedule**: Schedule to run the backup cron job.
- **cronjobs.\*.successfulJobsHistoryLimit**: Number of successful jobs to retain.
- **cronjobs.\*.failedJobsHistoryLimit**: Number of failed jobs to retain.
- **cronjobs.\*.daysToRetain**: Number of days to retain backups created under this cronjob.

```yaml
cronjobs:
  - name: daily
    enabled: true
    schedule: "0 0 * * *"
    successfulJobsHistoryLimit: 3
    failedJobsHistoryLimit: 3
    daysToRetain: 7
  - name: weekly
    enabled: true
    schedule: "0 4 * * 1"
    successfulJobsHistoryLimit: 1
    failedJobsHistoryLimit: 1
    daysToRetain: 30
  - name: monthly
    enabled: true
    schedule: "0 6 1 * *"
    successfulJobsHistoryLimit: 1
    failedJobsHistoryLimit: 1
    daysToRetain: 365
```

### Persistent Volume Claim (PVC) Configuration

- **pvc.existingClaim**: Whether an existing claim is being used. Default is `false`.
- **pvc.name**: Name of the backup container PVC. Default is `backup-pvc`.
- **pvc.storageClass**: Storage class for the PVC. Should be a file backup storage class. Default is `netapp-file-backup`.
- **pvc.accessModes**: Access mode for the PVC. Default is `ReadWriteMany`.
- **pvc.storage**: Requested storage size for the backup container PVC. Default is `200Mi`.

```yaml
pvc:
  existingClaim: false
  name: backup-pvc
  storageClass: netapp-file-backup
  storage: 200Mi
```

### Service Account Configuration

- **serviceAccount.existingServiceAccount**: Whether an existing service account is being used. Default is `false`.
- **serviceAccount.exists**: Whether the service account exists. Default is `false`.
- **serviceAccount.name**: Name of the service account. Default is `backup-sa`.

```yaml
serviceAccount:
  existingServiceAccount: false
  name: backup-sa
```

### Volume Configuration

- **volumes.dokuwikiData.claimName**: Name of the persistent volume claim for the DokuWiki instance. Default is `dokuwiki-pvc-name`.

```yaml
volumes:
  dokuwikiData:
    claimName: dokuwiki-pvc-name
```

## Usage

To deploy the Helm chart with the default values, run:

```sh
helm install my-dokuwiki-backup ./dokuwiki-backup
```

To customize the deployment, create a `values.yaml` file with your desired configurations and run:

```sh
helm install my-dokuwiki-backup ./dokuwiki-backup -f values.yaml
```

This will set up the backup container and the cron jobs according to your specified configuration, ensuring that your DokuWiki data is backed up regularly.
