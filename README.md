# University Carpooling Timetabling Optimization System

A system that optimizes university timetabling while considering carpooling opportunities using OSRM (Open Source Routing Machine) for routing calculations and a custom heuristic algorithm for timetable optimization. Both components are containerized using Docker for easy deployment and scalability.

## System Architecture

The system consists of two main containerized components:
- OSRM Server: Handles routing calculations and distance matrices
- Heuristic Algorithm Service: Processes timetabling optimization with carpooling constraints

### Prerequisites

- Docker and Docker Compose
- At least 8GB RAM
- 20GB available disk space
- Linux/Unix-based OS (recommended)

## Getting Started

### 1. OSRM Container Setup

```bash
# Pull the OSRM container
docker pull osrm/osrm-backend

# Create a directory for map data
mkdir osrm_data
cd osrm_data

# Download and process map data (example for a specific region)
wget http://download.geofabrik.de/[region]-latest.osm.pbf
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-extract -p /opt/car.lua /data/[region]-latest.osm.pbf
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/[region]-latest.osrm
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-customize /data/[region]-latest.osrm
```

### 2. Heuristic Algorithm Container Setup

```bash
# Build the heuristic algorithm container
docker build -t timetabling-heuristic .
```

### 3. Running the System

Create a `docker-compose.yml` file:

```yaml
version: '3'
services:
  osrm:
    image: osrm/osrm-backend
    ports:
      - "5000:5000"
    volumes:
      - ./osrm_data:/data
    command: "osrm-routed --algorithm mld /data/[region]-latest.osrm"

  timetabling:
    image: timetabling-heuristic
    depends_on:
      - osrm
    environment:
      - OSRM_HOST=osrm
      - OSRM_PORT=5000
```

Start the services:
```bash
docker-compose up -d
```

## System Configuration

### OSRM Configuration
- Default port: 5000
- Supported profiles: car
- Memory usage: 4GB minimum recommended
- Customize `car.lua` profile if needed

### Heuristic Algorithm Parameters
- Population size: 100 (default)
- Maximum iterations: 1000 (default)
- Mutation rate: 0.1 (default)
- Crossover rate: 0.8 (default)

Adjust these parameters in the configuration file based on your specific needs.

## Input Data Format

### Timetabling Data
```json
{
    "courses": [
        {
            "id": "COURSE101",
            "duration": 120,
            "required_rooms": ["LECTURE_HALL"]
        }
    ],
    "students": [
        {
            "id": "STU001",
            "courses": ["COURSE101"],
            "location": [latitude, longitude]
        }
    ]
}
```

### Constraints
- Time windows for classes
- Room capacity
- Student travel time
- Carpooling group size limits

## API Endpoints

### OSRM Service
- `GET /route/v1/driving/{coordinates}`: Get route between points
- `GET /table/v1/driving/{coordinates}`: Get distance matrix

### Timetabling Service
- `POST /optimize`: Submit timetabling problem
- `GET /status/{job_id}`: Check optimization status
- `GET /result/{job_id}`: Get optimization results

## Output Format

```json
{
    "timetable": {
        "courses": {
            "COURSE101": {
                "time": "2024-01-01T09:00:00",
                "room": "LECTURE_HALL_1"
            }
        },
        "carpools": [
            {
                "driver": "STU001",
                "passengers": ["STU002", "STU003"],
                "route": {
                    "distance": 12.5,
                    "duration": 900,
                    "waypoints": [[lat1, lon1], [lat2, lon2]]
                }
            }
        ]
    }
}
```

## Error Handling

- OSRM service errors (5xx): Retry with exponential backoff
- Invalid input data: Returns 400 with error details
- Optimization timeout: Configurable, defaults to 1 hour

## Performance Considerations

- OSRM container requires significant memory for routing data
- Consider using multiple OSRM instances for large-scale deployments
- Cache frequently requested routes
- Use batch processing for large timetabling problems

## Troubleshooting

Common issues and solutions:
1. OSRM memory errors: Increase container memory limit
2. Slow optimization: Adjust heuristic parameters
3. Connection issues: Check network configuration between containers

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with detailed description

## License

This project is licensed under the MIT License - see the LICENSE file for details.
