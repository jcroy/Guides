
## Dockerfile

Create a `Dockerfile` in your project root with the following content:

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Set work directory
WORKDIR /app

# Install dependencies
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . /app/
```
Docker-compose.yml
```


version: '3.8'

services:
  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    env_file:
      - ./.env  # Use your .env file for environment variables

  web:
    build: .
    command: bash -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    env_file:
      - ./.env  # Use your .env file for environment variables
    depends_on:
      - db

volumes:
  postgres_data:
```
Ensure you're .env file is set correctly. You will need to set the postgres user data as well. 
```
POSTGRES_USER=yourusername
POSTGRES_PASSWORD=yourpassword
POSTGRES_DB=yourdbname
```
When you're ready to launch it it: docker-compose up --build
