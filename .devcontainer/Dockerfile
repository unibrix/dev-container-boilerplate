FROM node:22-bookworm

USER root

RUN apt-get update && apt-get install -y \
    mc \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g @openai/codex

USER node
WORKDIR /workspaces/app
