# Additional notes
## Creating and using containers
### Assignment Answers: DNS Round Robin Testing
1. Use an updated Elasticsearch image and limit its Java memory:
NOTE: This no longer works on Play-With-Docker due to memory limits. If using PWD, use Option 2.

Use a more updated version of elasticsearch, but you'll have to set memory requirements and change some default environment variables to make it work as a simple single container. Remember, you'll also want to change the network and alias for this assignment, but you can add the -e options similar to this: `docker run -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "xpack.security.enabled=false" --network <your_network> -d --network-alias search elasticsearch:8.4.3`

Then you could start another container to run nslookup and curl. Here's the full answer:
```
# create the bridge network
docker network create dude-net
# create multiple containers on the same network with the same DNS alias
docker run -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "xpack.security.enabled=false" --network dude-net -d --network-alias search elasticsearch:8.18.2
docker run -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "xpack.security.enabled=false" --network dude-net -d --network-alias search elasticsearch:8.18.2
# use alpine, which has nslookup built-in and you can use apk to add curl
docker run --rm -it --network dude-net alpine
# use nslookup to see both container IPs for the same DNS name
nslookup search
# install curl and run it 2-5 times to see different cluster_uuid for each container
# this shows that in the background, docker is delivering both IPs for the DNS name
# and that curl is rotating between them in round-robin style
apk add curl
curl -s search:9200
curl -s search:9200
```

You'll notice the new elasticsearch no longer has random **cluster_name**, so you'll need to look at **cluster_uuid** to tell them apart.

2. Replace Elasticsearch with httpenv, a simple HTTP server:
You can replace the elasticsearch image name with the much simpler `bretfisher/httpenv` (which runs on port 8888, not 9200) and get the same effect in this assignment. It's just a simple web server that returns the environment variables (including the container hostnames, which are their Docker ID's, to tell them apart) in HTTP.


```
# create the bridge network
docker network create dude-net
# create multiple containers on the same network with the same DNS alias
docker container run -d --network dude-net --net-alias search bretfisher/httpenv
docker container run -d --network dude-net --net-alias search bretfisher/httpenv
docker container run -d --network dude-net --net-alias search bretfisher/httpenv
# Curl the DNS alias, notice the different HOSTNAMES in the response from httpenv (it returns the container ID as a env var)
docker container run --rm --network dude-net rockylinux/rockylinux:10 curl -s search:8888
docker container run --rm --network dude-net rockylinux/rockylinux:10 curl -s search:8888
docker container run --rm --network dude-net rockylinux/rockylinux:10 curl -s search:8888
```

#### More Info on elasticsearch changes
The *only* reason I used elasticsearch in this demo was because 1. it uses HTTP, which we can easily curl for a response, and 2. Because each elasticsearch instance has a random ID, we can see that in a HTTP response to prove that multiple curl commands cause Docker to round-robin those connections to different containers.

I was using the older :2 tag for elasticsearch because newer versions use up *a lot* of memory and require complex environment variables to tune it for a typical laptop or Play With Docker.

However, the :2 tag for elasticsearch is old and only works on x86_64 machines. The older v2  doesn't have an arm64 build (M1, M2, Apple Silicon, Raspberry Pi, etc.) It's one of the last lectures for me to replace with an image that has an arm64 variant as well as the typical x86_64, so if you're on a arm64 machine, you can use one of these two options to finish the assignment: