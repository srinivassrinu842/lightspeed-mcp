# Red Hat Lightspeed MCP Remediation Tools Enablement Guide

## Overview

The `mcp_lightspeed-mc_remediations__create_vuln_playbook` tool is a write operation that is **disabled by default** in the Red Hat Lightspeed MCP server. This guide provides clear steps for administrators to enable and configure this tool.

---

## Key Facts

- **Default Mode**: The MCP server runs in **read-only mode** by default
- **Tool Status**: Remediation playbook creation tools are **write operations** and are disabled by default
- **Enablement Method**: Use the `--all-tools` startup flag to enable write tools
- **RBAC Control**: Permissions can be further restricted using Red Hat's Role-Based Access Control (RBAC) system
- **Service Account Role Required**: `Remediations user`

---

## Step 1: Enable Write Tools on MCP Server Startup

### Option A: Command-Line Flag (Direct Startup)

Add the `--all-tools` flag when starting the MCP server:

```bash
# For STDIO transport
podman run \
  --env LIGHTSPEED_CLIENT_ID=<YOUR_CLIENT_ID> \
  --env LIGHTSPEED_CLIENT_SECRET=<YOUR_CLIENT_SECRET> \
  --interactive \
  --rm \
  quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest \
  --all-tools
```

### Option B: HTTP/SSE Transport with Write Tools

```bash
# For HTTP transport
podman run \
  --env LIGHTSPEED_CLIENT_ID=<YOUR_CLIENT_ID> \
  --env LIGHTSPEED_CLIENT_SECRET=<YOUR_CLIENT_SECRET> \
  --net host \
  --rm \
  quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest \
  http --all-tools

# For SSE transport
podman run \
  --env LIGHTSPEED_CLIENT_ID=<YOUR_CLIENT_ID> \
  --env LIGHTSPEED_CLIENT_SECRET=<YOUR_CLIENT_SECRET> \
  --net host \
  --rm \
  quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest \
  sse --all-tools
```

### Option C: VS Code Configuration

Update `.vscode/mcp.json` to include the `--all-tools` flag:

```json
{
    "inputs": [
        {
            "id": "lightspeed_client_id",
            "type": "promptString",
            "description": "Enter the Red Hat Lightspeed Client ID",
            "password": true
        },
        {
            "id": "lightspeed_client_secret",
            "type": "promptString",
            "description": "Enter the Red Hat Lightspeed Client Secret",
            "password": true
        }
    ],
    "servers": {
        "lightspeed-mcp": {
            "type": "stdio",
            "command": "podman",
            "args": [
                "run",
                "--env",
                "LIGHTSPEED_CLIENT_ID",
                "--env",
                "LIGHTSPEED_CLIENT_SECRET",
                "--interactive",
                "--rm",
                "quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest",
                "--all-tools"
            ],
            "env": {
                "LIGHTSPEED_CLIENT_ID": "${input:lightspeed_client_id}",
                "LIGHTSPEED_CLIENT_SECRET": "${input:lightspeed_client_secret}"
            }
        }
    }
}
```

### Option D: Cursor Configuration

Update `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "lightspeed-mcp": {
        "type": "stdio",
        "command": "podman",
        "args": [
            "run",
            "--env",
            "LIGHTSPEED_CLIENT_ID",
            "--env",
            "LIGHTSPEED_CLIENT_SECRET",
            "--interactive",
            "--rm",
            "quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest",
            "--all-tools"
        ],
        "env": {
            "LIGHTSPEED_CLIENT_ID": "<YOUR_CLIENT_ID>",
            "LIGHTSPEED_CLIENT_SECRET": "<YOUR_CLIENT_SECRET>"
        }
    }
  }
}
```

---

## Step 2: Configure Service Account with Required RBAC Role

### 2.1 Create or Identify Service Account

1. Navigate to **https://console.redhat.com**
2. Click **Settings (⚙️ Gear Icon)** → **"Service Accounts"**
3. Create a new service account or use an existing one
4. Record the **Client ID** and **Client Secret**

### 2.2 Grant "Remediations user" Role

The `mcp_lightspeed-mc_remediations__create_vuln_playbook` tool requires the **`Remediations user`** role.

**Steps to Assign the Role:**

1. **Log in as Organization Administrator** with the **User Access administrator** role
2. Navigate to **Settings (⚙️ Gear Icon)** → **"User Access"** → **"Groups"**
3. Choose one of the following options:

#### Option A: Create New Group for MCP Service Accounts

```
1. Click "Create group"
2. Enter group name (e.g., "mcp-remediations-accounts")
3. In the "Add roles" section, select:
   - Remediations user
4. Go to the "Service accounts" tab
5. Add your MCP service account to the group
```

#### Option B: Use Existing Group

```
1. Find an existing group that has the "Remediations user" role
2. Go to the "Service accounts" tab
3. Add your MCP service account to the group
```

**Result**: Your service account will inherit the `Remediations user` role and gain access to create remediation playbooks.

### 2.3 Verify Permissions Assignment

1. Go to **Settings** → **"Service Accounts"**
2. Click on your service account
3. Verify that it shows the **"Remediations user"** role in its assigned permissions

---

## Step 3: Set Environment Variables

Configure the MCP server with your service account credentials:

```bash
export LIGHTSPEED_CLIENT_ID="<your_client_id>"
export LIGHTSPEED_CLIENT_SECRET="<your_client_secret>"
```

These environment variables are used by the MCP server to authenticate with Red Hat Lightspeed services.

---

## Step 4: Verify Tool Availability

### Test 1: Check Tool List

Run a query to list available tools:

```bash
# Using the MCP CLI (if available)
# This should show "create_vuln_playbook" in the remediations toolset
```

### Test 2: Create a Test Remediation Playbook

Once configured, you should be able to create a vulnerability remediation playbook:

```
Request:
{
  "tool": "mcp_lightspeed-mc_remediations__create_vuln_playbook",
  "params": {
    "playbook_name": "Test Remediation",
    "cves": ["CVE-2024-XXXX"],
    "uuids": ["<system_uuid>"]
  }
}
```

---

## Troubleshooting

### Issue 1: Tool Not Available / "Tool is disabled" Error

**Symptoms**: 
- Tool appears disabled or unavailable
- Error message: "This tool is disabled by default"

**Resolution**:
1. ✅ Verify `--all-tools` flag is included in startup command
2. ✅ Restart the MCP server with the flag
3. ✅ Check Docker/Podman logs for startup messages

### Issue 2: Permission Denied / "Access Denied"

**Symptoms**:
- Error: "User does not have permission"
- Tool executes but fails with RBAC error

**Resolution**:
1. ✅ Verify service account has **"Remediations user"** role assigned
2. ✅ Confirm the service account is in the correct group
3. ✅ Wait a few minutes for RBAC cache to refresh
4. ✅ Try resetting the service account credentials

### Issue 3: Authentication Fails

**Symptoms**:
- Error: "Invalid credentials" or "Unauthorized"
- Connection fails to console.redhat.com

**Resolution**:
1. ✅ Verify `LIGHTSPEED_CLIENT_ID` and `LIGHTSPEED_CLIENT_SECRET` are correct
2. ✅ Ensure credentials haven't expired or been revoked
3. ✅ Check network connectivity to console.redhat.com
4. ✅ Regenerate credentials if needed

### Issue 4: Tool Still Not Appearing After Restart

**Symptoms**:
- Tool list doesn't show remediation tools despite configuration

**Resolution**:
1. ✅ Verify MCP server version is up to date: `podman pull quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest`
2. ✅ Check that `--all-tools` flag appears AFTER the container image name in the command
3. ✅ Verify no conflicting read-only environment variables are set
4. ✅ Check MCP server logs for startup errors: `podman logs <container_id>`

---

## Configuration File References

### Key Configuration Files

| File | Purpose | Location |
|------|---------|----------|
| `.vscode/mcp.json` | VS Code MCP configuration | Workspace root |
| `~/.cursor/mcp.json` | Cursor MCP configuration | User home directory |
| `~/.gemini/settings.json` | Gemini CLI configuration | User home directory |
| `~/.config/Claude/mcp.json` | Claude Desktop configuration | User config directory |

### Environment Variables

| Variable | Purpose | Required |
|----------|---------|----------|
| `LIGHTSPEED_CLIENT_ID` | Service account client ID | Yes |
| `LIGHTSPEED_CLIENT_SECRET` | Service account client secret | Yes |
| `LIGHTSPEED_BASE_URL` | Custom Red Hat Lightspeed URL | No (defaults to production) |
| `LIGHTSPEED_SSO_BASE_URL` | Custom SSO URL | No (defaults to production) |

---

## Security Considerations

### Important Security Notes

⚠️ **When Running Locally (Recommended)**:
- Ensure the container is NOT exposed to the internet
- It's acceptable to use client ID and secret in local Docker/Podman configuration
- Delete or reset credentials after use

⚠️ **When Running Remotely**:
- Be aware that credentials are transmitted to the remote MCP server
- Trust the remote server not to leak credentials
- Consider using JWT Bearer tokens instead of client credentials for remote deployments
- Use encrypted channels (HTTPS/TLS) for remote connections

### Emergency Credential Revocation

If you suspect credential compromise:

1. **Immediate**: Stop the MCP server container
   ```bash
   podman stop <container_id>
   ```

2. **Revoke Credentials**: Go to https://console.redhat.com → **Service Accounts**
   - Click the affected service account
   - Select **"Delete"** or **"Reset"** to invalidate credentials

3. **Regenerate**: Create new credentials for the service account

4. **Update Configuration**: Update all client configurations with new credentials

---

## Testing the Configuration

### Quick Verification Script

```bash
#!/bin/bash

# Test 1: Check if --all-tools is in the startup command
echo "✓ Verify --all-tools flag is present in startup command"

# Test 2: Check service account permissions
echo "✓ Verify service account has 'Remediations user' role"
echo "  Navigate to: console.redhat.com → Settings → User Access → Groups"

# Test 3: Check credentials
echo "✓ Verify LIGHTSPEED_CLIENT_ID is set: ${LIGHTSPEED_CLIENT_ID:?Not set}"
echo "✓ Verify LIGHTSPEED_CLIENT_SECRET is set: ${LIGHTSPEED_CLIENT_SECRET:?Not set}"

# Test 4: Test MCP server startup
echo "✓ Starting MCP server with --all-tools flag..."
podman run \
  --env LIGHTSPEED_CLIENT_ID \
  --env LIGHTSPEED_CLIENT_SECRET \
  --interactive \
  --rm \
  quay.io/redhat-services-prod/insights-management-tenant/insights-mcp/red-hat-lightspeed-mcp:latest \
  --all-tools

echo "If no errors appear, the remediation tools are now enabled!"
```

---

## Additional Resources

- **Official Documentation**: https://docs.redhat.com/en/documentation/red_hat_lightspeed/
- **Remediations Guide**: https://docs.redhat.com/en/documentation/red_hat_lightspeed/1-latest/html/red_hat_lightspeed_remediations_guide
- **GitHub Repository**: https://github.com/RedHatInsights/insights-mcp
- **Service Account Setup Video**: https://www.youtube.com/watch?v=UvNcmJsbg1w

---

## Summary Checklist

- [ ] Add `--all-tools` flag to MCP server startup command
- [ ] Create or identify service account in console.redhat.com
- [ ] Assign **"Remediations user"** role to service account
- [ ] Set `LIGHTSPEED_CLIENT_ID` environment variable
- [ ] Set `LIGHTSPEED_CLIENT_SECRET` environment variable
- [ ] Restart MCP server with new configuration
- [ ] Verify remediation tools appear in tool list
- [ ] Test creating a remediation playbook
- [ ] Document credentials securely
- [ ] Review security considerations for your deployment

---

**Last Updated**: May 28, 2026  
**Applies To**: Red Hat Lightspeed MCP v1.0 and later
