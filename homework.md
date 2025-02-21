```markdown
# Object Counter API

A Flask-based API that performs object detection on images using TensorFlow Serving, with support for both MongoDB and PostgreSQL storage.

## Architecture

This application follows Hexagonal Architecture with three main layers:

- **Entrypoints**: API endpoints and request handling
- **Domain**: Core business logic and interfaces
- **Adapters**: External service implementations (databases, object detection)

## Features

- Object detection using RFCN model
- Object counting with confidence threshold
- Raw prediction endpoint
- Support for both MongoDB and PostgreSQL storage
- Docker Compose setup for easy deployment

## Prerequisites

- Docker and Docker Compose
- Python 3.11+
- wget (for model download)

## Quick Start with Docker Compose

1. Clone the repository:
```bash
git clone <repository-url>
cd object-counter
```

2. Run the setup script:
```bash
chmod +x setup.sh
./setup.sh
```

3. Start all services:
```bash
docker-compose up -d
```

4. Test the API:
```bash
curl -F "threshold=0.9" -F "file=@resources/images/cat.jpg" http://localhost:5000/object-count
```

## Manual Setup

### 1. Download RFCN Model
```bash
wget https://storage.googleapis.com/intel-optimized-tensorflow/models/v1_8/rfcn_resnet101_fp32_coco_pretrained_model.tar.gz
tar -xzvf rfcn_resnet101_fp32_coco_pretrained_model.tar.gz -C tmp
mkdir -p tmp/model/rfcn/1
mv tmp/rfcn_resnet101_coco_2018_01_28/saved_model/saved_model.pb tmp/model/rfcn/1
```

### 2. Start Required Services

#### Start TensorFlow Serving:
```bash
docker run -d \
    --name=tfserving \
    -p 8500:8500 \
    -p 8501:8501 \
    -v "$(pwd)/tmp/model:/models" \
    -e MODEL_NAME=rfcn \
    -e MODEL_BASE_PATH=/models \
    tensorflow/serving:latest
```

#### Start Database (Choose one):

For PostgreSQL:
```bash
docker run -d \
    --name object-counter-postgres \
    -e POSTGRES_DB=object_counter \
    -e POSTGRES_USER=postgres \
    -e POSTGRES_PASSWORD=postgres \
    -p 5432:5432 \
    postgres:14
```

<!-- For MongoDB:
```bash
docker run -d \
    --name test-mongo \
    -p 27017:27017 \
    mongo:latest
``` -->

### 3. Python Setup
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Environment Configuration

For PostgreSQL:
```bash
export DB_TYPE=postgres

export ENV=prod
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=object_counter
export DB_USER=postgres
export DB_PASSWORD=postgres
```

For MongoDB:
```bash
export DB_TYPE=mongo
export ENV=prod
```

### 5. Run Application
```bash
python -m counter.entrypoints.webapp
```

## API Endpoints

### 1. Object Count
```bash
POST /object-count
```
Parameters:
- `file`: Image file
- `threshold`: Confidence threshold (0.0-1.0)

Response:
```json
{
  "current_objects": [
    {
      "count": 1,
      "object_class": "cat"
    }
  ],
  "total_objects": [
    {
      "count": 1,
      "object_class": "cat"
    }
  ]
}
```

### 2. Raw Predictions
```bash
POST /predict
```
Parameters:
- `file`: Image file
- `threshold`: Confidence threshold (0.0-1.0)

Response:
```json
{
  "predictions": [
    {
      "class_name": "cat",
      "score": 0.999190748,
      "box": {
        "xmin": 0.367288858,
        "ymin": 0.278333426,
        "xmax": 0.735821366,
        "ymax": 0.6988855
      }
    }
  ]
}
```

## Project Structure
```
object-counter/
├── counter/
│   ├── adapters/
│   │   ├── count_repo.py
│   │   ├── count_repo_postgres.py
│   │   └── object_detector.py
│   ├── domain/
│   │   ├── actions.py
│   │   ├── models.py
│   │   ├── ports.py
│   │   └── predictions.py
│   └── entrypoints/
│       └── webapp.py
├── tests/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── setup.sh
```

## Development

### Running Tests
```bash
pytest
```

### Development Mode
```bash
export ENV=dev
python -m counter.entrypoints.webapp
```

## Troubleshooting

### Common Issues

1. **TensorFlow Serving Connection Error**
   - Verify TensorFlow container is running: `docker ps`
   - Check logs: `docker logs tfserving`

2. **Database Connection Issues**
   - Verify correct DB_TYPE environment variable
   - Check database container status
   - Verify connection parameters

3. **Model Loading Error**
   - Ensure model files are in correct location
   - Check TensorFlow Serving logs


This README.md now provides:
1. Complete setup instructions
2. API documentation
3. Project structure
4. Environment configuration
5. Troubleshooting guides
6. Development instructions

