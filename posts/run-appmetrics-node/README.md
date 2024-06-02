# Solve build image docker with appmetrics-dash failed on node bullseys-slim:18.xx

Found error **omr-agentcore/libhcmqtt.so** while build the image docker Error, I couldn't install package on docker node bullseys-slim:18.10

e.g. snippet error
```
23.80 npm ERR! collect2: error: ld returned 1 exit status
23.80 npm ERR! make: *** [omr-agentcore/hcmqtt.target.mk:180: Release/obj.target/omr-agentcore/libhcmqtt.so] Error 1
23.80 npm ERR! gyp ERR! build error
23.80 npm ERR! gyp ERR! stack Error: make failed with exit code: 2
23.80 npm ERR! gyp ERR! stack at ChildProcess.onExit (/app/node_modules/node-gyp/lib/build.js:194:23)
23.80 npm ERR! gyp ERR! stack at ChildProcess.emit (node:events:513:28)
23.80 npm ERR! gyp ERR! stack at ChildProcess._handle.onexit (node:internal/child_process:291:12)
23.80 npm ERR! gyp ERR! System Linux 6.4.16-linuxkit
23.80 npm ERR! gyp ERR! command "/usr/local/bin/node" "/app/node_modules/.bin/node-gyp" "rebuild"
23.80 npm ERR! gyp ERR! cwd /app/node_modules/appmetrics
23.80 npm ERR! gyp ERR! node -v v18.10.0
23.80 npm ERR! gyp ERR! node-gyp -v v5.1.1
23.80 npm ERR! gyp ERR! not ok
23.80
23.80 npm ERR! A complete log of this run can be found in:
23.80 npm ERR! /root/.npm/_logs/2024-05-15T04_50_37_854Z-debug-0.log
```

e.g my dockerfile
```
FROM node:18.10-bullseye-slim

WORKDIR /app

RUN apt-get update \
  && apt-get install -y g++ gcc-9 make python \
  && apt-get clean

ENV CC=/usr/bin/gcc-9

RUN npm install node-gyp -g

COPY --chown=node:node ./package*.json ./
RUN npm ci --only=production
.
.
```

Solved By Change Environment variable CC
```
CC=/usr/bin/gcc-9
```

e.g Code snippet put appmetrics-dash

```
//must config at top level of node excute

import { CONTEXT_PATH, globalOptionSequelize } from './common/constant';

require('appmetrics-dash').attach({ url: `${CONTEXT_PATH}/monitoring`, nodereport: null });
```