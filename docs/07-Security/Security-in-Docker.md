La sécurité dans docker

Partons du principe que nous avons un host avec Docker installé.

Cette host à déjà un nombre de process divers en cours, maintenant, lancçons un container Ubuntu qui va faire un sleep de 3600 secondes

```
docker run ubuntu sleep 3600
```

Nous savons que les coantiner ne sont pas complétement isolé de l'host comme une machine virtualle. Le container et l'host partage le même kernel.

Containers are isolated using namespaces.

In Linux, the host has a namespace and the containers have their own namespace.

All the processes run by the containers are in fact run on the host itself, but in their own namespace.

As far as the Docker container is concerned, it is in its own namespace and it can see its own processes

only.

It cannot see anything outside of it or in any other namespace.

So when you list the processes from within the Docker container, you see the sleep process with a process

ID of one for the Docker host, all processes of its own, as well as those in the child.

```
$ ps aux
USER   PID  %CPU  %MEM   VSZ   RSS  TTY  STAT  START   TIME  COMMAND
root     1   0.0   0.0  4528   828  ?    Ss    03:06   0:00  sleep 3600
```
Namespaces are visible as just another process in the system.

So when you list the processes on the host, you see a list of processes, including the sleep command,

but with a different process ID.

This is because the processes can have different process IDs in different namespaces.

And that's how Docker isolates containers within a system.

So that's process isolation.

```
$ ps aux
USER     PID   %CPU  %MEM    VSZ   RSS  TTY    STAT  START   TIME  COMMAND
project  3720   0.1   0.1  95500  4916  ?      R     06:06   0:00  sshd: project@pts/0
project  3725   0.0   0.1  95196  4132  ?      S     06:06   0:00  sshd: project@notty
project  3727   0.2   0.1  21352  5340  pts/0  Ss    06:06   0:00  -bash
root     3802   0.0   0.0   8924  3616  ?      Sl    06:06   0:00  docker-containerd-shim -namespace m
root     3816   1.0   0.0   4528   828  ?      Ss    06:06   0:00  sleep 3600
```

Let us now look at users in context of security.

The Docker host has a set of users A root user as well as a number of Non-root users.

By default, Docker runs processes within containers as the root user.

This can be seen in the output of the commands we ran earlier, both within the container and outside

the container on the host.

The process is run as the root user.

Now, if you do not want the process within the container to run as the root user, you may set the

user using the user option within the docker run command and specify the new user ID.

You will see that the process now runs with the new user ID.

```
docker run --user=1001 ubuntu sleep 3600
```

Another way to enforce user security is to have this defined in the Docker image itself at the time

of creation.


For example, we will use the default ubuntu image and set the user id to 1000 using the user instruction.

Then build the custom image.

We can now run this image without specifying the user id, and the process will be run with the user id 1000

```
FROM UBUNTU

USER 1000
```


```
$ docker build -t my-ubuntu-image .
```

```
$ docker run my-ubuntu-image sleep 3600
```

```
$ ps aux
USER   PID  %CPU  %MEM   VSZ   RSS  TTY  STAT  START   TIME  COMMAND
1000     1   0.0   0.0  4528   828  ?    Ss    03:06   0:00  sleep 3600
```

What happens when you run containers as the root user?

Is the root user within the container the same as the root user on the host?

Can the process inside the container do anything that the root user can do on the system?

If so, isn't that dangerous?

Well, Docker implements a set of security features that limits the abilities of the root user within

the container.

So the root user within the container isn't really like the root user on the host.

Docker uses Linux capabilities to implement this.

The root user can literally do anything, and so does a process run by the root user.

It has unrestricted access to the system from modifying files and permissions on files, access control,

creating or killing processes, setting group ID or user ID.

Performing network related operations such as binding to network ports.

Broadcasting on a network.

controlling network ports system related operations like rebooting the host, manipulating system clock,

and many more.

Linux Capabilities :
CHOWN   
DAC   
KILL   
SETFCAP   
SETPCAP   
SETGID   
SETUID   
NET_BIND   
NET_RAW   
MAC_ADMIN   
BROADCAST   
NET_ADMIN   
SYS_ADMIN   
SYS_CHROOT   
AUDIT_WRITE   
MANY MORE...

All of these are the different capabilities on a Linux system, and you can see a full list at this

location:

/usr/include/linux/capability.h


You can now control and limit what capabilities are made available to a user.

By default, Docker runs a container with a limited set of capabilities, and so the processes running

within the container do not have the privileges to say, reboot the host or perform operations that

can disrupt the host or other containers running on the same host.

If you wish to override this behavior and provide additional privileges than what is available, use

the **--cap-add** option in the docker run command.

```
docker run --cap-add MAC_ADMIN ubuntu
```

Similarly, you can drop privileges as well using the **--cap-drop** option 

```
docker run --cap-add KILL ubuntu
```

or in case you wish to run the container with all **--privileges** enabled, use the privileged flag. 
