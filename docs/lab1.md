---
layout: default
title: 1. Deploy ToDo stand-alone
---

We will use a simple todo list app throughout this workshop. It is written in Node.js. You downloaded the code in the [Get the Code](../index.md#get-the-code) section previously.

At this point, your development team is quite small and you're simply building an app to prove out your MVP (minimum viable product). You want to show how it works and what it's capable of doing without needing to think about how it will work for a large team, multiple developers, etc.

![Todo App](images/todo-app.png)

## Building the Container Image

In order to build the application, we need to use a Dockerfile. A Dockerfile is simply a text-based script of instructions that is used to create a container image. 

1. Create a file named Dockerfile **in the same folder as the file package.json** with the following contents.

   ```
   FROM node:12-alpine
   RUN apk add --no-cache python2 g++ make
   WORKDIR /app
   COPY . .
   RUN yarn install --production
   EXPOSE 3000
   CMD ["node", "src/index.js"]
   ```

   > Please check that the file Dockerfile has no file extension like .txt. Some editors may append this file extension automatically and this would result in an error in the next step.

   What do the commands in the Dockerfile mean?

   <dl>
   <dt>FROM</dt>
     <dd>This the base image you are building upon</dd>
   <dt>RUN</dt>
     <dd>This command will be executed during the BUILD of the container image</dd>
   <dt>WRKDIR</dt>
     <dd>This will be the working directory in the container image</dd>
   <dt>COPY</dt>
     <dd>Copy data from your notebook into the container image</dd>
   <dt>EXPOSE</dt>
     <dd>This is the port your application is using</dd>
   <dt>CMD</dt>
     <dd>This command starts your application when the container starts</dd>   
   </dl>

2. If you haven't already done so, open a terminal and go to the app directory with the Dockerfile. Now build the container image using the docker build command.

   ```
   docker build -t getting-started .
   ```

   This command used the Dockerfile to build a new container image. You might have noticed that a lot of "layers" were downloaded. This is because we instructed the builder that we wanted to start FROM the node:12-alpine image. But, since we didn't have that on our machine, that image needed to be downloaded.

   After the image was downloaded, we copied in our application and used yarn to install our application's dependencies. The CMD directive specifies the default command to run when starting a container from this image.

   Finally, the -t flag 'tags' our image. Think of this simply as a human-readable name for the final image. Since we named the image getting-started, we can refer to that image when we run a container.

   Don't forget the . at the end of the docker build command! It tells that Docker should look for the Dockerfile in the current directory.

## Starting an App Container

Now that we have an image, let's run the application! To do so, we will use the docker run command.

1. Start your container using the docker run command and specify the name of the image we just created:

   ```
   docker run -dp 3000:3000 getting-started
   ```

   What are the -d and -p flags?

   <dl>
   <dt>-d</dt>
     <dd>This is a server type application. Run the new container in "detached" mode = in the background.</dd>
   <dt>-p</dt>
     <dd>Map the host's port 3000 to the container's port 3000. Without the port mapping, we wouldn't be able to access the application.</dd>
   </dl>    
   
   Note: the first port number (before the ':') is the host port on your notebook etc. If this port is already taken by another app, you could try another port. 
   
   The second port (after the ':') is the container port. How do we know that the container port is 3000? If you look in file `source/index.js`  you can find this statement:

   ```
   app.listen(3000, () => console.log('Listening on port 3000'));
   ```

   So the ToDo app is listening on port 3000. And in the Dockerfile we specified `EXPOSE 3000` which means we tell Docker the container listens on the specified network port at runtime.


2. After a few seconds, open your web browser to http://localhost:3000. You should see our app!

   ![Empty Todo List](images/empty-todo-app.png)

   Go ahead and add an item or two and see that it works as you expect. You can mark items as complete and remove items. Your frontend is successfully storing items in the backend! 

   You can see your running container in Docker Dashboard (Mac, Windows) or via commandline:

   ```
   docker ps
   ```

   Result:

   ```
   CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS          PORTS                                       NAMES
   b46846361370   getting-started          "docker-entrypoint.s…"   6 minutes ago    Up 6 minutes    0.0.0.0:3000->3000/tcp, :::3000->3000/tcp   focused_kepler
   ```

---

**Next Step:** [Update the app, build a new image](lab2.md) 