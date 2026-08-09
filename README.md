# Docker Desktop: GitLab Runner Setup and Registration

#### docker-compose.yml
> hostname: my-gitlab-server , restart: always add koro ar new server add koro nam daw gitlab-runner
```bash
services:
  gitlab-server:
    image: 'gitlab/gitlab-ce'
    container_name: my-gitlab-server
    hostname: my-gitlab-server
    restart: always
    ports:
      - '8000:80'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = 'w@sIm1997'
        puma['worker_processes'] = 0 
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab

  gitlab-runner:
    image: 'gitlab/gitlab-runner:latest'
    container_name: my-gitlab-runner
    restart: always
    depends_on:
      - gitlab-server
    volumes:
      - ./gitlab-runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
    privileged: true
```
---

#### vs terminal e command daw
```bash
docker compose up -d
```
> my-gitlab-server and my-gitlab-runner running hoye jabe.
![](https://imgur.com/YW9BDYl.png)

---

#### browser e http://localhost:8000/ open koro
#### project create koro--->Project name, Project URL, Project slug, Visibility Level: public etc value diye project create koro.
#### click koro: Admin --->click koro: CI/CD --->click koro: Runners --->copy koro: registration tokens
![](https://imgur.com/ZUrq5JE.png)

---

#### vs terminal e command daw
```bash
docker exec -it my-gitlab-runner gitlab-runner register
```
#### Enter the GitLab instance URL (for example, https://gitlab.com/):
```bash
http://my-gitlab-server
```
#### Enter the registration token:
```bash
paste kore daw
```
#### Enter a description for the runner:
> description ja iccah deya jai
```bash
my-docker-runner
```
#### Enter tags for the runner (comma-separated):
```bash
docker
```
#### Enter optional maintenance note for the runner:
```bash
docker
```
#### Enter an executor: docker, docker-windows, docker+machine, kubernetes, virtualbox, shell, custom, instance, docker-autoscaler, ssh, parallels:
```bash
docker
```
#### Enter the default Docker image (for example, ruby:3.3):
```bash
alpine:latest
```
> successfully runner register hoye jabe.
---

#### vs terminal e command daw
```bash
docker restart my-gitlab-runner
```
#### check
```bash
docker exec -it my-gitlab-runner gitlab-runner list
```
---

#### online e runner run hoyeche
![](https://imgur.com/RB4MSsr.png)

#### external url e jate na jai tai add koro
> external_url 'http://my-gitlab-server' add koro.
```bash
version: '5.4'
services:
  gitlab-server:
    image: 'gitlab/gitlab-ce'
    container_name: my-gitlab-server
    hostname: my-gitlab-server
    restart: always
    ports:
      - '8000:80'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        gitlab_rails['initial_root_password'] = 'w@sIm1997'
        puma['worker_processes'] = 0 
        external_url 'http://my-gitlab-server'
    volumes:
      - ./gitlab/config:/etc/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/data:/var/opt/gitlab

  gitlab-runner:
    image: 'gitlab/gitlab-runner:latest'
    container_name: my-gitlab-runner
    restart: always
    depends_on:
      - gitlab-server
    volumes:
      - ./gitlab-runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
    privileged: true
```
#### vs terminal e daw
```bash
docker compose down
```
```bash
docker compose up -d
```
---
