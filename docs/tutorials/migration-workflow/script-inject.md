---
layout: default
title: "Custom transformer for injecting a script into the container image"
permalink: /tutorials/migration-workflow/script-inject
parent: "Migration workflow"
grand_parent: Tutorials
nav_order: 4
---

# Custom transformer for injecting a script into the container image

## Big picture

In this example, we illustrate how we could include our own scripts into the container image that could be built using the docker file generated by Move2Kube. 

1. Let us start by creating a workspace directory `WORKSPACE_DIR`. We will assume all commands are run within this workspace and all sub-directories are created within this workspace.

2. To dump the input at an accessible location, create an input sub-directory `input` and copy the `e2e-demo` into this folder.

3. To see what we get **without** any customization, let us run `move2kube transform -s input/ --qa-skip`. The output dockerfile generated in the path  `myproject/source/e2e-demo/frontend/Dockerfile` looks like this:
```
FROM registry.access.redhat.com/ubi8/nodejs-12
COPY . .
RUN npm install
RUN npm run build
EXPOSE 8080
CMD npm run start
```
and the enclosing folder `myproject/source/e2e-demo/frontend/` does not contain anything but the source files of the `frontend` service and the Dockerfile mentioned above:
```
myproject/source/e2e-demo/frontend/
├── Dockerfile
├── LICENSE
├── README.md
├── __mocks__
│   ├── fileMock.js
│   └── styleMock.js
├── dr-surge.js
├── jest.config.js
├── nodemon.json
├── package-lock.json
├── package.json
├── server.js
├── src
│   ├── app
│   ├── favicon.png
│   ├── index.html
│   ├── index.tsx
│   └── typings.d.ts
├── stories
│   ├── Dashboard.stories.tsx
│   └── Support.stories.tsx
├── stylePaths.js
├── test-setup.js
├── tsconfig.json
├── webpack.common.js
├── webpack.dev.js
└── webpack.prod.js
```

Let us say, we want to add a simple start-up script (shown below) into the container image, and make the container execute this script while it is brought up:
```
echo "[Injected script]: Starting the application"
npm run start
```
This can be achieved by customizing Move2Kube using custom transformer that is available in [here](https://github.com/konveyor/move2kube-transformers/tree/main/script-inject).

4. To use this custom transformer, create a `customizations` sub-directory and copy the `script-inject` from [here](https://github.com/konveyor/move2kube-transformers/tree/main/script-inject) into this folder.

5. Run the following CLI command: `move2kube transform -s input/ -c customizations/ --qa-skip`. Note that this command has a `-c customizations/` option which is meant to tell Move2Kube to pick up the custom transformer `script-inject`. 

Once the output is generated, we can observe the same dockerfile mentioned before i.e. `myproject/source/e2e-demo/frontend/Dockerfile` contains the custom image. The contents are shown below:
```
FROM registry.access.redhat.com/ubi8/nodejs-12
COPY . .
RUN npm install
RUN npm run build
EXPOSE 8080
CMD sh hello-nodejs.sh
```
and the output folder `myproject/source/e2e-demo/frontend/` now contains the `hello-nodejs.sh` which did not exist before.
```
myproject/source/e2e-demo/frontend/
├── Dockerfile
├── LICENSE
├── README.md
├── __mocks__
│   ├── fileMock.js
│   └── styleMock.js
├── dr-surge.js
├── hello-nodejs.sh
├── jest.config.js
├── nodemon.json
├── package-lock.json
├── package.json
├── server.js
├── src
│   ├── app
│   ├── favicon.png
│   ├── index.html
│   ├── index.tsx
│   └── typings.d.ts
├── stories
│   ├── Dashboard.stories.tsx
│   └── Support.stories.tsx
├── stylePaths.js
├── test-setup.js
├── tsconfig.json
├── webpack.common.js
├── webpack.dev.js
└── webpack.prod.js
```
## Anatomy of `script-inject` transformer
If you are wondering where did `hello-nodejs.sh` come from, it can be see in the custom transformer folder that was provided to Move2Kube. Contents of this folder are shown below:
```
script-inject/
├── nodejs.yaml
└── templates
    ├── Dockerfile
    └── hello-nodejs.sh
```
Any file that is put in the `templates` sub-directory seen above, would get transferred to output and eventually into the container image when the docker file is used to build the image. As a side note, `script-inject` transformer is generating a custom dockerfile for any NodeJS service detected within `e2e-demo` project.

In the above transformer folder, the `nodejs.yaml` is the configuration for the transformer. As can be seen from the contents below, the name of our custom transformer is `Nodejs-ScriptInject` (see `name` field in the `metadata` section). In the specification section, let us look at the important fields of interest:
- Specifying that we are going to use an in-built transformer class (see `class` field in `spec` section) `NodejsDockerfileGenerator` as the basis of our transformer (Note here that there could different **transformers** implementing the same **transformer class**. The difference could be in the way they are configured.)
- we are asking Move2Kube to pass artifacts of type `Service` to this transformer (see `consumes` field).
- we are stating that the output produced by this transformer is a `Dockerfile` (see `produce` field). 
- We are also specifying an `override` section which is asking Move2Kube to override the in-built transformer named `Nodejs-Dockerfile` and use our custom transformer in its place.
```
apiVersion: move2kube.konveyor.io/v1alpha1
kind: Transformer
metadata:
  name: Nodejs-ScriptInject
  labels:
    move2kube.konveyor.io/task: containerization
    move2kube.konveyor.io/inbuilt: true
spec:
  class: "NodejsDockerfileGenerator"
  directoryDetect:
    levels: -1
  consumes:
    Service: 
      merge: false
  produces:
    Dockerfile:
      disabled: false
    DockerfileForService:
      disabled: false
  override:
    matchLabels: 
      move2kube.konveyor.io/name: Nodejs-Dockerfile
  config:
    defaultNodejsVersion: "12"
```
Also, note that `Dockerfile` template in the `templates` folder is modified to include the shell script as start-up script:
```
FROM registry.access.redhat.com/ubi8/nodejs-{{ .NodeVersion }}
COPY . .
RUN npm install
{{- if .Build }}
RUN npm run build
{{- end}}
EXPOSE {{ .Port }}
CMD sh hello-nodejs.sh
```

Next step [Custom Helm-chart Generator](/tutorials/migration-workflow/custom-helmchart-gen)