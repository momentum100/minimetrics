# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

minimetrics is a lightweight statsd server with a built-in metrics dashboard. It's designed to be simple, zero-config, and suitable for development environments or small deployments where you need quick metrics without the overhead of complex systems like Prometheus or Grafana.

## Architecture

The project is a single-file Go application (`server.go`) that:

- Runs a statsd UDP server to receive metrics
- Stores metrics in SQLite database (in-memory or file-based)
- Serves a web dashboard from embedded static files in `public/`
- Provides REST API endpoints for querying metrics data

Key components:
- **UDP Server**: Listens for statsd protocol messages (counters, gauges, timers, sets)
- **HTTP Server**: Serves the dashboard and API endpoints
- **Database Layer**: SQLite with goqu query builder for metrics storage
- **Frontend**: Vue.js dashboard with Chart.js for visualization

## Development Commands

### Build and Run
```bash
# Build the application
make build

# Build and run
make run

# Build for Linux using Docker
make buildlinux
```

### Direct Go Commands
```bash
# Build with SQLite JSON support
go build -tags sqlite_json -x -o minimetrics server.go

# Run with SQLite JSON support
go run -tags sqlite_json -x server.go
```

### Configuration
The application accepts these command-line flags:
- `-bind`: Web server bind address (default: "0.0.0.0:8070")
- `-statsd`: Statsd server bind address (default: "0.0.0.0:8125") 
- `-db`: Database path or "memory" for in-memory storage (default: "./metrics.db")

Environment variables:
- `MM_BIND`: Override web server bind address
- `MM_STATSD`: Override statsd bind address
- `MM_DB`: Override database path

## Key Implementation Details

### Database Schema
The application uses SQLite with tables for:
- `counters`: Stores metric values with timestamps and labels
- `labels`: Tracks available metric names and types

### API Endpoints
- `/data/labels`: Returns all available metric labels and types
- `/data/query/`: Queries metric data with filtering and grouping

### Statsd Protocol Support
Supports standard statsd message formats:
- Counters: `metric:value|c`
- Gauges: `metric:value|g`
- Timers: `metric:value|ms`
- Sets: `metric:value|s`

### Frontend Structure
The `public/` directory contains:
- `index.html`: Main dashboard page
- Vue.js components for chart rendering and UI
- Chart.js for data visualization
- CSS for dark theme support

## Dependencies

- `github.com/doug-martin/goqu/v9`: SQL query builder
- `github.com/mattn/go-sqlite3`: SQLite driver
- Uses Go's `embed` package for static files

## Testing

No formal test suite is currently present in the codebase. Testing is done manually by:
1. Running the server
2. Sending statsd messages
3. Viewing results in the web dashboard