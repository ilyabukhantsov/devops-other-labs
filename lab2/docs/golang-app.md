
```
FROM golang:1.22

WORKDIR /app

COPY . .

RUN go build -o fizzbuzz

CMD ["./fizzbuzz", "serve"]
```

```
[+] Building 44.8s (9/9) FINISHED 
➜  deploy.lab-containers-starter-project-golang git:(main) ✗ sudo docker run -it golang-app sh 
# ls -la    
total 10688
drwxr-xr-x 1 root root     4096 Apr 23 14:29 .
drwxr-xr-x 1 root root     4096 Apr 23 14:31 ..
drwxr-xr-x 1 root root     4096 Apr 23 14:29 .git
-rw-r--r-- 1 root root     2415 Apr 23 14:09 .gitignore
-rw-r--r-- 1 root root       95 Apr 23 14:28 Dockerfile
-rw-r--r-- 1 root root     1073 Apr 23 14:09 README.rst
drwxr-xr-x 2 root root     4096 Apr 23 14:09 cmd
-rwxr-xr-x 1 root root 10817773 Apr 23 14:29 fizzbuzz
-rw-r--r-- 1 root root      181 Apr 23 14:09 go.mod
-rw-r--r-- 1 root root    75705 Apr 23 14:09 go.sum
drwxr-xr-x 2 root root     4096 Apr 23 14:09 lib
-rw-r--r-- 1 root root      119 Apr 23 14:09 main.go
drwxr-xr-x 2 root root     4096 Apr 23 14:09 templates
```


```
FROM golang:1.22 AS builder

WORKDIR /app
COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -o fizzbuzz

FROM scratch

COPY --from=builder /app/fizzbuzz /fizzbuzz

EXPOSE 8080

CMD ["/fizzbuzz", "serve"]
```

```
[+] Building 13.0s (10/10) FINISHED
golang-app:latest   3d78dce4c5b6       16.7MB         5.98MB    U  
```


```
[+] Building 19.0s (12/12) FINISHED    
golang-app:latest   3cb0f705966b       49.7MB         14.1MB    U   
```