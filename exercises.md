# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn:*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Ngụy Quang Hùng  Mã học viên: 2A202602998

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> *Câu trả lời của bạn:* Nếu để mặc định là `"changeme"`, lúc deploy lên cloud ứng dụng vẫn chạy bình thường. Ai cũng có thể mò ra key này hoặc gọi API tự do, dẫn đến việc tài khoản LLM của bạn bị dùng chùa và trừ sạch tiền. "Chết sớm" giúp ta lập tức nhận ra mình quên cấu hình bảo mật trước khi hệ thống lên mạng.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> *Câu trả lời của bạn:* Dòng log: `{"timestamp": "2026-09-28T08:50:11.000000Z", "level": "info", "event": "ask_completed", "user_id": "sv-test", "cost_usd": 0.0001}`. Hai việc làm được: 1. Đưa vào hệ thống như Kibana/Datadog để vẽ biểu đồ tổng chi phí (`sum(cost_usd)`) theo từng User. 2. Lọc cực nhanh và chính xác mọi hành động của một `user_id` cụ thể bằng câu truy vấn hệ thống mà không cần dùng Regex dò tìm chuỗi.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản                 | Dung lượng |
| -------------------- | ------------ |
| 1 stage (bản đầu) | ~1000 MB     |
| Multi-stage          | ~150 MB      |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *Câu trả lời của bạn:* Phần dung lượng khổng lồ bị cắt giảm chính là các công cụ biên dịch (gcc, make...), mã nguồn gốc của các thư viện (dependency) và các file cache của `pip`. Multi-stage chỉ sao chép các file nhị phân và thư viện đã được build xong ở môi trường chạy (runtime) nên rất nhẹ.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> *Câu trả lời của bạn:* Với cấu trúc hiện tại, các layer cài đặt thư viện (`pip install`) được dùng lại từ cache. Chỉ có layer `COPY . .` và các bước cấu hình sau đó chạy lại. Nếu đặt `COPY . .` lên đầu, chỉ cần ta sửa một dấu phẩy trong code, cache của `COPY` sẽ bị vô hiệu hóa, kéo theo `pip install` phía dưới phải tải và cài lại toàn bộ thư viện từ số 0, cực kỳ tốn thời gian build.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> *Câu trả lời của bạn:* Nếu code bị lỗi thực thi lệnh (RCE), hacker chiếm quyền shell trong container. Vì container chạy quyền root, hacker có thể lợi dụng mount volume hoặc khai thác các lỗi Linux kernel để thoát khỏi sandbox (container breakout) và chiếm luôn máy chủ vật lý. Lệnh `USER <non-root>` ép app chạy bằng quyền user thường, nên dù hacker vào được container cũng không có đặc quyền để thực hiện leo thang chiếm máy chủ.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> *Câu trả lời của bạn:* Có thể gửi tới 20 request trong vòng 2 giây. User chỉ cần gửi 10 request vào lúc 10:00:59 (chưa chạm giới hạn phút 00) và lập tức gửi thêm 10 request vào lúc 10:01:01 (hệ thống vừa reset sang phút 01 nên lại cho qua).

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> *Câu trả lời của bạn:* Rate Limit chặn theo số lượng tần suất request, còn Cost Guard chặn theo tổng chi phí.
>
> 1. Rate Limit cho qua nhưng Cost Guard chặn: User chỉ gửi 1 request nhưng đính kèm lượng text khổng lồ tốn cực nhiều token. Số lượng chưa bị limit chặn, nhưng số tiền vượt ngân sách tháng nên Cost Guard chặn.
> 2. Cost Guard cho qua nhưng Rate Limit chặn: User gửi 50 request trong 1 phút, mỗi request chỉ dài 1 từ (tốn cực ít tiền). Tổng tiền vẫn an toàn, nhưng spam quá nhanh nên Rate Limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> *Câu trả lời của bạn:* Khi Redis rớt mạng, cả 3 container sẽ đồng loạt báo lỗi 503 cho gộp `/health`. Orchestrator (Docker/K8s) gọi `/health` thấy fail liên tục sẽ tưởng container đã chết hoàn toàn, liền ra lệnh **kill và restart** lại cả 3 container. Hậu quả là toàn bộ hệ thống bị sập (downtime). Tách riêng giúp app báo `/ready` 503 để ngưng nhận việc, nhưng `/health` vẫn 200 để giữ mạng sống chờ Redis hồi phục.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> *Câu trả lời của bạn:* Nếu lưu bằng dict Python, biến số sẽ bị phân mảnh độc lập ở 3 vùng nhớ của 3 container. Khi load balancer chuyển request loạn xạ, ta sẽ thấy `history_length` nhảy cóc và thụt lùi (ví dụ: 1, 1, 2, 1, 3...). Các câu hỏi bị chia năm xẻ bảy làm Bot mất trí nhớ. Nếu xài Redis, dữ liệu tập trung 1 chỗ nên `history_length` sẽ tăng tịnh tiến (1, 2, 3...).

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn:* Trong lúc Deploy Railway, lúc đầu mình chưa gắn biến `REDIS_URL`. App liền bị crash và lỗi fail fast ở file config với nội dung báo thiếu field `redis_url`. Mình đã khắc phục bằng cách truy cập tab Variables của dự án Agent trên Railway, add tham chiếu biến `REDIS_URL` từ dịch vụ Redis nội bộ qua. Sau đó Railway tự động redeploy và chạy ổn định.
