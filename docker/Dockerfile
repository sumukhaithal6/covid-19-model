FROM python:3.6

WORKDIR /usr/src/app

COPY requirements.txt ./

RUN pip install --no-cache-dir -r requirements.txt

# Don't put this in requirements.txt. It fails because of cython issues.
RUN pip install pomegranate==0.12.2

CMD [ "python", "-c", "import pomegranate; print(pomegranate.__version__);" ]
