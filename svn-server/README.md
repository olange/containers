[![Docker Image](https://img.shields.io/badge/docker%20image-available-green.svg)](https://hub.docker.com/r/olange/svn-server/)

# Alpine ‹SVN Server› Docker image

Lightweight container providing a [**Subversion server**](https://subversion.apache.org), based on latest [Alpine Linux](https://hub.docker.com/_/alpine), with latest [Apache2 HTTPD Server](https://pkgs.alpinelinux.org/packages?name=apache2) packages for Alpine. The Subversion server can be accessed via the **WebDav protocol** (http://).

## Run

To run the image, you can use the following command:

```shell
docker run -d --name svn-server -p 80:80 -p 443:443 \
           -v ‹host_svnroot›:/home/svn \
           -v ‹host_svn_auth_file›:/etc/subversion/svn-auth-file.htpasswd
           -v ‹host_svn_authz_file›:/etc/subversion/svn-authz-file.conf
           olange/svn-server
```

Alternatively, you can execute the [`./scripts/run`](scripts/run) shell script.

`/home/svn` stores your repositories and can use either bind mount or named volume. `/etc/subversion` stores subversion configuration and `/etc/apache2/` stores the Apache HTTPD configuration.

To verify that everything is up and running, open your browser and connect to `http://localhost/`. The system should ask you for the username and password — see below —, then it will show you an empty folder — there are no repos yet.

## Configure

You need to **setup usernames and passwords** for the access via WebDav protocol, within the `/etc/subversion/svn-authz-file.conf` file. You can use the following command from your host machine to add a user to this file:

```shell
docker exec -t svn-server htpasswd -b /etc/subversion/svn-auth-file.htpasswd ‹username› ‹password›
```

To enable **write access to the repositories**, you'll need to also configure the `/etc/subversion/svn-authz-file.conf` file.

For further information on how to configure Subversion, please refer to the [official web page](https://subversion.apache.org/).

## Build

```shell
export GIT_SHA=$(git rev-parse --short HEAD)
export SVN_VERSION=1.14.2

docker build --platform linux/arm/v8 -t olange/svn-server:$SVN_VERSION-armv8 --build-arg GIT_SHA --build-arg SVN_VERSION .
docker build --platform linux/amd64 -t olange/svn-server:$SVN_VERSION-amd64 --build-arg GIT_SHA --build-arg SVN_VERSION .

docker push --all-tags olange/svn-server
```

Alternatively, you can execute the [`./scripts/build`](scripts/build) and [`./scripts/publish`](scripts/publish) shell scripts.

## License

Distributed under the Apache-2.0 license. See [LICENSE](../LICENSE) for details.