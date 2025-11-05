# Authelia

## Installation

```
secrets:
  JWT_SECRET:
    file: '$DOCKER_DATA_DIR/authelia/secrets/JWT_SECRET'
  SESSION_SECRET:
    file: '$DOCKER_DATA_DIR/authelia/secrets/SESSION_SECRET'
  STORAGE_PASSWORD:
    file: '$DOCKER_DATA_DIR/authelia/secrets/STORAGE_PASSWORD'
  STORAGE_ENCRYPTION_KEY:
    file: '$DOCKER_DATA_DIR/authelia/secrets/STORAGE_ENCRYPTION_KEY'
  IDENTITY_PROVIDERS_OIDC_HMAC_SECRET:
    file: '$DOCKER_DATA_DIR/authelia/secrets/IDENTITY_PROVIDERS_OIDC_HMAC_SECRET'
services:
  authelia:
    container_name: 'authelia'
    image: 'docker.io/authelia/authelia:latest'
    restart: 'unless-stopped'
    networks:
      - nginx_nginx_default
      - mysql
    expose:
      - 9091
    secrets: ['JWT_SECRET', 'SESSION_SECRET', 'STORAGE_PASSWORD', 'STORAGE_ENCRYPTION_KEY', 'IDENTITY_PROVIDERS_OIDC_HMAC_SECRET']
    environment:
      AUTHELIA_IDENTITY_VALIDATION_RESET_PASSWORD_JWT_SECRET_FILE: '/run/secrets/JWT_SECRET'
      AUTHELIA_SESSION_SECRET_FILE: '/run/secrets/SESSION_SECRET'
      AUTHELIA_STORAGE_MYSQL_PASSWORD_FILE: '/run/secrets/STORAGE_PASSWORD'
      AUTHELIA_STORAGE_ENCRYPTION_KEY_FILE: '/run/secrets/STORAGE_ENCRYPTION_KEY'
      TZ: $TZ
    volumes:
      - '$DOCKER_DATA_DIR/authelia/config:/config'

networks:
  nginx_nginx_default:
    external: true
  mysql:
    external: true
```

## Password generation

Generate password for user file:

```
sudo docker run authelia/authelia:latest authelia crypto hash generate argon2 --password 'password'
```

Output will be like this:

```
Digest: $argon2id$v=19$m=65536,t=3,p=4$Hjc8e7WYcBFcJmEDUOsS9A$ozM7RyZR1EyDR8cuyVpDDfmLrGPGFgo5E2NNqRumui4
```

Copy to file.
