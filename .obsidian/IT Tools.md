from https://github.com/CorentinTh/it-tools

```
version: "3.6"
services:
  it-tools:
    container_name: "it-tools"
    image: "corentinth/it-tools:latest"
    ports:
      - "8888:80"
    restart: "unless-stopped"
```

Starts a container and you can open the tools by calling `http://whatever:8888`.