FROM python:3.8-slim
<<<<<<< HEAD

WORKDIR /app

COPY ./requirements.txt /app/requirements.txt

RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean

RUN pip install -r /app/requirements.txt \
    && rm -rf /root/.cache/pip

COPY . /app/
=======
WORKDIR /app
COPY ./requirements.txt /app/requirements.txt
RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean
RUN pip install -r /app/requirements.txt \
    && rm -rf /root/.cache/pip
COPY . /app/
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
>>>>>>> 718e5f5 (Mon commit v.1.0)
