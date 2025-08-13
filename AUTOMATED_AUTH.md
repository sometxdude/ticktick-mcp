# TickTick MCP - Requests-Only Authentication Guide

This guide explains how to set up and use lightweight requests-only authentication for the TickTick MCP server, eliminating the need for Selenium or complex browser automation.

## Overview

The TickTick MCP server now supports two authentication methods:

1. **Manual Browser Authentication** (original) - Opens a browser window for manual OAuth flow
2. **Requests-Only Authentication** (new) - Uses only the `requests` library with a simple local server

## Prerequisites for Requests-Only Authentication

### No Additional Dependencies Required!

The requests-only authentication uses only Python standard library modules plus `requests`, which is already included in your requirements. No Selenium, no Chrome installation, no ChromeDriver needed!

## Setup Steps

### 1. Register Your TickTick Application

1. Go to [TickTick Developer Portal](https://developer.ticktick.com/manage)
2. Create a new application
3. Note your **Client ID** and **Client Secret**
4. Set the redirect URI to: `http://localhost:8002`

### 2. Configure Environment Variables

Create or update your `.env` file:

```bash
# Required for all authentication methods
TICKTICK_CLIENT_ID=your_client_id_here
TICKTICK_CLIENT_SECRET=your_client_secret_here
```

**That's it!** No usernames or passwords need to be stored. The requests-only method handles the OAuth flow automatically.

## Usage

### Method 1: Interactive Setup

Run the authentication command and choose your preferred method:

```bash
uv run -m ticktick_mcp.cli auth
```

You'll be presented with options:
- Option 1: Manual browser authentication (original method)  
- Option 2: Requests-only authentication (lightweight, recommended)

### Method 2: Direct Requests-Only Authentication

Use command-line flags for requests-only authentication:

```bash
# Requests-only auth (opens browser automatically)
uv run -m ticktick_mcp.cli auth --requests-only

# Requests-only auth without opening browser automatically
uv run -m ticktick_mcp.cli auth --requests-only --no-browser
```

### Method 3: Check Existing Credentials

Check if you already have valid authentication tokens:

```bash
uv run -m ticktick_mcp.cli auth --check-only
```

## Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `TICKTICK_CLIENT_ID` | Yes | Your TickTick application client ID |
| `TICKTICK_CLIENT_SECRET` | Yes | Your TickTick application client secret |
| `TICKTICK_USERNAME` | No | Your TickTick email (for automated auth convenience) |
| `TICKTICK_ACCESS_TOKEN` | Auto | Generated access token (created automatically) |
| `TICKTICK_REFRESH_TOKEN` | Auto | Generated refresh token (created automatically) |

## How Requests-Only Authentication Works

The requests-only authentication process:

1. **Start Local Server**: Starts a lightweight HTTP server on localhost to handle OAuth callbacks
2. **Generate OAuth URL**: Creates the TickTick authorization URL with proper parameters
3. **Open Browser**: Opens your default browser to the OAuth URL (or displays URL for manual opening)
4. **User Authorization**: You log in to TickTick and approve the application (one-time manual step)
5. **Capture Callback**: The local server captures the authorization code from TickTick's redirect
6. **Exchange for Token**: Uses `requests` to trade the authorization code for access and refresh tokens
7. **Save Tokens**: Stores tokens securely in your `.env` file
8. **Cleanup**: Shuts down the local server

## Security Considerations

### OAuth Security
- No passwords are ever stored or handled by the application
- Uses standard OAuth 2.0 flow with secure token exchange
- Only authorization codes and tokens are handled programmatically

### Token Storage
- Access tokens are stored in your `.env` file
- Refresh tokens allow automatic token renewal
- Tokens are validated before use

### Local Server Security
- Temporary local server only runs during authentication
- Server automatically shuts down after receiving the callback
- No persistent connections or data storage

## Troubleshooting

### Common Issues

#### Port Already in Use
```bash
# If port 8002 is in use, the system will show an error
# You can change the port in the .env file:
TICKTICK_REDIRECT_URI=http://localhost:8003
```

#### Browser Won't Open
- Try using `--no-browser` flag and copy the URL manually
- Check your default browser settings
- Ensure you have a browser installed

#### Authentication Failed
- Verify your Client ID and Client Secret are correct  
- Check that your redirect URI is set to `http://localhost:8002`
- Ensure you have an internet connection
- Try manual authentication as a fallback

#### Local Server Issues
- Check if your firewall allows localhost connections
- Make sure no other application is using port 8002
- Try running with debug logging for more information

### Debug Mode

For troubleshooting, you can enable debug logging:

```bash
# Run with debug logging
uv run -m ticktick_mcp.cli run --debug

# Or test the requests-only auth module directly
PYTHONPATH=. python -m ticktick_mcp.src.requests_auth
```

## Fallback Strategy

If requests-only authentication fails, the system automatically falls back to manual browser authentication, ensuring you can always complete the setup process.

## Example: Complete Setup

```bash
# 1. Check if already authenticated
uv run -m ticktick_mcp.cli auth --check-only

# 2. If not authenticated, run requests-only setup
uv run -m ticktick_mcp.cli auth --requests-only

# 3. Verify authentication worked
uv run -m ticktick_mcp.cli auth --check-only

# 4. Start the MCP server
uv run -m ticktick_mcp.cli run
```

## Benefits of Requests-Only Authentication

- **Lightweight**: No Selenium or browser drivers required
- **Fast Setup**: OAuth flow completes in ~30 seconds
- **Reliable**: Uses only HTTP requests and standard library
- **CI/CD Friendly**: Can be automated in deployment pipelines
- **Secure**: Standard OAuth 2.0 flow with no credential storage

## Comparison: Manual vs Requests-Only

| Feature | Manual Auth | Requests-Only Auth |
|---------|-------------|-------------------|
| Dependencies | Basic | Basic (requests only) |
| Browser Required | Yes (visible) | Yes (one-time OAuth) |
| User Interaction | Full manual process | One-time authorization |
| Setup Time | ~2-3 minutes | ~30 seconds |
| CI/CD Suitable | No | Yes |
| Credentials Storage | None | None (tokens only) |
| Fallback Available | No | Yes (to manual) |

Choose requests-only authentication for faster, more reliable setup with minimal dependencies!
