FROM python:3.10-slim-buster

WORKDIR /workspace

COPY ./requirements.txt  requirements.txt
RUN apt update -y && apt install -y build-essential libpq-dev 
RUN pip install --upgrade pip setuptools wheel &&  pip install -r requirements.txt

COPY ./app.py app.py
COPY ./config.py config.py

CMD python app.py
EXPOSE 5153
