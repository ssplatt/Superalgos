# Run Superalgos with `docker-compose` for Production(-like) Deployments

Follow the link to [install docker-compose](https://docs.docker.com/compose/install/). Learn more at the [docker-compose reference](https://docs.docker.com/compose/reference/).

Pull a pre-built container and run it:

```
docker-compose pull
docker-compose up -d
```

Or build a container and run the application:

```
docker-compose up --build -d
```

## Configure the Container Environment

### Available image tags

To avoid breaking changes, the `shasum hash` and the `release` tags are the best to use. These will generally ensure you are always running the same code. The other tags will change which code they are pointing to more frequently and without notice.

- `latest` : the absolute latest build
- `master` : the latest master branch build
- `develop` : the latest develop branch build
- `<shasum hash>` : a specific git commit hash
- `<release>` : corresponds with a Github Release (git tag), i.e. `beta-10`

### Environment variables

`PUID` and `PGID` environment variables can be used to help avoid permissions issues in the mounted volumes between the container environment and the local OS environment. The default `PUID` and `GUID` is `1000`. You can view the current user's PUID and GUID with the `id` command.

## Using a Secure Web Application Gateway (swag)

Since Superalgos itself does not have encryption or authentication built in, exposing the application to the public internet is not recommended. However, a reverse proxy can be used to encrypt data in transit and provide simple authentication.

**Exposing Superalgos to the internet will expose you to hackers and must be done with caution. Do so at your own risk. These directions are provided for informational purposes only.**

[linuxserver/swag](https://github.com/linuxserver/docker-swag) is one method to provide a secure ingress to the Superalgos application over the public internet. `swag` includes [nginx as a reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/), [certbot](https://certbot.eff.org/), and [fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page) which can be configured to provide password auth (htpasswd), valid certificates for tls/ssl encryption (https), and the ability to ban bots and/or hackers.

- [View the swag readme for more details and configuration options](https://github.com/linuxserver/docker-swag#readme)

Run the Superalgos application and swag containers:

```
docker-compose -f docker-compose.yml -f docker-compose.swag.yml up -d
```

**create a username and password** for authentication:

```
docker-compose exec swag htpasswd -c /config/nginx/.htpasswd <username>
```

**Configure certbot for ssl/tls encryption (https)**, copy the `template.env` file to `.env` and edit the environment variables in `.env`. Change any other environment variables in `.env` as needed. You will need to own the domain you are trying to create certificates for. You can buy your own domain at websites like [Namecheap](https://www.namecheap.com/), for example.

`swag` configurations will persist to the `swag/` directory on the local filesystem.

The included nginx site configuration, `swag/nginx/site-confs/superalgos`, assumes you will be accessing the application using a subdomain, i.e. https://superalgos.example.com. To make modifications, copy this file to another location and create your own custome configuration.
