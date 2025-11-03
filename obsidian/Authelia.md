Generate password for user file:
```
sudo docker run authelia/authelia:latest authelia crypto hash generate argon2 --password 'password'
```

Output will be like this:

```
Digest: $argon2id$v=19$m=65536,t=3,p=4$Hjc8e7WYcBFcJmEDUOsS9A$ozM7RyZR1EyDR8cuyVpDDfmLrGPGFgo5E2NNqRumui4
```

Copy to file.