---
title: Lessons I learned from a cab drive about Serverless
author: Camal Cakar
patat:
    wrap: true
    margins:
        left: 0
        right: 10
    theme:
        syntaxHighlighting:
            decVal: [bold, onDullRed]
        bulletListMarkers: '-+'
        emph: [vividWhite, underline]
        header: [vividBlue, bold]
        codeBlock: [vividWhite, bold]
        code: [dullRed, bold]
        quoted: [bold, onDullRed]
...

# We will start when the music stops

# Setting the scene

## Judgment

golang pro tip:
Don’t **blindly apply** dogmatic advice, use your judgment each and every time.
 
- *Jaana B. Dogan*

## Context

Was talking recently with a friend about how hard it is to write blog posts
and give talks these days. The longer you’re in the tech industry the more
your opinions end up being **“well it depends”** and **“I’m not sure”.** 
These don’t quite make for good blog posts, hah.

- *Zach Holman*

# Mixing the elixir of this talk 🍵

## Ingredients

* In a startup or corporate enviroment
* Public cloud is an option
* No realtime critical system
* Serving highly mutable business requirements
* Caring about fast business value delivery

(Web,  
"OpenFaaS",  
"Fn Project")

## No value

```zsh
RFC for 700 HTTP Status Codes
     . . . .
     2.2.  Novelty Implementations . . . . . . . . . . . . .    3
     . . . .

 2.2.  Novelty Implementations
   o  710 - PHP
   o  719 - Haskell
```

(GitHub,  
"7XX-rfc")

# Adopt a "Shall I" not a "Can I" mentality 

## My attempt for this talk

Give you a hard time writing notes about:

. . .

- 🔽 🔽 🔽 🔽 🔽 🔽 🔽 🔽
  - Things Camal did wrong
  - Articles to read
  - Tools to try
  - Video Clips to watch
  - Suggestions to start

# Travel 🗺

## Types of travel

* ✈️

* 🚋
   
* 🚵🏽‍♀️

# Nowadays I like to drive a car 🚗

# Expect in Paris 👨🏽‍🍳🇫🇷

# Giving away responsibility

## Computer Science in a nutshell

From the first programming languages, to object-oriented programming,
to the development of virtualisation and cloud infrastructure, the
history of computer science **is a history of the development of abstractions**
that hide complexity and empower your to build ever more sophisticated applications.

- *Kubernetes - Up & Running*

## Computer Science in a nutshell

```zsh
┌──────────────┬─────┬───────┬──────┬────┬───┐
│ on-premise   │ VM  │ IaaS  │ PaaS │ ?  │ ? │
└──────────────┴─────┴───────┴──────┴────┴───┘
```

## Function-as-a-Service

```zsh
                                                  
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│              │ │ Micro Service│ │     FaaS     │
│              │ │              │ └──────────────┘
│   Monolith   │ └──────────────┘ ┌──────────────┐
│              │                  │     FaaS     │
│              │ ┌──────────────┐ └──────────────┘
│              │ │ Micro Service│ ┌──────────────┐
│              │ │              │ │     FaaS     │
└──────────────┘ └──────────────┘ └──────────────┘
```

## PaaS != FaaS

```zsh
┌────────────────────────┬────────────┬──────────────────────┐
│                        │ PaaS       │ FaaS                 │
┌────────────────────────┬────────────┬──────────────────────┐
│ Startup time           │ Minutes    │ Milliseconds         │
┌────────────────────────┬────────────┬──────────────────────┐
│ Running time           │ 24/7       │ Runs incoming req.   │
┌────────────────────────┬────────────┬──────────────────────┐
│ Cost                   │ Interval   │ Pay for usage        │
┌────────────────────────┬────────────┬──────────────────────┐
│ Unit of Code           │ Monolithic │ Single-purpose func  │
```


## Serverless Bingo!

“Serverless” is **just a name**. We could have called it “Jeff”
- *Jeff Conf*

(Twitter,  
"JeffConf")

## Serverless Bingo!

A Serverless solution is one that costs you nothing to run if
nobody is using it (*excluding data storage*)

## Serverless Bingo!

```zsh
┌────────────────────────┬──────────┬────────────────────┐
│ Functions as a Service + Patterns + 3rd Party Services │
└────────────────────────┴──────────┴────────────────────┘
```

# Embrace third-party services

## Searching for Core

Speed is king.

- *Boxing proverb*

## K8s example

```js
// app.js
const express = require('express')
const app = express()
app.get('/', async (req, res, next) => {
  res.status(200).send('Hello, DDD Cologne!')
})
app.listen(3000, () => console.log('Server is running on port 3000'))
```

## K8s example

```zsh
# Dockerfile
FROM node:alpine

# Create app directory
WORKDIR /usr/src/app

# COPY package.json .
# For npm@5 or later, copy package-lock.json as well
COPY package.json package-lock.json ./

# Install app dependencies
RUN npm install

# Bundle app source
COPY . .

EXPOSE 3000

# Start Node server
CMD [ "npm", "start" ]
```

## K8s example

```zsh
docker build . -t <docker_hub_username>/<image_name>
docker push <docker_hub_username>/<image_name>
```

## K8s example

```zsh
$ export ORGANIZATION_NAME=your-org-name

# create state store
$ export BUCKET_NAME=${ORGANIZATION_NAME}-state-store
$ aws s3api create-bucket \
    --bucket ${BUCKET_NAME} \
    --region eu-central-1 \
    --create-bucket-configuration LocationConstraint=eu-central-1
$ aws s3api put-bucket-versioning \
    --bucket ${BUCKET_NAME} \
    --versioning-configuration Status=Enabled

# create cluster
$ export KOPS_CLUSTER_NAME=${ORGANIZATION_NAME}.k8s.local
$ export KOPS_STATE_STORE=s3://${BUCKET_NAME}

# define cluster configuration
$ kops create cluster \
    --master-count=1 --master-size=t2.micro \
    --node-count=1 --node-size=t2.micro \
    --zones=eu-central-1a \
    --name=${KOPS_CLUSTER_NAME}

# if you want to edit config
$ kops edit cluster --name ${KOPS_CLUSTER_NAME}

# apply and create cluster
$ kops update cluster --name ${KOPS_CLUSTER_NAME} --yes

# validate cluster is running
$ kops validate cluster
```

## K8s example

```zsh
# node-deployment.yml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: node
spec:
  selector:
    matchLabels:
      app: node
      tier: backend
  replicas: 9
  template:
    metadata:
      labels:
        app: node
        tier: backend
    spec:
      containers:
        - name: node
          image: <docker_hub_username>/<image_name>
          ports:
            - containerPort: 3000
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

## K8s example

```zsh
# node-service.yml
apiVersion: v1
kind: Service
metadata:
  name: node
  labels:
    app: node
    tier: backend
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 3000
  selector:
    app: node
    tier: backend
```

## K8s example

```zsh
$ kubectl apply -f node-deployment.yml
$ kubectl apply -f node-service.yml
```

# Done 🎉

## Serverless example

```
# Framework
$ npm i -g serverless 
# Express.js router proxy module
$ npm i serverless-http
```

## Serverless example

```js
// app.js
const express = require('express')
const sls = require('serverless-http')
const app = express()
app.get('/', async (req, res, next) => {
  res.status(200).send('Hello, DDD Cologne!')
})
module.exports.server = sls(app)
```

## Serverless example

```
# serverless.yml
service: express-sls-app

provider:
  name: aws
  runtime: nodejs8.10
  stage: dev
  region: eu-central-1

functions:
  app:
    handler: app.server
    events:
      - http:
          path: /
          method: ANY
      - http:
          path: /{proxy+}
          method: ANY
```

## Serverless example

```zsh
$ serverless deploy
```

# Done 🎉

## Junior Developer thunderstorm

Imagine a new-gen developer that starts her/his career learning
and working on **nothing but AWS Lambda.**

That experience will shape **forever** her/his worldwide and expectations
about how IT works. Everything we know and do today will be irrelevant.

This is the issue.

- *Alessandro Perilli*

# Learning: If you run fast, you can also run fast into the wrong direction

## Project structure (fine-granular)

```zsh
                                                   ┌──────────────┐
                       ┌──────────────┐ ┌──────────└──────────────┘
                       │     API      │/            ┌──────────────┐
┌──────────────┐       │              │─────────────└──────────────┘
│ Incoming req.│------>│   Gateway    │              ┌──────────────┐
└──────────────┘       │              │──────────────└──────────────┘
                       │              │\              ┌──────────────┐
                       └──────────────┘ └─────────────└──────────────┘

```

# Learning: No one really knows how to structure Serverless systems

## Project structure (resource orientated)

```zsh
                                       ┌──────────────┐
       ┌──────────────┐            ┌───└──────────────┘
       │     API      │           /
       │              │   ┌────────┐
------>│   Gateway    │───└────────┘
       │              │           \     ┌──────────────┐
       │              │            └────└──────────────┘
       └──────────────┘
```

# Shameless plug: Careless is coming

## If not controlled, event driven systems will be dangerous ☠️

. . . The danger is that it's very easy to make nicely decoupled systems with event notification, without realizing that you're losing sight of that larger-scale flow, and thus set yourself up for trouble in future years. The pattern is still very useful, but you have to be careful of the trap.

- *Martin Fowler*

## AWS Step Functions to the rescue

A step machine with:

- Visual feedback
- Condition programming

**Already used by Coca Cola shrink down a processing task from 18 hours to 10 seconds!**

(Youtube,  
"AWS re:Invent 2017: State Machines in the Wild! How Customers use AWS Step Functions")

# Learning: There is a big gap between the cloud providers

## Well design through communication

```csharp
Art by Joan Stark
               _.===========================._
            .'`  .-  - __- - - -- --__--- -.  `'.
        __ / ,'`     _|--|_________|--|_     `'. \
      /'--| ;    _.'\ |  '         '  | /'._    ; |
     //   | |_.-' .-'.'    -  -- -    '.'-. '-._| |
    (\)   \"` _.-` /                     \ `-._ `"/
    (\)    `-`    /      .---------.      \    `-`
    (\)           |      ||1||2||3||      |
   (\)            |      ||4||5||6||      |
  (\)             |      ||7||8||9||      |
 (\)           ___|      ||*||0||#||      |
 (\)          /.--|      '---------'      |
  (\)        (\)  |\_  _  __   _   __  __/|
 (\)        (\)   |                       |
 (\\\\jgs\\\)      '.___________________.'
  '-'-'-'--'
```

(YouTube,  
“What is Success? ElmConf”)

## Where are we right now?

New platform capabilities lead to new architectural style and new operational practices

- *Simon Wardley*

(YouTube,  
“Why the fuzz about Serverless”)

## Where are we right now?

**Netflix** -> Cloud *VM-native* architecture  
**Airbnb** -> Cloud *Container-native* architecture  
**?,you?** -> Cloud *Serverless-native* architecture  

(Slideshare,   
"Asher Sterkin")

# Learning: Ignorance fuels frustration

# Sociotechnical (I believe we will hear this term a lot in the next time)

# Learning: Have a trust and safe-to-fail enviroment for the project

# Learning: Don't be afraid of the Black Box just put some observability sugar on it

## W/O OpenCensus

<!-- 
For most of our systems, good observability is simply more important than good testing, because good observability enables smart organizations to focus on fast deployment and rollback, optimizing for mean time to recovery (MTTR) instead of mean time between failure (MTBF). 
-->

```go
package hello

import (
 "fmt"
 "net/http"
)

func HelloWorld(w http.ResponseWriter, r *http.Request) {
 fmt.Fprintf(w, "Hello, DDD Cologne!")
}
```

## W/ OpenCensus

```go
import (
   "fmt"
   "net/http"
   "go.opencensus.io/plugin/ochttp"
)

func HelloWorld(w http.ResponseWriter, r *http.Request) {
    fn := func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Hello, DDD Cologne!")
    }
    traced := &ochttp.Handler{
        Handler: http.HandlerFunc(fn),
    }
    traced.ServeHTTP(w, r)
}
```

# Learning: Have a look at Epsagon DataBird Sensu and others

# Learning: Escape the WebUI ASAP

# Take a look at Serverless APEX SAM

# Learning: Embrace the automation mantra 🧘🏽‍♂️

## Automation mantra

- Document the steps  
- Create automation equivalents  
- Create automation  
- Self-service and autonomous systems

(acmequeue,
"Manual work is a bug" Thomas A. Limoncelli)

# Learning: New platform same rules (Use Env. Variables!)

# Learning: New platform new rules

## Anti-Pattern use the same storage bucket

The 200$ return statement ... just use a different storage bucket every time :)

(Sourcebox,  
"https://sourcebox.be/blog/2017/08/07/serverless-a-lesson-learned-the-hard-way/"
)


# Learning: It doesnt scale automaticlly

## Example

Kinesis is just scaling to a specific limit!

(DevZone,  
"Why It's Important to Try and Break Your Serverless Application")

# A new tribe is coming. Financial x Dev

## Refactoring based on 💸

(runbook,  
"How We Massively Reduced Our AWS Lambda Bill With Go")

# Learning: Architecture documentation is 🔑

## Architecture documentation styles

Arc42 (Dr. Gernot Starke und Peter Hruschka) or C4 (Simon Brown)

# Consensus is poisonous for innovation ☠️

## The slide with all links :)

**Platforms**  
OpenFaaS  
Fn Project  

**Tools**  
Epsagon  
DataBird  
Sensu  
Cloudcraft (AWS only)  

**Frameworks**  
Serverless  
Apex  
SAM  
Careless  

**Books**  
Serverless Architecture on AWS (Peter Sbarski)  
Hands-On Serverless Applications with Go  

**YouTube**  
Why the fuss about Serverless? Simon Wardley

# EOF

## Appendix: How to decide to go Serverless?

```zsh
  ^----------------+------------------+
  |                |                  |
  |                |                  |
M |   Re-Design    |    Not you cup   |
E |   or not       |                  |
M |   a good fit   |    of tea        |
O |                |                  |
R |                |                  |
Y |                |                  |
  +-----------------------------------+
  |                |                  |
  |                |                  |
  |                |                  |
  |   Perfect      |   Re-Design?     |
  |                |                  |
  |                |                  |
  |                |                  |
  +----------------+------------------>
                  TIME
```

## Appendix: KanDDDinsky discount code

Get your ticket at kandddinsky.com ( https://www.kandddinsky.com/ )
We’ve created a discount code for your local DDD community in particular:
**“KanDDDinsky_LOVES_Cologne”**
