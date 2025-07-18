# WiFi Network Automation with n8n and Airtable

## Overview

This guide demonstrates how to set up an automated WiFi network with a captive portal that collects user data through forms and automatically sends the information to Airtable using n8n workflows and webhooks.

## Architecture Overview

```
WiFi User → Captive Portal → Form Submission → Webhook → n8n → Airtable
                                              ↓
                                         Additional Actions:
                                         - Email notifications
                                         - CRM updates
                                         - Marketing automation
```

## Components Required

### 1. Network Infrastructure
- **Router/Access Point**: MikroTik, Ubiquiti, pfSense, or cloud-managed solution
- **Captive Portal**: Custom HTML form or platform-based solution
- **Internet Connection**: Stable broadband for webhook processing

### 2. Software Stack
- **n8n**: Workflow automation platform (self-hosted or cloud)
- **Airtable**: Database for storing user information
- **Web Server**: To host captive portal (Apache/Nginx or cloud hosting)

### 3. Services
- **Webhook endpoint**: For receiving form submissions
- **SSL Certificate**: For secure data transmission
- **Domain name**: For professional appearance

## Implementation Steps

### Step 1: Set Up Airtable Database

#### Create Airtable Base Structure

```javascript
// Airtable Base: "WiFi Users Database"
// Table: "User Registrations"

Fields:
- Name (Single line text)
- Email (Email)
- Phone (Phone number)
- Device Type (Single select: Mobile, Laptop, Tablet, Other)
- MAC Address (Single line text)
- Connection Time (Date and time)
- Location (Single line text)
- Terms Accepted (Checkbox)
- Marketing Consent (Checkbox)
- Session Duration (Duration)
- Return Visitor (Checkbox)
```

#### Get Airtable API Credentials
1. Go to https://airtable.com/api
2. Select your base
3. Copy Base ID and API Key
4. Note the table name and field names

### Step 2: Set Up n8n Workflow

#### Install n8n

**Self-hosted (Docker):**
```bash
# Create docker-compose.yml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your_secure_password
      - WEBHOOK_URL=https://your-domain.com/
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - n8n-network

volumes:
  n8n_data:

networks:
  n8n-network:
    driver: bridge
```

**Cloud option**: Use n8n.cloud for managed hosting

#### Create n8n Workflow

```json
{
  "name": "WiFi User Registration Automation",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "wifi-registration",
        "responseMode": "responseNode"
      },
      "name": "Webhook - WiFi Form",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "resource": "append",
        "table": "User Registrations",
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Name": "={{$json.name}}",
            "Email": "={{$json.email}}",
            "Phone": "={{$json.phone}}",
            "Device Type": "={{$json.device_type}}",
            "MAC Address": "={{$json.mac_address}}",
            "Connection Time": "={{$now}}",
            "Location": "={{$json.location}}",
            "Terms Accepted": "={{$json.terms_accepted}}",
            "Marketing Consent": "={{$json.marketing_consent}}"
          }
        }
      },
      "name": "Airtable - Save User Data",
      "type": "n8n-nodes-base.airtable",
      "position": [450, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json.marketing_consent}}",
              "operation": "equal",
              "value2": "true"
            }
          ]
        }
      },
      "name": "Check Marketing Consent",
      "type": "n8n-nodes-base.if",
      "position": [650, 300]
    },
    {
      "parameters": {
        "fromEmail": "wifi@your-domain.com",
        "toEmail": "={{$node['Webhook - WiFi Form'].json.email}}",
        "subject": "Welcome to Our WiFi Network!",
        "text": "Thank you for connecting to our WiFi network. We hope you enjoy your visit!"
      },
      "name": "Send Welcome Email",
      "type": "n8n-nodes-base.emailSend",
      "position": [850, 200]
    },
    {
      "parameters": {
        "respondWith": "json",
        "responseBody": {
          "status": "success",
          "message": "Registration successful",
          "redirect_url": "https://your-success-page.com"
        }
      },
      "name": "Response - Success",
      "type": "n8n-nodes-base.respondToWebhook",
      "position": [850, 400]
    }
  ],
  "connections": {
    "Webhook - WiFi Form": {
      "main": [
        [
          {
            "node": "Airtable - Save User Data",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Airtable - Save User Data": {
      "main": [
        [
          {
            "node": "Check Marketing Consent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Check Marketing Consent": {
      "main": [
        [
          {
            "node": "Send Welcome Email",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Response - Success",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Send Welcome Email": {
      "main": [
        [
          {
            "node": "Response - Success",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

### Step 3: Create Captive Portal Form

#### HTML Form Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WiFi Access - Registration Required</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .logo {
            text-align: center;
            margin-bottom: 30px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="text"], input[type="email"], input[type="tel"], select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
        }
        .checkbox-group {
            margin: 20px 0;
        }
        .checkbox-group input[type="checkbox"] {
            margin-right: 10px;
        }
        .submit-btn {
            background-color: #007bff;
            color: white;
            padding: 12px 30px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
            font-size: 16px;
        }
        .submit-btn:hover {
            background-color: #0056b3;
        }
        .loading {
            display: none;
            text-align: center;
            color: #666;
        }
        .error {
            color: red;
            margin-top: 10px;
            display: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="logo">
            <h1>Welcome to Our WiFi Network</h1>
            <p>Please complete this quick registration to access free WiFi</p>
        </div>

        <form id="wifiForm">
            <div class="form-group">
                <label for="name">Full Name *</label>
                <input type="text" id="name" name="name" required>
            </div>

            <div class="form-group">
                <label for="email">Email Address *</label>
                <input type="email" id="email" name="email" required>
            </div>

            <div class="form-group">
                <label for="phone">Phone Number</label>
                <input type="tel" id="phone" name="phone">
            </div>

            <div class="form-group">
                <label for="device_type">Device Type</label>
                <select id="device_type" name="device_type">
                    <option value="Mobile">Mobile Phone</option>
                    <option value="Laptop">Laptop</option>
                    <option value="Tablet">Tablet</option>
                    <option value="Other">Other</option>
                </select>
            </div>

            <div class="checkbox-group">
                <label>
                    <input type="checkbox" name="terms_accepted" required>
                    I agree to the <a href="/terms" target="_blank">Terms of Service</a> *
                </label>
            </div>

            <div class="checkbox-group">
                <label>
                    <input type="checkbox" name="marketing_consent">
                    I'd like to receive promotional offers and updates
                </label>
            </div>

            <button type="submit" class="submit-btn">Connect to WiFi</button>
            
            <div class="loading">
                Processing your registration...
            </div>
            
            <div class="error" id="errorMessage">
                Registration failed. Please try again.
            </div>
        </form>
    </div>

    <script>
        document.getElementById('wifiForm').addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const submitBtn = document.querySelector('.submit-btn');
            const loading = document.querySelector('.loading');
            const errorDiv = document.getElementById('errorMessage');
            
            // Show loading state
            submitBtn.style.display = 'none';
            loading.style.display = 'block';
            errorDiv.style.display = 'none';
            
            // Get form data
            const formData = new FormData(this);
            const data = {
                name: formData.get('name'),
                email: formData.get('email'),
                phone: formData.get('phone'),
                device_type: formData.get('device_type'),
                terms_accepted: formData.get('terms_accepted') ? true : false,
                marketing_consent: formData.get('marketing_consent') ? true : false,
                mac_address: await getMacAddress(),
                location: 'Main Location', // Set your location
                user_agent: navigator.userAgent,
                timestamp: new Date().toISOString()
            };
            
            try {
                const response = await fetch('https://your-n8n-instance.com/webhook/wifi-registration', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify(data)
                });
                
                const result = await response.json();
                
                if (result.status === 'success') {
                    // Redirect to success page or grant internet access
                    window.location.href = result.redirect_url || 'http://google.com';
                } else {
                    throw new Error('Registration failed');
                }
                
            } catch (error) {
                console.error('Error:', error);
                submitBtn.style.display = 'block';
                loading.style.display = 'none';
                errorDiv.style.display = 'block';
            }
        });
        
        // Function to get MAC address (limited in browsers for security)
        async function getMacAddress() {
            // This is a placeholder - MAC address detection is limited in browsers
            // You might need to get this from the router/network level
            return 'Browser-Limited';
        }
    </script>
</body>
</html>
```

### Step 4: Configure Router/Network

#### For MikroTik RouterOS

```bash
# Enable hotspot
/ip hotspot setup

# Configure hotspot profile with custom login page
/ip hotspot user profile
set [find default=yes] login-by=http-post
set [find default=yes] http-proxy=0.0.0.0:0

# Set custom login page
/ip hotspot walled-garden
add dst-host=your-domain.com
add dst-host=your-n8n-instance.com

# Configure hotspot server
/ip hotspot
set [find default=yes] login-timeout=00:05:00
set [find default=yes] idle-timeout=00:30:00
```

#### For pfSense

1. Navigate to **Services > Captive Portal**
2. Enable captive portal
3. Set **Portal page contents** to your custom HTML
4. Configure **Allowed Hostnames**: your-domain.com, your-n8n-instance.com
5. Set authentication method to **No Authentication** (handled by form)

#### For Ubiquiti UniFi

1. Go to **Settings > Guest Control**
2. Enable **Guest Portal**
3. Choose **External Portal Server**
4. Set portal URL to your custom form
5. Configure **Post-authentication redirect**

### Step 5: Enhanced n8n Workflow Features

#### Advanced Workflow with Multiple Actions

```json
{
  "name": "Advanced WiFi Automation",
  "nodes": [
    {
      "name": "Webhook - WiFi Form",
      "type": "n8n-nodes-base.webhook"
    },
    {
      "name": "Validate Email",
      "type": "n8n-nodes-base.function",
      "parameters": {
        "functionCode": "const email = items[0].json.email;\nconst emailRegex = /^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$/;\n\nif (!emailRegex.test(email)) {\n  throw new Error('Invalid email format');\n}\n\nreturn items;"
      }
    },
    {
      "name": "Check Duplicate User",
      "type": "n8n-nodes-base.airtable",
      "parameters": {
        "operation": "list",
        "table": "User Registrations",
        "filterByFormula": "={{`{Email} = \"${$json.email}\"`}}"
      }
    },
    {
      "name": "Is New User?",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json.records.length}}",
              "operation": "equal",
              "value2": 0
            }
          ]
        }
      }
    },
    {
      "name": "Create New User",
      "type": "n8n-nodes-base.airtable"
    },
    {
      "name": "Update Existing User",
      "type": "n8n-nodes-base.airtable",
      "parameters": {
        "operation": "update",
        "table": "User Registrations"
      }
    },
    {
      "name": "Send to CRM",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://your-crm.com/api/contacts",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer your_crm_token"
            }
          ]
        }
      }
    },
    {
      "name": "Trigger Marketing Automation",
      "type": "n8n-nodes-base.httpRequest"
    }
  ]
}
```

### Step 6: Security and Compliance

#### SSL/HTTPS Configuration

```nginx
# Nginx configuration for captive portal
server {
    listen 443 ssl;
    server_name your-domain.com;
    
    ssl_certificate /path/to/certificate.pem;
    ssl_certificate_key /path/to/private.key;
    
    location / {
        root /var/www/captive-portal;
        index index.html;
    }
    
    location /webhook {
        proxy_pass https://your-n8n-instance.com;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Privacy Compliance

```javascript
// Add to form JavaScript for GDPR compliance
function showPrivacyNotice() {
    const notice = `
        <div id="privacyNotice" style="background: #f8f9fa; padding: 15px; border-radius: 5px; margin: 20px 0;">
            <h4>Privacy Notice</h4>
            <p>We collect your data to provide WiFi access and improve our services. 
               Your data will be stored securely and won't be shared with third parties without consent.</p>
            <p><a href="/privacy-policy" target="_blank">Read our full Privacy Policy</a></p>
        </div>
    `;
    document.querySelector('.container').insertAdjacentHTML('afterbegin', notice);
}

// Call on page load
document.addEventListener('DOMContentLoaded', showPrivacyNotice);
```

### Step 7: Monitoring and Analytics

#### n8n Workflow for Analytics

```javascript
// Function node to track user behavior
const userData = items[0].json;

// Calculate session metrics
const sessionData = {
    user_id: userData.id,
    connection_time: userData.timestamp,
    device_info: {
        type: userData.device_type,
        user_agent: userData.user_agent
    },
    location: userData.location,
    is_return_visitor: userData.return_visitor
};

// Send to analytics service
return [{
    json: {
        ...userData,
        analytics: sessionData
    }
}];
```

#### Dashboard Setup in Airtable

1. Create **Views** for different data segments:
   - New users today
   - Return visitors
   - Marketing consent rates
   - Device type distribution

2. Set up **Automations** in Airtable:
   - Daily summary emails
   - Welcome email sequences
   - Data backup routines

### Step 8: Testing and Deployment

#### Testing Checklist

- [ ] Form submission works correctly
- [ ] Webhook receives data properly
- [ ] Airtable records are created
- [ ] Email notifications are sent
- [ ] WiFi access is granted after submission
- [ ] Mobile responsiveness
- [ ] SSL certificate works
- [ ] Privacy compliance elements

#### Deployment Steps

1. **Set up hosting** for captive portal
2. **Configure domain** and SSL
3. **Deploy n8n** workflow
4. **Configure router** settings
5. **Test end-to-end** flow
6. **Monitor** for issues
7. **Optimize** based on usage

## Benefits of This Approach

### Technical Benefits
- **Scalable**: n8n can handle high volumes of submissions
- **Flexible**: Easy to add new automation steps
- **Reliable**: Webhook-based architecture is robust
- **Maintainable**: Visual workflow editor in n8n

### Business Benefits
- **Data Collection**: Comprehensive user database
- **Marketing Automation**: Immediate engagement opportunities
- **Analytics**: Real-time insights into user behavior
- **Compliance**: Built-in privacy and consent management

## Advanced Features

### 1. Progressive Profiling
```javascript
// n8n function to implement progressive profiling
const existingUser = items[0].json.records[0];
const newData = items[0].json.formData;

if (existingUser) {
    // Update only new fields
    const updatedFields = {};
    for (const field in newData) {
        if (!existingUser.fields[field] || existingUser.fields[field] === '') {
            updatedFields[field] = newData[field];
        }
    }
    return [{ json: { id: existingUser.id, fields: updatedFields } }];
}
```

### 2. Conditional Workflows
```javascript
// Different workflows based on user type
const userEmail = items[0].json.email;
const domain = userEmail.split('@')[1];

const workflowRouting = {
    'company.com': 'employee_workflow',
    'gmail.com': 'guest_workflow',
    'student.edu': 'student_workflow'
};

return [{
    json: {
        ...items[0].json,
        workflow_type: workflowRouting[domain] || 'default_workflow'
    }
}];
```

### 3. Integration Examples

#### Slack Notifications
```json
{
  "name": "Slack - New User Alert",
  "type": "n8n-nodes-base.slack",
  "parameters": {
    "channel": "#wifi-registrations",
    "text": "New WiFi user registered: {{$json.name}} ({{$json.email}})"
  }
}
```

#### Google Sheets Backup
```json
{
  "name": "Google Sheets - Backup Data",
  "type": "n8n-nodes-base.googleSheets",
  "parameters": {
    "operation": "append",
    "documentId": "your_sheet_id",
    "sheetName": "WiFi Users"
  }
}
```

## Troubleshooting

### Common Issues

1. **Webhook not receiving data**
   - Check firewall settings
   - Verify SSL certificate
   - Test webhook URL directly

2. **Airtable API errors**
   - Validate API key and base ID
   - Check field name mappings
   - Verify table permissions

3. **Form not submitting**
   - Check JavaScript console for errors
   - Verify CORS settings
   - Test network connectivity

4. **Users not getting internet access**
   - Check router captive portal settings
   - Verify redirect URLs
   - Test authentication flow

### Monitoring and Logs

```javascript
// n8n logging function
console.log('WiFi Registration:', {
    timestamp: new Date().toISOString(),
    user: items[0].json.email,
    status: 'success',
    workflow_id: $workflow.id
});
```

## Conclusion

This WiFi automation setup using n8n and Airtable provides:

- **Automated data collection** from WiFi users
- **Real-time processing** of registrations
- **Flexible workflow** automation
- **Comprehensive user database**
- **Marketing automation** capabilities
- **Compliance-ready** privacy features

The system is scalable, maintainable, and can be easily extended with additional features as your needs grow. The visual workflow editor in n8n makes it easy to modify and expand the automation without complex coding.