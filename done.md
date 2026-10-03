| Tiêu chí              | Docker Compose                           | Docker Swarm                                               |
| --------------------- | ---------------------------------------- | ---------------------------------------------------------- |
| **Mục đích chính**    | Chạy/quản lý nhiều container             | Điều phối container trên nhiều máy                         |
| **Phạm vi**           | Thường 1 Docker Host                     | Nhiều Docker Host                                          |
| **Đơn vị quản lý**    | Container                                | Service → Task → Container                                 |
| **Cluster**           | Không có cluster                         | Có cluster                                                 |
| **Manager / Worker**  | Không có                                 | Có                                                         |
| **Scheduling**        | Không có scheduler cấp cluster           | Manager tự quyết định task chạy node nào                   |
| **Scale**             | Scale container trên host hiện tại       | Scale service trên toàn cluster                            |
| **Self-Healing**      | Hạn chế, chủ yếu trong phạm vi host      | Có, duy trì desired replicas                               |
| **Failover node**     | Không chuyển workload sang máy khác      | Có thể tạo task trên node còn sống                         |
| **Load Balancing**    | Có thể dùng reverse proxy bên ngoài      | Có routing mesh/service load balancing                     |
| **Service Discovery** | Có network nội bộ                        | Có service discovery trong Swarm                           |
| **Overlay Network**   | Không phải orchestration multi-node      | Có, kết nối service giữa các node                          |
| **Rolling Update**    | Không phải chức năng orchestration chính | Hỗ trợ                                                     |
| **Desired State**     | Không phải mô hình cluster               | Cốt lõi của Swarm                                          |
| **High Availability** | Không ở cấp cluster                      | Có khả năng ở cấp service; cần nhiều manager để HA manager |
| **Độ phức tạp**       | Dễ                                       | Cao hơn                                                    |
| **Phù hợp**           | Dev, test, app nhỏ, single server        | Multi-node deployment, orchestration                       |
| **Quản lý**           | `docker compose`                         | `docker service`, `docker node`, `docker stack`            |

---

## Test 1 – Self-Healing & Failover

4 replicas  
↓  
2 node  
↓  
Worker bị down  
↓  
Manager phát hiện  
↓  
Task trên Worker bị Shutdown  
↓  
Manager tạo Task mới  
↓  
Service quay về desired state = 4 replicas

<img width="1857" height="869" alt="ảnh" src="https://github.com/user-attachments/assets/1258b726-8367-4373-8dad-22ae1344d4cc" />

---

## Test 2 – Load Balancing giữa các Replicas

4 replicas  
↓  
Client gửi request  
↓  
Swarm Routing Mesh  
↓  
Phân phối request  
↓  
Replica 1 → Replica 2 → Replica 3 → Replica 4...

<img width="935" height="912" alt="ảnh" src="https://github.com/user-attachments/assets/22df3e92-49d9-48df-a6f7-c98cc42805a3" />
