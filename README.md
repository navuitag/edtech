# EdTech Hub

<p align="center">
  <img src="icons/logo.svg" alt="EdTech Hub" width="360">
</p>

Monorepo làm việc cho bộ ứng dụng học tập PWA dành cho học sinh Việt Nam. Mỗi môn học là một ứng dụng web độc lập (HTML/CSS/JavaScript), chạy offline, lưu tiến độ trên thiết bị và hỗ trợ **nhiều người học** trên cùng máy.

Repo gốc này (`edtech`) chứa **trang hub** để chọn môn học. Các dự án con nằm trong thư mục con, mỗi dự án có git repository riêng.

---

## Ứng dụng

| Môn | Thư mục | Repository | Mô tả ngắn |
|-----|---------|------------|------------|
| 📐 Toán | `mathflow/` | [MathFlow](https://github.com/navuitag/MathFlow) | Học Toán lớp 1–9, ôn hè, chuyên đề, phân tích lỗi sai |
| 🧬 Sinh học | `bioflow/` | [bioflow](https://github.com/navuitag/bioflow) | Sinh học THCS, sơ đồ tư duy, chuyên đề |
| ⚗️ Hóa học | `chemflow/` | [chemflow](https://github.com/navuitag/chemflow) | Hóa học THCS, thí nghiệm trực quan |
| ⚡ Vật lí | `phyflow/` | [phyflow](https://github.com/navuitag/phyflow) | Vật lí THCS, mô phỏng 3D |
| 📖 Ngữ văn | `literatureflow/` | [literatureflow](https://github.com/navuitag/literatureflow) | Ngữ văn, ôn hè, sơ đồ tư duy |
| 💻 Tin học | `ITflow/` | [ITflow](https://github.com/navuitag/ITflow) | Tin học, luyện chuột/bàn phím, Scratch |
| 🇬🇧 Tiếng Anh | `englishflow/` | [englishflow](https://github.com/navuitag/englishflow) | Tiếng Anh theo lớp, nghe-nói-viết |
| 🗣️ Học tiếng Anh trong 60 ngày | `engcoach/` | [engcoach](https://github.com/navuitag/engcoach) | Lộ trình tiếng Anh 60 ngày, flashcard SRS |

> **EduFlash** (`eduflash/`) là dự án Next.js riêng (PostgreSQL, Vercel), không nằm trong nhóm PWA *Flow.

---

## Chạy nhanh (trang hub)

Clone repo gốc và các dự án con vào cùng thư mục, rồi phục vụ từ **thư mục gốc** `edtech/`:

```bash
cd edtech
python3 -m http.server 8000
```

Mở trình duyệt: **http://localhost:8000** — trang `index.html` liệt kê tất cả môn học.

Cấu trúc thư mục mong đợi:

```text
edtech/
├── index.html          # Hub chọn môn (repo edtech)
├── mathflow/
├── bioflow/
├── chemflow/
├── phyflow/
├── literatureflow/
├── ITflow/
├── englishflow/
└── engcoach/
```

---

## Chạy từng ứng dụng

Mỗi app cần **HTTP server** (ES Modules, Service Worker — không mở trực tiếp `file://`).

```bash
# Ví dụ MathFlow
cd mathflow
python3 -m http.server 8000
# → http://localhost:8000

# Ví dụ EngCoach
cd engcoach
python3 -m http.server 8080
# → http://localhost:8080
```

Chi tiết từng dự án: xem `README.md` trong thư mục con (nếu có).

---

## Liên kết chéo giữa các môn

Trong mỗi ứng dụng:

- Nút **「Môn học」** trên thanh điều hướng — chuyển sang môn khác
- Lưới **「Khám phá môn học khác」** trên trang chủ
- EngCoach: thêm danh sách môn ở sidebar

URL được tạo tự động theo môi trường:

| Môi trường | Cách hoạt động |
|------------|----------------|
| Monorepo local | Liên kết tương đối `../mathflow/index.html#/home` |
| GitHub Pages | `https://<user>.github.io/<repo>/index.html#/home` |
| Tùy chỉnh | `localStorage.setItem('edtech_hub_base', 'https://domain.com/edtech')` |

Logic dùng chung: `edtechApps.js` + `edtechHub.js` (trong từng dự án).

---

## Kiến trúc chung (*Flow apps)

7 ứng dụng *Flow (MathFlow, BioFlow, …) dùng cùng mô hình:

```text
{project}/
├── index.html
├── manifest.json
├── service-worker.js
├── assets/js/          # app.js, router.js, state.js, profileStore.js
├── assets/css/
├── components/         # navbar, learnerSwitcher, edtechHub, …
├── modules/            # lessonEngine, quizEngine, mindMap, …
├── data/               # JSON nội dung bài học
└── scripts/            # (một số dự án) generator nội dung
```

**EngCoach** dùng kiến trúc script tag (không ES modules), sidebar tĩnh trong `index.html`.

### Tính năng chung

- **PWA + offline** — Service Worker cache shell
- **Hash router** — `#/home`, `#/skills`, …
- **Multi-learner** — nhiều hồ sơ trên cùng thiết bị (`localStorage`)
- **Giờ học** — ghi nhận thời gian học theo phút (tab đang mở)
- **Game hóa** — XP, streak, daily quest, mastery
- **Sơ đồ tư duy** — mind map theo chương/chủ đề

---

## Cấu trúc repo Git

| Repo | Nội dung |
|------|----------|
| [navuitag/edtech](https://github.com/navuitag/edtech) | Hub (`index.html`), `.gitignore` |
| `navuitag/{MathFlow,bioflow,…}` | Từng ứng dụng môn học (repo độc lập) |

Thư mục con bị **gitignore** trong repo gốc để tránh nested git conflict. Khi làm việc:

1. **Commit hub** → repo `edtech`
2. **Commit từng app** → repo tương ứng (`cd mathflow && git commit …`)

---

## Scripts công cụ (local)

Thư mục `scripts/` (không commit vào repo `edtech`) chứa công cụ phát triển monorepo:

```text
scripts/
├── shared/                 # Module dùng chung (mindMap, studyTime, edtechHub, …)
├── patch-mindmap.mjs       # Triển khai mind map sang các *Flow
├── patch-edtech-hub.mjs    # Triển khai liên kết chéo môn học
├── patch-workbook-ui.mjs
└── generate-favicons.py
```

Chạy patch (ví dụ):

```bash
node scripts/patch-edtech-hub.mjs
```

---

## Yêu cầu hệ thống

- Trình duyệt hiện đại (Chrome, Firefox, Safari, Edge)
- Python 3 hoặc Node.js để chạy HTTP server local
- Không cần backend, database hay đăng nhập cho các app *Flow / EngCoach

---

## Giấy phép & liên hệ

Mỗi dự án con có thể có giấy phép riêng. Mã nguồn được host tại tổ chức GitHub [navuitag](https://github.com/navuitag).

---

## Tác giả

- **Nguyễn Anh Vũ**
- Email: [navuitag@gmail.com](mailto:navuitag@gmail.com)
- Điện thoại: [0986201079](tel:+84986201079)
