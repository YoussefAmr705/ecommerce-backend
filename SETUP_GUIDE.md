# Deployment Guide: Scalable Ecommerce Backend on AWS EC2 (Docker Swarm)

This guide will help you deploy the microservices architecture to a single AWS EC2 instance using Docker Swarm.

## 1. Infrastructure Requirements

### AWS EC2 Specification
- **Instance Type**: `t3.medium` (Recommended) or `t3.large`. 
  - *Why?* We are running ~14 containers (including replicas). A `t3.micro` will run out of memory.
- **OS**: Ubuntu 22.04 LTS.
- **Security Group Rules**:
  - **Port 22 (SSH)**: Allow from your IP.
  - **Port 80 (HTTP)**: Allow from anywhere (0.0.0.0/0).

### GitHub Secrets
Add the following to your GitHub Repository Secrets (`Settings` -> `Secrets and variables` -> `Actions`):
- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub access token.

---

## 2. EC2 Initial Setup

Once you have launched your EC2 instance, SSH into it and run the following commands to install Docker and initialize Swarm:

```bash
# Update and install Docker
sudo apt-get update
sudo apt-get install -y docker.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Allow your user to run docker without sudo
sudo usermod -aG docker $USER
# IMPORTANT: Log out and log back in for this to take effect!

# Initialize Docker Swarm (Single Node)
docker swarm init
```

---

## 3. Deployment Steps

### Step A: Prepare Environment Variables
Create a `.env` file on your EC2 instance in the folder where you will clone the project.

```bash
nano .env
```

Paste the following and fill in your values:
```env
MONGO_URI_USERS=mongodb://mongo:27017/users
MONGO_URI_PRODUCTS=mongodb://mongo:27017/products
MONGO_URI_CART=mongodb://mongo:27017/cart
MONGO_URI_ORDER=mongodb://mongo:27017/orders
MONGO_URI_PAYMENT=mongodb://mongo:27017/payments
JWT_SECRET=your_super_secret_jwt_key
STRIPE_SECRET_KEY=your_stripe_key
NODEMAILER_EMAIL=your_email@gmail.com
NODEMAILER_PASSWORD=your_app_password
TWILIO_ACCOUNT_SID=your_sid
TWILIO_AUTH_TOKEN=your_token
TWILIO_PHONE_NUMBER=your_twilio_number
```

### Step B: Deploy the Stack
1. Clone the repository to your EC2:
   ```bash
   git clone <your-repo-url>
   cd scalable-ecommerce-backend
   ```
2. Deploy the stack using the compose file:
   ```bash
   docker stack deploy -c docker-compose.yml ecommerce
   ```

### Step C: Verify Deployment
Check the status of your services:
```bash
docker stack ps ecommerce
```
Your API should now be accessible at `http://<your-ec2-public-ip>/api/`

---

## 4. How to Update
When you push new code to GitHub:
1. The `build-and-push.yml` workflow will automatically push the latest images to Docker Hub.
2. To update the services on EC2, run:
   ```bash
   docker stack deploy -c docker-compose.yml ecommerce
   ```
   Docker Swarm will perform a rolling update of the containers.
