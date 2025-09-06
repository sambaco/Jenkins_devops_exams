FROM nginx:latest

WORKDIR /app

COPY ./requirements.txt /app/requirements.txt

RUN apt-get update \
    && apt-get install gcc -y \
    && apt-get clean

RUN pip install -r /app/requirements.txt \
    && rm -rf /root/.cache/pip

COPY . /app/


EXPOSE 8000

CMD ["uvicorn", "main:app","--reload", "--host", "0.0.0.0", "--port", "8000"]
