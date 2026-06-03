# 🔍 Complete Detection Queries Library

## Query 1: Failed SSH Attempts by Source IP
```spl
index=main "Failed password" 
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)" 
| stats count by src_ip 
| sort - count
