# Data Engineering Docker Workshop - Pipeline

A containerized data pipeline that ingests NYC yellow taxi trip data into PostgreSQL. This project demonstrates best practices for building reproducible data pipelines using Docker and Python.

## Overview

This pipeline downloads historical NYC taxi trip data from the DataTalks.Club repository and loads it into a PostgreSQL database. It's designed to be:

- **Reproducible**: Uses `uv` for deterministic dependency management with lock file
- **Scalable**: Processes data in configurable chunks to handle large datasets
- **Containerized**: Runs in Docker for consistent environments across machines
- **Configurable**: CLI options for customizing database connections, data sources, and table names

## Features

- Downloads compressed CSV data directly from GitHub
- Streams data in chunks for memory-efficient processing
- Automatically creates database schema on first run
- Progress tracking with tqdm
- Configurable database parameters and table naming

## Prerequisites

- Docker and Docker Compose (for running containerized)
- PostgreSQL database (local or remote)
- Python 3.13+ (for local development)
- `uv` package manager (optional, for local development)

## Quick Start

### Using Docker

1. **Start PostgreSQL** (if not already running):
   ```bash
   docker run --name postgres-db \
     -e POSTGRES_USER=root \
     -e POSTGRES_PASSWORD=root \
     -e POSTGRES_DB=ny_taxi \
     -p 5432:5432 \
     -d postgres:16
   ```

2. **Build the pipeline image**:
   ```bash
   docker build -t ny-taxi-pipeline .
   ```

3. **Run the pipeline**:
   ```bash
   docker run --network host ny-taxi-pipeline \
     --pg-user root \
     --pg-pass root \
     --pg-host localhost \
     --pg-port 5432 \
     --pg-db ny_taxi \
     --year 2021 \
     --month 1
   ```

### Local Development

1. **Install dependencies**:
   ```bash
   uv sync --locked
   ```

2. **Run the pipeline**:
   ```bash
   python ingest_data.py \
     --pg-user root \
     --pg-pass root \
     --pg-host localhost \
     --pg-port 5432 \
     --pg-db ny_taxi \
     --year 2021 \
     --month 1
   ```

## Configuration Options

The pipeline accepts the following CLI options:

| Option | Default | Description |
|--------|---------|-------------|
| `--pg-user` | `root` | PostgreSQL username |
| `--pg-pass` | `root` | PostgreSQL password |
| `--pg-host` | `localhost` | PostgreSQL host |
| `--pg-port` | `5432` | PostgreSQL port |
| `--pg-db` | `ny_taxi` | Target database name |
| `--year` | `2021` | Year of taxi data to download |
| `--month` | `1` | Month of taxi data (1-12) |
| `--chunk-size` | `100000` | Rows per batch for database inserts |
| `--target-table` | `yellow_taxi_data` | Target table name |

## Project Structure

```
pipeline/
├── Dockerfile              # Multi-stage Docker image definition
├── pyproject.toml         # Project metadata and dependencies
├── uv.lock                # Locked dependency versions
├── .python-version        # Python version specification
├── ingest_data.py         # Main data ingestion script (entry point)
├── pipeline.py            # Example pipeline script
├── main.py                # Basic main module
└── README.md              # This file
```

## Data Schema

The pipeline creates a `yellow_taxi_data` table (or custom name) with the following columns:

- `VendorID` (Int64)
- `tpep_pickup_datetime` (Datetime)
- `tpep_dropoff_datetime` (Datetime)
- `passenger_count` (Int64)
- `trip_distance` (Float64)
- `RatecodeID` (Int64)
- `store_and_fwd_flag` (String)
- `PULocationID`, `DOLocationID` (Int64) - Pickup/Dropoff location IDs
- `payment_type` (Int64)
- `fare_amount`, `extra`, `mta_tax`, `tip_amount`, `tolls_amount`, `improvement_surcharge`, `congestion_surcharge`, `total_amount` (Float64)

## Technologies Used

- **Python 3.13**: Latest stable Python version
- **pandas**: Data manipulation and CSV reading
- **SQLAlchemy**: ORM and database abstraction
- **psycopg2**: PostgreSQL adapter
- **click**: CLI framework
- **tqdm**: Progress bars
- **uv**: Fast Python package manager
- **Docker**: Containerization

## Docker Best Practices

This project uses several Docker best practices:

- **Multi-stage builds**: The Dockerfile copies the `uv` binary from the official image
- **Layer caching**: Dependencies are installed before application code
- **Slim base image**: Uses `python:3.13.10-slim` for smaller image size
- **Reproducible builds**: Uses `uv.lock` for deterministic dependency installation
- **Optimal layer ordering**: Rarely changed files (dependencies) before frequently changed files (code)

## Development

To modify the pipeline locally:

1. Install dependencies: `uv sync`
2. Update code as needed
3. Test locally before building Docker image
4. Rebuild Docker image: `docker build -t ny-taxi-pipeline .`

## Troubleshooting

- **Connection refused**: Ensure PostgreSQL is running and accessible at the specified host/port
- **Out of memory**: Reduce `--chunk-size` for large datasets
- **Missing data**: Verify the year/month combination exists in the DataTalks.Club releases

## License

Part of the DataTalks.Club Data Engineering Zoomcamp
