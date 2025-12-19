# Palo Alto Networks MCP Server

A tool that lets AI assistants (like Claude) manage your Palo Alto Networks firewall through natural conversation.

## What is this?

This is an **MCP Server** (Model Context Protocol Server) - a bridge that allows AI assistants to securely interact with your Palo Alto Networks firewall. Instead of manually logging into your firewall's admin panel, you can ask an AI assistant to:

- View your firewall's system information
- List security rules, addresses, and network configurations
- View specific configuration settings
- Move or copy configuration items

Think of it as giving your AI assistant controlled access to manage your firewall on your behalf.

## Is it safe?

**Yes.** This server was designed with security in mind:

| Security Feature | Description |
|-----------------|-------------|
| **No hardcoded passwords** | Your API key is stored securely in environment variables, never in the code |
| **No hidden connections** | The server only connects to YOUR firewall - no data is sent anywhere else |
| **Trusted dependencies** | Uses only well-known, widely-used libraries (axios for HTTP requests, official MCP SDK) |
| **No install scripts** | No hidden code runs when you install the package |
| **Open source** | All code is visible and auditable |
| **Read-focused** | Most operations are read-only; write operations require explicit action |

## Requirements

Before you start, you'll need:

1. **A Palo Alto Networks firewall** with API access enabled
2. **An API key** from your firewall (see [How to get an API key](#how-to-get-an-api-key))
3. **Node.js** installed on your computer (version 18 or higher)

## Quick Start

1. **Download the code**
   ```bash
   git clone https://github.com/danielitus/paloalto-mcp-server.git
   cd paloalto-mcp-server
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Build the project**
   ```bash
   npm run build
   ```

4. **Set up your credentials**

   Create a file called `.env` in the project folder with:
   ```
   PANOS_API_KEY=your-api-key-here
   PANOS_API_BASE_URL=https://your-firewall-address/restapi/v11.0
   ```

5. **Start the server**
   ```bash
   npm start
   ```

## How to Get an API Key

1. Log into your Palo Alto firewall's web interface
2. Go to **Device** → **Administrators**
3. Click on your admin username
4. Look for **API Key** section and generate a new key
5. Copy and save this key securely - you'll need it for configuration

> **Important:** Keep your API key secret! Anyone with this key can access your firewall's API.

## What Can It Do?

Once set up, you can ask your AI assistant to perform these tasks:

### View System Information
Ask: *"What virtual systems are configured on my firewall?"*

### List Resources
Ask: *"Show me all the address objects"* or *"List my security rules"*

Available categories:
- **Objects**: Addresses, Services, Tags, Security Profiles, and more
- **Policies**: Security Rules, NAT Rules, QoS Rules, and more
- **Network**: Interfaces, Zones, VPNs, DHCP settings, and more
- **Devices**: Virtual Systems, Server Profiles, and more

### View Configuration Details
Ask: *"Show me the configuration for the trust zone"*

### Move or Copy Configurations
Ask: *"Clone the web-server address object to the DMZ"*

## Real-World Examples

Here are practical examples of how you can use this tool with your AI assistant:

### Finding Redundant Security Rules

Redundant rules are rules that overlap with other rules, making them unnecessary. They clutter your firewall and can cause confusion.

**Example conversation:**

> **You:** "List all my security rules and check for any redundant ones"
>
> **AI Assistant:** *Retrieves your security rules and analyzes them*
>
> "I found 3 potentially redundant rules:
> 1. **Rule 'allow-web-old'** (line 15) - This rule allows HTTP from 'internal-net' to 'any', but **Rule 'allow-all-web'** (line 8) already covers this traffic
> 2. **Rule 'temp-ssh-access'** (line 23) - Appears to be a temporary rule from 6 months ago that duplicates 'admin-ssh-access'
> 3. **Rule 'legacy-ftp'** (line 31) - Allows FTP but is shadowed by 'block-legacy-protocols' (line 5) which denies all legacy protocols"

### Finding Shadowed Rules

Shadowed rules are rules that never get hit because a rule above them matches the same traffic first.

**Example conversation:**

> **You:** "Are any of my security rules being shadowed by other rules?"
>
> **AI Assistant:** *Analyzes rule order and matching criteria*
>
> "Yes, I found 2 shadowed rules:
> 1. **Rule 'allow-marketing-web'** (line 20) will never match because **Rule 'allow-department-web'** (line 12) already matches all department subnets including marketing
> 2. **Rule 'deny-guest-wifi'** (line 45) is shadowed by **Rule 'guest-limited-access'** (line 40) - both match guest-wifi zone but line 40 processes first"

### Checking for Overly Permissive Rules

Find rules that might be too broad and pose security risks.

**Example conversation:**

> **You:** "Find any security rules that are too permissive or use 'any' too broadly"
>
> **AI Assistant:** *Reviews rules for security best practices*
>
> "I found these potentially risky rules:
> 1. **Rule 'allow-all-outbound'** - Allows ANY application from trust zone to untrust zone. Consider restricting to specific applications
> 2. **Rule 'vendor-access'** - Source is 'any' with destination to your internal servers. Recommend limiting source to specific vendor IPs
> 3. **Rule 'legacy-app-rule'** - Uses 'any' for both service and application. This bypasses App-ID inspection"

### Analyzing Rule Usage

Identify rules that may no longer be needed.

**Example conversation:**

> **You:** "Which security rules haven't been hit recently and might be candidates for removal?"
>
> **AI Assistant:** *Retrieves rules and checks hit counts where available*
>
> "Based on the rule data, these rules have zero or very low hit counts:
> 1. **Rule 'contractor-access-2023'** - 0 hits, last modified January 2023
> 2. **Rule 'old-app-server'** - 0 hits, references address object 'app-srv-retired'
> 3. **Rule 'test-rule-delete-me'** - Name suggests temporary rule, 0 hits"

### Comparing Address Objects

Find duplicate or overlapping address definitions.

**Example conversation:**

> **You:** "Do I have any duplicate or overlapping address objects?"
>
> **AI Assistant:** *Retrieves and compares address objects*
>
> "Found these issues:
> 1. **'web-server-1'** (10.0.1.100) and **'webserver1'** (10.0.1.100) - Same IP, different names
> 2. **'dev-subnet'** (10.0.50.0/24) overlaps with **'dev-servers'** (10.0.50.0/28) - The /28 is fully contained within the /24
> 3. **'old-dns'** and **'dns-primary'** both point to 8.8.8.8"

### Security Audit Summary

Get a high-level overview of your firewall's security posture.

**Example conversation:**

> **You:** "Give me a security audit summary of my firewall rules"
>
> **AI Assistant:** *Performs comprehensive analysis*
>
> "**Security Audit Summary:**
>
> | Category | Count | Status |
> |----------|-------|--------|
> | Total Rules | 47 | - |
> | Rules using 'any' source | 5 | ⚠️ Review recommended |
> | Rules using 'any' destination | 3 | ⚠️ Review recommended |
> | Disabled rules | 8 | Consider cleanup |
> | Rules without logging | 12 | 🔴 Enable logging |
> | Potentially redundant | 4 | Review for removal |
>
> **Top recommendations:**
> 1. Enable logging on the 12 rules currently not logging
> 2. Review the 5 rules with 'any' source for tighter restrictions
> 3. Consider removing or archiving the 8 disabled rules"

## Configuration Options

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `PANOS_API_KEY` | Yes | Your firewall API key | `LUFRPT...` |
| `PANOS_API_BASE_URL` | No | Your firewall's API address | `https://192.168.1.1/restapi/v11.0` |

## Troubleshooting

### "PANOS_API_KEY environment variable is required"
You haven't set your API key. Make sure your `.env` file exists and contains your key.

### Connection errors
- Check that your firewall is accessible from your computer
- Verify the `PANOS_API_BASE_URL` is correct
- Ensure your API key is valid and hasn't expired

### Permission errors
Your API key may not have sufficient permissions. Check with your firewall administrator.

## For Developers

### Project Structure
```
paloalto-mcp-server/
├── src/
│   └── index.ts      # Main server code
├── build/
│   └── index.js      # Compiled JavaScript
├── package.json      # Project dependencies
└── tsconfig.json     # TypeScript configuration
```

### Dependencies

This project uses minimal, trusted dependencies:

| Package | Purpose | Weekly Downloads |
|---------|---------|-----------------|
| `@modelcontextprotocol/sdk` | Official MCP protocol library | Maintained by Anthropic |
| `axios` | HTTP requests to firewall API | 45M+ weekly downloads |
| `typescript` | Development only | Standard tooling |

### Building from Source

```bash
npm install
npm run build
```

### Running with Docker

```bash
docker build -t paloalto-mcp-server .
docker run -e PANOS_API_KEY=your-key -e PANOS_API_BASE_URL=https://your-firewall/restapi/v11.0 paloalto-mcp-server
```

## Getting Help

- **Issues**: Report problems on [GitHub Issues](https://github.com/danielitus/paloalto-mcp-server/issues)
- **Questions**: Check the troubleshooting section above

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Make your changes
4. Submit a Pull Request

## License

MIT License - see LICENSE file for details.

---

**Note:** This tool is not officially affiliated with Palo Alto Networks. Use at your own discretion and always follow your organization's security policies.
