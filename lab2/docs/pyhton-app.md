# Python part

## First Dockerfile

```
FROM python:3.7

WORKDIR /app

COPY requirements/backend.in ./requirements.txt

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080

CMD ["uvicorn", "spaceship.main:app", "--host=0.0.0.0", "--port=8080"] 
```

#### What we did:

1. We used Python 3.7 and set the working directory to /app.
2. We copied requirements.txt and installed all dependencies using pip.
3. We copied the project source code and started the application.

#### First time and size of image:

```
[+] Building 49.4s (10/10) FINISHED   

IMAGE               ID             DISK USAGE   CONTENT SIZE   EXTRA
python-app:latest   5088e84125a8       1.51GB          386MB  

```

#### What we changed:

We added comment in app.py 
```
# New change 
```


#### Second time and size of image:

```
[+] Building 4.4s (10/10) FINISHED 
IMAGE               ID             DISK USAGE   CONTENT SIZE   EXTRA
python-app:latest   85d69da53f0d       1.51GB          386MB    
```

We have already optimized the Dockerfile by installing dependencies first and copying the code afterward.

#### What we changed:

1. We deleted docker image
2. We added light version of python

```
[+] Building 25.2s (10/10) FINISHED 
IMAGE               ID             DISK USAGE   CONTENT SIZE   EXTRA
python-app:latest   2859e8706c10        244MB         58.6MB     
```
#### What we changed:

1. We added numpy
2. We added matrix multiply

```
[+] Building 33.4s (10/10) FINISHED
IMAGE         ID             DISK USAGE   CONTENT SIZE   EXTRA
lab2:latest   fd44715235cc        344MB         80.2MB   
```
