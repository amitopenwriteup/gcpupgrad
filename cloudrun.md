# Cloud Run Fundamentals and Advanced Features
## 2-Hour Hands-On Workshop — Google Cloud Console (UI)

---

Duration: 2 Hours
Level: Beginner to Intermediate
Prerequisites: A Google Cloud account with billing enabled and an existing project
What you'll build: A microservices application deployed on Cloud Run with Pub/Sub event triggers, Secret Manager integration, and Cloud Tasks background processing

---

## Workshop Agenda

| Time | Module |
|------|--------|
| 0:00 – 0:10 | Setup: Enable APIs and open Cloud Shell |
| 0:10 – 0:35 | Module 1: Deploy Your First Cloud Run Service |
| 0:35 – 0:55 | Module 2: Scaling, Concurrency and Custom Domains |
| 0:55 – 1:15 | Module 3: VPC Integration, Secrets and Environment Variables |
| 1:15 – 1:35 | Module 4: Event-Driven Architecture with Pub/Sub |
| 1:35 – 1:55 | Module 5: Background Processing with Cloud Tasks |
| 1:55 – 2:00 | Cleanup and Summary |

---

## Setup: Enable APIs and Open Cloud Shell (0:00 – 0:10)

### Step 1 — Enable Required APIs

1. Go to [https://console.cloud.google.com](https://console.cloud.google.com) and select your project
2. In the left sidebar, click **APIs & Services > Library**
3. Search for and **Enable** each of the following APIs (click the API, then click **Enable**):
   - `Cloud Run API`
   - `Cloud Build API`
   - `Pub/Sub API`
   - `Cloud Tasks API`
   - `Secret Manager API`
   - `Artifact Registry API`

Tip: You can open each API in a new browser tab to enable them simultaneously.

### Step 2 — Open Cloud Shell

1. Click the **Cloud Shell icon** (terminal icon `>_`) in the top-right toolbar
2. Click **Authorize** if prompted
3. Wait for the shell to initialize — you will see a terminal at the bottom of the screen

---

## Module 1: Deploy Your First Cloud Run Service (0:10 – 0:35)

### Step 3 — Create a Simple Web Application

In Cloud Shell, run the following commands one by one:

```bash
# Create a working directory
mkdir ~/cloudrun-workshop && cd ~/cloudrun-workshop
```

```bash
# Create a simple Node.js web app
cat > index.js << 'EOF'
const express = require('express');
const app = express();
app.use(express.json());

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Cloud Run!',
    service: process.env.SERVICE_NAME || 'hello-service',
    version: '1.0.0'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
EOF
```

```bash
# Create package.json
cat > package.json << 'EOF'
{
  "name": "cloudrun-demo",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "express": "^4.18.0"
  },
  "scripts": { "start": "node index.js" }
}
EOF
```

```bash
# Create a Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-slim
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 8080
CMD ["node", "index.js"]
EOF
```

### Step 4 — Build and Push the Container Image

```bash
# Set your project ID variable
export PROJECT_ID=$(gcloud config get-value project)
echo "Project ID: $PROJECT_ID"
```

```bash
# Create an Artifact Registry repository
gcloud artifacts repositories create cloudrun-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Cloud Run Workshop Repository"
```

```bash
# Build and push the image using Cloud Build
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/hello-service:v1
```

This step takes about 1-2 minutes. You will see build logs streaming in the terminal.

### Step 5 — Deploy to Cloud Run via the Console

1. In the GCP Console, go to **Cloud Run** (use the left sidebar or search bar)
2. Click **Create Service**
3. Select **Deploy one revision from an existing container image**
4. Click **Select** next to the container image field
5. Navigate: **Artifact Registry > cloudrun-repo > hello-service > v1**
6. Click **Select**

Configure the service with the following values:

| Field | Value |
|-------|-------|
| Service name | `hello-service` |
| Region | `us-central1` |
| CPU allocation | `CPU is only allocated during request processing` |
| Minimum instances | `0` |
| Maximum instances | `10` |
| Ingress | `Allow all traffic` |
| Authentication | `Allow unauthenticated invocations` |

7. Click **Create** and wait for the deployment (about 1 minute)
8. Once deployed, click the **URL** shown at the top — you should see a JSON response

---

## Module 2: Scaling, Concurrency and Custom Domains (0:35 – 0:55)

### Step 6 — Explore Service Configuration in the Console

1. In **Cloud Run**, click your `hello-service`
2. Click **Edit & Deploy New Revision**
3. Open the **Capacity** section and set:
   - **Memory:** `512Mi`
   - **CPU:** `1`
   - **Request timeout:** `300` seconds
   - **Maximum concurrent requests per instance:** `80`

Note on concurrency: when set to 80, each container instance handles up to 80 simultaneous requests. Cloud Run scales out new instances automatically when demand exceeds this number.

4. Expand **Autoscaling** and set:
   - **Minimum number of instances:** `1` (keeps one warm instance to avoid cold starts)
   - **Maximum number of instances:** `100`

5. Click **Deploy** to apply the changes

### Step 7 — View Autoscaling in Action

1. Click your service and go to the **Metrics** tab
2. You will see graphs for request count, instance count, and request latency
3. In Cloud Shell, generate some load:

```bash
# Get your service URL
SERVICE_URL=$(gcloud run services describe hello-service \
  --region=us-central1 \
  --format='value(status.url)')

# Send 50 requests to trigger scaling
for i in $(seq 1 50); do curl -s $SERVICE_URL > /dev/null & done
wait
echo "Done! Check the Metrics tab in the console."
```

4. Refresh the **Metrics** tab and watch the instance count increase, then scale back down

### Step 8 — Configure a Custom Domain (Optional)

1. In Cloud Run, click **Manage Custom Domains** at the top of the Cloud Run page
2. Click **Add Mapping**
3. Select your service: `hello-service`
4. Enter your domain (e.g., `hello.yourdomain.com`)
5. Follow the DNS verification steps shown — GCP provides the exact DNS records to add
6. SSL certificates are automatically provisioned and renewed by Google via managed certificates — no manual certificate management is required

---

## Module 3: VPC Integration, Secrets and Environment Variables (0:55 – 1:15)

### Step 9 — Create a Secret in Secret Manager

1. Go to **Security > Secret Manager** in the left sidebar (or search "Secret Manager")
2. Click **Create Secret** and fill in:
   - **Name:** `db-password`
   - **Secret value:** `my-super-secret-password-123`
3. Leave all other settings as default
4. Click **Create Secret**

### Step 10 — Attach the Secret to Cloud Run

1. Go to **Cloud Run > hello-service**
2. Click **Edit & Deploy New Revision**
3. Click the **Variables & Secrets** tab
4. Under **Secrets**, click **Reference a Secret** and configure:
   - **Secret:** `db-password`
   - **Reference method:** `Exposed as environment variable`
   - **Variable name:** `DB_PASSWORD`
   - **Version:** `latest`
5. Click **Done**

Add a regular environment variable:

6. Under **Environment variables**, click **Add Variable** and set:
   - **Name:** `SERVICE_NAME`
   - **Value:** `hello-service-prod`
7. Click **Deploy**

### Step 11 — Grant Secret Manager Access to Cloud Run

1. Go to **IAM & Admin > IAM**
2. Find the Cloud Run service account (it looks like `[PROJECT_NUMBER]-compute@developer.gserviceaccount.com`)
3. Click the pencil icon to edit it
4. Click **Add Another Role**
5. Search for and add: **Secret Manager Secret Accessor**
6. Click **Save**

### Step 12 — Update the App to Use Secrets

```bash
cd ~/cloudrun-workshop

cat > index.js << 'EOF'
const express = require('express');
const app = express();
app.use(express.json());

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Cloud Run!',
    service: process.env.SERVICE_NAME || 'hello-service',
    secretLoaded: !!process.env.DB_PASSWORD,
    version: '2.0.0'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', uptime: process.uptime() });
});

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
EOF
```

```bash
# Build and deploy the updated image
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/hello-service:v2

gcloud run deploy hello-service \
  --image us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/hello-service:v2 \
  --region us-central1
```

### Step 13 — Configure a VPC Connector

1. Go to **VPC network > Serverless VPC Access**
2. Click **Create Connector** and fill in:
   - **Name:** `workshop-connector`
   - **Region:** `us-central1`
   - **Network:** `default`
   - **IP range:** `10.8.0.0/28`
3. Click **Create** (takes about 1 minute)

Attach the VPC connector to Cloud Run:

4. Go to **Cloud Run > hello-service > Edit & Deploy New Revision**
5. Click the **Connections** tab
6. Under **VPC**, select **Connect to a VPC for outbound traffic**
7. Select `workshop-connector`
8. Routing: select **Route only requests to private IPs through the VPC**
9. Click **Deploy**

This allows your Cloud Run service to securely access resources inside your VPC such as Cloud SQL, Redis, or internal services without exposing them to the public internet.

---

## Module 4: Event-Driven Architecture with Pub/Sub (1:15 – 1:35)

### Step 14 — Create a Pub/Sub Topic

1. Go to **Pub/Sub** in the left sidebar
2. Click **Create Topic** and fill in:
   - **Topic ID:** `order-events`
3. Leave **Add a default subscription** checked
4. Click **Create**

### Step 15 — Build a Pub/Sub Subscriber Service

```bash
cd ~/cloudrun-workshop
mkdir pubsub-service && cd pubsub-service

cat > index.js << 'EOF'
const express = require('express');
const app = express();
app.use(express.json());

app.post('/pubsub/push', (req, res) => {
  const message = req.body?.message;

  if (!message) {
    console.log('No message received');
    return res.status(400).send('No message');
  }

  // Pub/Sub messages are base64-encoded
  const data = message.data
    ? Buffer.from(message.data, 'base64').toString('utf-8')
    : 'No data';

  console.log(`Received event: ${data}`);
  console.log(`Message ID: ${message.messageId}`);
  console.log(`Publish time: ${message.publishTime}`);

  // Return 200 to acknowledge the message (prevents redelivery)
  res.status(200).send('Message processed');
});

app.get('/health', (req, res) => res.json({ status: 'ok' }));

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => console.log(`Pub/Sub service on port ${PORT}`));
EOF

cat > package.json << 'EOF'
{
  "name": "pubsub-service",
  "version": "1.0.0",
  "dependencies": { "express": "^4.18.0" },
  "scripts": { "start": "node index.js" }
}
EOF

cp ../Dockerfile .
```

```bash
# Build and deploy the Pub/Sub service
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/pubsub-service:v1

gcloud run deploy pubsub-service \
  --image us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/pubsub-service:v1 \
  --region us-central1 \
  --no-allow-unauthenticated
```

### Step 16 — Create a Push Subscription

1. Go to **Pub/Sub > Subscriptions**
2. Click **Create Subscription** and fill in:
   - **Subscription ID:** `order-events-push`
   - **Topic:** `order-events`
   - **Delivery type:** `Push`
3. Get your service URL from Cloud Shell:

```bash
PUBSUB_URL=$(gcloud run services describe pubsub-service \
  --region=us-central1 \
  --format='value(status.url)')
echo $PUBSUB_URL
```

4. **Endpoint URL:** Paste your service URL followed by `/pubsub/push`
   - Example: `https://pubsub-service-xxx-uc.a.run.app/pubsub/push`
5. Check **Enable authentication**
6. Set **Acknowledgement deadline:** `60 seconds`
7. Click **Create**

### Step 17 — Grant Pub/Sub Permissions

```bash
# Create a service account for Pub/Sub to invoke Cloud Run
gcloud iam service-accounts create pubsub-invoker \
  --display-name="Pub/Sub Cloud Run Invoker"

# Grant it permission to invoke the pubsub-service
gcloud run services add-iam-policy-binding pubsub-service \
  --region=us-central1 \
  --member="serviceAccount:pubsub-invoker@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/run.invoker"
```

Update the subscription to use the service account:

1. Go to **Pub/Sub > Subscriptions > order-events-push**
2. Click **Edit**
3. Under authentication, set the service account to `pubsub-invoker@[PROJECT_ID].iam.gserviceaccount.com`
4. Click **Update**

### Step 18 — Test the Event Flow

```bash
# Publish a test message
gcloud pubsub topics publish order-events \
  --message='{"orderId": "ORD-001", "product": "Widget", "quantity": 5}'
```

View the logs:

1. Go to **Cloud Run > pubsub-service**
2. Click the **Logs** tab
3. You should see: `Received event: {"orderId": "ORD-001", ...}`

---

## Module 5: Background Processing with Cloud Tasks (1:35 – 1:55)

### Step 19 — Create a Cloud Tasks Queue

1. Go to **Cloud Tasks** in the left sidebar
2. Click **Create Queue** and fill in:
   - **Queue name:** `order-processing-queue`
   - **Region:** `us-central1`
3. Configure rate limits:
   - **Maximum dispatches per second:** `100`
   - **Maximum concurrent dispatches:** `10`
4. Configure retry settings:
   - **Maximum attempts:** `5`
   - **Minimum backoff:** `10s`
   - **Maximum backoff:** `300s`
5. Click **Create Queue**

### Step 20 — Build a Task Worker Service

```bash
cd ~/cloudrun-workshop
mkdir task-worker && cd task-worker

cat > index.js << 'EOF'
const express = require('express');
const app = express();
app.use(express.json());

app.post('/tasks/process-order', async (req, res) => {
  const task = req.body;
  const taskName = req.headers['x-cloudtasks-taskname'];
  const retryCount = req.headers['x-cloudtasks-taskretrycount'];

  console.log(`Processing task: ${taskName} (attempt ${retryCount})`);
  console.log(`Order data: ${JSON.stringify(task)}`);

  // Simulate order processing
  await new Promise(resolve => setTimeout(resolve, 500));

  // Simulate occasional failures for retry demo (5% chance)
  if (Math.random() < 0.05) {
    console.error('Simulated processing error!');
    return res.status(500).send('Processing failed');
  }

  console.log(`Order ${task.orderId} processed successfully!`);
  res.status(200).json({ processed: true, orderId: task.orderId });
});

app.get('/health', (req, res) => res.json({ status: 'ok' }));

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => console.log(`Task worker on port ${PORT}`));
EOF

cat > package.json << 'EOF'
{
  "name": "task-worker",
  "version": "1.0.0",
  "dependencies": { "express": "^4.18.0" },
  "scripts": { "start": "node index.js" }
}
EOF

cp ../Dockerfile .
```

```bash
# Build and deploy the task worker
gcloud builds submit --tag us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/task-worker:v1

gcloud run deploy task-worker \
  --image us-central1-docker.pkg.dev/$PROJECT_ID/cloudrun-repo/task-worker:v1 \
  --region us-central1 \
  --no-allow-unauthenticated
```

### Step 21 — Grant Cloud Tasks Permission to Invoke Cloud Run

```bash
# Create a service account for Cloud Tasks
gcloud iam service-accounts create cloudtasks-invoker \
  --display-name="Cloud Tasks Cloud Run Invoker"

# Grant it permission to invoke the task-worker service
gcloud run services add-iam-policy-binding task-worker \
  --region=us-central1 \
  --member="serviceAccount:cloudtasks-invoker@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/run.invoker"
```

### Step 22 — Create and Dispatch Tasks

```bash
# Get the task worker URL
WORKER_URL=$(gcloud run services describe task-worker \
  --region=us-central1 \
  --format='value(status.url)')

# Create a single task in the queue
gcloud tasks create-http-task \
  --queue=order-processing-queue \
  --url=$WORKER_URL/tasks/process-order \
  --method=POST \
  --header="Content-Type:application/json" \
  --body-content='{"orderId":"ORD-101","product":"Gadget","amount":99.99}' \
  --location=us-central1 \
  --oidc-service-account-email=cloudtasks-invoker@$PROJECT_ID.iam.gserviceaccount.com
```

```bash
# Create multiple tasks to simulate a queue burst
for i in {101..110}; do
  gcloud tasks create-http-task \
    --queue=order-processing-queue \
    --url=$WORKER_URL/tasks/process-order \
    --method=POST \
    --header="Content-Type:application/json" \
    --body-content="{\"orderId\":\"ORD-$i\",\"product\":\"Item-$i\"}" \
    --location=us-central1 \
    --oidc-service-account-email=cloudtasks-invoker@$PROJECT_ID.iam.gserviceaccount.com
  echo "Queued order ORD-$i"
done
```

### Step 23 — Monitor the Queue and Worker

Monitor the queue:

1. Go to **Cloud Tasks > order-processing-queue**
2. Watch the **Tasks dispatched** and **Tasks completed** counters update in real time
3. Click **View task details** on any pending task to inspect its headers and body

Monitor the worker:

1. Go to **Cloud Run > task-worker**
2. Click the **Logs** tab to see log entries for each processed order
3. Check the **Metrics** tab to see request count and instance scaling

---

## Architecture Summary

Here is what you built in this workshop:

```
Internet
    |
    v
[hello-service]          <- Public-facing API with secrets + VPC
    |
    +-- Secret Manager   <- DB credentials injected at runtime
    +-- VPC Connector    <- Private resource access
    |
[Pub/Sub Topic: order-events]
    |
    v
[pubsub-service]         <- Event-driven push subscriber
    |
    v
[Cloud Tasks Queue: order-processing-queue]
    |
    v
[task-worker]            <- Async background processor
```

---

## Cleanup (1:55 – 2:00)

To avoid charges, delete all resources created during this workshop:

```bash
# Delete Cloud Run services
gcloud run services delete hello-service --region=us-central1 --quiet
gcloud run services delete pubsub-service --region=us-central1 --quiet
gcloud run services delete task-worker --region=us-central1 --quiet

# Delete Cloud Tasks queue
gcloud tasks queues delete order-processing-queue --location=us-central1 --quiet

# Delete Pub/Sub topic and subscription
gcloud pubsub subscriptions delete order-events-push --quiet
gcloud pubsub topics delete order-events --quiet

# Delete secrets
gcloud secrets delete db-password --quiet

# Delete Artifact Registry repository
gcloud artifacts repositories delete cloudrun-repo \
  --location=us-central1 --quiet

# Delete service accounts
gcloud iam service-accounts delete pubsub-invoker@$PROJECT_ID.iam.gserviceaccount.com --quiet
gcloud iam service-accounts delete cloudtasks-invoker@$PROJECT_ID.iam.gserviceaccount.com --quiet
```

Alternatively, delete the entire project from: **IAM & Admin > Manage Resources > Select project > Delete**

---

## Key Takeaways

**Cloud Run Fundamentals**
- Cloud Run runs stateless containers and scales from zero automatically
- Concurrency controls the number of simultaneous requests per instance
- Setting minimum instances above 0 eliminates cold starts but incurs constant billing

**Security Best Practices**
- Always use `--no-allow-unauthenticated` for internal services
- Inject secrets via Secret Manager — never hardcode them in code or images
- Use dedicated service accounts with least-privilege IAM roles

**Integration Patterns**
- Pub/Sub push — real-time event processing triggered by external events
- Cloud Tasks — controlled async processing with rate limiting, retries, and scheduling
- VPC Connector — bridge between serverless Cloud Run and private VPC resources

**Cost Optimization**
- Set min-instances to 0 for dev and test workloads
- Use CPU-only-during-requests allocation for intermittent traffic patterns
- Monitor idle instance time in Cloud Monitoring

---

## Further Reading

- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Cloud Run Pricing](https://cloud.google.com/run/pricing)
- [Pub/Sub Push Subscriptions](https://cloud.google.com/pubsub/docs/push)
- [Cloud Tasks HTTP Targets](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
- [Secret Manager Best Practices](https://cloud.google.com/secret-manager/docs/best-practices)
- [Serverless VPC Access](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access)

---

Workshop — Google Cloud Run 2-Hour Hands-On Lab
All steps performed via GCP Console UI and Cloud Shell
