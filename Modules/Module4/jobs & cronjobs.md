# Jobs & CronJobs 

## Overview

Jobs and CronJobs in Kubernetes v1.35 provide powerful mechanisms for running batch workloads and scheduled tasks with enhanced reliability, 
monitoring, and management capabilities.

### Migration Complexity

**Job Migration**
```yaml
# BEFORE (v1.34): Basic Job
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing-v134
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: processor
        image: data/processor:v1.0
        # v1.34: Basic configuration
        command: ["python", "process_data.py"]
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        env:
        - name: INPUT_PATH
          value: "/data/input"
        - name: OUTPUT_PATH
          value: "/data/output"
```

```yaml
# AFTER (v1.35): Enhanced Job
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing-v135
  labels:
    app: data-processor
    version: v1.35
  annotations:
    job.kubernetes.io/version: "v1.35"
spec:
  completions: 10
  parallelism: 3
  backoffLimit: 3
  activeDeadlineSeconds: 3600
  # v1.35: Enhanced cleanup
  ttlSecondsAfterFinished: 86400
  # v1.35: Indexed completion mode
  completionMode: Indexed
  template:
    metadata:
      labels:
        app: data-processor
        batch-type: analytics
    spec:
      restartPolicy: OnFailure
      # v1.35: Enhanced security
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        fsGroup: 10001
      containers:
      - name: processor
        image: data/processor:v2.0-v135
        # v1.35: Enhanced processing logic
        command:
        - /bin/bash
        - -c
        - |
          echo "Starting data processing job ${JOB_COMPLETION_INDEX}"
          python process_data.py \
            --index=${JOB_COMPLETION_INDEX} \
            --total-jobs=${JOB_COMPLETIONS} \
            --input-path=${INPUT_PATH} \
            --output-path=${OUTPUT_PATH}
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
            ephemeral-storage: "5Gi"
          limits:
            memory: "2Gi"
            cpu: "1000m"
            ephemeral-storage: "10Gi"
        env:
        - name: JOB_COMPLETION_INDEX
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
        - name: JOB_COMPLETIONS
          value: "10"
        - name: INPUT_PATH
          value: "/data/input"
        - name: OUTPUT_PATH
          value: "/data/output"
        # v1.35: Enhanced health monitoring
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - pgrep -f process_data.py
          periodSeconds: 30
        # v1.35: Progress tracking
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - test -f /tmp/processing-started
          periodSeconds: 10
        volumeMounts:
        - name: data-volume
          mountPath: /data
        - name: temp-volume
          mountPath: /tmp
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: data-processing-pvc
      - name: temp-volume
        emptyDir:
          sizeLimit: 5Gi
```
