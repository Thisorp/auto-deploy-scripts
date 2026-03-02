# auto-deploy-scripts

---

# 🧠 Tổng Quan Script Deploy Menu

Script của bạn là một **menu deploy bán-tự-động** dùng cho:

* VM mới
* Server staging / production
* Không cần CI/CD phức tạp
* Deploy image từ registry
* Có kiểm tra health + logs

---

# ⚙️ Cách Hoạt Động Tổng Thể

Script chia thành 4 nhóm chức năng:

```
1️⃣ Quản lý cấu hình
2️⃣ Quản lý Git source code
3️⃣ Quản lý Docker Compose
4️⃣ Hỗ trợ kiểm tra & debug
```

---

# 1️⃣ Nhóm Cấu Hình

## 1) Set/Update Config

Lưu vào:

```
~/.deploy_menu.conf
```

Lưu các biến:

* APP_DIR
* REPO_URL
* COMPOSE_PATH

→ Các chức năng sau không hỏi lại nữa.

---

# 2️⃣ Nhóm Git

## 2) Clone code (first time)

Dùng khi VM mới.

Thực hiện:

```
git clone <REPO_URL> <APP_DIR>
git checkout main
```

⚠ Không clone nếu thư mục không rỗng (đã được chuẩn hoá).

---

## 3) Pull code (update)

Thực hiện:

```
git fetch
git checkout main
git reset --hard origin/main
```

Nếu remote là HTTPS → hỏi GitHub user/token.

👉 Đảm bảo server luôn đồng bộ đúng code remote.

---

# 3️⃣ Nhóm Docker Compose

## 4) Docker compose up -d (ALL services)

Chạy:

```
docker compose up -d --remove-orphans
```

Dùng khi:

* Bạn đổi tag image
* Hoặc deploy lại toàn bộ stack

---

## 5) Docker compose up -d (ONE service)

Chạy:

```
docker compose up -d <service>
```

Dùng khi:

* Chỉ bump version 1 service
* Muốn tránh restart cả hệ thống

---

## 6) Check status

In:

```
docker compose ps
```

Và kiểm tra:

* running / exited
* healthy / unhealthy
* no-healthcheck

---

## 7) Wait health

Chờ tối đa 120s:

* Tất cả container phải running
* Nếu có HEALTHCHECK → phải healthy

Nếu timeout → cảnh báo.

---

## 8) View logs

Cho phép:

* Tail logs all
* Follow logs all
* Tail 1 service
* Follow 1 service

Dùng debug khi service lỗi.

---

## 9) Full Deploy (QUAN TRỌNG)

Flow:

```
Pull code (git)
docker compose up -d
Wait health
Status
Logs tail
```

🚫 Không docker pull riêng.

Dùng khi:

* Dev push code
* Hoặc bump version image trong compose

---

## 10) Restart Service

2 option:

```
docker compose restart
docker compose restart <service>
```

Dùng khi:

* App treo
* Config thay đổi
* Không thay đổi image

---

# 4️⃣ Các Cơ Chế Thông Minh Trong Script

### ✔ Không hỏi lại compose path

Nhờ `ensure_compose_path`

---

### ✔ Chỉ hỏi GitHub credentials khi cần

Nếu remote HTTPS

---

### ✔ Không docker pull thừa

Chỉ pull khi cần (theo logic tag thay đổi)

---

### ✔ Có health wait

Giúp production ổn định hơn

---

# 🎯 Cách Deploy Thực Tế Bạn Nên Dùng

## Trường hợp 1 – Dev push code mới

→ Chọn:

```
8) Full deploy
```

---

## Trường hợp 2 – Chỉ bump version backend

→ Chọn:

```
5) Docker compose up -d (one service)
```

---

## Trường hợp 3 – App treo

→ Chọn:

```
10) Restart service
```

---

## Trường hợp 4 – Server mới

```
1) Set config
2) Clone
4) Compose up
```

---

# 🏗 Kiến Trúc Deploy Hiện Tại Của Bạn

Mô hình của bạn hiện đang là:

```
Developer push code
        ↓
VM pull code
        ↓
Docker compose up
        ↓
Pull image nếu tag mới
        ↓
Recreate container
```

→ Đây là mô hình semi-manual DevOps, hoàn toàn ổn cho team nhỏ hoặc internal system.

---

# 📌 Mức độ “chuẩn production” hiện tại

| Tiêu chí               | Trạng thái |
| ---------------------- | ---------- |
| Config tách riêng      | ✅          |
| Không hỏi lại path     | ✅          |
| Pull code an toàn      | ✅          |
| Không pull image thừa  | ✅          |
| Có health check        | ✅          |
| Có restart riêng       | ✅          |
| Có deploy từng service | ✅          |


---

