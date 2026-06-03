# 🔍 Complete Detection Queries Library

## Query 1: Failed SSH Attempts by Source IP
```spl
index=main "Failed password" 
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" 
| stats count by src_ip 
| sort - count

index=main "Failed password" 
| bucket _time span=5m
| stats count by src_ip, _time
| where count > 10

index=main "Failed password" 
| bucket _time span=1m
| stats count by src_ip, _time
| where count > 20

index=main "Failed password for invalid user" 
| rex "invalid user (?<username>\w+)" 
| stats count by username, src_ip
| where count > 3

index=main ("Failed password" OR "Accepted password")
| transaction src_ip maxspan=5m
| where like(events, "%Failed%") AND like(events, "%Accepted%")

index=main "Failed password" 
| timechart count by src_ip limit=10
