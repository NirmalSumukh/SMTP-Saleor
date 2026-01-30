# Saleor SMTP App - Setup & Usage Guide

## 1. How the App Works
This is a **Saleor App** designed to handle email notifications for your e-commerce store.

### Core Functionality
- **Event Listener**: It listens for webhooks from Saleor (e.g., `ORDER_CREATED`, `ORDER_FULFILLED`).
- **SMTP Gateway**: When an event occurs, it generates an email using a template (Handlebars) and sends it via your configured SMTP server.
- **Configuration**: Unlike hardcoded settings, this app allows you to configure different SMTP providers for different events directly from the Saleor Dashboard.

### Architecture
- **Frontend**: A React/Next.js interface that loads inside the Saleor Dashboard (via iframe). This is where you configure SMTP settings and email templates.
- **Backend**: Next.js API routes that handle:
  - `POST /api/webhooks/*`: Receiving event payloads from Saleor.
  - `POST /api/register`: Registering the app with Saleor.
  - `POST /trpc/*`: API for the frontend UI.

## 2. Prerequisites
To use this app, you need:
1.  **SMTP Server**: Credentials (Host, Port, User, Password) for an email provider (e.g., SendGrid, AWS SES, Gmail, Mailgun).
2.  **Public URL**: Since Saleor needs to send webhooks to this app, the app must be accessible via the internet (https).
    - **Local Development**: Use a tunnel like ngrok or Cloudflare Tunnel.
    - **Production (Hostinger)**: A domain name pointing to your VPS IP.
3.  **Saleor Permissions**: The app requires permissions to manage apps and read orders/users (handled automatically during installation).

## 3. Local Development (Port 3010)
I have configured the app to run on port **3010**.

### Steps to Run Locally
1.  **Configure Environment**:
    Ensure your `.env` file has the basic setup (already created).
2.  **Start the Server**:
    ```bash
    pnpm run dev
    ```
    The app will start at `http://localhost:3010`.
3.  **Expose to Internet** (Crucial for Saleor Connection):
    Use a tunnel to make your localhost accessible:
    ```bash
    # Example using ngrok
    ngrok http 3010
    ```
    Copy the https URL (e.g., `https://random-id.ngrok-free.app`).

## 4. Connecting to Saleor Dashboard
1.  **Go to Saleor Dashboard** > **Apps**.
2.  Click **"Install App"**.
3.  **App Manifest URL**: Enter your public URL + `/api/manifest`.
    - Local: `https://your-ngrok-id.ngrok-free.app/api/manifest`
    - Production: `https://smtp.yourdomain.com/api/manifest`
4.  Follow the prompts to install.
5.  Once installed, open the app from the Dashboard. You will see the configuration UI where you can add your SMTP details.

## 5. Deployment Guide (Hostinger VPS)
Since you have a Hostinger VPS (Ubuntu/Linux), using **Docker** is the most reliable way to run this standalone app.

### Option A: Docker (Recommended)
You can run this app as a container alongside your Saleor containers.

1.  **Prepare Dockerfile**: (I can create one for you if needed, or use the standard Node image).
2.  **Build & Run**:
    ```bash
    # Build the image
    docker build -t saleor-app-smtp .

    # Run the container
    docker run -d \
      -p 3010:3010 \
      -e SECRET_KEY=your_secure_key \
      -e APP_API_BASE_URL=https://smtp.yourdomain.com \
      --name saleor-app-smtp \
      saleor-app-smtp
    ```

### Option B: Node.js + PM2 (Direct on VPS)
If you don't want to use Docker:

1.  **Install Node.js 18+ & pnpm** on your VPS.
2.  **Upload Code**: Use SCP or Git to copy this `smtp` folder to your VPS.
3.  **Install & Build**:
    ```bash
    cd smtp
    pnpm install
    pnpm run build
    ```
4.  **Run with PM2**:
    ```bash
    npm install -g pm2
    pm2 start npm --name "saleor-smtp" -- start
    ```
5.  **Reverse Proxy (Traefik/Nginx)**:
    Configure Traefik (since you mentioned it) or Nginx to point a domain (e.g., `smtp.leemasmart.com`) to `http://localhost:3010`.

## 6. Important Notes
- **App Tunnel**: For the app to work within the Saleor Dashboard iframe, your browser must be able to load the app's URL. Mixed content (HTTP app in HTTPS dashboard) is blocked by browsers. **Always use HTTPS**.
- **Webhooks**: If the app is not receiving emails, check the "Webhooks" section in Saleor Dashboard to see if delivery failed.
