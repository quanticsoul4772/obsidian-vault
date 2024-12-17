# System Backup Scripts Guide

## Overview
This guide provides comprehensive backup solutions for system administrators managing HPC and data center environments. These scripts cover various backup scenarios, from simple file backups to complete system snapshots, with focus on automation and reliability.

### Key Features
- Automated backup scheduling
- Incremental and full backup support
- Backup verification and logging
- Remote backup capabilities
- Error handling and notifications
- Compression and encryption options

## Basic Backup Scripts

### File System Backup
```bash
#!/bin/bash
# filesystem_backup.sh
# Creates incremental backups of specified directories

# Configuration
BACKUP_ROOT="/backup"
SOURCE_DIRS=(
    "/etc"
    "/home"
    "/var/www"
)
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/var/log/backup/backup_${TIMESTAMP}.log"
RETENTION_DAYS=30

# Initialize
mkdir -p "$BACKUP_ROOT" "$(dirname "$LOG_FILE")"
exec 1> >(tee -a "$LOG_FILE") 2>&1

# Functions
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

create_backup() {
    local source_dir=$1
    local backup_name=$(basename "$source_dir")
    local backup_dir="${BACKUP_ROOT}/${backup_name}"
    local latest_link="${backup_dir}/latest"
    
    mkdir -p "$backup_dir"
    
    if [ -L "$latest_link" ]; then
        # Incremental backup
        rsync -av --delete \
            --link-dest="$latest_link" \
            "$source_dir/" \
            "${backup_dir}/${TIMESTAMP}/"
    else
        # Full backup
        rsync -av "$source_dir/" "${backup_dir}/${TIMESTAMP}/"
    fi
    
    # Update latest link
    ln -snf "${backup_dir}/${TIMESTAMP}" "$latest_link"
}

cleanup_old_backups() {
    local dir=$1
    find "$dir" -maxdepth 1 -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;
}

# Main execution
log_message "Starting backup process"

for dir in "${SOURCE_DIRS[@]}"; do
    log_message "Backing up $dir"
    create_backup "$dir"
    cleanup_old_backups "${BACKUP_ROOT}/$(basename "$dir")"
done

log_message "Backup process completed"
```

### Database Backup
```bash
#!/bin/bash
# database_backup.sh
# Backs up MySQL/PostgreSQL databases

# Configuration
BACKUP_DIR="/backup/databases"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/var/log/backup/db_backup_${TIMESTAMP}.log"
RETENTION_DAYS=14
EMAIL_RECIPIENT="admin@example.com"

# Database credentials (consider using .my.cnf or .pgpass for security)
DB_USER="backup_user"
DB_PASS="backup_password"
DB_HOST="localhost"

# Initialize
mkdir -p "$BACKUP_DIR" "$(dirname "$LOG_FILE")"
exec 1> >(tee -a "$LOG_FILE") 2>&1

# MySQL backup function
mysql_backup() {
    local db_name=$1
    local backup_file="${BACKUP_DIR}/mysql_${db_name}_${TIMESTAMP}.sql.gz"
    
    mysqldump --single-transaction \
        -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" \
        "$db_name" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        echo "MySQL backup successful: $backup_file"
    else
        echo "MySQL backup failed: $db_name" >&2
        return 1
    fi
}

# PostgreSQL backup function
postgres_backup() {
    local db_name=$1
    local backup_file="${BACKUP_DIR}/postgresql_${db_name}_${TIMESTAMP}.sql.gz"
    
    PGPASSWORD="$DB_PASS" pg_dump -h "$DB_HOST" -U "$DB_USER" \
        "$db_name" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        echo "PostgreSQL backup successful: $backup_file"
    else
        echo "PostgreSQL backup failed: $db_name" >&2
        return 1
    fi
}

# Cleanup function
cleanup_old_backups() {
    find "$BACKUP_DIR" -name "*.sql.gz" -type f -mtime +$RETENTION_DAYS -delete
}

# Main execution
echo "Starting database backup process at $(date)"

# MySQL databases
mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" -N -e \
    "SHOW DATABASES" | while read -r db; do
    case "$db" in
        "information_schema"|"performance_schema"|"mysql"|"sys")
            continue
            ;;
        *)
            mysql_backup "$db"
            ;;
    esac
done

# PostgreSQL databases
PGPASSWORD="$DB_PASS" psql -h "$DB_HOST" -U "$DB_USER" -l -t | \
    cut -d'|' -f1 | sed 's/ //g' | grep -v '^$' | while read -r db; do
    case "$db" in
        "template0"|"template1"|"postgres")
            continue
            ;;
        *)
            postgres_backup "$db"
            ;;
    esac
done

# Cleanup old backups
cleanup_old_backups

# Send completion email
{
    echo "Subject: Database Backup Report - $(date '+%Y-%m-%d')"
    echo "Content-Type: text/plain"
    echo
    cat "$LOG_FILE"
} | sendmail "$EMAIL_RECIPIENT"
```

## Advanced Backup Scripts

### System State Backup
```bash
#!/bin/bash
# system_state_backup.sh
# Creates a comprehensive system state backup

# Configuration
BACKUP_ROOT="/backup/system-state"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/var/log/backup/system_state_${TIMESTAMP}.log"

# State files to backup
STATE_FILES=(
    "/etc/fstab"
    "/etc/passwd"
    "/etc/group"
    "/etc/shadow"
    "/etc/hosts"
    "/etc/resolv.conf"
    "/etc/network"
    "/etc/systemd"
    "/var/spool/cron"
)

# Package management
backup_package_state() {
    local backup_dir="$1/packages"
    mkdir -p "$backup_dir"
    
    if command -v dpkg > /dev/null; then
        # Debian/Ubuntu
        dpkg --get-selections > "$backup_dir/package_selections"
        apt-mark showhold > "$backup_dir/package_holds"
        cp /etc/apt/sources.list* "$backup_dir/"
    elif command -v rpm > /dev/null; then
        # RHEL/Rocky
        rpm -qa > "$backup_dir/package_list"
        cp /etc/yum.repos.d/* "$backup_dir/"
    fi
}

# Service state
backup_service_state() {
    local backup_dir="$1/services"
    mkdir -p "$backup_dir"
    
    systemctl list-unit-files > "$backup_dir/unit_files"
    systemctl list-units --all > "$backup_dir/units_all"
}

# Main execution
main_backup_dir="${BACKUP_ROOT}/${TIMESTAMP}"
mkdir -p "$main_backup_dir"

# Backup system state files
for file in "${STATE_FILES[@]}"; do
    if [ -e "$file" ]; then
        parent_dir="$main_backup_dir$(dirname "$file")"
        mkdir -p "$parent_dir"
        cp -a "$file" "$parent_dir/"
    fi
done

# Backup package and service states
backup_package_state "$main_backup_dir"
backup_service_state "$main_backup_dir"

# Create tarball
tar czf "${main_backup_dir}.tar.gz" -C "$main_backup_dir" .
rm -rf "$main_backup_dir"
```

## Integration Examples

### Prometheus Integration
```yaml
# prometheus.yml
- job_name: 'backup_monitoring'
  static_configs:
    - targets: ['localhost:9100']
  metrics_path: '/metrics'
  metric_relabel_configs:
    - source_labels: [__name__]
      regex: 'node_backup_.*'
      action: keep
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "panels": [
      {
        "title": "Backup Status",
        "type": "gauge",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "backup_success_gauge",
            "legendFormat": "Last Backup Status"
          }
        ]
      }
    ]
  }
}
```

## Best Practices

### 1. Backup Strategy
- Implement 3-2-1 rule (3 copies, 2 different media, 1 offsite)
- Regular backup testing and verification
- Document recovery procedures
- Automate backup processes

### 2. Security Considerations
- Encrypt sensitive backups
- Secure backup storage locations
- Regular permission audits
- Access control implementation

### 3. Performance Optimization
- Schedule during off-peak hours
- Use compression appropriately
- Implement incremental backups
- Monitor backup performance

### 4. Monitoring and Maintenance
- Regular backup verification
- Automated status reporting
- Storage capacity monitoring
- Retention policy enforcement

## Troubleshooting Guide

### Common Issues

1. Backup Failures
   - Check disk space
   - Verify permissions
   - Monitor backup logs
   - Test backup paths

2. Performance Issues
   - Review compression settings
   - Check network bandwidth
   - Monitor system resources
   - Optimize scheduling

3. Storage Problems
   - Monitor capacity
   - Check mount points
   - Verify backup chains
   - Test storage health

## Related Documentation
- [[Tools - Performance Monitoring Scripts]]
- [[Tools - Essential Linux Commands]]
- [[Troubleshooting Decision Tree]]

## Tags
#backup #scripts #automation #system-admin #disaster-recovery