---
layout: default
title: 5. Image Building Best Practises
---

Here is some useful information and best practises for Docker Images and Image Building.

## Image Layering

* A Docker image is built up from a series of layers. 
* Each layer represents an instruction in the image’s Dockerfile. 
* Each layer except the very last one is read-only. 
* Each layer is only a set of differences from the layer before it. Note that both adding, and removing files will result in a new layer.
* The layers are stacked on top of each other. 
* A method called [union mounting](https://en.wikipedia.org/wiki/Union_mount) is used to combine these layers into one filesystem.

You can look at what makes up an image using the `docker image history` command. You can see the instruction that was used to create each layer within an image.

Use the `docker image history` command to see the layers in the todo-app image you created earlier in the tutorial.

```
docker image history todo-app
```

You should get output that looks something like this (dates/IDs may be different).

```
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
f78b96b14576   6 hours ago    /bin/sh -c #(nop)  CMD ["node" "src/index.js…   0B        
c8b30f41e4a6   6 hours ago    /bin/sh -c #(nop)  EXPOSE 3000                  0B        
c8c8e3943f9b   6 hours ago    /bin/sh -c yarn install --production            83.8MB    
f73ddab641af   6 hours ago    /bin/sh -c #(nop) COPY dir:af1af34f90a8195a8…   58.1MB    
738ef65ba08b   23 hours ago   /bin/sh -c #(nop) WORKDIR /app                  0B        
62f34aa2c196   23 hours ago   /bin/sh -c apk add --no-cache python2 g++ ma…   223MB     
1b156b4c3ee8   8 weeks ago    /bin/sh -c #(nop)  CMD ["node"]                 0B        
<missing>      8 weeks ago    /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B        
<missing>      8 weeks ago    /bin/sh -c #(nop) COPY file:4d192565a7220e13…   388B      
<missing>      8 weeks ago    /bin/sh -c apk add --no-cache --virtual .bui…   7.84MB    
<missing>      8 weeks ago    /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.17     0B        
<missing>      8 weeks ago    /bin/sh -c addgroup -g 1000 node     && addu…   77.6MB    
<missing>      8 weeks ago    /bin/sh -c #(nop)  ENV NODE_VERSION=12.22.10    0B        
<missing>      4 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B        
<missing>      4 months ago   /bin/sh -c #(nop) ADD file:9233f6f2237d79659…   5.59MB    
```

Each of the lines represents a layer in the image. The display here shows the layers that are part of the base image (`FROM node:12-alpine`) at the bottom (the lines where IMAGE is missing plus IMAGE=1b156b4c3ee8) and the newest layer at the top. Using this, you can also quickly see the size of each layer, helping diagnose large images.

## Layer Caching

Now that you've seen the layering in action, there's an important lesson to learn to help decrease build times for your container images.

> Once a layer changes, all downstream layers have to be recreated as well

Let's look at the Dockerfile we were using one more time...

```
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
EXPOSE 3000
CMD ["node", "src/index.js"]
```

Going back to the image history output, we see that **each command in the Dockerfile becomes a new layer in the image**. 

You might remember that when we made a very small change to the image (we simply changed a string in one file), the Node.js dependencies had to be reinstalled which takes a long time. Is there a way to fix this? 

To fix this, we need to restructure our Dockerfile to help support the caching of the dependencies. For Node-based applications, those dependencies are defined in the package.json file. So, what if we copied only that file in first, install the dependencies, and then copy in everything else? Then, we only recreate the yarn dependencies if there was a change to the package.json. Make sense?

1. Update the Dockerfile to copy in the `package.json` (and yarn.lock) first, install dependencies, and then copy everything else in.

   ```
   FROM node:12-alpine
   RUN apk add --no-cache python2 g++ make
   EXPOSE 3000
   WORKDIR /app
   COPY package.json yarn.lock ./
   RUN yarn install --production
   COPY . .
   CMD ["node", "src/index.js"]
   ```

2. Create a file named `.dockerignore` in the same folder as the Dockerfile with the following contents:

   ```
   node_modules
   ```

    `.dockerignore` files are an easy way to selectively copy only image relevant files. You can read more about this [here](https://docs.docker.com/engine/reference/builder/#dockerignore-file). In this case, the local `node_modules` folder should be omitted in the second `COPY` step because otherwise, it would possibly overwrite files which were created by the command in the RUN step. 
    
    For further details on why this is recommended for Node.js applications and other best practices, have a look at their guide on [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/).

    If you develop in Python, I found this [blog](https://www.docker.com/blog/containerized-python-development-part-1/) for you.

3. Build a new image.

    ```
    docker build -t todo-app .
    ```

    You should see output like this...

    ```
    Sending build context to Docker daemon  4.642MB
    Step 1/8 : FROM node:12-alpine
    ---> 1b156b4c3ee8
    Step 2/8 : RUN apk add --no-cache python2 g++ make
    ---> Using cache
    ---> fbb1cd3d25c1
    Step 3/8 : EXPOSE 3000
    ---> Running in 486f4a802a1d
    Removing intermediate container 486f4a802a1d
    ---> 7bd84b5c203a
    Step 4/8 : WORKDIR /app
    ---> Running in d548cf01b62c
    Removing intermediate container d548cf01b62c
    ---> 53567a4244f7
    Step 5/8 : COPY package.json yarn.lock ./
    ---> f3c4768c34c1
    Step 6/8 : RUN yarn install --production
    ---> Running in 594b1ea3bc04
    yarn install v1.22.17
    [1/4] Resolving packages...
    warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^2.0.0"
    warning Resolution field "ansi-regex@5.0.1" is incompatible with requested version "ansi-regex@^3.0.0"
    warning sqlite3 > node-gyp > tar@2.2.2: This version of tar is no longer supported, and will not receive security updates. Please upgrade asap.
    warning sqlite3 > node-gyp > request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
    warning sqlite3 > node-gyp > request > har-validator@5.1.5: this library is no longer supported
    warning sqlite3 > node-gyp > request > uuid@3.4.0: Please upgrade  to version 7 or higher.  Older versions may use Math.random() in certain circumstances, which is known to be problematic.  See https://v8.dev/blog/math-random for details.
    [2/4] Fetching packages...
    [3/4] Linking dependencies...
    [4/4] Building fresh packages...
    success Saved lockfile.
    Done in 13.74s.
    Removing intermediate container 594b1ea3bc04
    ---> 6c36ed52bcaa
    Step 7/8 : COPY . .
    ---> 984157ecbeee
    Step 8/8 : CMD ["node", "src/index.js"]
    ---> Running in 1c3defcd9716
    Removing intermediate container 1c3defcd9716
    ---> 1dc587135308
    Successfully built 1dc587135308
    Successfully tagged todo-app:latest
    ```

    You'll see that most layers were rebuilt. Perfectly fine since we changed the Dockerfile quite a bit. 

4. Now, make a change to the `src/static/index.html` file (e.g. change the <title> to say "The Awesome Todo App").

5. Rebuild the Docker image again using `docker build -t todo-app .` again. This time, your output should look a little different.

    ```
    Sending build context to Docker daemon  4.642MB
    Step 1/8 : FROM node:12-alpine
    ---> 1b156b4c3ee8
    Step 2/8 : RUN apk add --no-cache python2 g++ make
    ---> Using cache
    ---> fbb1cd3d25c1
    Step 3/8 : EXPOSE 3000
    ---> Using cache
    ---> 7bd84b5c203a
    Step 4/8 : WORKDIR /app
    ---> Using cache
    ---> 53567a4244f7
    Step 5/8 : COPY package.json yarn.lock ./
    ---> Using cache
    ---> f3c4768c34c1
    Step 6/8 : RUN yarn install --production
    ---> Using cache
    ---> 6c36ed52bcaa
    Step 7/8 : COPY . .
    ---> e05a0675d218
    Step 8/8 : CMD ["node", "src/index.js"]
    ---> Running in ec937d594973
    Removing intermediate container ec937d594973
    ---> a1f2fb9dbec3
    Successfully built a1f2fb9dbec3
    Successfully tagged todo-app:latest
    ```

    First off, you should notice that the build was MUCH faster! And, you'll see that steps 2-6 all have 'Using cache'. So we are using the build cache. Pushing and pulling this image and updates to it will be much faster as well. 

## Multi-Stage Builds

While we are not going to dive into it too much in this tutorial, multi-stage builds are an incredibly powerful tool to help use multiple stages to create an image. There are several advantages for them:

* Separate build-time dependencies from runtime dependencies
* Reduce overall image size by shipping only what your app needs to run

### Maven/Tomcat Example

When building Java-based applications, a JDK is needed to compile the source code to Java bytecode. However, that JDK isn't needed in production. Also, you might be using tools like Maven or Gradle to help build the app. Those also aren't needed in our final image. Multi-stage builds help.

```
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps 
```

In this example, we use one stage (called `build`) to perform the actual Java build using Maven. In the second stage (starting at `FROM tomcat`), we copy in files from the build stage. The final image is only the last stage being created (which can be overridden using the `--target` flag).

### React Example

When building React applications, we need a Node environment to compile the JS code (typically JSX), SASS stylesheets, and more into static HTML, JS, and CSS. If we aren't doing server-side rendering, we don't even need a Node environment for our production build. Why not ship the static resources in a static nginx container?

```
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

Here, we are using a `node:12` image to perform the build (maximizing layer caching) and then copying the output into an nginx container. 

## Recap

By understanding a little bit about how images are structured, we can build images faster and ship fewer changes. Multi-stage builds help us reduce overall image size and increase final container security by separating build-time dependencies from runtime dependencies.    

**Congratulations!** This concludes the workshop! You may want to have a looks at the last topic:

---

**Last Topic:** [Tips and useful commands](lab6.md) 