

# Docker Compose vs Docker Swarm — Lab 2 VPS

## 1. Mục tiêu

Lab này nhằm tìm hiểu sự khác nhau giữa **Docker Compose** và **Docker Swarm** thông qua việc triển khai thực tế trên 2 VPS.

Sau khi hoàn thành lab, cần hiểu được:

* Docker Compose là gì.
* Docker Swarm là gì.
* Sự khác nhau giữa Compose và Swarm.
* Cách tạo Docker Swarm Cluster.
* Vai trò của Manager và Worker.
* Cách deploy Service.
* Cách scale Service.
* Cách kiểm tra container/task đang chạy trên node nào.
* Cách Swarm xử lý khi một node gặp sự cố.
* Cách deploy ứng dụng bằng Docker Stack.
* Khái niệm Rolling Update.
* Khi nào nên sử dụng Compose và khi nào có thể sử dụng Swarm.

---

# 2. Docker Compose

## 2.1 Docker Compose là gì?

Docker Compose là công cụ dùng để định nghĩa và chạy các ứng dụng gồm nhiều container thông qua file YAML.

Ví dụ một ứng dụng có:

```text
Application
├── Nginx
├── Backend
├── Redis
└── MySQL
```

Có thể định nghĩa các service trong file:

```text
compose.yaml
```

Sau đó triển khai bằng:

```bash
docker compose up -d
```

## 2.2 Đặc điểm

Docker Compose thường được sử dụng để:

* Development
* Testing
* Local environment
* Single-server deployment
* Dựng nhanh nhiều container có liên quan với nhau

Ví dụ:

```text
VPS 1
│
├── nginx
├── app
├── redis
└── mysql
```

Các container đều chạy trên cùng một Docker Host.

---

# 3. Docker Swarm

## 3.1 Docker Swarm là gì?

Docker Swarm là tính năng orchestration được tích hợp trong Docker, cho phép nhiều Docker Host kết hợp thành một cluster.

Ví dụ:

```text
             Docker Swarm Cluster
                      │
             ┌────────┴────────┐
             │                 │
          VPS 1              VPS 2
         Manager             Worker
             │                 │
          Docker              Docker
```

Thay vì quản lý từng VPS độc lập, Swarm cho phép quản lý các node như một cluster.

---

# 4. Manager và Worker

Docker Swarm có các node.

## Manager Node

Manager chịu trách nhiệm quản lý cluster.

Các nhiệm vụ chính:

* Quản lý trạng thái cluster.
* Scheduling task.
* Quản lý service.
* Quản lý node.
* Duy trì desired state.

Ví dụ:

```text
VPS 1
Role: Manager
```

## Worker Node

Worker chịu trách nhiệm chạy các task/container được Manager phân phối.

Ví dụ:

```text
VPS 2
Role: Worker
```

Kiến trúc lab:

```text
                 Docker Swarm
                      │
          ┌───────────┴───────────┐
          │                       │
       VPS 1                    VPS 2
      Manager                  Worker
          │                       │
       Docker                  Docker
```

---

# 5. So sánh Docker Compose và Docker Swarm

| Tiêu chí           | Docker Compose              | Docker Swarm                    |
| ------------------ | --------------------------- | ------------------------------- |
| Mục đích           | Chạy nhiều container        | Orchestration nhiều Docker Host |
| Docker Host        | Thường một host             | Nhiều host                      |
| Cluster            | Không                       | Có                              |
| Manager/Worker     | Không                       | Có                              |
| Service scheduling | Không                       | Có                              |
| Scaling            | Có nhưng chủ yếu trên host  | Có trên cluster                 |
| Replicas           | Hạn chế                     | Có                              |
| Service discovery  | Có network                  | Có                              |
| Load balancing     | Hạn chế                     | Có                              |
| Self-healing       | Hạn chế                     | Có                              |
| Rolling update     | Hạn chế                     | Có                              |
| Overlay network    | Không phải mục tiêu chính   | Có                              |
| Độ phức tạp        | Thấp                        | Cao hơn                         |
| Phù hợp            | Development / single server | Cluster / orchestration         |

---

# 6. Ưu điểm Docker Compose

## 6.1 Dễ sử dụng

Chỉ cần file YAML:

```yaml
services:
  nginx:
    image: nginx

  redis:
    image: redis
```

Sau đó:

```bash
docker compose up -d
```

## 6.2 Dễ debug

Có thể sử dụng:

```bash
docker ps
docker logs
docker exec
```

để kiểm tra container.

## 6.3 Phù hợp development

Có thể nhanh chóng dựng môi trường:

```text
App
├── Backend
├── MySQL
├── Redis
└── Nginx
```

trên một máy.

---

# 7. Nhược điểm Docker Compose

Docker Compose không phải là một cluster orchestrator.

Ví dụ:

```text
VPS 1
│
├── nginx
├── app
└── mysql
```

Nếu VPS 1 gặp sự cố:

```text
VPS 1
   X
```

thì các container trên VPS đó cũng không thể tiếp tục hoạt động.

Docker Compose không tự động chuyển workload sang VPS 2 theo cơ chế cluster như Swarm.

---

# 8. Ưu điểm Docker Swarm

## 8.1 Cluster

Có thể kết hợp nhiều Docker Host:

```text
              Swarm Cluster
                    │
          ┌─────────┴─────────┐
          │                   │
       VPS 1               VPS 2
      Manager              Worker
```

## 8.2 Scaling

Có thể tăng số lượng replica:

```bash
docker service scale web=4
```

Ví dụ:

```text
VPS 1
├── web
└── web

VPS 2
├── web
└── web
```

## 8.3 Self-healing

Swarm cố gắng duy trì số lượng replica theo desired state.

Ví dụ yêu cầu:

```text
replicas = 4
```

Nếu một task/container gặp sự cố, Swarm sẽ cố gắng tạo task mới để đưa service trở lại trạng thái mong muốn.

## 8.4 Service Discovery

Các service trong Swarm có thể giao tiếp với nhau thông qua service name.

Ví dụ:

```text
frontend → backend
```

## 8.5 Rolling Update

Swarm hỗ trợ cập nhật service theo từng task, giúp giảm downtime khi triển khai phiên bản mới.

---

# 9. Nhược điểm Docker Swarm

## 9.1 Phức tạp hơn Compose

Cần hiểu thêm các khái niệm:

* Node
* Manager
* Worker
* Service
* Task
* Replica
* Stack
* Overlay Network
* Routing Mesh

## 9.2 Cần quản lý cluster

Cần quan tâm đến:

* Network
* Firewall
* Node
* Manager
* Worker
* Cluster state
* Quorum khi triển khai nhiều Manager

## 9.3 Hệ sinh thái nhỏ hơn Kubernetes

Swarm đơn giản hơn Kubernetes nhưng Kubernetes hiện được sử dụng rộng rãi hơn trong nhiều môi trường orchestration lớn.

---

# 10. Kiến trúc Lab

Lab sử dụng 2 VPS.

```text
VPS 1
Role: Manager

VPS 2
Role: Worker
```

Kiến trúc:

```text
                     Internet
                        │
                        │
                ┌───────▼───────┐
                │     VPS 1     │
                │    Manager    │
                │               │
                │    Docker     │
                └───────┬───────┘
                        │
                  Swarm Cluster
                        │
                ┌───────▼───────┐
                │     VPS 2     │
                │    Worker     │
                │               │
                │    Docker     │
                └───────────────┘
```

---

# 11. Lab A — Docker Compose

## 11.1 Mục tiêu

Triển khai một số container bằng Docker Compose trên VPS 1.

Mục tiêu là chứng minh:

> Docker Compose quản lý nhiều container trên một Docker Host.

## 11.2 Ví dụ Compose file

Tạo:

```text
compose.yaml
```

Nội dung:

```yaml
services:

  web:
    image: nginx:latest
    ports:
      - "80:80"

  redis:
    image: redis:latest
```

## 11.3 Deploy

```bash
docker compose up -d
```

Kiểm tra:

```bash
docker compose ps
```

Hoặc:

```bash
docker ps
```

Kết quả có thể giống:

```text
CONTAINER ID   IMAGE          STATUS
xxxx           nginx:latest   Up
xxxx           redis:latest   Up
```

Tất cả container đang chạy trên VPS 1.

---

# 12. Lab B — Tạo Docker Swarm Cluster

## 12.1 Kiểm tra Docker

Trên cả hai VPS:

```bash
docker --version
```

Kiểm tra Docker service:

```bash
systemctl status docker
```

Nếu Docker chưa chạy:

```bash
systemctl enable --now docker
```

---

# 13. Khởi tạo Swarm Manager

Trên VPS 1:

```bash
docker swarm init --advertise-addr <IP-VPS-1>
```

Ví dụ:

```bash
docker swarm init --advertise-addr 10.0.0.10
```

Docker sẽ trả về command tương tự:

```bash
docker swarm join \
  --token SWMTKN-xxxx \
  10.0.0.10:2377
```

Command này dùng để cho node khác join vào cluster.

---

# 14. Cho VPS 2 Join Cluster

Trên VPS 2 chạy command được trả về từ VPS 1:

```bash
docker swarm join \
  --token SWMTKN-xxxx \
  10.0.0.10:2377
```

Nếu thành công sẽ có thông báo tương tự:

```text
This node joined a swarm as a worker.
```

---

# 15. Kiểm tra Cluster

Trên VPS 1:

```bash
docker node ls
```

Ví dụ:

```text
ID        HOSTNAME   STATUS   AVAILABILITY   MANAGER STATUS
xxxx      vps1       Ready    Active         Leader
yyyy      vps2       Ready    Active
```

Giải thích:

```text
vps1
└── Manager
    └── Leader

vps2
└── Worker
```

Lúc này 2 VPS đã trở thành một Docker Swarm Cluster.

---

# 16. Deploy Service

Trên Manager:

```bash
docker service create \
  --name web \
  --publish 80:80 \
  nginx:latest
```

Kiểm tra:

```bash
docker service ls
```

Ví dụ:

```text
ID        NAME   MODE        REPLICAS
xxxx      web    replicated  1/1
```

---

# 17. Kiểm tra Task

```bash
docker service ps web
```

Ví dụ:

```text
NAME      IMAGE         NODE
web.1     nginx:latest  vps1
```

Điều này cho biết task `web.1` đang được chạy trên node nào.

---

# 18. Scale Service

Tăng service lên 4 replicas:

```bash
docker service scale web=4
```

Kiểm tra:

```bash
docker service ls
```

Kết quả:

```text
NAME   MODE        REPLICAS
web    replicated  4/4
```

Kiểm tra vị trí các task:

```bash
docker service ps web
```

Ví dụ:

```text
web.1    vps1
web.2    vps2
web.3    vps1
web.4    vps2
```

Swarm scheduler quyết định task được chạy trên node nào dựa trên trạng thái cluster và các constraint được cấu hình.

---

# 19. So sánh với Compose

Compose:

```text
VPS 1
│
├── web
├── web
├── web
└── web
```

Swarm:

```text
Swarm Cluster
│
├── VPS 1
│   ├── web
│   └── web
│
└── VPS 2
    ├── web
    └── web
```

Đây là một trong những điểm quan trọng nhất của lab.

---

# 20. Test Node Failure

## 20.1 Kiểm tra trạng thái ban đầu

```bash
docker node ls
```

Kiểm tra service:

```bash
docker service ps web
```

Giả sử:

```text
VPS 1
├── web.1
└── web.3

VPS 2
├── web.2
└── web.4
```

## 20.2 Dừng Docker trên Worker

Trên VPS 2:

```bash
systemctl stop docker
```

## 20.3 Kiểm tra từ Manager

Trên VPS 1:

```bash
docker node ls
```

Sau đó:

```bash
docker service ps web
```

Quan sát trạng thái task.

Swarm sẽ cố gắng duy trì desired state của service.

Nếu yêu cầu:

```text
replicas = 4
```

và VPS 2 không còn khả dụng, Swarm có thể tạo lại task trên node còn khả dụng nếu node đó đủ điều kiện chạy task.

---

# 21. Khởi động Worker lại

Trên VPS 2:

```bash
systemctl start docker
```

Kiểm tra:

```bash
docker node ls
```

Node sẽ quay lại trạng thái hoạt động khi kết nối cluster được khôi phục.

---

# 22. Docker Stack

Sau khi hiểu `docker service`, có thể sử dụng Docker Stack để deploy nhiều service bằng YAML.

Ví dụ:

```yaml
version: "3.8"

services:

  web:
    image: nginx:latest

    ports:
      - "80:80"

    deploy:
      replicas: 4

  redis:
    image: redis:latest
```

Deploy:

```bash
docker stack deploy -c docker-compose.yml myapp
```

Kiểm tra stack:

```bash
docker stack ls
```

Kiểm tra service:

```bash
docker stack services myapp
```

Kiểm tra task:

```bash
docker stack ps myapp
```

---

# 23. Scale bằng Stack

Có thể cấu hình số replica:

```yaml
deploy:
  replicas: 4
```

Hoặc thay đổi service bằng:

```bash
docker service scale myapp_web=6
```

Kiểm tra:

```bash
docker stack services myapp
```

---

# 24. Overlay Network

Trong Swarm có thể tạo overlay network:

```bash
docker network create \
  --driver overlay \
  app-network
```

Kiểm tra:

```bash
docker network ls
```

Overlay network cho phép các container/service trên các node khác nhau giao tiếp với nhau trong cùng một Swarm network.

Ví dụ:

```text
VPS 1                      VPS 2

frontend                   backend
   │                          │
   └──────── Overlay ─────────┘
```

---

# 25. Rolling Update

Ví dụ service đang sử dụng:

```text
nginx:1.25
```

Có thể update sang:

```text
nginx:1.26
```

Bằng:

```bash
docker service update \
  --image nginx:1.26 \
  web
```

Kiểm tra:

```bash
docker service ps web
```

Swarm sẽ thực hiện quá trình update theo cấu hình của service.

---

# 26. Một số command quan trọng

## Docker

```bash
docker ps
docker images
docker network ls
docker volume ls
```

## Compose

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs
```

## Swarm

```bash
docker swarm init
docker swarm join
docker swarm leave
docker node ls
```

## Service

```bash
docker service ls
docker service create
docker service ps
docker service scale
docker service update
docker service rm
```

## Stack

```bash
docker stack ls
docker stack deploy
docker stack services
docker stack ps
docker stack rm
```

---

# 27. Các khái niệm cần nhớ

## Node

Một Docker Host tham gia Swarm Cluster.

```text
VPS = Node
```

## Manager

Node quản lý Swarm Cluster.

## Worker

Node chạy workload được scheduler phân phối.

## Service

Định nghĩa workload cần chạy.

Ví dụ:

```text
web service
```

## Task

Một instance của service.

Ví dụ:

```text
web.1
web.2
web.3
web.4
```

## Replica

Số lượng instance mong muốn của service.

```yaml
deploy:
  replicas: 4
```

## Stack

Một nhóm nhiều service được deploy cùng nhau.

---

# 28. Bài học chính của Lab

## Docker Compose

Mô hình:

```text
Docker Host
│
├── Container
├── Container
└── Container
```

Mục tiêu:

> Quản lý nhiều container trên một host.

---

## Docker Swarm

Mô hình:

```text
             Swarm Cluster
                   │
          ┌────────┴────────┐
          │                 │
       Node 1             Node 2
       Manager            Worker
          │                 │
       Tasks               Tasks
```

Mục tiêu:

> Quản lý workload trên nhiều Docker Host.

---

# 29. Kết luận

Có thể ghi nhớ ngắn gọn:

```text
Docker Compose
    ↓
Multi-container application
    ↓
Một Docker Host
```

Trong khi:

```text
Docker Swarm
    ↓
Container orchestration
    ↓
Nhiều Docker Host
    ↓
Cluster
    ↓
Scheduling
    ↓
Scaling
    ↓
Service Discovery
    ↓
Self-healing
    ↓
Rolling Update
```

Câu quan trọng cần nhớ:

> **Compose tập trung vào việc định nghĩa và chạy nhiều container, còn Swarm tập trung vào việc orchestration các service trên nhiều Docker Host.**

---

# 30. Checklist hoàn thành Lab

* [ ] Cài Docker trên VPS 1
* [ ] Cài Docker trên VPS 2
* [ ] Chạy Docker Compose trên VPS 1
* [ ] Kiểm tra container bằng `docker ps`
* [ ] Khởi tạo Swarm Manager
* [ ] Cho VPS 2 join Swarm
* [ ] Kiểm tra bằng `docker node ls`
* [ ] Deploy nginx service
* [ ] Kiểm tra bằng `docker service ls`
* [ ] Kiểm tra task bằng `docker service ps`
* [ ] Scale service lên 4 replicas
* [ ] Quan sát task trên 2 VPS
* [ ] Test Worker failure
* [ ] Kiểm tra khả năng khôi phục service
* [ ] Tạo Overlay Network
* [ ] Deploy Docker Stack
* [ ] Test Rolling Update
* [ ] Ghi lại sự khác nhau giữa Compose và Swarm

---

# 31. Cấu trúc Repository đề xuất

```text
docker-swarm-lab/
│
├── README.md
│
├── compose/
│   └── compose.yaml
│
├── swarm/
│   ├── stack.yaml
│   └── commands.md
│
└── screenshots/
    ├── compose.png
    ├── swarm-node.png
    ├── service.png
    ├── scaling.png
    └── failover.png
```

Mục tiêu cuối cùng của lab:

```text
             Docker Compose
                    │
                    │
              Single Host
                    │
                    ▼
             docker compose
                    │
                    │
                    ▼
             Multiple Containers


             Docker Swarm
                    │
                    │
                    ▼
               Cluster
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Manager              Worker
          │                   │
          └─────────┬─────────┘
                    ▼
                 Services
                    │
                    ▼
                 Tasks
                    │
                    ▼
              Scaling / HA /
          Service Discovery /
             Self-healing
```

