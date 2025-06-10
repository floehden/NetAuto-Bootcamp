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
Think about your own variables and pass them to the go-env-server!


## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%202%20of%20the%20NetAuto%20Bootcamp%20on%20Linux!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @NetAutoGroupRM (or something else)



