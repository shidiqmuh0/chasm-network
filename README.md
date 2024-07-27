# Chasm Network Node Setup Guide

This guide provides step-by-step instructions to set up and run a node for the Chasm Network, leveraging Docker and ngrok for hosting and networking. Please ensure your system meets the minimum requirements before proceeding.

## Minimum System Requirements
- **CPU:** 2 vCPU
- **Memory:** 4GB RAM
- **Storage:** 50GB SSD

## Step 1: Obtain API Key from Groq
1. Visit [Groq API Key](https://console.groq.com/keys).
2. Sign up or log in to your account.
3. Save the generated API Key for later use.

## Step 2: Mint SCOUT NFT
1. Visit [Chasm Private Mint](https://scout.chasm.net/private-mint).
2. Connect your wallet and mint the SCOUT NFT with a 0.025 $MNT gas fee.
3. After minting, click on the _mint(scout)_ button.
4. Copy the `SCOUT_UID` and `WEBHOOK_API_KEY` displayed. These are crucial for your node configuration.

## Step 3: Set Up ngrok
1. Visit [ngrok Signup](https://dashboard.ngrok.com/signup) and create an account.
2. After signing in, navigate to "Your Authtoken" in the dashboard.
3. Click on "Show Authtoken" and copy the token. This token will be used to authenticate your ngrok client.

## Step 4: Install Dependencies

1. **Update and Upgrade System Packages:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Remove Existing Docker Packages:**
   ```bash
   for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
   ```

3. **Install Required Packages:**
   ```bash
   sudo apt-get update
   sudo apt-get install ca-certificates curl
   ```

4. **Set Up Docker Repository:**
   ```bash
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **Install Docker:**
   ```bash
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

6. **Install Screen and ngrok:**
   ```bash
   sudo apt install screen -y
   curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list && sudo apt update && sudo apt install ngrok
   ```

## Step 5: Start ngrok
1. Open a new terminal session using `screen`:
   ```bash
   screen
   ```
2. Start ngrok for HTTP port 3001:
   ```bash
   ngrok http 3001
   ```
3. Copy the forwarding URL provided by ngrok (e.g., `https://1bXX.XX.XXX.XXX.ngrok-free.app`). This URL will be used as the `WEBHOOK_URL` in your environment configuration.

## Step 6: Configure Environment Variables
1. Create a `.env` file in your working directory:
   ```bash
   nano .env
   ```
2. Populate the `.env` file with the following content, replacing placeholder values with the actual values you obtained earlier:
   ```bash
   PORT=3001
   LOGGER_LEVEL=debug
   ORCHESTRATOR_URL=https://orchestrator.chasm.net
   SCOUT_NAME=YourNodeName
   SCOUT_UID=Your_SCOUT_UID
   WEBHOOK_API_KEY=Your_WEBHOOK_API_KEY
   WEBHOOK_URL=Your_ngrok_URL
   PROVIDERS=groq
   MODEL=gemma2-9b-it
   GROQ_API_KEY=Your_GROQ_API_KEY
   ```
3. Save and exit the file (press `Ctrl + X`, then `Y`, and `Enter`).

## Step 7: Open Firewall for Port 3001
```bash
sudo ufw allow 3001
```

## Step 8: Run the Scout Docker Container
1. Pull the latest Chasm Scout Docker image:
   ```bash
   docker pull chasmtech/chasm-scout:latest
   ```
2. Run the Docker container with your environment variables:
   ```bash
   docker run -d --restart=always --env-file ./.env -p 3001:3001 --name scout chasmtech/chasm-scout
   ```
3. Check the container logs to ensure it is running correctly:
   ```bash
   docker logs scout
   ```

## Step 9: Test the Setup
1. Verify the local server is responding:
   ```bash
   curl localhost:3001
   ```
2. Send a test request to your webhook URL:
   ```bash
   source ./.env
   curl -X POST \
        -H "Content-Type: application/json" \
        -H "Authorization: Bearer $WEBHOOK_API_KEY" \
        -d '{"body":"{\"model\":\"gemma2-9b-it\",\"messages\":[{\"role\":\"system\",\"content\":\"You are a helpful assistant.\"}]}"}' \
        $WEBHOOK_URL
   ```

## Step 10: Check Node Status
- Monitor your node's status on the [Chasm Scout Dashboard](https://scout.chasm.net/dashboard) after approximately 2 hours.

---

This concludes the setup process for your Chasm Network Node. Ensure all configurations are correctly set and your services are running smoothly. If you encounter any issues, refer to the official documentation or support channels for assistance.
