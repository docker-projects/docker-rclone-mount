## docker-rclone-mount
## Configuring rclone.conf
### Copying existing conf
	mkdir p $(pwd)/config/rclone-mount
	cp ~/.config/rclone/rclone.conf $(pwd)/config/rclone-mount/.rclone.conf

### Creating a new conf
	docker run -it --rm -v $(pwd)/config/rclone-mount:/config pknw1/rclone-mount rclone --config="/config/.rclone.conf" config

## Running a configured instance
### docker-cli

	REMOTE=yourremotename
	docker run -d \
		--name=rclone-$REMOTE \
		-v $(pwd)/config/rclone-$REMOTE:/config \
		--cap-add SYS_ADMIN \
		--device /dev/fuse \
		--security-opt apparmor:unconfined \
		-e RCLONE_REMOTE_MOUNT=$REMOTE:/ \
		-e PUID=666 \
		-e PGID=666 \
		-e TZ=Europe/London \
		-v $(pwd)/config/rclone-$REMOTE:/config \
		-v /mnt/rclone-$REMOTE:/data:shared \
		-v /etc/localtime:/etc/localtime \
		-v /tmp:/tmp \
		--restart=unless-stopped \
		pknw1/rclone-mount

### docker-compose

	version: '3'
	services:
	  rclone-ovh:
	    image: pknw1/rclone-mount
	    container_name: rclone-mount
	    security_opt:
	      - apparmor:unconfined
	    cap_add:
	      - SYS_ADMIN
	    devices:
	      - "/dev/fuse:/dev/fuse"
	    environment:
	      - PUID=666
	      - PGID=666
	      - PUMASK=0000
	      - TZ=Europe/London
	      - RCLONE_REMOTE_MOUNT=remote:/
	    volumes:
	      - ./config/rclone-mount:/config
	      - /mnt/rclone-remote:/data:shared
	      - /tmp:/tmp
	      - /etc/localtime:/etc/localtime
	    restart: unless-stopped
