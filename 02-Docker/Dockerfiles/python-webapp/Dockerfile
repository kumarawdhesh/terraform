FROM python:2.7-slim

WORKDIR /app

ADD ./code /app

RUN pip install --trusted-host pypi.python.org -r req.txt

EXPOSE 5001

CMD ["python", "app.py"]
