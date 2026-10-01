# TradeIQ Development Setup Guide

A complete, step-by-step guide to build TradeIQ from scratch. This is designed for developers new to the stack.

---

## Table of Contents

1. [Prerequisites & System Setup](#prerequisites--system-setup)
2. [Phase 1: Environment Configuration](#phase-1-environment-configuration)
3. [Phase 2: Backend Setup (Java/Spring Boot)](#phase-2-backend-setup-javaspring-boot)
4. [Phase 3: Database Setup (PostgreSQL + TimescaleDB)](#phase-3-database-setup-postgresql--timescaledb)
5. [Phase 4: Frontend Setup (Next.js)](#phase-4-frontend-setup-nextjs)
6. [Phase 5: Running the Full Stack](#phase-5-running-the-full-stack)
7. [Development Workflow](#development-workflow)

---

## Prerequisites & System Setup

### What You'll Need

- **Computer**: Mac, Linux, or Windows (with WSL2)
- **Time**: 1-2 hours for initial setup
- **Internet**: Stable connection for downloading tools
- **Text Editor**: VS Code (recommended) — https://code.visualstudio.com

### Step 1: Install Git

Git is how you manage code versions.

**Mac:**
```bash
brew install git
```

**Windows (in PowerShell as Administrator):**
```powershell
choco install git
```

Or download from: https://git-scm.com/download/win

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install git
```

**Verify installation:**
```bash
git --version
```

Should output something like: `git version 2.42.0`

---

### Step 2: Install Node.js & npm

Node.js is the runtime for JavaScript/TypeScript. npm is the package manager.

Go to: https://nodejs.org

- Download the **LTS (Long Term Support)** version
- Follow the installer
- This will install both Node.js and npm

**Verify installation:**
```bash
node --version
npm --version
```

You should see version numbers (e.g., `v20.10.0` and `10.2.3`).

---

### Step 3: Install Java Development Kit (JDK)

We'll use Java 21 LTS for Spring Boot.

**Option A: Using Homebrew (Mac/Linux)**
```bash
brew install openjdk@21
```

**Option B: Download directly**
Go to: https://adoptium.net

- Select Java 21 LTS
- Choose your operating system
- Run the installer

**Verify installation:**
```bash
java -version
javac -version
```

Both should show Java 21.

---

### Step 4: Install Docker

Docker lets you run PostgreSQL without manually installing the database.

Go to: https://www.docker.com/products/docker-desktop

- Download Docker Desktop for your OS
- Install and run it
- You should see the Docker icon in your system tray/menu

**Verify installation:**
```bash
docker --version
```

Should output something like: `Docker version 24.0.0`

---

### Step 5: Install an IDE (Optional but Recommended)

**IntelliJ IDEA Community** (for Java development):
https://www.jetbrains.com/idea/download/

Or use **VS Code** (simpler, lighter):
https://code.visualstudio.com

---

## Phase 1: Environment Configuration

### Step 1: Clone Your Repository

First, navigate to where you want to store your project:

```bash
cd ~/projects
```

Then clone your repository:

```bash
git clone https://github.com/Calvin-nyauyanga/TradeIQ.git
cd TradeIQ
```

### Step 2: Create a `.env` File

This file stores sensitive configuration (API keys, database passwords, etc.).

In the root of your TradeIQ folder, create a file called `.env`:

```bash
touch .env
```

Add this content (we'll populate real values later):

```env
# Database
POSTGRES_DB=tradeiq_dev
POSTGRES_USER=tradeiq_user
POSTGRES_PASSWORD=your_secure_password_here
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Java/Spring Boot
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/tradeiq_dev
SPRING_DATASOURCE_USERNAME=tradeiq_user
SPRING_DATASOURCE_PASSWORD=your_secure_password_here
SERVER_PORT=8080

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8080/api

# API Keys (add these after signing up)
OPENAI_API_KEY=your_key_here
TWELVE_DATA_API_KEY=your_key_here
TRADING_ECONOMICS_API_KEY=your_key_here
```

**Important:** Add `.env` to `.gitignore` so you never commit secrets:

```bash
echo ".env" >> .gitignore
```

---

### Step 3: Create Project Folder Structure

From the root of TradeIQ, create these folders:

```bash
mkdir -p backend frontend shared/models
```

Your structure should look like:

```
TradeIQ/
├── backend/          (Spring Boot application)
├── frontend/         (Next.js application)
├── shared/
│   └── models/       (Shared data models)
├── .env
├── .gitignore
└── SETUP_GUIDE.md
```

---

## Phase 2: Backend Setup (Java/Spring Boot)

### Step 1: Generate a Spring Boot Project

Go to: https://start.spring.io

Configure as follows:

- **Project**: Maven
- **Language**: Java
- **Spring Boot**: 3.2.1 (or latest)
- **Project Metadata**:
  - Group: `com.tradeiq`
  - Artifact: `backend`
  - Name: `TradeIQ Backend`
  - Packaging: Jar
  - Java: 21

**Add Dependencies** (search and select these):
- Spring Web
- Spring Data JPA
- Spring Security
- PostgreSQL Driver
- Spring Boot DevTools
- Lombok
- Validation
- Spring WebSocket

Click **GENERATE** — this downloads a zip file.

### Step 2: Extract and Place Backend

- Extract the downloaded zip
- Move the contents into your `backend/` folder
- Your structure should now be:

```
backend/
├── src/
│   ├── main/
│   │   ├── java/com/tradeiq/
│   │   └── resources/
│   └── test/
├── pom.xml
├── mvnw
└── mvnw.cmd
```

### Step 3: Open Backend in IDE

Open VS Code or IntelliJ in the `backend/` directory:

```bash
cd backend
code .
```

Or in IntelliJ, open the `pom.xml` as a project.

### Step 4: Update `application.yml`

Replace `src/main/resources/application.yml` with:

```yaml
spring:
  application:
    name: TradeIQ Backend

  datasource:
    url: ${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5432/tradeiq_dev}
    username: ${SPRING_DATASOURCE_USERNAME:tradeiq_user}
    password: ${SPRING_DATASOURCE_PASSWORD:tradeiq_pass}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
    show-sql: false

  security:
    jwt:
      secret: ${JWT_SECRET:your-secret-key-change-in-production}
      expiration: 86400000

server:
  port: ${SERVER_PORT:8080}
  servlet:
    context-path: /api

logging:
  level:
    root: INFO
    com.tradeiq: DEBUG
```

### Step 5: Create Basic Controller

Create `src/main/java/com/tradeiq/controller/HealthController.java`:

```java
package com.tradeiq.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/health")
public class HealthController {

    @GetMapping
    public String health() {
        return "TradeIQ Backend is running!";
    }
}
```

### Step 6: Test Backend Startup

From the `backend/` directory:

```bash
./mvnw spring-boot:run
```

You should see Spring Boot starting up. If successful, visit:
```
http://localhost:8080/api/health
```

You should see: `TradeIQ Backend is running!`

**To stop the server:** Press `Ctrl+C`

---

## Phase 3: Database Setup (PostgreSQL + TimescaleDB)

### Step 1: Create `docker-compose.yml`

In the **root of TradeIQ** (not inside backend or frontend), create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: tradeiq-postgres
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  adminer:
    image: adminer:latest
    container_name: tradeiq-adminer
    ports:
      - "8081:8080"
    depends_on:
      - postgres
    environment:
      ADMINER_DEFAULT_SERVER: postgres

volumes:
  postgres_data:
```

### Step 2: Create Database Initialization Script

In the **root of TradeIQ**, create `init-db.sql`:

```sql
-- Enable TimescaleDB extension
CREATE EXTENSION IF NOT EXISTS timescaledb;

-- Create users table
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    subscription_tier VARCHAR(50) DEFAULT 'FREE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create trades table
CREATE TABLE IF NOT EXISTS trades (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    instrument VARCHAR(50) NOT NULL,
    direction VARCHAR(10) NOT NULL,
    entry_price DECIMAL(20, 8) NOT NULL,
    exit_price DECIMAL(20, 8),
    stop_loss DECIMAL(20, 8),
    take_profit DECIMAL(20, 8),
    position_size DECIMAL(20, 8),
    result DECIMAL(20, 8),
    r_multiple DECIMAL(10, 2),
    status VARCHAR(50) DEFAULT 'OPEN',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    closed_at TIMESTAMP
);

-- Create hypertable for historical price data (TimescaleDB)
CREATE TABLE IF NOT EXISTS market_data (
    time TIMESTAMP NOT NULL,
    instrument VARCHAR(50) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    open DECIMAL(20, 8),
    high DECIMAL(20, 8),
    low DECIMAL(20, 8),
    close DECIMAL(20, 8),
    volume DECIMAL(20, 2)
);

SELECT create_hypertable('market_data', 'time', if_not_exists => TRUE);

-- Create indexes
CREATE INDEX ON market_data (instrument, timeframe, time DESC);

-- Create journals table
CREATE TABLE IF NOT EXISTS journals (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    trade_id INTEGER REFERENCES trades(id),
    notes TEXT,
    emotion VARCHAR(50),
    confidence INTEGER,
    fomo BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Step 3: Start PostgreSQL

From the **root of TradeIQ**:

```bash
docker-compose up -d
```

This starts PostgreSQL in the background.

**Verify it's running:**
```bash
docker ps
```

You should see `tradeiq-postgres` running.

### Step 4: Access the Database (Optional)

Open your browser and go to:
```
http://localhost:8081
```

Login with:
- Server: `postgres`
- Username: `tradeiq_user`
- Password: (from your `.env` file)
- Database: `tradeiq_dev`

---

## Phase 4: Frontend Setup (Next.js)

### Step 1: Create Next.js App

From your `frontend/` directory:

```bash
cd frontend

npx create-next-app@latest . --typescript --tailwind --eslint
```

When prompted:
- TypeScript? **Yes**
- ESLint? **Yes**
- Tailwind CSS? **Yes**
- App Router? **Yes**
- Source dir? **No**
- Import alias? **Yes** (keep default `@/*`)

### Step 2: Install Additional Dependencies

```bash
npm install axios zustand react-hot-toast
npm install -D tailwindcss postcss autoprefixer
```

- **axios**: HTTP client
- **zustand**: State management
- **react-hot-toast**: Toast notifications

### Step 3: Install TradingView Charts

```bash
npm install lightweight-charts
```

### Step 4: Install shadcn/ui Components

```bash
npx shadcn-ui@latest init
```

When prompted, use defaults.

Then add specific components:

```bash
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
npx shadcn-ui@latest add label
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add chart
```

### Step 5: Create Basic Directory Structure

From `frontend/`, create:

```bash
mkdir -p src/{components,pages,store,services,hooks,lib}
```

### Step 6: Create API Service

Create `src/services/api.ts`:

```typescript
import axios from 'axios';

const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8080/api';

const api = axios.create({
  baseURL: API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Add token to requests if it exists
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

### Step 7: Create Basic Store

Create `src/store/authStore.ts`:

```typescript
import { create } from 'zustand';

interface User {
  id: number;
  email: string;
  firstName: string;
  lastName: string;
  subscriptionTier: string;
}

interface AuthStore {
  user: User | null;
  token: string | null;
  setUser: (user: User | null) => void;
  setToken: (token: string | null) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  token: localStorage.getItem('token') || null,
  
  setUser: (user) => set({ user }),
  
  setToken: (token) => {
    if (token) {
      localStorage.setItem('token', token);
    } else {
      localStorage.removeItem('token');
    }
    set({ token });
  },
  
  logout: () => {
    localStorage.removeItem('token');
    set({ user: null, token: null });
  },
}));
```

### Step 8: Create Home Page

Replace `src/app/page.tsx`:

```typescript
'use client';

import Link from 'next/link';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';

export default function Home() {
  return (
    <main className="min-h-screen bg-gradient-to-br from-slate-900 to-slate-800">
      <div className="container mx-auto px-4 py-12">
        {/* Header */}
        <div className="flex justify-between items-center mb-16">
          <div>
            <h1 className="text-4xl font-bold text-white mb-2">TradeIQ</h1>
            <p className="text-slate-300">AI-Powered Trading Intelligence</p>
          </div>
          <div className="space-x-4">
            <Link href="/login">
              <Button variant="outline">Login</Button>
            </Link>
            <Link href="/signup">
              <Button>Sign Up</Button>
            </Link>
          </div>
        </div>

        {/* Hero */}
        <div className="grid md:grid-cols-2 gap-12 mb-16 items-center">
          <div>
            <h2 className="text-3xl font-bold text-white mb-4">
              Trade Smarter. Learn Faster.
            </h2>
            <p className="text-slate-300 mb-6">
              TradeIQ combines your trading history, real-time market data, 
              macroeconomic events, and AI reasoning to help you understand 
              your decisions and the market environment.
            </p>
            <Link href="/signup">
              <Button size="lg">Get Started</Button>
            </Link>
          </div>
          <div className="bg-slate-700 rounded-lg h-64 flex items-center justify-center">
            <p className="text-slate-400">Dashboard Preview</p>
          </div>
        </div>

        {/* Features */}
        <div className="grid md:grid-cols-3 gap-6">
          <Card className="bg-slate-700 border-slate-600">
            <CardHeader>
              <CardTitle className="text-white">Trade Journal</CardTitle>
              <CardDescription>Track every trade with details</CardDescription>
            </CardHeader>
            <CardContent>
              <p className="text-slate-300">
                Log your trades, entry/exit points, and performance metrics
                automatically.
              </p>
            </CardContent>
          </Card>

          <Card className="bg-slate-700 border-slate-600">
            <CardHeader>
              <CardTitle className="text-white">AI Analysis</CardTitle>
              <CardDescription>Get intelligent insights</CardDescription>
            </CardHeader>
            <CardContent>
              <p className="text-slate-300">
                Receive detailed reviews of what went right and what went wrong
                in each trade.
              </p>
            </CardContent>
          </Card>

          <Card className="bg-slate-700 border-slate-600">
            <CardHeader>
              <CardTitle className="text-white">Market Intelligence</CardTitle>
              <CardDescription>Real-time data & news</CardDescription>
            </CardHeader>
            <CardContent>
              <p className="text-slate-300">
                Track XAUUSD, US30, NASDAQ, and other instruments with live
                updates.
              </p>
            </CardContent>
          </Card>
        </div>
      </div>
    </main>
  );
}
```

### Step 9: Install shadcn/ui CLI

If it hasn't been installed yet:

```bash
npm install -g shadcn-ui@latest
```

### Step 10: Test Frontend

From `frontend/`:

```bash
npm run dev
```

Visit: `http://localhost:3000`

You should see the TradeIQ landing page.

---

## Phase 5: Running the Full Stack

### Step 1: Terminal Setup

You'll need **3 terminals** running simultaneously:

**Terminal 1 (PostgreSQL):**
```bash
cd TradeIQ
docker-compose up
```

**Terminal 2 (Backend):**
```bash
cd TradeIQ/backend
./mvnw spring-boot:run
```

**Terminal 3 (Frontend):**
```bash
cd TradeIQ/frontend
npm run dev
```

### Step 2: Verify All Services

- **Frontend**: http://localhost:3000
- **Backend**: http://localhost:8080/api/health
- **Database Admin**: http://localhost:8081

### Step 3: Stop Everything

**To stop any service:** Press `Ctrl+C` in its terminal.

**To stop PostgreSQL (keeping data):**
```bash
docker-compose stop
```

**To stop PostgreSQL (deleting data):**
```bash
docker-compose down
```

---

## Development Workflow

### Adding a New Feature

1. **Create a branch:**
```bash
git checkout -b feature/your-feature-name
```

2. **Make changes** in backend, frontend, or both

3. **Commit your work:**
```bash
git add .
git commit -m "Add: description of what you added"
```

4. **Push to GitHub:**
```bash
git push origin feature/your-feature-name
```

5. **Create a Pull Request** on GitHub

### Common Commands

**Backend (Maven):**
```bash
./mvnw clean install      # Build everything
./mvnw test               # Run tests
./mvnw spring-boot:run    # Start the application
```

**Frontend (Next.js):**
```bash
npm install               # Install dependencies
npm run dev               # Start development server
npm run build             # Build for production
npm test                  # Run tests
```

**Database (Docker):**
```bash
docker-compose up -d      # Start in background
docker-compose down       # Stop and remove
docker-compose logs       # View logs
```

### Troubleshooting

**Port already in use:**
```bash
# Find what's using port 3000
lsof -i :3000
# Kill it
kill -9 <PID>
```

**PostgreSQL won't start:**
```bash
docker-compose down
docker-compose up
```

**Backend won't connect to DB:**
Check your `.env` file — ensure credentials match `docker-compose.yml`.

---

## Next Steps

Once everything is running:

1. **Create a user endpoint** in Spring Boot
2. **Connect frontend to backend** with API calls
3. **Build authentication** (login/signup)
4. **Implement trade journal** creation and listing
5. **Add market data integration** (Twelve Data API)

This foundation is solid. Start here, then build each phase incrementally.

---

## Getting Help

- **Spring Boot Docs**: https://spring.io/projects/spring-boot
- **Next.js Docs**: https://nextjs.org/docs
- **PostgreSQL Docs**: https://www.postgresql.org/docs/
- **Stack Overflow**: Search your error message

Good luck building TradeIQ!
