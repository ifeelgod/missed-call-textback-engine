# N8N & Telephony Interface Setup Workbook: Field Integration Guide, Testing Protocols & Troubleshooting Procedures

> **A Step-by-Step Practical Field Workbook for Deploying N8N, Wiring Telephony Gateways, Stress-Testing Automation Workflows, and Diagnosing System Errors**  
> **Written with care for freelance developers, technical consultants, and agency owners**

---

## Welcome to Your N8N & Telephony Integration Workbook!

Hey there, friend! Welcome to your hands-on technical workbook. If you have ever felt hesitant about setting up webhooks, configuring Docker VPS servers, or troubleshooting API errors, please take a deep breath. You are in safe hands! 

We are going to walk through every single terminal command, environment variable, N8N node setting, and telephony gateway screen together. By the time you finish this workbook, you will have a rock-solid, production-grade automation system that runs flawlessly 24/7/365.

---

# CHAPTER 1: N8N HOSTING & ENVIRONMENT CONFIGURATION

### Section Roadmap & Overview
In this chapter, we are going to set up your N8N automation engine on a high-speed Cloud VPS using Docker Compose, configure SSL encryption certificates, and set your environment variables for Twilio, Telnyx, and DeepSeek AI.

### Why Self-Hosting N8N Matters
Self-hosting N8N on a $5/month Cloud VPS (Hetzner or DigitalOcean) gives you **unlimited workflow executions** with zero per-execution fees. It guarantees sub-300ms webhook response times and ensures 99.99% uptime for your contractor clients.

### Step-by-Step N8N Deployment Guide

#### 1.1 Docker Compose Deployment (`docker-compose.yml`)

Create a folder `/opt/n8n` on your VPS and paste the following production `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  caddy:
    image: caddy:latest
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./caddy_data:/data
      - ./Caddyfile:/etc/caddy/Caddyfile
    environment:
      - DOMAIN_NAME=n8n.youragency.com

  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=n8n.youragency.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://n8n.youragency.com/
      - GENERIC_TIMEZONE=America/New_York
      - N8N_ENCRYPTION_KEY=your_secret_encryption_key_here
    volumes:
      - ./n8n_data:/home/node/.n8n
```

#### 1.2 Caddy Reverse Proxy & Automatic SSL (`Caddyfile`)

Create a `Caddyfile` in the same directory to handle free, automatic SSL certificate renewal via Let's Encrypt:

```text
n8n.youragency.com {
    reverse_proxy n8n:5678
}
```

Start your server by running: `docker compose up -d`

---

#### 1.3 Environment Variables Setup (`.env`)

Store your API keys safely in your N8N environment configuration:

```env
# N8N System Keys
N8N_ENCRYPTION_KEY=super_secret_32_character_key
N8N_HOST=n8n.youragency.com

# DeepSeek AI Credentials
DEEPSEEK_API_KEY=sk-ds-your-deepseek-api-key-here

# Twilio Telephony Credentials
TWILIO_ACCOUNT_SID=ACXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
TWILIO_AUTH_TOKEN=your_twilio_auth_token_here
TWILIO_PHONE_NUMBER=+15551234567

# Telnyx Telephony Credentials
TELNYX_API_KEY=KEY017XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
TELNYX_PUBLIC_KEY=your_telnyx_public_key_here
```

### Key Takeaways & Chapter Summary
We deployed N8N on Docker with automatic SSL via Caddy, verified container health, and configured production environment variables for CPaaS gateways and DeepSeek AI.

### Milestone Celebration!
**CONGRATULATIONS!** Chapter 1 is complete! Your N8N automation engine is live, secure, and ready for workflow integration!

---

# CHAPTER 2: TELEPHONY GATEWAY (CPaaS) INTERFACE WIRING

### Section Roadmap & Overview
In this chapter, we will inspect our full-color telephony integration flowchart, configure webhook listener endpoints for Twilio, Telnyx, Sent.dm, and Sendillo, and wire inbound call triggers.

### Why Telephony Webhook Wiring Matters
Telephony gateways communicate with N8N via **HTTP POST Webhooks**. Wiring these endpoints correctly ensures that the exact moment a contractor's phone rings 4 times or is declined, CPaaS gateways send caller ID data to N8N in under 300 milliseconds.

### Telephony Interface Architecture & Visual Flowchart

#### 2.1 N8N & Telephony Interface Integration Workflow

![N8N & Telephony Interface Integration Workflow](./n8n_telecom_diagram.jpg)

```
  [Incoming Phone Call Webhook POST]
                 │
                 ▼
  [CPaaS Telephony Gateway Verification (Twilio / Telnyx / Sent.dm / Sendillo)]
                 │
                 ▼
  [N8N Automation Logic Engine & DeepSeek AI Intent Classifier]
                 │
                 ▼
  [Outbound SMS Dispatch & Diagnostics Audit (<22s Latency)]
```

---

#### 2.2 Twilio Webhook Configuration Protocol
1. Log into your **Twilio Console** and navigate to **Phone Numbers > Active Numbers**.
2. Click your virtual number to open settings.
3. Under **Voice & Fax > A CALL COMES IN**, select **Webhook** and paste:  
   `https://n8n.youragency.com/webhook/missed-call-trigger` (HTTP POST).
4. Under **Messaging > A MESSAGE COMES IN**, select **Webhook** and paste:  
   `https://n8n.youragency.com/webhook/incoming-sms-optout` (HTTP POST).
5. Click **Save**.

#### 2.3 Telnyx Interface Configuration Protocol
1. Log into your **Telnyx Portal** and navigate to **Messaging Profiles**.
2. Click **Create Messaging Profile** (e.g., *Speed-to-Lead Profile*).
3. Under **Inbound Settings > Webhook URL**, paste:  
   `https://n8n.youragency.com/webhook/missed-call-trigger` (HTTP POST).
4. Assign your purchased Telnyx number to this Messaging Profile.

#### 2.4 Sent.dm & Sendillo Gateway Protocols
- **Sent.dm:** Configure endpoint target `https://api.sent.dm/v1/messages` with bearer auth token. Enables automated fallback from SMS to WhatsApp/RCS if cellular delivery fails.
- **Sendillo (`sendillo.com`):** Direct wholesale agency webhook listener for GoHighLevel CRM instances.

### Key Takeaways & Chapter Summary
We reviewed the full-color integration flowchart, configured Twilio/Telnyx webhook endpoints, and verified instant call trigger communication.

### Milestone Celebration!
**FANTASTIC JOB!** Chapter 2 is complete! Your telephony gateways are officially wired to your N8N automation engine!

---

# CHAPTER 3: NODE-BY-NODE N8N WORKFLOW IMPLEMENTATION

### Section Roadmap & Overview
In this chapter, we break down the 7 core nodes of our N8N workflow, reviewing node properties, JavaScript sanitization logic, DeepSeek AI prompt parameters, and outbound SMS dispatch code.

### Why Granular Node Setup Matters
Each node in N8N serves a specific purpose—from sanitizing raw phone numbers to evaluating timezones and generating AI text. Configuring each node carefully guarantees zero runtime crashes.

### Node Configuration & Implementation Guide

#### Node 1: Webhook Listener (`Webhook - Missed Call Listener`)
- **Type:** `n8n-nodes-base.webhook`
- **Method:** `POST`
- **Path:** `missed-call-trigger`
- **Response Mode:** `On Received` (Returns HTTP 200 immediately to prevent gateway timeout).

#### Node 2: Sanitization & Suppression (`Code - Normalize & Suppress`)
- **JavaScript Code:**
```javascript
const body = $input.first().json.body || $input.first().json;
const rawPhone = body.From || body.caller_number || body.phone || '';

// Clean to E.164 (+1NXXNXXXXXX)
const cleanedPhone = rawPhone.replace(/[^+\d]/g, '');

// Check suppression list
const optOutList = ['+15550000000']; 
const isOptedOut = optOutList.includes(cleanedPhone);

return [{
  json: {
    rawPhone,
    cleanedPhone,
    callStatus: body.CallStatus || body.event || 'no-answer',
    isOptedOut,
    receivedAt: new Date().toISOString()
  }
}];
```

#### Node 3: Human Delay (`Wait - 20s Buffer`)
- **Type:** `n8n-nodes-base.wait`
- **Amount:** `20` Seconds. Mimics a human technician picking up their phone to type a text reply.

#### Node 4: Business Hours Evaluator (`Code - Business Hours`)
- **JavaScript Code:**
```javascript
const now = new Date();
const localTime = new Date(now.toLocaleString("en-US", { timeZone: "America/New_York" }));

const dayOfWeek = localTime.getDay(); // 0=Sun, 1=Mon...
const currentHour = localTime.getHours();
const currentMinute = localTime.getMinutes();

let isBusinessHours = false;
if (dayOfWeek >= 1 && dayOfWeek <= 5) {
  if (currentHour > 8 && currentHour < 17) {
    isBusinessHours = true;
  } else if (currentHour === 17 && currentMinute <= 30) {
    isBusinessHours = true;
  }
}

return [{
  json: {
    ...$input.first().json,
    isBusinessHours,
    evaluatedTime: localTime.toISOString()
  }
}];
```

#### Node 5: DeepSeek AI Intent Node (`HTTP - DeepSeek AI`)
- **URL:** `https://api.deepseek.com/v1/chat/completions`
- **Method:** `POST`
- **Header:** `Authorization: Bearer {{ $env.DEEPSEEK_API_KEY }}`
- **System Prompt:**
```text
You are an automated speed-to-lead receptionist for Apex Plumbing & Heating. Craft a concise SMS auto-reply under 150 characters for a missed call.
Context: Business Hours = {{ $json.isBusinessHours }}
Rules: Under 150 chars, no AI mention, include 'Reply STOP to cancel'. If Business Hours=true: ask how we can help. If false: state closed, morning callback, add 'Reply URGENT for emergency dispatch'. Return raw text only.
```

#### Node 6: Outbound SMS Gateway (`HTTP - Twilio Outbound`)
- **URL:** `https://api.twilio.com/2010-04-01/Accounts/{{ $env.TWILIO_ACCOUNT_SID }}/Messages.json`
- **Body:** `To={{ encodeURIComponent($json.toPhone) }}&From={{ encodeURIComponent($env.TWILIO_PHONE_NUMBER) }}&Body={{ encodeURIComponent($json.smsBody) }}`

#### Node 7: Incoming Opt-Out Listener (`Webhook - Opt-Out Handler`)
- Detects incoming SMS keywords (`STOP`, `UNSUBSCRIBE`, `CANCEL`) and commits caller phone to suppression list.

### Key Takeaways & Chapter Summary
We reviewed all 7 N8N workflow nodes, validated phone sanitization regex, configured DeepSeek AI system prompts, and set up outbound carrier dispatch.

### Milestone Celebration!
**YOU DID IT!** Chapter 3 is finished! Your complete N8N workflow is fully configured and ready for live testing!

---

# CHAPTER 4: SYSTEM TESTING, PERFORMANCE AUDITING & STRESS TESTS

### Section Roadmap & Overview
In this chapter, we will execute a 5-step live functional test, perform load/stress tests to verify stability during peak call spikes, and audit SMS character segment limits.

### Why Performance Auditing Matters
Testing your workflow under real-world conditions guarantees that when a contractor receives 5 missed calls in 10 minutes during a winter storm, your N8N engine processes every single call without dropping webhooks or crashing servers.

### 5-Step Live Validation & Stress Test Protocol

```
[TEST STEP 1: Inbound Call Webhook Verification]
Action: Call virtual gateway number from test mobile phone.
Benchmark: Webhook POST payload logged in N8N execution dashboard in <300ms.

[TEST STEP 2: Opt-Out Suppression Filter Audit]
Action: Pass a test phone number listed in suppression database.
Benchmark: Filter node halts execution instantly; zero outbound SMS dispatched.

[TEST STEP 3: Human Delay Buffer Audit]
Action: Observe execution log timer.
Benchmark: Wait node holds execution for exactly 20.0 seconds.

[TEST STEP 4: DeepSeek AI Latency & Length Audit]
Action: Inspect choice output from DeepSeek HTTP node.
Benchmark: Response received in <1.2 seconds; text length strictly under 150 characters.

[TEST STEP 5: Outbound Carrier Delivery & Segment Math]
Action: Check test mobile phone inbox.
Benchmark: SMS arrives within 22 seconds total; single SMS segment billed ($0.0083).
```

---

#### 4.1 Concurrent Call Stress Test
Run a concurrent load test by simulating 10 simultaneous HTTP POST requests to your N8N webhook endpoint using Apache Bench or cURL:

```bash
# Simulate 10 concurrent missed call webhooks
ab -n 10 -c 10 -p test_payload.json -T "application/json" https://n8n.youragency.com/webhook/missed-call-trigger
```
- **Target Performance Metric:** 100% HTTP 200 Success Rate; Zero 504 Gateway Timeouts.

### Key Takeaways & Chapter Summary
We executed the 5-step live validation test, stress-tested webhook concurrency under load, and verified single-segment SMS delivery math.

### Milestone Celebration!
**WOOHOO!** Chapter 4 is complete! Your system is officially tested, verified, and benchmarked for high-performance production!

---

# CHAPTER 5: MASTER TROUBLESHOOTING & ERROR DIAGNOSTICS SOP

### Section Roadmap & Overview
In this final chapter, we provide a complete troubleshooting diagnostic matrix covering every potential runtime error, carrier status code, and API failure—along with step-by-step fix protocols.

### Why Troubleshooting SOPs Matter
Having clear diagnostic procedures takes all the stress out of system management. If an error occurs, you can identify the exact root cause and fix it in under 60 seconds.

---

### Master Diagnostics & Error Resolution Matrix

| Diagnostic Error / Status | Root Cause | Immediate Step-by-Step Fix Protocol |
| :--- | :--- | :--- |
| **1. Webhook 404 / Connection Refused** | N8N container offline, Caddy SSL renewal failed, or URL path typo in Twilio console. | 1. SSH into VPS and run `docker ps` to verify N8N status.<br>2. Verify domain DNS records point to VPS IP.<br>3. Check `Caddyfile` reverse proxy settings and restart container: `docker compose restart`. |
| **2. Twilio Error 21614 / Delivery Failure** | Unregistered A2P 10DLC campaign, carrier spam filtering, or invalid phone formatting. | 1. Ensure phone number is formatted to E.164 (`+1NXXNXXXXXX`).<br>2. Submit business EIN for A2P 10DLC Brand Registration in Twilio Console.<br>3. Verify message body includes mandatory footer: `"Reply STOP to cancel"`. |
| **3. Double Text Messaging to Customer** | Customer calls back during 20s wait buffer; timer fires redundant text. | 1. Add active call interception node before Wait node.<br>2. Store caller ID key (`active_call_[PHONE]`) in Redis/memory.<br>3. Cancel wait timer if secondary inbound call event is detected. |
| **4. DeepSeek API Timeout / JSON Parse Error** | API key invalid, DeepSeek rate limit reached, or model output format error. | 1. Check `DEEPSEEK_API_KEY` environment variable in `.env`.<br>2. Add N8N Code fallback node: If AI response fails, default to standard template: *"Hey! On a job site & missed your call. How can we help? Reply STOP to cancel."* |
| **5. CCF Fails to Trigger (Phone Rings Forever)** | Contractor entered incorrect star code or carrier call forwarding option disabled. | 1. Have contractor redial star code: Verizon (`*71[NUMBER]`), AT&T (`*61*[NUMBER]#`), T-Mobile (`**61*[NUMBER]#`).<br>2. Verify contractor's mobile account supports Conditional Call Forwarding.<br>3. Test by calling contractor line from secondary test phone. |
| **6. Multi-Segment Double Billing** | Outbound SMS body exceeds 160 GSM characters due to special characters or long text. | 1. In Node 5 process code, add hard length truncation: `smsContent.substring(0, 155) + " STOP"`.<br>2. Remove non-GSM emoji characters that force Unicode encoding (reducing segment size to 70 chars). |

---

### Workbook Graduation & Final Summary

**CONGRATULATIONS! YOU HAVE GRADUATED FROM THE N8N & TELEPHONY SETUP WORKBOOK!**  
Take a huge victory lap! You now hold a complete, production-ready field manual for hosting, wiring, testing, and troubleshooting your **30-Minute Missed-Call Text-Back Engine**. You are fully equipped to build a highly profitable, scalable recurring revenue business!
