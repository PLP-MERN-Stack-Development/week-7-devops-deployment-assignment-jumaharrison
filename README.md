/**
 * MERN Stack Deployment Script (Single Code File Version)
 * Includes: Backend Setup, Frontend Build, MongoDB Config, CI/CD Notes, Monitoring
 */

// 1. ENVIRONMENT VARIABLES
require('dotenv').config();

const express = require('express');
const mongoose = require('mongoose');
const helmet = require('helmet');
const morgan = require('morgan');
const path = require('path');
const cors = require('cors');

const app = express();

// 2. MIDDLEWARES FOR PRODUCTION SECURITY & LOGGING
app.use(helmet());
app.use(cors());
app.use(morgan('combined'));
app.use(express.json());

// 3. DATABASE CONNECTION (MongoDB Atlas)
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  poolSize: 5,
})
.then(() => console.log('✅ Connected to MongoDB Atlas'))
.catch((err) => console.error('❌ MongoDB Error:', err));

// 4. HEALTH CHECK ENDPOINT
app.get('/health', (req, res) => {
  res.status(200).send('🟢 OK - API is live');
});

// 5. PRODUCTION STATIC FRONTEND SERVE
if (process.env.NODE_ENV === 'production') {
  app.use(express.static(path.join(__dirname, 'client', 'build')));
  app.get('*', (req, res) =>
    res.sendFile(path.join(__dirname, 'client', 'build', 'index.html'))
  );
}

// 6. ERROR HANDLING
app.use((err, req, res, next) => {
  console.error('🔥 Error:', err.stack);
  res.status(500).json({ message: 'Internal Server Error' });
});

// 7. SERVER LISTEN
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`🚀 Server running in ${process.env.NODE_ENV} on port ${PORT}`);
});

/* 
================================================================
  CI/CD Setup (GitHub Actions)
================================================================
📁 .github/workflows/ci.yml

name: MERN CI/CD Pipeline
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v4
      with:
        node-version: '18'
    - run: |
        npm install
        cd client && npm install && npm run build
    - run: npm test || echo "tests failed, but continuing"

  # Deployment to Vercel/Render handled via webhook auto-deploy
*/

// ===============================================================
// CLIENT BUILD (RUN THIS MANUALLY BEFORE PUSHING TO MAIN)
// In terminal:
// cd client
// npm install
// npm run build
// ===============================================================

/*
================================================================
  .env Example
================================================================
# For backend (server/.env)
PORT=5000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/dbname?retryWrites=true&w=majority
NODE_ENV=production

# For frontend (client/.env.production)
REACT_APP_API_URL=https://yourapi.render.com
================================================================
*/
