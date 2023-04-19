# A personal Jupyter Hub for teaching

> :warning: **HEAVY DEVELOPMENT NOT READY FOR DAILY USE** :warning:

A easy to deploy JupyterHub (multi-user notebook) for individual or small classroom usage.
Can be used as a complement for an on premise large scale install sharing the same images.
It is based on docker, the hub and each individual servers run in a container. Data are persisted in named volumes.

> :warning: Its is **not intended for production** (no scaling, minimal security).
> :warning: By default, sign on is open **expose wisely**.

## Usage

1. Clone this repository
1. Launch with `docker compose up -d`
1. Use the local jupyterhub <http://localhost:8000>
1. Or Use a link [nbgitpuller link](https://nbgitpuller.readthedocs.io/en/latest/link.html) with http://localhost:8000 to bootstrap with any github repository.
   * for example a [quickstart](http://localhost:8000/hub/user-redirect/git-pull?repo=https%3A%2F%2Fgithub.com%2Febpro%2Fnotebook-qs-base&urlpath=lab%2Ftree%2Fnotebooks%2Fnotebook-qs-base%2Fquickstart.ipynb)

## Configuration

The configuration is done in `jupyterhub_config.py`.

The following jupyter docker images are provided :

1. Python ([brunoe/jupyter-base](github.com/ebpro/jupyter-base/))
1. Java ([brunoe/jupyter-java](github.com/ebpro/jupyter-java))
1. PostgreSQL ([brunoe/jupyter-db-pg](https://github.com/ebpro/jupyter-db-pg))
1. SciPy ([jupyter/scipy-notebook](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-scipy-notebook))

The images 1. to 3. includes the vscode IDE and a a Linux window manager in the browser (XFCE).

## Storage

Data for the hub and the users are stored in volumes.

```bash
docker volume ls|grep jupyterhub-
```

To backup users volumes one solution is to use tar in another container.

```bash
USERS=$(docker volume ls|tr -s ' '|cut -f 2 -d ' '|sed -n s'/^jupyterhub-user-\(.*\)/\1/p')
for USER_TO_BACKUP in $USERS; do
  echo --Backing up $USER_TO_BACKUP--
  docker run --rm \
          -v $PWD/jupyterhub-backups/:/backups \
          -v jupyterhub-user-$USER_TO_BACKUP:/work \
          jupyter/minimal-notebook \
          tar --exclude={".m2",".codeserver/data/CachedExtension*","/work/.codeserver/data/logs/"} \
            -zcf \
            /backups/jupyterhub-user-$USER_TO_BACKUP-backup.tar.gz \
            /work
done
```
