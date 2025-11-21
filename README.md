# Backend Setup Files

## package.json
```json
{
  "name": "prescription-backend",
  "version": "1.0.0",
  "description": "Digital Prescription Management System Backend",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^8.0.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.2",
    "dotenv": "^16.3.1",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "express-rate-limit": "^7.1.5",
    "twilio": "^4.19.0",
    "multer": "^1.4.5-lts.1",
    "cloudinary": "^1.41.0",
    "express-validator": "^7.0.1",
    "winston": "^3.11.0",
    "compression": "^1.7.4",
    "express-mongo-sanitize": "^2.2.0",
    "hpp": "^0.2.3"
  },
  "devDependencies": {
    "nodemon": "^3.0.2"
  }
}
```

## .env
```env
# Server Configuration
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:3000

# Database
MONGODB_URI=mongodb://localhost:27017/prescription_system

# JWT Secret
JWT_SECRET=your_super_secret_jwt_key_change_this_in_production
JWT_EXPIRE=30d

# Twilio Configuration
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=+1234567890

# Cloudinary Configuration (for image uploads)
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Payment Configuration
PRESCRIPTION_CHARGE=4.50

# Super Admin Default Credentials
SUPER_ADMIN_EMAIL=superadmin@prescriptapp.superadmin.com
SUPER_ADMIN_PASSWORD=SuperSecure123!@#
```

## src/server.js
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');
const mongoSanitize = require('express-mongo-sanitize');
const hpp = require('hpp');
const rateLimit = require('express-rate-limit');
require('dotenv').config();

const authRoutes = require('./routes/auth');
const doctorRoutes = require('./routes/doctor');
const frontdeskRoutes = require('./routes/frontdesk');
const hospitalAdminRoutes = require('./routes/hospitalAdmin');
const superAdminRoutes = require('./routes/superAdmin');
const logger = require('./utils/logger');

const app = express();

// Security Middleware
app.use(helmet());
app.use(cors({
  origin: process.env.CLIENT_URL,
  credentials: true
}));
app.use(mongoSanitize());
app.use(hpp());
app.use(compression());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use('/api/', limiter);

// Body parser
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Database connection
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
  .then(() => logger.info('MongoDB Connected'))
  .catch(err => logger.error('MongoDB Connection Error:', err));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/doctor', doctorRoutes);
app.use('/api/frontdesk', frontdeskRoutes);
app.use('/api/hospital-admin', hospitalAdminRoutes);
app.use('/api/super-admin', superAdminRoutes);

// Health check
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'OK', timestamp: new Date() });
});

// Error handling middleware
app.use((err, req, res, next) => {
  logger.error(err.stack);
  res.status(err.status || 500).json({
    success: false,
    message: err.message || 'Internal Server Error'
  });
});

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`);
});

module.exports = app;
```

## src/config/database.js
```javascript
const mongoose = require('mongoose');
const logger = require('../utils/logger');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    logger.info(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    logger.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

## src/config/auth.js
```javascript
module.exports = {
  jwtSecret: process.env.JWT_SECRET,
  jwtExpire: process.env.JWT_EXPIRE || '30d',
  roles: {
    SUPER_ADMIN: 'superadmin',
    HOSPITAL_ADMIN: 'hospital_admin',
    DOCTOR: 'doctor',
    FRONTDESK: 'frontdesk'
  },
  emailDomains: {
    SUPER_ADMIN: '.superadmin.com',
    HOSPITAL_ADMIN: '.admin.com',
    DOCTOR: '.doctor.com',
    FRONTDESK: '.frontdesk.com'
  }
};
```

## src/config/twilio.js
```javascript
const twilio = require('twilio');

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

module.exports = client;
```
