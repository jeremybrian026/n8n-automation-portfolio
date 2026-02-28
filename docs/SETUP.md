# Setup Guide

This guide will help you set up the n8n Enterprise Automation Suite for demonstration or adaptation to your own use case.

## Quick Start

### Prerequisites

- n8n installed (see [n8n.io](https://n8n.io) for installation)
- Google account (for Gmail and Sheets integration)
- Notion account (optional, for handover workflow)

### Basic Installation

```bash
# Install n8n globally
npm install -g n8n

# Start n8n
n8n start

# Access at http://localhost:5678
```

## Workflow Import

### Option 1: Import via UI

1. Open n8n at `http://localhost:5678`
2. Click **Workflows** → **Import from File**
3. Select a workflow JSON file
4. Click **Import**

### Option 2: Import All at Once

```bash
# From project directory
for file in workflows/**/*.json; do
  echo "Import $file into n8n manually via UI"
done
```

## Configuration

### 1. Google Sheets Setup

**Create Your Spreadsheet**:

1. Create a new Google Sheet
2. Add these column headers:

```
For Monitoring Workflows:
DEVICE NAME | ACCOUNT NUMBER | CID | SERIAL | TERMINATION | POC | CONTACT | ADDRESS | TO | CC

For Database Workflows:
STATUS | ACCOUNT NUMBER | CID | ONU SN | SITE NAME | CLIENT | SITE ADDRESS | 
DEVICE MODEL | SERIAL | MODEL | IP BLOCK | WAN PARENT SUBNET | VLAN | 
TERMINATION | DEVICE | NPE UPLINK | NPE REMOTE IP | CIRCUIT ID | PLAN | POC | CONTACT
```

**Sample Data**:

```csv
Active,ACC-001,CID-001,SN001,Office-A,Client-A,123 Main St,...
Active,ACC-002,CID-002,SN002,Office-B,Client-B,456 Oak Ave,...
```

**Get Spreadsheet ID**:

From URL: `https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit`

###  2. Gmail Setup

**Create Labels**:

1. Open Gmail → Settings → Labels
2. Create these labels:
   - `monitoring-alerts`
   - `device-offline`
   - `system-notifications`

**Get Label IDs**:

Use Gmail API or manually from the workflows (they're visible in execution logs)

### 3. n8n Data Tables (for Auth)

**Create Tables**:

1. In n8n, go to **Data Tables** (if available)
2. Create `Users` table:

```
Columns: ID | USERNAME | PASSWORD | FULL_NAME | ROLE | ACTIVE
Sample: 1 | admin | password123 | Admin User | admin | YES
```

3. Create `Audit_Logs` table:

```
Columns: TIMESTAMP | USER_ID | USERNAME | ACTION | IP
```

### 4. Credentials Setup

**Add Google Sheets OAuth2**:

1. n8n → Credentials → New → Google Sheets OAuth2 API
2. Enter Client ID and Secret (from Google Cloud Console)
3. Authenticate

**Add Gmail OAuth2**:

1. n8n → Credentials → New → Gmail OAuth2
2. Enter Client ID and Secret
3. Authenticate

**Add Notion API** (optional):

1. n8n → Credentials → New → Notion API
2. Enter Integration Token

## Customization for Your Use Case

### Monitoring Workflows

**Modify for Your Alerts**:

1. Open `Auto-Ticketing_Site-A.json`
2. Update the JavaScript parser to match your email format:

```javascript
// Customize cell indices based on your HTML table
return {
  deviceName : cells[0] || "",  // Column 1
  alarmType  : cells[4] || "",  // Column 5 (adjust as needed)
  timestamp  : cells[6] || ""   // Column 7 (adjust as needed)
};
```

3. Update email template with your branding
4. Change recipient email addresses

### Database Workflows

**Adapt to Your Schema**:

1. Open `Database_v2.json`
2. Update column mappings to match your spreadsheet:

```javascript
grouped[client].push({
  status: row['STATUS'] || '',
  // Add your fields here
  customField: row['YOUR_COLUMN'] || '',
});
```

### API Endpoints

**Change Webhook Paths**:

In each workflow, update the webhook node:
- `/webhook/auth-login` → `/webhook/your-endpoint`
- `/webhook/db-fetch` → `/webhook/your-endpoint`

## Testing

### Test Monitoring Workflow

```bash
# Create a test email with HTML table
# Apply your Gmail label
# Check n8n execution history
```

### Test Auth API

```bash
curl -X POST http://localhost:5678/webhook/auth-login \
  -H "Content-Type: application/json" \
  -d '{"id_no": "admin", "password": "password123"}'
```

### Test Database API

```bash
curl http://localhost:5678/webhook/db-fetch
```

## Production Deployment

### Security Recommendations

- [ ] Enable HTTPS
- [ ] Add API authentication
- [ ] Implement rate limiting
- [ ] Use environment variables for sensitive data
- [ ] Hash passwords
- [ ] Restrict CORS to specific domains

### Docker Deployment

```yaml
version: '3.7'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=change-this-password
      - N8N_HOST=your-domain.com
      - N8N_PROTOCOL=https
    volumes:
      - ~/.n8n:/home/node/.n8n
    restart: unless-stopped
```

## Troubleshooting

### Common Issues

**Workflow doesn't trigger**:
- Check if workflow is activated
- Verify credentials are valid
- Review label IDs match

**API returns 404**:
- Ensure workflow is active
- Check webhook path is correct
- Verify n8n is running

**Data not updating**:
- Check Google Sheets permissions
- Verify OAuth tokens haven't expired
- Test connection manually

## Portfolio Tips

When showcasing this project:

1. **Highlight Impact**: Mention time saved, errors reduced
2. **Explain Decisions**: Why n8n? Why this approach?
3. **Show Technical Depth**: Discuss the JavaScript parsing logic
4. **Demonstrate Problem-Solving**: How you handled edge cases
5. **Code Quality**: Point out error handling, logging

## Adaptation Guide

This suite can be adapted for various industries:

**IT Help Desk**: Ticket automation  
**E-commerce**: Order processing alerts  
**Healthcare**: Patient notification systems  
**Manufacturing**: Equipment monitoring  
**Finance**: Transaction alerts  

---

For questions or feedback, feel free to reach out!

**Last Updated**: February 2026
