# Runbook: Troubleshooting an Unresponsive Linux Service

**Objective:** Identify why a Linux service is unresponsive, determine the root cause, and restore functionality while minimizing disruption.

**Example:** Troubleshooting `nginx.service` on a systemd-based Linux server.

## 1. Check the service status

```bash
sudo systemctl status nginx
sudo systemctl is-active nginx
```

Check whether the service is `active`, `inactive`, or `failed`. Look for exit codes, recent errors, failed startup attempts, or repeated restarts.

## 2. Inspect service logs

```bash
sudo journalctl -u nginx -n 100 --no-pager
sudo journalctl -u nginx --since "30 minutes ago"
```

Look for configuration errors, permission issues, missing dependencies, resource exhaustion, or application crashes.

For additional context, check application-specific logs, such as `/var/log/nginx/error.log`.

## 3. Verify the listening port

```bash
sudo ss -tulnp | grep ':80'
```

Confirm that the expected port is listening and associated with the correct process.

If the port is listening but the service remains unresponsive, test locally:

```bash
curl -I --max-time 5 http://localhost:80
```

If the local test succeeds but remote connections fail, investigate firewall rules, network connectivity, and upstream load balancers.

## 4. Determine whether a restart is appropriate

**Restart when:**
- The service has crashed or entered a failed state.
- A known transient issue has been identified.
- Configuration changes require a restart and have been validated.
- The restart is approved and its impact is understood.

**Avoid restarting when:**
- The root cause is unknown and restarting could destroy useful diagnostic evidence.
- The service is production-critical and downtime hasn't been approved.
- The underlying issue, such as disk exhaustion or invalid configuration, remains unresolved.
- Restarting could interrupt active transactions or workloads.

For Nginx, validate configuration before restarting:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

## 5. Verify recovery

```bash
sudo systemctl status nginx
sudo ss -tulnp | grep ':80'
curl -I --max-time 5 http://localhost:80
sudo journalctl -u nginx -n 20 --no-pager
```

Confirm the service is running, accepting connections, and no longer generating errors. Test from a client as well if the original problem involved remote access.

## 6. Document the incident

Record the symptoms, relevant log entries, root cause (if identified), corrective actions, and verification results. If the issue is recurring, recommend preventive measures such as monitoring, alerting, or configuration changes.

**Key principle:** Investigate first, restart only when justified, and always verify that the original problem has been resolved.
