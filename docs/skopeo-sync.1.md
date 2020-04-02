% skopeo-sync(1)

## NAME
skopeo\-sync - Synchronize images between container registries and local directories.


## SYNOPSIS
**skopeo sync** --src _transport_ --dest _transport_ _source_ _destination_

## DESCRIPTION
Synchronize images between container registries and local directories.
The synchronization is achieved by copying all the images found at _source_ to _destination_.

Useful to synchronize a local container registry mirror, and to to populate registries running inside of air-gapped environments.

Differently from other skopeo commands, skopeo sync requires both source and destination transports to be specified separately from _source_ and _destination_.
One of the problems of prefixing a destination with its transport is that, the registry `docker://hostname:port` would be wrongly interpreted as an image reference at a non-fully qualified registry, with `hostname` and `port` the image name and tag.

Available _source_ transports:
 - _docker_ (i.e. `--src docker`): _source_ is a repository hosted on a container registry (e.g.: `registry.example.com/busybox`).
 If no image tag is specified, skopeo sync copies all the tags found in that repository.
 - _dir_ (i.e. `--src dir`): _source_ is a local directory path (e.g.: `/media/usb/`). Refer to skopeo(1) **dir:**_path_ for the local image format.
 - _yaml_ (i.e. `--src yaml`): _source_ is local YAML file path.
 The YAML file should specify the list of images copied from different container registries (local directories are not supported). Refer to EXAMPLES for the file format.

Available _destination_ transports:
 - _docker_ (i.e. `--dest docker`): _destination_ is a container registry (e.g.: `my-registry.local.lan`).
 - _dir_ (i.e. `--dest dir`): _destination_ is a local directory path (e.g.: `/media/usb/`).
 One directory per source 'image:tag' is created for each copied image.

When the `--scoped` option is specified, images are prefixed with the source image path so that multiple images with the same
name can be stored at _destination_.

## OPTIONS
**--authfile** _path_

Path of the authentication file. Default is ${XDG\_RUNTIME\_DIR}/containers/auth.json, which is set using `podman login`.
If the authorization state is not found there, $HOME/.docker/config.json is checked, which is set using `docker login`.

**--src-authfile** _path_

Path of the authentication file for the source registry. Uses path given by `--authfile`, if not provided.

**--dest-authfile** _path_

Path of the authentication file for the destination registry. Uses path given by `--authfile`, if not provided.

**--src** _transport_ Transport for the source repository.

**--dest** _transport_ Destination transport.

**--scoped** Prefix images with the source image path, so that multiple images with the same name can be stored at _destination_.

**--remove-signatures** Do not copy signatures, if any, from _source-image_. This is necessary when copying a signed image to a destination which does not support signatures.

**--sign-by=**_key-id_ Add a signature using that key ID for an image name corresponding to _destination-image_.

**--src-creds** _username[:password]_ for accessing the source registry.

**--dest-creds** _username[:password]_ for accessing the destination registry.

**--src-cert-dir** _path_ Use certificates (*.crt, *.cert, *.key) at _path_ to connect to the source registry or daemon.

**--src-no-creds** _bool-value_ Access the registry anonymously.

**--src-tls-verify** _bool-value_ Require HTTPS and verify certificates when talking to a container source registry or daemon (defaults to true).

**--dest-cert-dir** _path_ Use certificates (*.crt, *.cert, *.key) at _path_ to connect to the destination registry or daemon.

**--dest-no-creds** _bool-value_  Access the registry anonymously.

**--dest-tls-verify** _bool-value_ Require HTTPS and verify certificates when talking to a container destination registry or daemon (defaults to true).

## EXAMPLES

### Synchronizing to a local directory
```
$ skopeo sync --src docker --dest dir registry.example.com/busybox /media/usb
```
Images are located at:
```
/media/usb/busybox:1-glibc
/media/usb/busybox:1-musl
/media/usb/busybox:1-ubuntu
...
/media/usb/busybox:latest
```

### Synchronizing to a local directory, scoped
```
$ skopeo sync --src docker --dest dir --scoped registry.example.com/busybox /media/usb
```
Images are located at:
```
/media/usb/registry.example.com/busybox:1-glibc
/media/usb/registry.example.com/busybox:1-musl
/media/usb/registry.example.com/busybox:1-ubuntu
...
/media/usb/registry.example.com/busybox:latest
```

### Synchronizing to a container registry
```
skopeo sync --src docker --dest docker registry.example.com/busybox my-registry.local.lan
```
Destination registry content:
```
REPO                         TAGS
registry.local.lan/busybox   1-glibc, 1-musl, 1-ubuntu, ..., latest
```

### Synchronizing to a container registry keeping the repository
```
skopeo sync --src docker --dest docker registry.example.com/repo/busybox my-registry.local.lan/repo
```
Destination registry content:
```
REPO                              TAGS
registry.local.lan/repo/busybox   1-glibc, 1-musl, 1-ubuntu, ..., latest
```

### YAML file content (used _source_ for `**--src yaml**`)

```yaml
registry.example.com:
    images:
        busybox: []
        redis:
            - "1.0"
            - "2.0"
            - "sha256:0000000000000000000000000000000011111111111111111111111111111111"
    credentials:
        username: john
        password: this is a secret
    tls-verify: true
    cert-dir: /home/john/certs
quay.io:
    tls-verify: false
    images:
        coreos/etcd:
            - latest
```

This will copy the following images:
- Repository `registry.example.com/busybox`: all images, as no tags are specified.
- Repository `registry.example.com/redis`: images tagged "1.0" and "2.0" along with image with digest "sha256:0000000000000000000000000000000011111111111111111111111111111111".
- Repository `quay.io/coreos/etcd`: images tagged "latest".

For the registry `registry.example.com`, the "john"/"this is a secret" credentials are used, with server TLS certificates located at `/home/john/certs`.

TLS verification is normally enabled, and it can be disabled setting `tls-verify` to `true`.
In the above example, TLS verification is enabled for `reigstry.example.com`, while is
disabled for `quay.io`.

## SEE ALSO
skopeo(1), podman-login(1), docker-login(1)

## AUTHORS

Flavio Castelli <fcastelli@suse.com>, Marco Vedovati <mvedovati@suse.com>

