# Cloud Support Troubleshooting Lab

Five support tickets worked end to end on a live AWS EC2 instance — each one opened in Spiceworks in the customer's words, diagnosed from the command line, fixed, verified, and closed.

Covers Linux (SSH, systemd, Nginx, inodes), AWS (IAM policies, VPC routing, security groups vs. NACLs) and structured root cause analysis, with screenshots of every error and every fix.

**Start with [Ticket 5](aws/05-route-table-drop/RCA.md):** an instance that's "running and healthy" but unreachable, solved by ruling out each network layer in order.

**Environment:** Ubuntu 26.04 LTS on AWS EC2 (us-west-1) · Spiceworks Cloud Help Desk · sensitive values redacted

## How each ticket was worked

1. Ticket opened in Spiceworks in the customer's words, before any diagnosis
2. Hypotheses listed and ruled in or out with commands, cheapest check first
3. Fixed, then verified by repeating the customer's exact failing action
4. Internal note (root cause / fix / verified) logged, ticket closed, RCA published

## Tickets

| # | Title | Category | Ticket | Status |
|---|-------|----------|--------|--------|
| 1 | [SSH Connection Timed Out vs. Permission Denied](linux/01-ssh-timeout-vs-permission-denied/RCA.md) | Linux | Spiceworks #3 | Closed |
| 2 | [Nginx: Service Fails to Start After Config Change](linux/02-nginx-broken-config/RCA.md) | Linux | Spiceworks #4 | Closed |
| 3 | [Disk "Full" While Disk Usage Shows Free Space](linux/03-inode-exhaustion/RCA.md) | Linux | Spiceworks #5 | Closed |
| 4 | [S3 Upload Fails With AccessDenied While Listing Still Works](aws/04-iam-s3-accessdenied/RCA.md) | AWS | Spiceworks #6 | Closed |
| 5 | [Instance Unreachable While Running and Healthy](aws/05-route-table-drop/RCA.md) | AWS | Spiceworks #7 | Closed |
