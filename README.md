Mock Tuya Cloud API Server

This project sets up a mock server for the Tuya Cloud API using TypeScript and Node.js. The server simulates key endpoints of the Tuya Cloud, allowing you to develop and test applications without needing access to the actual Tuya Cloud services.

Table of Contents

	•	Introduction
	•	Prerequisites
	•	Project Structure
	•	Installation
	•	Configuration
	•	Running the Server
	•	API Endpoints
	•	Token Generation
	•	Device Status
	•	Device Control
	•	Authentication
	•	Testing the Endpoints
	•	Error Handling
	•	Logging
	•	Building for Production
	•	Version Control
	•	Final Notes

Introduction

This guide walks you through creating a mock Tuya Cloud API server from scratch. The server is built using:
	•	TypeScript: For type-safe JavaScript.
	•	Express: Web framework for handling HTTP requests.
	•	Node.js: JavaScript runtime environment.
	•	Additional Libraries: body-parser, cors, morgan, and others for enhanced functionality.

By the end of this guide, you will have a fully functional mock server with authentication, protected routes, error handling, and logging.

Prerequisites

	•	Node.js and npm: Ensure you have Node.js (which includes npm) installed.

node -v
npm -v

If not installed, download from the official website.

	•	Git: Recommended for version control.

Project Structure

mock-tuya-cloud/
├── node_modules/
├── src/
│   └── index.ts
├── .gitignore
├── package.json
├── package-lock.json
├── tsconfig.json
└── README.md

Installation

Follow these steps to set up the project:
	1.	Create a New Project Directory

mkdir mock-tuya-cloud
cd mock-tuya-cloud


	2.	Initialize npm

npm init -y


	3.	Install Dependencies

npm install express body-parser cors morgan
npm install typescript ts-node nodemon @types/express @types/node @types/body-parser @types/cors @types/morgan --save-dev

Configuration

	1.	Initialize TypeScript Configuration

npx tsc --init


	2.	Update tsconfig.json

{
  "compilerOptions": {
    "target": "ES6",
    "module": "CommonJS",
    "rootDir": "./src",
    "outDir": "./dist",
    "esModuleInterop": true,
    "strict": true
  },
  "include": ["src"]
}


	3.	Create Source Directory

mkdir src


	4.	Create Entry Point File

touch src/index.ts

Running the Server

Development Mode

Use nodemon and ts-node for live reloading:
	1.	Update package.json Scripts

"scripts": {
  "start": "node dist/index.js",
  "build": "tsc",
  "dev": "nodemon src/index.ts --exec ts-node"
}


	2.	Start the Server

npm run dev



Production Mode

	1.	Build the Project

npm run build


	2.	Run the Compiled Code

npm start

API Endpoints

Token Generation

	•	Endpoint: GET /v1.0/token
	•	Description: Generates a mock access token.
	•	Parameters:
	•	client_id
	•	client_secret

Device Status

	•	Endpoint: GET /v1.0/devices/:device_id/status
	•	Description: Retrieves the status of a device.
	•	Protected: Yes (requires valid access_token)

Device Control

	•	Endpoint: POST /v1.0/devices/:device_id/commands
	•	Description: Sends commands to control a device.
	•	Protected: Yes (requires valid access_token)

Authentication

The server uses token-based authentication. You must first obtain a token using the /v1.0/token endpoint with valid client_id and client_secret.
	•	Valid Credentials:
	•	client_id: your_client_id
	•	client_secret: your_client_secret

Use the access_token received in subsequent requests by including it in the headers:

access_token: mock_access_token

Testing the Endpoints

Obtain Access Token

curl 'http://localhost:3000/v1.0/token?client_id=your_client_id&client_secret=your_client_secret'

Response:

{
  "result": {
    "access_token": "mock_access_token",
    "expire_time": 7200,
    "refresh_token": "mock_refresh_token",
    "uid": "mock_uid"
  },
  "success": true,
  "t": 1638316800000
}

Access Device Status

curl http://localhost:3000/v1.0/devices/1234567890/status \
-H "access_token: mock_access_token"

Response:

{
  "result": [
    {
      "code": "switch_led",
      "value": true
    },
    {
      "code": "brightness",
      "value": 80
    }
  ],
  "success": true,
  "t": 1638316800000
}

Control Device

curl -X POST http://localhost:3000/v1.0/devices/1234567890/commands \
-H "Content-Type: application/json" \
-H "access_token: mock_access_token" \
-d '{
  "commands": [
    {
      "code": "switch_led",
      "value": false
    }
  ]
}'

Response:

{
  "result": true,
  "success": true,
  "t": 1638316800000
}

Error Handling

	•	401 Unauthorized: Invalid client_id or client_secret.
	•	403 Forbidden: Missing or invalid access_token.
	•	404 Not Found: Endpoint does not exist.

Example of Unauthorized Access:

curl http://localhost:3000/v1.0/devices/1234567890/status

Response:

{
  "success": false,
  "msg": "Forbidden: Invalid access token",
  "t": 1638316800000
}

Logging

The server uses Morgan for logging HTTP requests in the Apache combined format.

Building for Production

	1.	Compile TypeScript to JavaScript

npm run build


	2.	Run the Server

npm start

Version Control

Initialize a git repository and make your initial commit:

git init
git add .
git commit -m "Initial commit"

Final Notes

	•	Extensibility: You can add more endpoints to simulate additional Tuya Cloud functionalities.
	•	Security: Remember that this is a mock server; do not expose it publicly without proper security measures.
	•	Updates: Keep your dependencies updated.

npm update

Full Source Code

src/index.ts

import express from 'express';
import bodyParser from 'body-parser';
import cors from 'cors';
import morgan from 'morgan';

const app = express();
const port = 3000;

app.use(bodyParser.json());
app.use(cors());
app.use(morgan('combined'));

// Authentication middleware
function authenticateToken(req: express.Request, res: express.Response, next: express.NextFunction) {
  const token = req.headers['access_token'];

  if (token === 'mock_access_token') {
    next();
  } else {
    res.status(403).json({
      success: false,
      msg: 'Forbidden: Invalid access token',
      t: Date.now(),
    });
  }
}

// Mock endpoint for token generation (with client ID and secret)
app.get('/v1.0/token', (req, res) => {
  const { client_id, client_secret } = req.query;

  if (client_id === 'your_client_id' && client_secret === 'your_client_secret') {
    const response = {
      result: {
        access_token: 'mock_access_token',
        expire_time: 7200,
        refresh_token: 'mock_refresh_token',
        uid: 'mock_uid'
      },
      success: true,
      t: Date.now(),
    };
    res.json(response);
  } else {
    res.status(401).json({
      success: false,
      msg: 'Invalid client ID or secret',
      t: Date.now(),
    });
  }
});

// Protected route: Mock endpoint for device status
app.get('/v1.0/devices/:device_id/status', authenticateToken, (req, res) => {
  const { device_id } = req.params;

  const response = {
    result: [
      {
        code: "switch_led",
        value: true
      },
      {
        code: "brightness",
        value: 80
      }
    ],
    success: true,
    t: Date.now(),
  };
  res.json(response);
});

// Protected route: Mock endpoint for device control
app.post('/v1.0/devices/:device_id/commands', authenticateToken, (req, res) => {
  const { device_id } = req.params;
  const { commands } = req.body;

  console.log(`Received command for device ${device_id}:`, commands);

  const response = {
    result: true,
    success: true,
    t: Date.now(),
  };
  res.json(response);
});

// Handle 404 errors
app.use((req, res) => {
  res.status(404).json({
    success: false,
    msg: 'Endpoint not found',
    t: Date.now(),
  });
});

// Start the server
app.listen(port, () => {
  console.log(`Mock Tuya Cloud API server is running on http://localhost:${port}`);
});

Visual Directory Overview

mock-tuya-cloud/
├── node_modules/          # Installed dependencies
├── src/                   # Source code directory
│   └── index.ts           # Main application file
├── .gitignore             # Git ignore file
├── package.json           # npm configuration file
├── package-lock.json      # npm lock file
├── tsconfig.json          # TypeScript configuration file
└── README.md              # Project documentation (this file)

Additional Configuration Files

.gitignore

/node_modules
/dist

package.json

{
  "name": "mock-tuya-cloud",
  "version": "1.0.0",
  "description": "Mock Tuya Cloud API Server using TypeScript and Express",
  "main": "dist/index.js",
  "scripts": {
    "start": "node dist/index.js",
    "build": "tsc",
    "dev": "nodemon src/index.ts --exec ts-node"
  },
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "body-parser": "^1.20.2",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "morgan": "^1.10.0"
  },
  "devDependencies": {
    "@types/body-parser": "^1.19.2",
    "@types/cors": "^2.8.13",
    "@types/express": "^4.17.17",
    "@types/morgan": "^1.9.4",
    "@types/node": "^20.4.2",
    "nodemon": "^3.0.1",
    "ts-node": "^10.9.1",
    "typescript": "^5.1.6"
  }
}

tsconfig.json

{
  "compilerOptions": {
    "target": "ES6",
    "module": "CommonJS",
    "rootDir": "./src",
    "outDir": "./dist",
    "esModuleInterop": true,
    "strict": true
  },
  "include": ["src"]
}

Final Notes

	•	Documentation: Keep your README updated as you add new features.
	•	Testing: Implement tests using frameworks like Jest or Mocha.
	•	Collaboration: If working with a team, consider setting up a repository on GitHub or GitLab.
	•	Further Development: Explore adding more complex features like data persistence, more detailed device simulations, and integration with front-end applications.

Enjoy building your mock Tuya Cloud API server!
