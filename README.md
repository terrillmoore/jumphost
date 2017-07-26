# Jumphost

Jumphost is a simple service for use in connecting `autossh`-using embedded
systems to `ansible`-using clients. 

The embedded systems log into the jumphost and do nothing, but as part of
the login they provide a reverse ssh channel to an sshd port on the target.

The ansible clients log into the jumphost and similarly do nothing; but they 
are able to provide a forward map from a port on the client system to the 
reverse ssh channel(s) offered by the logged-in embedded systems. 

The ansible playbooks refer to the embedded systems as (for example) 
`localhost:20000`, `localhost:20001`, etc. The embedded systems, when they log
in, map jumphost port 30000, 30001, etc. to themselves (each embedded system uses a different port.) The clients include `-L 20000:localhost:30000 -L 20001:localhost:30001` and so forth on the `ssh` command line. The ssh daemon on jumphost 
then bridges from 20000 on the client through 30000 on jumphost and then on to the relevant sshd port on the relevant embedded system.

To set this up requires two steps:

1. create `./.env` with variables as needed by `docker-compose.yml`.
2. populate `./jumphost` with suitable `id_jumphost_admin.pub`, `id_jumphost_system.pub` and `id_jumphost_ansible.pub` files, containing the authorized public keys for each form of login.

File | Key for User
-----|------------
`id_jumphost_admin.pub` | root
`id_jumphost_system.pub` | embedded systems
`id_jumphost_ansible.pub` | ansible clients

Then (as usual):
```shell
$ docker-compose build
$ docker-compose up -d
```

## Configuration Values in `.env`

Name | Default | Description
-----|---------|------------
`JUMPHOST_PHUSION_VERSION` | `latest` | The tag for the desired Phusion base version. For example, `0.9.22`
`JUMPHOST_SSH_SYSTEM_USER` | `jumpsystem` | The login to be used by the embedded systems when setting up the reverse tunnel.
`JUMPHOST_SSH_ANSIBLE_USER` | `jumpansible` | The login to be used by the Ansible clients. This login is possibly more fully featured, and may include the ability to sudo.
`JUMPHOST_SSH_OUTER` | 2022 | The port number on the container system to be used for all ssh connections. If you use different port numbers, you can run multiple jumphost instances on the same host system.

