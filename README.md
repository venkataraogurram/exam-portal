# Exam Portal - AWS Elastic Beanstalk Deployment

A modern, full-stack exam portal application with separate frontend and backend deployments on AWS Elastic Beanstalk, integrated with RDS MySQL database.

## 🌟 Features

### Student Features
- ✅ User registration and authentication
- ✅ Browse and take exams
- ✅ View exam results and history
- ✅ Profile management
- ✅ Password change functionality
- ✅ Statistics dashboard

### Admin Features
- ✅ Secure admin authentication
- ✅ Dashboard with statistics
- ✅ Create, edit, and delete exams
- ✅ Add multiple-choice questions
- ✅ Manage students (add, activate, deactivate, delete)
- ✅ Reset student passwords
- ✅ View all exam results
- ✅ Profile management

### Technical Features
- ✅ Modern, responsive UI with Font Awesome icons
- ✅ RDS MySQL database integration
- ✅ RESTful API architecture
- ✅ CORS enabled for cross-origin requests
- ✅ Graceful error handling
- ✅ Professional design system

## 🏗️ Architecture

```
exam-portal/
├── backend/                    # Flask API Backend
│   ├── .ebextensions/         # EB configuration
│   │   ├── environment.config # Environment variables
│   │   └── python.config      # Python/WSGI config
│   ├── .platform/             # Platform hooks
│   ├── app.py                 # Main Flask application
│   ├── database.py            # Database operations
│   └── requirements.txt       # Python dependencies
│
├── frontend/                   # Static Frontend
│   ├── .ebextensions/         # EB configuration
│   ├── admin.html             # Admin console
│   ├── admin.js               # Admin logic
│   ├── student-auth.html      # Student login/register
│   ├── student-auth.js        # Auth logic
│   ├── student-dashboard.html # Student dashboard
│   ├── student-dashboard.js   # Dashboard logic
│   ├── student-profile.html   # Student profile
│   ├── student-profile.js     # Profile logic
│   ├── modern-style.css       # Modern design system
│   ├── application.py         # Simple Python server for EB
│   └── requirements.txt       # Python dependencies
│
├── setup_db.py                # Database initialization script
├── DEPLOYMENT.md              # Detailed deployment guide
├── FEATURES.md                # Feature documentation
└── UPDATES.md                 # Change log
```

## 📋 Prerequisites

Before deploying, ensure you have:

1. **AWS Account** with appropriate permissions
2. **AWS CLI** installed and configured
   ```bash
   aws --version
   aws configure
   ```

3. **EB CLI** (Elastic Beanstalk Command Line Interface)
   ```bash
   pip install awsebcli
   eb --version
   ```

4. **Python 3.8+** installed
   ```bash
   python3 --version
   ```

5. **Git** installed
   ```bash
   git --version
   ```

6. **RDS MySQL Database** (or create one during deployment)

## 🚀 Quick Start Deployment

### Step 1: Clone the Repository

```bash
git clone https://github.com/venkataraogurram/exam-portal.git
cd exam-portal
```

### Step 2: Set Up RDS MySQL Database

#### Option A: Create New RDS Instance
1. Go to AWS RDS Console
2. Click "Create database"
3. Choose MySQL
4. Select "Free tier" template (if eligible)
5. Set DB instance identifier: `exam-portal-db`
6. Set Master username: `admin`
7. Set Master password: (choose a secure password)
8. Note down the endpoint URL after creation

#### Option B: Use Existing RDS Instance
- Ensure you have the endpoint, username, password, and database name

### Step 3: Configure Database Security Group

1. Go to RDS Console → Your database → Connectivity & security
2. Click on the VPC security group
3. Edit inbound rules:
   - Add rule: Type: MySQL/Aurora, Port: 3306, Source: Your IP (for testing)
   - Add rule: Type: MySQL/Aurora, Port: 3306, Source: Backend EC2 security group (after backend deployment)

### Step 4: Initialize Database

```bash
# Update setup_db.py with your RDS credentials
python3 setup_db.py
```

Or manually create the database:
```sql
CREATE DATABASE IF NOT EXISTS examportal;
```

### Step 5: Deploy Backend

```bash
cd backend

# Initialize EB (first time only)
eb init -p python-3.8 exam-portal-backend --region us-east-1

# Create environment with environment variables
eb create exam-backend-env \
  --instance-type t2.micro \
  --envvars DB_HOST=your-rds-endpoint.rds.amazonaws.com,DB_USER=admin,DB_PASSWORD=your-password,DB_NAME=examportal,DB_PORT=3306

# Or if environment exists, just deploy
eb deploy

# Check status
eb status
eb open
```

**Important**: After backend deployment, note the backend URL (e.g., `http://exam-backend-env.eba-xxxxx.us-east-1.elasticbeanstalk.com`)

### Step 6: Update Frontend Configuration

Update the API URL in frontend JavaScript files:

```bash
# Update these files with your backend URL:
# - frontend/admin.js
# - frontend/student-auth.js
# - frontend/student-dashboard.js
# - frontend/student-profile.js

# Replace:
const API_URL = 'http://exam-backend-env.eba-eepuvjyu.us-east-1.elasticbeanstalk.com';

# With your actual backend URL
```

### Step 7: Deploy Frontend

```bash
cd ../frontend

# Initialize EB (first time only)
eb init -p python-3.8 exam-portal-frontend --region us-east-1

# Create environment with ALB
eb create exam-frontend-env \
  --instance-type t2.micro \
  --elb-type application

# Or if environment exists, just deploy
eb deploy

# Check status
eb status
eb open
```

### Step 8: Update Backend Security Group

After both deployments:

1. Go to EC2 Console → Security Groups
2. Find backend EC2 instance security group
3. Copy the security group ID (e.g., `sg-xxxxx`)
4. Go to RDS security group
5. Add inbound rule: Type: MySQL/Aurora, Port: 3306, Source: Backend security group ID

### Step 9: Initialize Database Tables

```bash
# Call the initialization endpoint
curl -X POST http://your-backend-url.elasticbeanstalk.com/api/init-db
```

Or visit in browser: `http://your-backend-url.elasticbeanstalk.com/api/init-db`

### Step 10: Access Your Application

**Admin Portal:**
- URL: `http://your-frontend-url.elasticbeanstalk.com/admin.html`
- Default credentials: `admin` / `admin123`

**Student Portal:**
- URL: `http://your-frontend-url.elasticbeanstalk.com/student-auth.html`
- Register a new account to get started

## 🔧 Configuration

### Backend Environment Variables

Set these in `.ebextensions/environment.config` or via EB console:

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    DB_HOST: "your-rds-endpoint.rds.amazonaws.com"
    DB_USER: "admin"
    DB_PASSWORD: "your-secure-password"
    DB_NAME: "examportal"
    DB_PORT: "3306"
```

### Frontend API Configuration

Update `API_URL` in these files:
- `frontend/admin.js`
- `frontend/student-auth.js`
- `frontend/student-dashboard.js`
- `frontend/student-profile.js`

## 📝 Database Schema

The application uses the following tables:

- **admin_users**: Admin authentication
- **students**: Student accounts and profiles
- **exams**: Exam metadata
- **questions**: Exam questions with options
- **results**: Student exam submissions and scores

## 🔐 Security Considerations

### For Production Deployment:

1. **Use HTTPS**: Configure SSL certificate via AWS Certificate Manager
2. **Hash Passwords**: Implement bcrypt for password hashing
3. **JWT Tokens**: Use JWT for session management
4. **Environment Variables**: Never commit credentials to Git
5. **Security Groups**: Restrict RDS access to backend EC2 only
6. **IAM Roles**: Use IAM roles instead of access keys
7. **WAF**: Consider AWS WAF for additional protection
8. **Backup**: Enable automated RDS backups

### Update Admin Password:

```sql
UPDATE admin_users SET password = 'new-secure-password' WHERE username = 'admin';
```

## 🛠️ Useful Commands

### EB CLI Commands

```bash
# View logs
eb logs

# SSH into instance
eb ssh

# View environment status
eb status

# View environment health
eb health

# Set environment variables
eb setenv KEY=VALUE

# Scale instances
eb scale 2

# Terminate environment
eb terminate exam-backend-env
```

### Database Commands

```bash
# Connect to RDS from EC2
mysql -h your-rds-endpoint.rds.amazonaws.com -u admin -p

# Test connection
telnet your-rds-endpoint.rds.amazonaws.com 3306

# View tables
SHOW TABLES;

# View students
SELECT * FROM students;

# View exams
SELECT * FROM exams;
```

## 🐛 Troubleshooting

### Backend Issues

**Problem**: Database connection failed
```bash
# Check environment variables
eb printenv

# Check security group allows connection
# Verify RDS endpoint is correct
# Test connection from EC2 instance
eb ssh
telnet your-rds-endpoint.rds.amazonaws.com 3306
```

**Problem**: 502 Bad Gateway
```bash
# Check application logs
eb logs

# Verify WSGI configuration
# Check if Flask app is running on port 8080
```

### Frontend Issues

**Problem**: CORS errors
- Ensure backend has CORS enabled
- Check API_URL is correct in frontend files

**Problem**: Admin login not working
- Verify Font Awesome CDN is loading
- Check browser console for JavaScript errors
- Ensure admin.js is loaded correctly

### Database Issues

**Problem**: Tables not created
```bash
# Call init endpoint
curl -X POST http://your-backend-url/api/init-db

# Or run setup_db.py
python3 setup_db.py
```

## 📊 Monitoring

### CloudWatch Logs
- Backend logs: `/aws/elasticbeanstalk/exam-backend-env/`
- Frontend logs: `/aws/elasticbeanstalk/exam-frontend-env/`

### Metrics to Monitor
- Request count
- Response time
- Error rate
- Database connections
- CPU utilization

## 💰 Cost Optimization

### Free Tier Eligible:
- t2.micro instances (750 hours/month)
- RDS db.t2.micro (750 hours/month)
- Application Load Balancer (750 hours/month)

### Cost Reduction Tips:
1. Use t2.micro instances
2. Enable RDS auto-scaling
3. Set up auto-scaling for EC2
4. Use CloudWatch alarms for cost monitoring
5. Terminate unused environments

## 🔄 Updates and Maintenance

### Deploy Updates

```bash
# Backend
cd backend
eb deploy

# Frontend
cd frontend
eb deploy
```

### Database Migrations

```bash
# Backup before migration
# Run migration scripts
# Test thoroughly
```

## 📚 Additional Resources

- [AWS Elastic Beanstalk Documentation](https://docs.aws.amazon.com/elasticbeanstalk/)
- [Flask Documentation](https://flask.palletsprojects.com/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [EB CLI Documentation](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project is licensed under the MIT License.

## 👥 Support

For issues and questions:
- Create an issue on GitHub
- Check DEPLOYMENT.md for detailed deployment guide
- Review UPDATES.md for recent changes

## 🎯 Default Credentials

**Admin:**
- Username: `admin`
- Password: `admin123`

**Note**: Change default credentials in production!

---

**Built with ❤️ using Flask, MySQL, and AWS Elastic Beanstalk**
