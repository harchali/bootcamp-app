# emailservice

Python service for managing emails

## How to use 

Requirements are installed via `pip install -r requirements.txt`, note that you will need g++ installed.

Then, execute the app with `python email_server.py` on port 8080.

For production, install the GRPC probes, and also add `PYTHONUNBUFFERED=1` and `ENABLE_PROFILER=1` for better debugging.