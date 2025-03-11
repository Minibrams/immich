# Immich

A self-contained Immich instance using cheap SFTP storage (e.g., a Hetzner Storage Box).

# Prerequisites
1. An Ubuntu server with Docker installed
2. An SFTP-enabled remote drive (e.g., Hetzner Storage Box)


# Setup

First, install rclone, a volume plugin for Docker that allows a volume to clone its contents to a remote destination (steps also found [here](https://rclone.org/docker/)):

```bash
sudo apt-get -y install fuse
sudo mkdir -p /var/lib/docker-plugins/rclone/config
sudo mkdir -p /var/lib/docker-plugins/rclone/cache

docker plugin install rclone/docker-volume-rclone:amd64 args="-v" --alias rclone --grant-all-permissions
docker plugin enable rclone

# Verify installation
docker plugin list
```

In Hetzner, create or find your Storage Box and enter its connection settings (found [here](https://robot.hetzner.com/storage)) in `/var/lib/docker-plugins/rclone/config/rclone.conf`:

```conf
[storagebox]
type = sftp
host = <your storage box server here>  # e.g., u365767.your-storagebox.de
user = <your storage box user here>    # e.g., u365767
port = 23
pass = <your storage box password here>
shell_type = unix
md5sum_command = md5 -r
sha1sum_command = sha1 -r
```

The storage box is used as a volume in the `docker-compose.yml` file:

```yaml
volumes:
  # Storage Box
  data:
    driver: rclone
    driver_opts:

      # Use the [storagebox] configuration, and write to the "immich-test" path inside it
      remote: "storagebox:immich-test"
```

Now, create some certificates for your server. I recommend using certbot:
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your_domain

# Run the command below to renew all certificates. Useful in a cron job:
sudo certbot renew
```

Remember to update the paths to the certificates (and your domains) in `./immich-app/nginx.conf`.


# Deploy

1. Copy or clone this project onto your server
2. Go through the setup
3. Run:
```bash
cd immich-app
docker compose up -d
```

Your app should be running on [https://your-domain](https://your-domain).