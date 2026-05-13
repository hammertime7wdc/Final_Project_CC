# LK Deliveries Deployment Guide

Complete step-by-step guide to deploy LK Deliveries to Railway.

---

## Prerequisites

Before deploying, make sure you have:

### Required Accounts
- **GitHub account** (free) - for version control
- **Railway account** (free) - for hosting
- **Google OAuth credentials** - for customer login
- **Cloudinary account** (free) - for image storage
- **SMTP email** (Gmail, SendGrid, etc.) - for notifications

### Local Setup
- Git installed on your machine
- Python 3.11 or higher
- VS Code or any code editor
- The LK Deliveries app code locally working

### Verify Local Setup
```bash
# Test that app runs locally
python ui/main.py
# Should open browser window on http://localhost:5000
```

---

## Step 1: Prepare Your GitHub Repository

### 1.1 Create a new GitHub repository
1. Go to [github.com](https://github.com)
2. Click **New** (or + icon)
3. Name it `lk-deliveries-cloud-project`
4. **Public** (so professor can view it)
5. Click **Create repository**

### 1.2 Push your code to GitHub
```bash
# From your project directory
git init
git add .
git commit -m "Initial commit: LK Deliveries app"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/lk-deliveries-cloud-project.git
git push -u origin main
```

### 1.3 Create a Procfile
Create a file called `Procfile` in the root of your repo (no extension):
```
web: python ui/main.py
```

This tells Railway how to start your app.

### 1.4 Create requirements.txt
Make sure your `requirements.txt` includes all dependencies:
```
flet==0.28.3
sqlalchemy==2.0.23
python-dotenv==1.0.0
flask==3.0.0
requests==2.31.0
bcrypt==4.1.1
email-validator==2.1.0
cloudinary==1.36.0
google-auth==2.25.2
google-auth-oauthlib==1.2.0
google-auth-httplib2==0.2.0
```

---

## Step 2: Create a Railway Project

### 2.1 Sign up / Log in to Railway
1. Go to [railway.app](https://railway.app)
2. Click **Sign up** (or log in if you already have account)
3. Choose **GitHub** authentication
4. Authorize Railway to access your GitHub

### 2.2 Create new project
1. Click **+ New Project**
2. Select **Deploy from GitHub repo**
3. Find and select `lk-deliveries-cloud-project`
4. Click **Deploy**

Railway will automatically start the deployment from your `Procfile`.

---

## Step 3: Configure Environment Variables in Railway

### 3.1 Open Railway project settings
1. In Railway dashboard, click on your project
2. Click the **Variables** tab
3. Add each environment variable below

### 3.2 Add these environment variables:

```
PORT=5000

CLOUDINARY_API_KEY=your_cloudinary_api_key_here
CLOUDINARY_API_SECRET=your_cloudinary_secret_here
CLOUDINARY_CLOUD_NAME=your_cloud_name_here

GOOGLE_CLIENT_ID=your_google_client_id_here
GOOGLE_CLIENT_SECRET=your_google_client_secret_here

SMTP_EMAIL=your_email@gmail.com
SMTP_PASSWORD=your_app_password_here

DEBUG=False
```

**How to get these:**

**Cloudinary:**
- Sign up at [cloudinary.com](https://cloudinary.com)
- Go to Dashboard → Settings
- Copy API Key, API Secret, Cloud Name

**Google OAuth:**
- Go to [Google Cloud Console](https://console.cloud.google.com)
- Create new project
- Create OAuth 2.0 credentials (Web application)
- Add authorized redirect URI: `https://your-railway-url.railway.app/auth/callback`
- Copy Client ID and Client Secret

**SMTP (Gmail):**
- Use your Gmail email
- Go to Google Account → Security
- Enable 2-factor authentication
- Generate **App Password**
- Use that as SMTP_PASSWORD (not your regular password)

---

## Step 4: Configure Railway Deployment Settings

### 4.1 Set start command (if not auto-detected)
1. In Railway project, click **Settings**
2. Under **Build**, add start command:
```
python ui/main.py
```

### 4.2 Add rootDirectory (if using monorepo)
- Leave blank if app is in root
- Set to `./` if app code is in root folder

### 4.3 Configure Port
- Railway should auto-detect PORT from environment variables
- Verify PORT=5000 is set in Variables tab

---

## Step 5: Deploy and Monitor

### 5.1 Trigger deployment
1. Push changes to GitHub:
```bash
git add .
git commit -m "Configure for Railway deployment"
git push
```

2. Railway auto-deploys when you push
3. Watch the deployment logs in Railway dashboard

### 5.2 Check deployment status
1. Go to Railway project
2. Click **Deployments** tab
3. Wait for status to show **Success** (green checkmark)

### 5.3 Get your public URL
1. Click **Environment** tab
2. Under **Domains**, copy your Railway URL
3. Example: `https://lk-deliveries-cloud-project-production.railway.app`

---

## Step 6: Test the Live Application

### 6.1 Access the app
1. Open the Railway URL in browser
2. You should see **LK Deliveries** splash screen or login page

### 6.2 Test main features
- **Customer:** Log in → Browse menu → Add to cart → Place order
- **Owner:** Log in → Manage menu → View orders
- **Admin:** Log in → View users → Monitor fraud

### 6.3 Verify external services
- Google OAuth login works
- Menu images load (from Cloudinary)
- Email notifications send on order placement

### 6.4 Check logs for errors
1. In Railway dashboard, click **Logs** tab
2. View real-time logs
3. Look for any error messages

---

## Step 7: Monitoring & Maintenance

### 7.1 View application logs
1. Railway dashboard → **Logs** tab
2. Filter by error level
3. Check for:
   - 500 errors (app errors)
   - 404 errors (missing endpoints)
   - Connection errors (database/services)

### 7.2 Update and redeploy
To make changes:
```bash
# Make code changes locally
git add .
git commit -m "Fix: description of changes"
git push
# Railway auto-redeploys!
```

### 7.3 Rollback if needed
1. In Railway **Deployments** tab
2. Click on previous successful deployment
3. Click **Redeploy**

### 7.4 Check resource usage
1. **Resources** tab shows:
   - Memory usage
   - CPU usage
   - Uptime
2. If resource limited, upgrade Railway plan

---

## Troubleshooting

### App won't start (Build failed)
- Check `requirements.txt` has all dependencies
- Check `Procfile` syntax: `web: python ui/main.py`
- View Railway logs for specific error

### App crashes after deploy (Runtime error)
- Check environment variables are set correctly
- Verify `PORT=5000` is set
- Check logs for missing imports or config errors

### Can't connect to database
- Ensure SQLite file exists in app
- Check database path in code: should be relative path like `./food_delivery.db`
- For Railway, database should be in container, not external

### Images not loading (Cloudinary)
- Verify `CLOUDINARY_API_KEY` is correct
- Check `CLOUDINARY_CLOUD_NAME` is correct
- Ensure images uploaded to correct folder

### Email not sending
- Verify `SMTP_EMAIL` and `SMTP_PASSWORD` correct
- For Gmail: must use App Password, not account password
- Check that 2FA is enabled on Gmail

### Google OAuth not working
- Verify redirect URI in Google Console matches: `https://your-railway-url/auth/callback`
- Check `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` in Railway variables
- Ensure Google APIs enabled in Cloud Console

---

## What Happens on Each Push

1. You push code to GitHub
2. Railway detects the push
3. Railway pulls latest code
4. Railway reads `Procfile` and `requirements.txt`
5. Railway installs dependencies
6. Railway starts your app with: `python ui/main.py`
7. App binds to PORT environment variable
8. App available at Railway URL

---

## Success Checklist

- [ ] Code pushed to GitHub
- [ ] Procfile created in root
- [ ] Railway project created
- [ ] All environment variables set
- [ ] Deployment successful (green checkmark)
- [ ] Can access Railway URL in browser
- [ ] Login works
- [ ] Menu loads
- [ ] Can place order
- [ ] Dashboard accessible
- [ ] Logs show no errors

---

**You're deployed! 🎉**

The app is now live on the internet at your Railway URL.

Next steps:
1. Record your video presentation using the Railway URL
2. Share the live demo link with your professor
3. Document your architecture and costs
