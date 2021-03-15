# Assignment 02
In this assignment I use the following docker image:
`rubigdata/course:a2`

In order to start the container with name "hello-hadoop" and ports 8088 and 9870 forwarded to the host the following command is given:
`docker create --name hello-hadoop -it -p 8088:8088 -p 9870:9870 rubigdata/course:a2`

For my convenience I add a volume mount. The volume mount allows me to edit the code files on Windows using my favorite text editor, while staying in the docker CLI of the docker container and not needing to copy files from the host to the container, but only inside the container.
`docker create --name hello-hadoop -it -p 8088:8088 -p 9870:9870 -v /home/josh/uni/big-data/hello-hadoop-2021-joshdev-de/:/mnt/shared:ro rubigdata/course:a2`

To start and attach to the container's CLI I use.
`docker start hello-hadoop`
`docker attach hello-hadoop`

Now that I am in the container I can just copy the .java file I want to use from `/mnt/shared` to `/opt/hadoop` using `cp`
