# Assignment 01
I am not sure yet on which machine I will deploy the scala docker as I do not know yet which kind of computational power is best suited, multi core, but low clock speed or less cores with higher clock speeds. The way I plan to include my scala files is a little different as copying the files to the docker container seems rediculus to me. Docker is designed to be stateless so copying to the docker needs to be done everytime the docker is restarted. My solution to this is volume mounts from my local machine that allow me to edit the file and use the saved file instantly within the container.
My docker run looks like this:
```bash
docker rm -f scala
docker run -it \
  --name scala \
  -v /home/josh/uni/bigdata_homework/scala-files:/scala-files \
  williamyeh/scala /bin/bash
```
