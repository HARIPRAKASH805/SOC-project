# ⚡ Alert Configuration

## SSH Brute Force Alert

### Settings
- **Name**: SSH_Brute_Force_Detection
- **Search**: `index=main "Failed password" | stats count by src_ip | where count > 10`
- **Time Range**: Last 5 minutes
- **Run**: Every 1 minute
- **Trigger**: When count > 0

### Response Actions
1. Log to `threat_intel` index
2. Send email alert
3. Trigger automated block (optional)
