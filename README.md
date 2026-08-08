# Docker Desktop: Register GitLab Runner with GitLab Server [GitLab Runner setup]

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
