FROM python:3.11-slim 
WORKDIR /app 
COPY requirements.txt . 

RUN apt-get update && apt-get install -y --no-install-recommends \
     gcc \
     g++ \ 
     python3-dev \ 
     && rm -rf /var/lib/apt/lists/* 

RUN pip install --no-cache-dir --upgrade pip setuptools wheel
RUN pip install --no-cache-dir -r requirements.txt 
COPY . . 
ENTRYPOINT ["python", "recommendation_server.py"]