# GitLab Runner Setup and Registration (Docker Desktop)

এই ডকুমেন্টে দেখানো হয়েছে কিভাবে Docker Compose ব্যবহার করে একটি local GitLab server এবং GitLab Runner সেটআপ করতে হয়, এবং কিভাবে Runner টিকে GitLab এর সাথে register করতে হয়।

---

## ১. docker-compose.yml ফাইল তৈরি

প্রথমে একটি `docker-compose.yml` ফাইল বানাতে হবে, যেখানে দুইটি service থাকবে — একটি হলো `gitlab-server` (GitLab CE image দিয়ে), আরেকটি হলো `gitlab-runner`।

- `hostname: my-gitlab-server` — container টির hostname সেট করা হয়েছে।
- `restart: always` — container বন্ধ হয়ে গেলে বা crash করলে automatically আবার start হবে।
- `gitlab-runner` service টি `gitlab-server` এর উপর `depends_on` হিসেবে নির্ভরশীল, অর্থাৎ আগে server, তারপর runner start হবে।

```yaml
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

## ২. Containers Start করা

Terminal এ নিচের command দিয়ে দুইটি container একসাথে start করতে হবে:

```bash
docker compose up -d
```

এই command রান করলে `my-gitlab-server` এবং `my-gitlab-runner` — দুইটি container-ই running অবস্থায় চলে যাবে।

![GitLab Docker Compose running](https://imgur.com/YW9BDYl.png)

---

## ৩. GitLab এ Project তৈরি এবং Registration Token সংগ্রহ

Browser এ গিয়ে GitLab instance open করতে হবে:

```
http://localhost:8000/
```

তারপর ধাপে ধাপে:

1. একটি নতুন **Project** create করতে হবে — Project name, Project URL, Project slug, এবং Visibility Level (public) ইত্যাদি value দিয়ে।
2. **Admin** → **CI/CD** → **Runners** এ গিয়ে সেখান থেকে **registration token** copy করতে হবে।

![CI/CD Runners page](https://imgur.com/ZUrq5JE.png)

---

## ৪. Runner Register করা

Terminal এ গিয়ে নিচের command দিয়ে runner registration শুরু করতে হবে:

```bash
docker exec -it my-gitlab-runner gitlab-runner register
```

Registration process এর সময় নিচের প্রশ্নগুলোর উত্তর একে একে দিতে হবে:

| ধাপ | প্রশ্ন | Value |
|---|---|---|
| ১ | GitLab instance URL | `http://my-gitlab-server` |
| ২ | Registration token | (copy করা token paste করতে হবে) |
| ৩ | Runner description | `my-docker-runner` (ইচ্ছামতো নাম দেওয়া যায়) |
| ৪ | Tags (comma-separated) | `docker` |
| ৫ | Maintenance note (optional) | `docker` |
| ৬ | Executor | `docker` |
| ৭ | Default Docker image | `alpine:latest` |

সবগুলো ধাপ ঠিকভাবে সম্পন্ন হলে runner টি সফলভাবে register হয়ে যাবে।

---

## ৫. Runner Restart এবং Verify করা

Register শেষে runner container টি restart করতে হবে:

```bash
docker restart my-gitlab-runner
```

তারপর runner list check করে verify করা যাবে register ঠিকভাবে হয়েছে কিনা:

```bash
docker exec -it my-gitlab-runner gitlab-runner list
```

GitLab এর online interface এ গেলেও দেখা যাবে runner টি running অবস্থায় আছে।

![Runner running online](https://imgur.com/RB4MSsr.png)

---

## ৬. External URL Fix করা

Default অবস্থায় GitLab UI এর কিছু link ভুল external URL এ redirect করতে পারে, এই সমস্যা এড়াতে `GITLAB_OMNIBUS_CONFIG` এর ভেতরে `external_url` সেট করে দিতে হবে:

```yaml
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

পরিবর্তন প্রয়োগ করার জন্য container গুলো down করে আবার up করতে হবে:

```bash
docker compose down
```

```bash
docker compose up -d
```

---

## সংক্ষিপ্ত Flow

```
docker-compose.yml তৈরি
        ↓
docker compose up -d (server + runner start)
        ↓
Browser এ GitLab open করে Project create
        ↓
Admin → CI/CD → Runners থেকে registration token নেওয়া
        ↓
gitlab-runner register command দিয়ে registration
        ↓
docker restart my-gitlab-runner
        ↓
runner list দিয়ে verify
        ↓
external_url add করে docker compose down → up
```
