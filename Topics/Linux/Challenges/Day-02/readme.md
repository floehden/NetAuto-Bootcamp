# Day 02
## Introduction
Using export, echo and docker

## Prerequisites
* Docker
* docker-compose

## Challenge


### Challenge 1

At first we want to use environment variables. For this we use the export command.
```sh
export NAME="Alice"
export AGE=25
```

We can display them again with using echo and the $<env-variable-name>
```sh
echo $NAME
echo $AGE
```


### Challenge 2
In the second challenge we use them in a web service.</br>
</br>
Run the following commands

```sh
# Build the image
docker build -t go-env-server .

# Run the container with custom environment variables
docker run -p 8080:8080 -e NAME=${NAME} -e AGE=${AGE} go-env-server 
```
Go in your Browser to http://localhost:8080

we can stop the server with Crtl + C

### Challenge 3
a docker-compose challange




