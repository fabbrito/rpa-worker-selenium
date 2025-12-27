# Simple RPA Worker with Docker Compose

Download and run RPA scripts from URL with automatic restart.

## Quick Start

1. **Create directories:**
```bash
mkdir -p app/src app/db app/tmp app/logs
```

2. **Set your script URL:**
```bash
# Create .env file
cp .env.example .env

# Edit .env and set SCRIPT_URL
nano .env
```

3. **Start the worker:**
```bash
docker compose -f docker-compose.worker.yml up -d
```

4. **View logs:**
```bash
docker compose -f docker-compose.worker.yml logs -f
```

## How It Works

1. Worker downloads your script from SCRIPT_URL
2. Script runs and does its work
3. When script exits (with error after ~1 hour), container restarts automatically
4. All data in `app/db`, `app/src`, `app/tmp`, and `app/logs` persists between restarts

## Your Script Requirements

Your Python script should:
- Run for your desired time (e.g., 1 hour)
- Raise an error or exit when done to trigger restart
- Save important data to `/app/db`, `/app/tmp`, or `/app/logs`

Example:
```python
import time
import sys

def main():
    start_time = time.time()
    max_runtime = 3600  # 1 hour in seconds
    
    while True:
        # Do your RPA work here
        do_work()
        
        # Check if 1 hour passed
        if time.time() - start_time >= max_runtime:
            print("Max runtime reached, exiting to restart")
            sys.exit(1)  # Exit with error to trigger restart
        
        time.sleep(60)  # Wait between tasks

if __name__ == "__main__":
    main()
```

## Persistent Data

The following directories persist between restarts:
- `app/db` - Database files
- `app/src` - Downloaded scripts (cached)
- `app/tmp` - Cache and temporary files
- `app/logs` - Log files

## Commands

```bash
# Start worker
docker compose -f docker-compose.worker.yml up -d

# Stop worker
docker compose -f docker-compose.worker.yml down

# View logs
docker compose -f docker-compose.worker.yml logs -f

# Restart worker
docker compose -f docker-compose.worker.yml restart

# Check status
docker compose -f docker-compose.worker.yml ps
```

## Troubleshooting

**Worker keeps restarting immediately:**
- Check logs: `docker compose -f docker-compose.worker.yml logs`
- Verify SCRIPT_URL is accessible
- Check your script for errors

**Script not found:**
- Verify SCRIPT_URL environment variable is set
- Check if URL is accessible from container

**Need to clear cache:**
```bash
# Stop worker and clear temp files
docker compose -f docker-compose.worker.yml down
rm -rf app/tmp/*
docker compose -f docker-compose.worker.yml up -d
```
