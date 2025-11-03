Create a docker container considering mapping local folders for the media, e.g. like this:
```
sudo docker run -d \
    --name embyserver \
    --volume /home/michael/emby/program:/config \
    --volume /media/ssd/Music:/mnt/music \
    --volume /media/ssd/Videos:/mnt/videos \
    --volume /media/ssd/Music-Videos:/mnt/music-videos \
    --volume /media/ssd/TV-Shows:/mnt/tv-shows \
    --volume /media/ssd/Audiobooks:/mnt/audiobooks \
    --volume /home/michael/emby/media:/mnt/media \
    --net=host \
    --device /dev/dri:/dev/dri \
    --publish 8096:8096 \
    --publish 8920:8920 \
    --publish 1900:1900/udp \
    --publish 1901:1901/udp \
    --env UID=1000 \
    --env GID=1000 \
    --env GIDLIST=1000 \
    --restart always \
    emby/embyserver:latest
```

Open the web page (e.g. `http://192.168.178.26:8096/`) and follow the instructions.
In Settings under Devices on the DLNA tab activate DLNA server.