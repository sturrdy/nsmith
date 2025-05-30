# Project Name

## Setup

1. Clone the repository
2. Install dependencies:
```bash
npm install
```

3. Copy environment variables:
```bash
cp .env.example .env
```

4. Fill in your environment variables in `.env`

5. Start development servers:
```bash
npm run dev
```

## Environment Variables

- `DATABASE_URL` - Your database connection string
- `JWT_SECRET` - Secret for JWT token signing
- `PORT` - Server port (default: 3000)

## Project Structure

- `client/` - Frontend application
- `server/` - Backend API
- `shared/` - Shared types and utilities

## API Documentation

See [API.md](./API.md) for endpoint documentation.
```

## 2. Enhanced .gitignore

```gitignore:.gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Build outputs
dist/
build/
.next/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
logs/
*.log
```

## 3. Security with Helmet

```bash
npm install helmet
```

```typescript:server/src/app.ts
import helmet from 'helmet';
import express from 'express';

const app = express();

// Security middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
}));
```

## 4. Enhanced Error Handling

```typescript:server/src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import fs from 'fs/promises';
import path from 'path';

interface AppError extends Error {
  statusCode?: number;
  isOperational?: boolean;
}

export const errorHandler = async (
  err: AppError,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';

  // Log error
  const errorLog = {
    timestamp: new Date().toISOString(),
    method: req.method,
    url: req.url,
    statusCode,
    message,
    stack: err.stack,
    userAgent: req.get('User-Agent'),
    ip: req.ip,
  };

  console.error('Error:', errorLog);

  // Log to file in production
  if (process.env.NODE_ENV === 'production') {
    try {
      const logPath = path.join(process.cwd(), 'logs', 'error.log');
      await fs.appendFile(logPath, JSON.stringify(errorLog) + '\n');
    } catch (logError) {
      console.error('Failed to write error log:', logError);
    }
  }

  res.status(statusCode).json({
    success: false,
    message: process.env.NODE_ENV === 'production' ? 'Something went wrong' : message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
};
```

## 5. Package.json Scripts

```json:package.json
{
  "scripts": {
    "dev": "concurrently \"npm run dev:server\" \"npm run dev:client\"",
    "dev:server": "cd server && npm run dev",
    "dev:client": "cd client && npm run dev",
    "build": "npm run build:client && npm run build:server",
    "build:client": "cd client && npm run build",
    "build:server": "cd server && npm run build",
    "start": "cd server && npm start",
    "lint": "eslint . --ext .ts,.tsx,.js,.jsx",
    "lint:fix": "eslint . --ext .ts,.tsx,.js,.jsx --fix",
    "format": "prettier --write \"**/*.{ts,tsx,js,jsx,json,md}\"",
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

## 6. ESLint Configuration

```bash
npm install -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin prettier eslint-config-prettier
```

```json:.eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "env": {
    "node": true,
    "es2022": true
  },
  "rules": {
    "no-console": "warn",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/explicit-function-return-type": "warn"
  }
}
```

## 7. Prettier Configuration

```json:.prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

## 8. Environment Variables Template

```bash:.env.example
# Database
DATABASE_URL=postgresql://username:password@localhost:5432/dbname

# Authentication
JWT_SECRET=your-super-secret-jwt-key

# Server
PORT=3000
NODE_ENV=development

# External APIs
API_KEY=your-api-key
```

## 9. API Documentation Template

```markdown:API.md
# API Documentation

## Authentication

All protected endpoints require a Bearer token in the Authorization header:
```
Authorization: Bearer <your-jwt-token>
```

## Endpoints

### POST /api/auth/login
Login user

**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "token": "jwt-token-here",
  "user": {
    "id": 1,
    "email": "user@example.com"
  }
}
```