# Local Installation Guide for Fitbit-Grafana

This guide provides instructions for setting up the Fitbit data fetcher and visualization stack locally without Docker.

## Prerequisites

- Python 3.7 or higher
- pip (Python package installer)
- InfluxDB 1.8+ (for database)
- Grafana (for visualization)

## Step 1: Install InfluxDB

```bash
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/ubuntu bionic stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt update
sudo apt install influxdb
```



## Step 2: Configure InfluxDB

1. Start InfluxDB service:
   ```bash
   # Linux/macOS
   sudo systemctl start influxdb
   # or
   influxd
   ```

2. Create database and user:
   ```bash
   influx
   
   # In InfluxDB shell:
   CREATE DATABASE FitbitHealthStats
   CREATE USER fitbit_user WITH PASSWORD 'fitbit_password'
   GRANT ALL ON FitbitHealthStats TO fitbit_user
   exit
   ```

## Step 3: Install Grafana

### Installation Methods

#### Ubuntu/Debian:
```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana
```

### Start Grafana:
```bash
# Linux
sudo systemctl start grafana-server

# Manual start
grafana-server
```

Grafana will be available at `http://0.0.0.0:3000` (admin/admin)

## Step 4: Set up Python Environment

3. Create a virtual environment (recommended):
   ```bash
   python -m venv fitbit-env
   source fitbit-env/bin/activate  
   ```

4. Install Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Step 5: Configure Fitbit Application


## Step 6: Configure the Python Script

Edit the `Fitbit_Fetch.py` file and update these variables:

```python
# File paths (create these directories first)
FITBIT_LOG_FILE_PATH = "/path/to/your/logs/fitbit.log"
TOKEN_FILE_PATH = "/path/to/your/tokens/fitbit.token"

# InfluxDB configuration
INFLUXDB_HOST = '0.0.0.0'
INFLUXDB_PORT = 8086
INFLUXDB_USERNAME = 'fitbit_user'
INFLUXDB_PASSWORD = 'fitbit_password'
INFLUXDB_DATABASE = 'FitbitHealthStats'

# Fitbit API credentials
client_id = "your_application_client_ID"
client_secret = "your_application_client_secret"
DEVICENAME = "Your_Device_Name"  # e.g., "Charge5"

# Timezone (optional)
LOCAL_TIMEZONE = "Automatic"  # or specify like "America/New_York"
```

Alternatively, you can set these as environment variables:
```bash
export FITBIT_LOG_FILE_PATH="/path/to/your/logs/fitbit.log"
export TOKEN_FILE_PATH="/path/to/your/tokens/fitbit.token"
export INFLUXDB_HOST="0.0.0.0"
export INFLUXDB_PORT="8086"
export INFLUXDB_USERNAME="fitbit_user"
export INFLUXDB_PASSWORD="fitbit_password"
export INFLUXDB_DATABASE="FitbitHealthStats"
export CLIENT_ID="your_application_client_ID"
export CLIENT_SECRET="your_application_client_secret"
export DEVICENAME="Your_Device_Name"
export LOCAL_TIMEZONE="Automatic"
```

## Step 7: Create Required Directories

```bash
mkdir -p logs tokens
```

## Step 8: Initial Setup and First Run

1. Run the script for the first time:
   ```bash
   python Fitbit_Fetch.py
   ```

2. Enter your refresh token when prompted

3. The script will start fetching data and should show successful API requests

4. Exit with `Ctrl+C` after confirming it works

## Step 9: Configure Grafana

1. Open Grafana at `http://0.0.0.0:3000`
2. Login with admin/admin
3. Add InfluxDB as data source:
   - URL: `http://0.0.0.0:8086`
   - Database: `FitbitHealthStats`
   - User: `fitbit_user`
   - Password: `fitbit_password`
4. Test the connection

## Step 10: Import Dashboard

1. Install the required Grafana plugin:
   ```bash
   grafana-cli plugins install marcusolsson-hourly-heatmap-panel
   ```
   Then restart Grafana.

2. Import the dashboard:
   - Use import code **23088** (for InfluxDB v1)
   - Or download the JSON file from the `Grafana_Dashboard` folder and import it

## Step 11: Running as a Service (Optional)

### Linux (systemd):

1. Copy the provided service file template (`extra/Fitbit_Fetch_Autostart.service`)
2. Edit paths in the service file
3. Install and start the service:
   ```bash
   sudo cp Fitbit_Fetch_Autostart.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable Fitbit_Fetch_Autostart.service
   sudo systemctl start Fitbit_Fetch_Autostart.service
   ```


## Historical Data Bulk Update

To fetch historical data:

1. Stop the regular script if running
2. Set `AUTO_DATE_RANGE=False` in the script or as environment variable
3. Run the script - it will prompt for start and end dates
4. Enter dates in YYYY-MM-DD format
5. Wait for completion (can take 24+ hours for years of data due to API limits)

## Troubleshooting

- **Permission errors**: Ensure the script has write access to log and token directories
- **InfluxDB connection issues**: Verify InfluxDB is running and credentials are correct
- **Fitbit API errors**: Check that your application is set to "personal" type
- **Missing data**: Ensure your Fitbit device is syncing regularly

## Dependencies

The script requires these Python packages (from `requirements.txt`):
- influxdb==5.3.1
- pytz==2022.1
- Requests==2.31.0
- schedule==1.2.0
- influxdb_client==1.39.0
- influxdb3-python==0.12.0

