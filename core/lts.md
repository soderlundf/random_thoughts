## 1. Use an LTS Version of Node.js

LTS stands for **Long-Term Support**.  
LTS releases receive security updates and critical fixes for a longer period, making them the safest choice for production environments.

> Example:  
> As of late 2025, **Node.js 22.x** is an active LTS release.  
> Pinning to `22.x` means you allow patch updates (e.g., 22.1 → 22.2) but not breaking major updates (e.g., 23.x).

---

## 2. Define "engines" in `package.json`

You can specify which versions of Node.js and npm your project supports.  
This helps developers and CI/CD environments ensure consistency and avoid version mismatches.

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "A robust Node.js project",
  "main": "index.js",
  "engines": {
    "node": "22.x",
    "npm": ">=10.0.0"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

## Tips

Run node -v and npm -v locally to confirm your versions.
Use a .nvmrc file or a Docker image (e.g., node:22-alpine) to guarantee version consistency across environments.

Example .nvmrc file:

`22`

## Locked LTS version in docker-compose

```
version: "3.9"

services:
  web:
    image: node:24-alpine
    container_name: my-node-app
    working_dir: /usr/src/app
    volumes:
      - ./:/usr/src/app
    command: ["npm", "start"]
    environment:
      NODE_ENV: production
    ports:
      - "3000:3000"
    restart: unless-stopped

```
