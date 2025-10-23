# Distributed SQL Load Balancer

[![Go Version](https://img.shields.io/badge/Go-1.23-blue.svg)](https://golang.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A production-grade intelligent load balancer that distributes SQL queries across multiple database backends with automatic failure detection, health monitoring, and dynamic scaling capabilities.

## 🚀 Features

- **Intelligent Query Distribution** - Parses SQL queries and distributes them optimally across backends
- **Circuit Breaker Pattern** - Automatic failure detection with 3-state health monitoring (Healthy/Failed/Recovering)
- **Docker Auto-Discovery** - Zero-config backend registration using Docker API
- **Manual Scaling** - Add/remove backends on-demand with cooldown protection
- **Real-Time Monitoring** - 4 comprehensive dashboards with live metrics and visualizations
- **Response-Time Routing** - Selects backends based on health scores (error rate, latency, circuit state)
- **Metrics Preservation** - Maintains request counts and health data across discovery cycles
- **Zero-Downtime Scaling** - Seamless backend addition/removal without service interruption

## 📋 Table of Contents

- [Architecture](#architecture)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [Dashboards](#dashboards)
- [API Endpoints](#api-endpoints)
- [Configuration](#configuration)
- [How It Works](#how-it-works)
- [Testing](#testing)
- [Development](#development)
- [Project Structure](#project-structure)
- [Contributing](#contributing)

## 🏗️ Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│   Load Balancer (Go)            │
│  - Query Parser                 │
│  - Circuit Breaker              │
│  - Health Monitor               │
│  - Docker Discovery             │
└────┬────────┬────────┬──────────┘
     │        │        │
     ▼        ▼        ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│Backend-1│ │Backend-2│ │Backend-N│
│(Node.js)│ │(Node.js)│ │(Node.js)│
│Data:    │ │Data:    │ │Data:    │
│1-1666   │ │1667-3332│ │...      │
└─────────┘ └─────────┘ └─────────┘
```

### Key Components

- **Load Balancer**: Go-based reverse proxy with SQL parsing and intelligent routing
- **Database Backends**: Scalable Node.js services managing data partitions
- **Docker Infrastructure**: Containerized services with automatic discovery
- **Monitoring Dashboards**: Real-time visualization with Chart.js

## ⚡ Quick Start

### Prerequisites

- Docker & Docker Compose
- Go 1.23+ (for local development)
- Node.js 18+ (for backend development)

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/distributed-sql-load-balancer.git
cd distributed-sql-load-balancer/simplelb
```

2. **Start the system with 5 backends**
```bash
docker-compose -f docker-compose-scalable.yml up -d --scale db-backend=5
```

3. **Access the dashboards**
- Main Dashboard: http://localhost:3030/dashboard
- Monitoring: http://localhost:3030/
- Scaling UI: http://localhost:3030/scaling-ui
- Query Dashboard: http://localhost:3030/query-dashboard

### Verify Installation

```bash
# Check running containers
docker ps

# Check metrics
curl http://localhost:3030/metrics
```

## 📖 Usage

### Executing SQL Queries

**Via API:**
```bash
curl -X POST http://localhost:3030/query \
  -H "Content-Type: application/json" \
  -d '{"sql": "SELECT * FROM users WHERE age > 30"}'
```

**Via Query Dashboard:**
1. Open http://localhost:3030/query-dashboard
2. Enter your SQL query
3. Click "Execute Distributed"
4. View results and execution details

### Scaling Backends

**Manual Scaling:**
```bash
# Scale up to 7 backends
docker-compose -f docker-compose-scalable.yml up -d --scale db-backend=7

# Scale down to 3 backends
docker-compose -f docker-compose-scalable.yml up -d --scale db-backend=3
```

**Via Scaling UI:**
1. Open http://localhost:3030/scaling-ui
2. Click "Add Backend" or "Remove Backend"
3. Wait for 30-second cooldown between operations

### Switching Routing Strategy

```bash
# Switch to Round Robin
curl -X POST "http://localhost:3030/strategy?strategy=round_robin"

# Switch to Response Time Based (default)
curl -X POST "http://localhost:3030/strategy?strategy=response_time"
```

## 📊 Dashboards

### 1. Main Dashboard (`/dashboard`)
- **Overview Tab**: Summary metrics and backend grid
- **Real-time Monitoring**: Live performance charts
- **Query Distribution**: SQL query analysis interface
- **Backend Management**: Detailed backend configuration

### 2. Monitoring Dashboard (`/`)
- Backend health status cards
- Performance metrics chart (10 minutes history)
- Query distribution visualization
- Backend load comparison
- Recent query log

### 3. Scaling UI (`/scaling-ui`)
- Current backend count display
- Add/Remove backend controls
- Backend status grid with health indicators
- Partition information
- Error rates and request counts

### 4. Query Dashboard (`/query-dashboard`)
- SQL query input interface
- Query analysis results
- Distribution strategy visualization
- Execution time tracking

## 🔌 API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Load balancer (proxies to backends) |
| `/health` | GET | Load balancer health check |
| `/metrics` | GET | Backend metrics and health data |
| `/query` | POST | Execute SQL query |
| `/strategy` | POST | Change routing strategy |
| `/dashboard` | GET | Main dashboard UI |
| `/scaling-ui` | GET | Scaling control UI |
| `/api/backends` | GET | Backend status information |
| `/api/scale` | POST | Scale backends up/down |

## ⚙️ Configuration

### Environment Variables

```bash
# Docker Discovery
DOCKER_DISCOVERY=true
BACKEND_LABEL=app=db-backend
BACKEND_NETWORK=simplelb_simplelb-network
TOTAL_RECORDS=10000

# Load Balancer
PORT=3030
```

### Circuit Breaker Settings

- **Failure Threshold**: 5 consecutive failures
- **Recovery Timeout**: 30 seconds
- **Recovery Test Requests**: 3 requests

### Health Check Settings

- **Interval**: 30 seconds
- **Timeout**: 3 seconds
- **Discovery Interval**: 10 seconds

## 🔧 How It Works

### SQL Query Distribution

1. **Query Parsing**: SQL parsed using `sqlparser` library
2. **Strategy Selection**: Determines full/partial/single backend distribution
3. **Query Modification**: Adds partition ranges (e.g., `id BETWEEN 1 AND 1666`)
4. **Parallel Execution**: Sends modified queries to backends concurrently
5. **Result Merging**: Combines results using appropriate merge strategy

### Circuit Breaker Pattern

**Three States:**
- **HEALTHY**: Normal operation, all requests forwarded
- **FAILED**: Too many failures (>5), requests blocked
- **RECOVERING**: Testing recovery with limited requests (3)

**Failure Detection:**
- Tracks response status codes (≥400 = error)
- Increments failure counter on errors
- Resets counter on success
- Trips circuit after threshold

**Recovery Process:**
1. Wait 30 seconds after failure
2. Transition to RECOVERING state
3. Send 3 test requests
4. If all succeed → HEALTHY
5. If any fails → back to FAILED

### Docker Auto-Discovery

1. Connects to Docker socket (`/var/run/docker.sock`)
2. Queries containers with label `app=db-backend`
3. Extracts IP addresses from shared network
4. Calculates data partitions dynamically
5. Updates backend pool every 10 seconds
6. **Preserves metrics** - doesn't recreate backends

### Health Monitoring

**Health Score Calculation:**
- Error Rate (40% weight): 0% errors = 1.0, 100% errors = 0.0
- Response Time (40% weight): <100ms = 1.0, >2000ms = 0.0
- Circuit State (20% weight): HEALTHY = 1.0, FAILED = 0.0

**Backend States:**
- **HEALTHY**: Error rate < 50%, circuit closed
- **DEGRADED**: Error rate 50-100%, circuit closed
- **FAILED**: Circuit breaker tripped
- **RECOVERING**: Testing recovery

## 🧪 Testing

### Run Demo Script

```bash
cd simplelb
.\UNHEALTHY_BACKEND_DEMO.ps1
```

This script:
1. Switches to Round Robin strategy
2. Sends 60 requests rapidly
3. Displays backend health status
4. Shows unhealthy backend detection
5. Opens dashboards automatically

### Test Queries

```sql
-- Basic SELECT
SELECT * FROM users WHERE age > 30

-- Aggregation
SELECT COUNT(*) FROM users

-- Sorting
SELECT * FROM users ORDER BY salary DESC LIMIT 10

-- Filtering
SELECT name, age FROM users WHERE department = 'Engineering'
```

### Manual Testing

```bash
# Send multiple requests
for i in {1..20}; do
  curl -s http://localhost:3030/ > /dev/null
done

# Check metrics
curl http://localhost:3030/metrics | jq '.backends'
```

## 🛠️ Development

### Build Locally

```bash
cd simplelb
go build -o simplelb.exe
./simplelb.exe
```

### Run Backend Locally

```bash
cd simplelb
node test-db-backend.js 8080 "Backend-1" 1 10000
```

### Docker Build

```bash
# Build load balancer
docker build -t simplelb:latest .

# Build backend
docker build -t db-backend:latest -f Dockerfile.backend .
```

## 📁 Project Structure

```
.
├── simplelb/
│   ├── main.go                          # Load balancer main
│   ├── query-engine.go                  # SQL query analysis
│   ├── query-distributor.go             # Query distribution logic
│   ├── docker-discovery.go              # Docker auto-discovery
│   ├── scaling-handlers.go              # Scaling API handlers
│   ├── unified-dashboard.html           # Main dashboard
│   ├── monitoring-dashboard.html        # Monitoring UI
│   ├── scaling-ui.html                  # Scaling control UI
│   ├── query-dashboard.html             # Query interface
│   ├── docker-compose-scalable.yml      # Docker Compose config
│   ├── Dockerfile                       # Load balancer image
│   ├── Dockerfile.backend               # Backend image
│   ├── Dockerfile.backend-unhealthy     # Unhealthy backend (testing)
│   └── test-db-backend.js               # Backend service
└── README.md
```

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [sqlparser](https://github.com/xwb1989/sqlparser) - SQL parsing library
- [Docker Go SDK](https://github.com/docker/docker) - Docker API client
- [Chart.js](https://www.chartjs.org/) - Data visualization

From RayanPinto




---

⭐ Star this repo if you find it helpful!
