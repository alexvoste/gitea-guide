GitHub is cooked? Spin up your own Gitea in 5 minutes.

Hey everyone. With all the recent talk about GitHub potentially getting blocked in Russia, now’s probably a good time to think about self-hosting.

Officially: State Duma deputy Anton Gorelkin said GitHub could become completely inaccessible in Russia. He urged developers to urgently move their projects to Russian platforms and accused GitHub’s administration of discriminating against Russian users.

At the end of the day, we devs still need somewhere to store code, share projects with the homies, and manage version control.

I recently picked up a couple of servers, and instead of letting them sit idle, I decided to spin up Gitea on one of them — a lightweight GitHub alternative. Here’s a quick setup guide.

## What you’ll need

* A clean server (doesn’t have to be some beefy datacenter machine). Gitea uses around 50 MB of RAM.
* Docker and docker-compose.
* PostgreSQL — we’ll run it in the same Docker stack.

## Open the firewall ports (ufw)

```bash
sudo ufw allow 3000/tcp # web interface
sudo ufw allow 2222/tcp # Git over SSH (port 22 is already used by the system)
```

## Create a folder and config

```bash
mkdir gitea && cd gitea
touch docker-compose.yml
```

## Add the contents to docker-compose.yml

see (docker-compose.yml)

Quick note: `2222:22` means Git inside the container listens on port 22, but externally it’s exposed on port 2222 so it doesn’t clash with the host SSH service (which already uses port 22).

When cloning repos, use:

```bash
ssh://git@your_server:2222/user/repository.git
```

## Start everything up

```bash
docker-compose up -d
```

Wait about 10–20 seconds for the database to initialize, then open this in your browser:

```text
http://YOUR_SERVER_IP:3000
```

## Security: HTTPS and nginx

Exposing port 3000 directly to the internet isn’t exactly safe. You’ll want TLS. Best practice is to put nginx in front of Gitea and set up Let’s Encrypt.

Example nginx config (HTTP → HTTPS only):

see (nginx.conf)

You can grab SSL certs with certbot:

```bash
sudo certbot --nginx -d git.yourdomain.com
```

Certbot will automatically issue the certificate and patch your nginx config. After that, Gitea will be available at:

```text
https://git.yourdomain.com
```

## What you get in the end

* Your own Git server with a web UI, SSH access, and PostgreSQL.
* Full control over your code, no dependency on external platforms or possible blocks.
* Unlimited users and repositories.

At this point, you’re no longer tied to GitHub. If you want to migrate existing repos, just do a:

```bash
git push --mirror
```

to the new server.

Happy self-hosting. If you’ve got questions — drop them in the comments.

[https://github.com/alexvoste](https://github.com/alexvoste)

