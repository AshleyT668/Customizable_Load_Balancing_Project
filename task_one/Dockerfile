FROM python:3.9-slim

WORKDIR /app

COPY . /app

RUN pip install -r requirements.txt

ENV SERVER_ID=default

EXPOSE 5000

CMD ["python", "server.py"]
