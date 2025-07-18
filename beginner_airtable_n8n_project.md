# Beginner's Guide: Simple Form to Airtable with n8n

## What We're Building

A simple contact form on a webpage that automatically saves submissions to an Airtable database using n8n automation. Perfect for beginners!

**What happens:**
1. User fills out a form on your webpage
2. Form sends data to n8n (automation tool)
3. n8n automatically saves data to Airtable
4. User gets a success message

## Step 1: Create Your Airtable Database (15 minutes)

### 1.1 Sign Up for Airtable
1. Go to [airtable.com](https://airtable.com)
2. Click "Sign up for free"
3. Create your account with email and password

### 1.2 Create Your First Base
1. After logging in, click "Create a base"
2. Choose "Start from scratch"
3. Name your base: "Contact Form Submissions"
4. Click "Create base"

### 1.3 Set Up Your Table
1. You'll see a table called "Table 1" - rename it to "Contacts"
2. Create these fields (columns):

**Click on each column header to rename:**
- Column A: Rename to "Name" (leave as "Single line text")
- Column B: Rename to "Email" (change type to "Email")
- Column C: Rename to "Phone" (change type to "Phone number")
- Column D: Rename to "Message" (change type to "Long text")
- Column E: Rename to "Submitted At" (change type to "Date and time")

**To change column types:**
1. Click the dropdown arrow next to column name
2. Click "Customize field type"
3. Select the appropriate type
4. Click "Save"

### 1.4 Get Your Airtable API Information
1. Go to [airtable.com/api](https://airtable.com/api)
2. Click on your "Contact Form Submissions" base
3. Copy and save these (we'll need them later):
   - **Base ID**: Starts with "app..." (e.g., appAbC123dEf456)
   - **Table Name**: "Contacts"

### 1.5 Create API Token
1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
2. Click "Create new token"
3. Name it: "n8n Integration"
4. Under scopes, select:
   - `data.records:read`
   - `data.records:write`
5. Under Access, select your base: "Contact Form Submissions"
6. Click "Create token"
7. **IMPORTANT**: Copy and save this token (starts with "pat...")

## Step 2: Set Up n8n (20 minutes)

### 2.1 Sign Up for n8n Cloud (Easiest Option)
1. Go to [n8n.cloud](https://n8n.cloud)
2. Click "Start for free"
3. Create your account
4. Choose the free plan

*Alternative: You can self-host n8n, but cloud is easier for beginners*

### 2.2 Create Your First Workflow
1. After logging in, click "New workflow"
2. You'll see a blank canvas with nodes

### 2.3 Add Webhook Node (This receives form data)
1. Click the "+" button on the canvas
2. Search for "Webhook"
3. Click on "Webhook" node
4. In the webhook settings:
   - **HTTP Method**: POST
   - **Path**: contact-form
   - **Response Mode**: "Respond to Webhook"
5. Click "Save"

### 2.4 Get Your Webhook URL
1. Click "Execute Node" on the webhook
2. Copy the webhook URL (it looks like: `https://your-instance.app.n8n.cloud/webhook/contact-form`)
3. Save this URL - we'll need it for our form

### 2.5 Add Airtable Node
1. Click the "+" after the webhook node
2. Search for "Airtable"
3. Click on "Airtable" node
4. Configure the Airtable node:
   - **Resource**: Record
   - **Operation**: Append
   - **Base ID**: Paste your Base ID from Step 1.4
   - **Table**: Contacts

### 2.6 Connect Airtable Credentials
1. In the Airtable node, click "Create New" next to Credentials
2. **Name**: My Airtable Account
3. **API Key**: Paste your API token from Step 1.5
4. Click "Save"

### 2.7 Map Form Data to Airtable Fields
In the Airtable node, under "Columns":
1. Click "Add Column"
2. Set up each field:

**Column 1:**
- **Column**: Name
- **Value**: `{{ $json.name }}`

**Column 2:**
- **Column**: Email  
- **Value**: `{{ $json.email }}`

**Column 3:**
- **Column**: Phone
- **Value**: `{{ $json.phone }}`

**Column 4:**
- **Column**: Message
- **Value**: `{{ $json.message }}`

**Column 5:**
- **Column**: Submitted At
- **Value**: `{{ $now }}`

### 2.8 Add Response Node
1. Click "+" after the Airtable node
2. Search for "Respond to Webhook"
3. Click on it
4. Configure:
   - **Response Body**: 
   ```json
   {
     "status": "success",
     "message": "Thank you! Your message has been received."
   }
   ```

### 2.9 Save Your Workflow
1. Click "Save" in the top right
2. Name it: "Contact Form to Airtable"
3. Click "Save"

## Step 3: Create Your HTML Form (10 minutes)

### 3.1 Create the HTML File
Create a new file called `contact-form.html` and copy this code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contact Us</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        
        .form-container {
            background: white;
            padding: 40px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        h1 {
            color: #333;
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
            color: #555;
        }
        
        input[type="text"],
        input[type="email"],
        input[type="tel"],
        textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
            box-sizing: border-box;
        }
        
        textarea {
            height: 120px;
            resize: vertical;
        }
        
        .submit-btn {
            background-color: #007bff;
            color: white;
            padding: 12px 30px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            width: 100%;
        }
        
        .submit-btn:hover {
            background-color: #0056b3;
        }
        
        .submit-btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .success-message {
            background-color: #d4edda;
            color: #155724;
            padding: 15px;
            border-radius: 5px;
            margin-top: 20px;
            display: none;
        }
        
        .error-message {
            background-color: #f8d7da;
            color: #721c24;
            padding: 15px;
            border-radius: 5px;
            margin-top: 20px;
            display: none;
        }
        
        .loading {
            display: none;
            text-align: center;
            color: #666;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Contact Us</h1>
        
        <form id="contactForm">
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
                <label for="message">Message *</label>
                <textarea id="message" name="message" placeholder="Tell us how we can help you..." required></textarea>
            </div>
            
            <button type="submit" class="submit-btn" id="submitBtn">
                Send Message
            </button>
            
            <div class="loading" id="loading">
                Sending your message...
            </div>
            
            <div class="success-message" id="successMessage">
                Thank you! Your message has been received. We'll get back to you soon.
            </div>
            
            <div class="error-message" id="errorMessage">
                Sorry, there was an error sending your message. Please try again.
            </div>
        </form>
    </div>

    <script>
        // IMPORTANT: Replace this URL with your actual webhook URL from n8n
        const WEBHOOK_URL = 'https://your-instance.app.n8n.cloud/webhook/contact-form';
        
        document.getElementById('contactForm').addEventListener('submit', async function(e) {
            e.preventDefault();
            
            // Get form elements
            const submitBtn = document.getElementById('submitBtn');
            const loading = document.getElementById('loading');
            const successMessage = document.getElementById('successMessage');
            const errorMessage = document.getElementById('errorMessage');
            
            // Hide previous messages
            successMessage.style.display = 'none';
            errorMessage.style.display = 'none';
            
            // Show loading state
            submitBtn.disabled = true;
            submitBtn.textContent = 'Sending...';
            loading.style.display = 'block';
            
            // Get form data
            const formData = new FormData(this);
            const data = {
                name: formData.get('name'),
                email: formData.get('email'),
                phone: formData.get('phone'),
                message: formData.get('message')
            };
            
            try {
                // Send data to n8n webhook
                const response = await fetch(WEBHOOK_URL, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify(data)
                });
                
                const result = await response.json();
                
                if (response.ok && result.status === 'success') {
                    // Success!
                    successMessage.style.display = 'block';
                    this.reset(); // Clear the form
                } else {
                    throw new Error('Form submission failed');
                }
                
            } catch (error) {
                console.error('Error:', error);
                errorMessage.style.display = 'block';
            } finally {
                // Reset button state
                submitBtn.disabled = false;
                submitBtn.textContent = 'Send Message';
                loading.style.display = 'none';
            }
        });
    </script>
</body>
</html>
```

### 3.2 Update the Webhook URL
1. In the HTML code above, find this line:
   ```javascript
   const WEBHOOK_URL = 'https://your-instance.app.n8n.cloud/webhook/contact-form';
   ```
2. Replace it with your actual webhook URL from Step 2.4

## Step 4: Test Your Project (5 minutes)

### 4.1 Test the Workflow in n8n
1. Go back to your n8n workflow
2. Click "Execute Workflow" button
3. The webhook should be "listening"

### 4.2 Test the Form
1. Open your `contact-form.html` file in a web browser
2. Fill out the form with test data
3. Click "Send Message"
4. You should see a success message

### 4.3 Check Airtable
1. Go back to your Airtable base
2. You should see a new record with your test data!

## Step 5: Make It Live (Optional)

### 5.1 Host Your Form Online
**Option 1: GitHub Pages (Free)**
1. Create a GitHub account
2. Create a new repository
3. Upload your HTML file
4. Enable GitHub Pages in repository settings

**Option 2: Netlify (Free)**
1. Go to [netlify.com](https://netlify.com)
2. Drag and drop your HTML file
3. Get a free URL

**Option 3: Local Testing**
- Just open the HTML file in your browser for testing

## Troubleshooting Common Issues

### ❌ Form doesn't submit
**Check:**
- Is the webhook URL correct in your HTML?
- Is your n8n workflow active?
- Check browser console for errors (F12 → Console)

### ❌ Data doesn't appear in Airtable
**Check:**
- Are your Airtable credentials correct?
- Do the column names match exactly?
- Is your Base ID correct?

### ❌ n8n shows errors
**Check:**
- API token permissions
- Base ID format
- Field mapping in the Airtable node

## What You've Learned

✅ How to create an Airtable database  
✅ How to set up n8n workflows  
✅ How to create forms that submit data  
✅ How to connect web forms to databases  
✅ Basic automation concepts  

## Next Steps (When You're Ready)

1. **Add more fields** to your form and Airtable
2. **Add email notifications** when forms are submitted
3. **Create conditional logic** (different actions for different types of submissions)
4. **Add file uploads** to your form
5. **Connect to other services** like Google Sheets, Slack, or email marketing tools

## Complete Checklist

- [ ] Created Airtable account and base
- [ ] Set up table with correct fields
- [ ] Got API credentials
- [ ] Created n8n account
- [ ] Built the workflow with webhook → Airtable → response
- [ ] Created HTML form
- [ ] Updated webhook URL in form
- [ ] Tested the complete flow
- [ ] Verified data appears in Airtable

**Congratulations! You've built your first automated form-to-database system! 🎉**

---

*This is a foundational project that teaches you the basics of no-code automation. Once you master this, you can build much more complex systems!*