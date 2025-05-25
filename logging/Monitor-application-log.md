# Application Troubleshooting Guide

## Overview

This document provides a step-by-step troubleshooting guide for resolving user access and purchase issues in a Kubernetes-deployed web application. The guide demonstrates how to identify and diagnose problems using pod logs analysis.

## Problem Statement

### Issue 1: User Access Problem

**Original Question:** A user - USER5 - has expressed concerns accessing the application. Identify the cause of the issue.

### Issue 2: Purchase Problem

**Original Question:** A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue.

## Prerequisites

- Kubernetes cluster access with `kubectl` configured
- Appropriate permissions to view pod logs
- Basic understanding of Kubernetes pod management

## Troubleshooting Steps

### Step 1: Identify Available Pods

Before investigating logs, ensure you know which pods are running in your application.

```bash
kubectl get pods
```

### Step 2: Investigate User Access Issues (USER5)

#### Command Used

```bash
kubectl logs webapp-1
```

#### Console Output

```
[2025-05-25 18:28:52,904] INFO in event-simulator: USER4 logged out
[2025-05-25 18:28:53,905] INFO in event-simulator: USER4 logged out
[2025-05-25 18:28:54,906] INFO in event-simulator: USER4 is viewing page1
[2025-05-25 18:28:55,908] INFO in event-simulator: USER2 is viewing page3
[2025-05-25 18:28:56,909] INFO in event-simulator: USER1 logged in
[2025-05-25 18:28:57,910] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:28:57,910] INFO in event-simulator: USER4 is viewing page1
[2025-05-25 18:28:58,911] INFO in event-simulator: USER2 logged out
[2025-05-25 18:28:59,912] INFO in event-simulator: USER4 logged in
[2025-05-25 18:29:00,913] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:29:00,913] INFO in event-simulator: USER4 logged out
[2025-05-25 18:29:01,914] INFO in event-simulator: USER2 is viewing page2
[2025-05-25 18:29:02,915] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:02,915] INFO in event-simulator: USER4 logged in
[2025-05-25 18:29:03,917] INFO in event-simulator: USER1 is viewing page1
[2025-05-25 18:29:04,918] INFO in event-simulator: USER1 is viewing page1
[2025-05-25 18:29:05,918] INFO in event-simulator: USER1 is viewing page3
[2025-05-25 18:29:06,919] INFO in event-simulator: USER2 logged in
[2025-05-25 18:29:07,920] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:07,920] INFO in event-simulator: USER4 is viewing page3
[2025-05-25 18:29:08,921] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:29:08,921] INFO in event-simulator: USER3 is viewing page2
[2025-05-25 18:29:09,922] INFO in event-simulator: USER2 logged in
[2025-05-25 18:29:10,923] INFO in event-simulator: USER2 logged out
[2025-05-25 18:29:11,924] INFO in event-simulator: USER4 is viewing page2
[2025-05-25 18:29:12,924] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:12,925] INFO in event-simulator: USER3 logged in
[2025-05-25 18:29:13,926] INFO in event-simulator: USER1 is viewing page1
[2025-05-25 18:29:14,927] INFO in event-simulator: USER2 is viewing page2
[2025-05-25 18:29:15,928] INFO in event-simulator: USER2 is viewing page1
[2025-05-25 18:29:16,930] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:29:16,930] INFO in event-simulator: USER3 is viewing page3
[2025-05-25 18:29:17,930] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:17,930] INFO in event-simulator: USER4 is viewing page3
[2025-05-25 18:29:18,931] INFO in event-simulator: USER3 logged in
[2025-05-25 18:29:19,932] INFO in event-simulator: USER3 is viewing page2
[2025-05-25 18:29:20,933] INFO in event-simulator: USER4 logged out
[2025-05-25 18:29:21,934] INFO in event-simulator: USER4 logged out
[2025-05-25 18:29:22,935] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:22,935] INFO in event-simulator: USER4 is viewing page3
[2025-05-25 18:29:23,936] INFO in event-simulator: USER1 logged out
[2025-05-25 18:29:24,937] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:29:24,937] INFO in event-simulator: USER3 is viewing page3
[2025-05-25 18:29:25,938] INFO in event-simulator: USER2 logged out
[2025-05-25 18:29:26,939] INFO in event-simulator: USER2 is viewing page2
[2025-05-25 18:29:27,940] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:27,940] INFO in event-simulator: USER3 is viewing page1
[2025-05-25 18:29:28,941] INFO in event-simulator: USER3 logged out
[2025-05-25 18:29:29,942] INFO in event-simulator: USER3 is viewing page3
[2025-05-25 18:29:30,943] INFO in event-simulator: USER3 is viewing page2
[2025-05-25 18:29:31,944] INFO in event-simulator: USER4 logged in
[2025-05-25 18:29:32,945] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:32,945] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:29:32,945] INFO in event-simulator: USER3 is viewing page3
[2025-05-25 18:29:33,946] INFO in event-simulator: USER2 is viewing page2
[2025-05-25 18:29:34,947] INFO in event-simulator: USER2 is viewing page3
[2025-05-25 18:29:35,948] INFO in event-simulator: USER4 is viewing page2
[2025-05-25 18:29:36,950] INFO in event-simulator: USER3 is viewing page1
[2025-05-25 18:29:37,950] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:37,950] INFO in event-simulator: USER2 is viewing page2
[2025-05-25 18:29:38,952] INFO in event-simulator: USER3 logged out
[2025-05-25 18:29:39,953] INFO in event-simulator: USER4 is viewing page2
[2025-05-25 18:29:40,954] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:29:40,955] INFO in event-simulator: USER2 logged in
[2025-05-25 18:29:41,956] INFO in event-simulator: USER4 is viewing page1
[2025-05-25 18:29:42,957] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:29:42,957] INFO in event-simulator: USER4 is viewing page3
[2025-05-25 18:29:43,959] INFO in event-simulator: USER1 is viewing page2
[2025-05-25 18:29:44,960] INFO in event-simulator: USER2 is viewing page2
[2025-05-25 18:29:45,961] INFO in event-simulator: USER4 logged in
```

#### Analysis

The logs from `webapp-1` clearly show that **USER5** is experiencing login failures due to account lockout:

- Multiple WARNING entries: `USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS`
- The failures occur consistently throughout the log timeline
- Pattern indicates repeated login attempts that are being blocked by the security system

### Step 3: Investigate Purchase Issues

#### Command Used

```bash
kubectl logs webapp-2 -c simple-webapp
```

#### Console Output

```
[2025-05-25 18:30:15,865] INFO in event-simulator: USER2 logged out
[2025-05-25 18:30:16,866] INFO in event-simulator: USER4 is viewing page2
[2025-05-25 18:30:17,868] INFO in event-simulator: USER4 is viewing page1
[2025-05-25 18:30:18,869] INFO in event-simulator: USER3 is viewing page1
[2025-05-25 18:30:19,870] INFO in event-simulator: USER1 is viewing page1
[2025-05-25 18:30:20,872] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:20,872] INFO in event-simulator: USER4 is viewing page3
[2025-05-25 18:30:21,873] INFO in event-simulator: USER3 logged out
[2025-05-25 18:30:22,874] INFO in event-simulator: USER4 logged in
[2025-05-25 18:30:23,874] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:30:23,875] INFO in event-simulator: USER4 logged out
[2025-05-25 18:30:24,876] INFO in event-simulator: USER4 logged out
[2025-05-25 18:30:25,877] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:25,877] INFO in event-simulator: USER2 is viewing page1
[2025-05-25 18:30:26,878] INFO in event-simulator: USER1 is viewing page2
[2025-05-25 18:30:27,879] INFO in event-simulator: USER3 is viewing page3
[2025-05-25 18:30:28,880] INFO in event-simulator: USER2 logged out
[2025-05-25 18:30:29,881] INFO in event-simulator: USER1 is viewing page3
[2025-05-25 18:30:30,882] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:30,882] INFO in event-simulator: USER1 logged out
[2025-05-25 18:30:31,883] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:30:31,884] INFO in event-simulator: USER2 is viewing page1
[2025-05-25 18:30:32,885] INFO in event-simulator: USER3 is viewing page2
[2025-05-25 18:30:33,885] INFO in event-simulator: USER1 is viewing page3
[2025-05-25 18:30:34,887] INFO in event-simulator: USER1 is viewing page3
[2025-05-25 18:30:35,888] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:35,888] INFO in event-simulator: USER3 is viewing page3
[2025-05-25 18:30:36,889] INFO in event-simulator: USER2 is viewing page3
[2025-05-25 18:30:37,890] INFO in event-simulator: USER3 is viewing page2
[2025-05-25 18:30:38,891] INFO in event-simulator: USER3 is viewing page2
[2025-05-25 18:30:39,892] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:30:39,892] INFO in event-simulator: USER4 is viewing page1
[2025-05-25 18:30:40,893] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:40,893] INFO in event-simulator: USER1 logged in
[2025-05-25 18:30:41,894] INFO in event-simulator: USER4 is viewing page2
[2025-05-25 18:30:42,895] INFO in event-simulator: USER2 is viewing page1
[2025-05-25 18:30:43,896] INFO in event-simulator: USER2 is viewing page3
[2025-05-25 18:30:44,897] INFO in event-simulator: USER2 is viewing page1
[2025-05-25 18:30:45,898] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:45,899] INFO in event-simulator: USER2 logged out
[2025-05-25 18:30:46,900] INFO in event-simulator: USER1 logged out
[2025-05-25 18:30:47,901] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:30:47,901] INFO in event-simulator: USER3 logged in
[2025-05-25 18:30:48,902] INFO in event-simulator: USER1 logged in
[2025-05-25 18:30:49,904] INFO in event-simulator: USER3 is viewing page1
[2025-05-25 18:30:50,905] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:50,905] INFO in event-simulator: USER2 is viewing page1
[2025-05-25 18:30:51,906] INFO in event-simulator: USER3 is viewing page1
[2025-05-25 18:30:52,907] INFO in event-simulator: USER3 logged in
[2025-05-25 18:30:53,908] INFO in event-simulator: USER1 is viewing page2
[2025-05-25 18:30:54,909] INFO in event-simulator: USER1 logged in
[2025-05-25 18:30:55,910] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2025-05-25 18:30:55,910] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2025-05-25 18:30:55,910] INFO in event-simulator: USER2 is viewing page2
```

#### Analysis

The logs from `webapp-2` reveal that **USER30** is experiencing purchase failures:

- Multiple WARNING entries: `USER30 Order failed as the item is OUT OF STOCK`
- The failures occur consistently throughout the log timeline
- Issue is inventory-related, not authentication-related

## Command Explanations

### `kubectl logs webapp-1`

- **Purpose**: Retrieves logs from the pod named `webapp-1`
- **Why it works**: Provides real-time application logs that contain user activity and error messages
- **What it reveals**: Authentication failures, user sessions, and system warnings

### `kubectl logs webapp-2 -c simple-webapp`

- **Purpose**: Retrieves logs from a specific container (`simple-webapp`) within the pod `webapp-2`
- **Why the `-c` flag**: When a pod contains multiple containers, you must specify which container's logs to view
- **What it reveals**: Similar user activity logs but from a different application instance

## Solution Summary

### Issue 1: USER5 Access Problem

**Root Cause**: Account lockout due to multiple failed login attempts
**Evidence**: Repeated WARNING messages showing "Failed to Login as the account is locked due to MANY FAILED ATTEMPTS"
**Solution Approach**:

- Unlock the user account through admin console
- Investigate potential security breach or password issues
- Consider implementing account lockout notifications

### Issue 2: Purchase Problem

**Affected User**: USER30
**Root Cause**: Inventory shortage - items are out of stock
**Evidence**: Repeated WARNING messages showing "Order failed as the item is OUT OF STOCK"
**Solution Approach**:

- Replenish inventory for the requested items
- Implement better inventory management and low-stock alerts
- Consider implementing waitlist functionality for out-of-stock items

## Why This Approach Works

1. **Direct Log Analysis**: Kubernetes logs provide real-time insights into application behavior and errors
2. **Pattern Recognition**: Repeated error messages help identify systemic issues rather than one-off problems
3. **Multi-Pod Investigation**: Checking logs across different pods ensures comprehensive troubleshooting
4. **Timestamp Correlation**: Log timestamps help understand the frequency and persistence of issues
5. **Error Categorization**: Different warning types (authentication vs. inventory) require different solution approaches

## Additional Troubleshooting Commands

```bash
# View logs with timestamps
kubectl logs webapp-1 --timestamps

# Follow logs in real-time
kubectl logs -f webapp-1

# View last N lines of logs
kubectl logs webapp-1 --tail=50

# View logs from all containers in a pod
kubectl logs webapp-2 --all-containers=true

# Export logs to file for analysis
kubectl logs webapp-1 > webapp-1-logs.txt
```

## Best Practices

1. **Regular Log Monitoring**: Implement log aggregation and monitoring solutions
2. **Alert Configuration**: Set up alerts for repeated error patterns
3. **Log Retention**: Ensure adequate log retention policies for historical analysis
4. **Security Monitoring**: Monitor for account lockout patterns that might indicate attacks
5. **Inventory Management**: Implement proactive inventory monitoring to prevent stock-out situations

## YAML Configuration Considerations

While no specific YAML configurations were provided in the original problem, typical configurations that could help prevent these issues include:

```yaml
# Example: Security Policy Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: security-config
data:
  max_login_attempts: "5"
  lockout_duration: "15m"
  enable_account_notifications: "true"
```

```yaml
# Example: Inventory Monitoring Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: inventory-config
data:
  low_stock_threshold: "10"
  enable_stock_alerts: "true"
  waitlist_enabled: "true"
```

This troubleshooting approach demonstrates the power of Kubernetes logging for identifying and resolving application issues efficiently.
