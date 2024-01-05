# login-vuejs-apiology-docker

auhor: m4r4v

---

# Description

A regular login, using apiology as the REST API that communicates with db. For the frontend is all vuejs using quasar and pinia.

The purpose is to authenticate a user and keeping the session alive during X time of inactivity or wrong token validation.

---

# Environment Setup

This settings are for a frontend and backend solution integrated in one container. In order to obtain the desired result, the following architecture is being used:

- apiology folder that container inside all the logic for the API, is located inside the `/var/www` folder, at the same level as the `html` folder that contains public data.
- the front end the `dist` folder, contains the *production* / *built* files from *vuejs*; and this folders content needs to be inside the `html` folder.


### Dev - Prod Rules

the port is the only factor being used to determine if is the dev or prod environment, so when going live into production, the port must change to 80:

- **dev port** : 8010
- **prod port** : 80

### Technical Environment

- **NodeJs**: v20.10.0
- **NPM**: v10.2.4
- **VueJS**: @vue/cli 5.0.8
- **Quasar**: @quasar/cli v2.3.0
- **PHP**: v8.2.10
- **Docker**: v24.0.6
- **Docker Image**: m4r4v/lamp-debian-bullseye:latest
- **OS**: GNU/Linux
- **Distro**: Debian Bullseye v11.8
- **DB**: MariaDB version 15.1 Distrib 10.5.21-MariaDB

---

## Create container - docker run

```bash
docker run -p 8010:80 -d -v ${PWD}:/var/www -e MYSQL_ROOT_PASSWORD=123456 --name login-vue-apiology m4r4v/lamp-debian-bullseye:latest
```

see more at: []()