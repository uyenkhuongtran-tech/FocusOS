# FocusOS — Repo mẫu deploy GitHub Pages (1 lần là xong)

Thư mục này đã đóng gói sẵn mọi thứ để đưa **FocusOS** lên web miễn phí.

```
focusos-repo/
├── index.html                      ← App (đã bọc HTML5 + viewport, đổi tên sẵn)
├── .nojekyll                       ← Tắt Jekyll (giữ nguyên file tĩnh)
├── .gitignore
├── .github/workflows/deploy.yml    ← Tự động deploy mỗi lần push
└── domain-is-a-dev/                ← Dùng KHI muốn domain đẹp (không bắt buộc)
    ├── focusos.json                ← File để PR đăng ký <tên>.is-a.dev
    └── CNAME                       ← Copy ra root SAU khi PR is-a.dev được merge
```

---

## BƯỚC 1 — Đưa lên GitHub (chọn 1 trong 2)

### Cách A — Web UI (không cần cài gì)
1. Tạo repo public: https://github.com/new → tên `focusos` → **Public** → Create.
2. **Add file → Upload files** → kéo thả **toàn bộ nội dung bên trong** `focusos-repo/` (cả thư mục `.github`).
   - Lưu ý: kéo *nội dung bên trong*, không kéo cả thư mục cha, để `index.html` nằm ở gốc repo.
3. Commit.

### Cách B — Dòng lệnh
```bash
cd focusos-repo
git init && git add . && git commit -m "FocusOS v1"
git branch -M main
git remote add origin https://github.com/<github-username>/focusos.git
git push -u origin main
```

## BƯỚC 2 — Bật GitHub Pages
Repo → **Settings → Pages → Source = GitHub Actions**.
(Đã có sẵn workflow `deploy.yml` nên mỗi lần push site tự cập nhật.)

✅ Sau ~1 phút có ngay (free + HTTPS):
```
https://<github-username>.github.io/focusos/
```
Dùng để **testing** là đủ. Nếu chỉ cần vậy thì BỎ QUA Bước 3.

---

## BƯỚC 3 — (Tùy chọn) Domain đẹp free: `<tên>.is-a.dev`

> Năm 2026 Freenom đã ngừng — dùng **is-a.dev** (free, qua GitHub PR). Schema có thể đổi, nên đối chiếu README mới nhất tại https://github.com/is-a-dev/register trước khi PR.

1. Mở `domain-is-a-dev/focusos.json`, sửa:
   - `username` → tài khoản GitHub của anh
   - `email` → email của anh
   - `records.CNAME` → `<github-username>.github.io`
   - (Muốn subdomain tên khác? Đổi tên file `focusos.json` → `<tên>.json`. Tên file = subdomain.)
2. **Fork** https://github.com/is-a-dev/register → copy file JSON vào thư mục `domains/` của bản fork → tạo **Pull Request**. Chờ merge.
3. Sau khi merge: copy file `domain-is-a-dev/CNAME` ra **gốc repo** `focusos` (sửa nội dung thành `<tên>.is-a.dev` nếu đổi tên). Commit & push.
   - Hoặc: Repo → **Settings → Pages → Custom domain** → nhập `<tên>.is-a.dev` → Save (GitHub tự tạo CNAME).
4. Đợi DNS + SSL vài phút → tick **Enforce HTTPS**.

✅ Truy cập:
```
https://<tên>.is-a.dev
```

> Vì sao tách CNAME ra thư mục riêng? Nếu để CNAME ở gốc ngay từ đầu, GitHub ép dùng custom domain và **domain mặc định github.io sẽ ngừng hoạt động** trước khi DNS sẵn sàng. Tách ra → domain mặc định chạy ngay, khi nào xong is-a.dev thì mới kích hoạt.

---

## Kiểm tra nhanh sau deploy
- Mở trên desktop + mobile (đã có viewport → co giãn đúng).
- Sửa code → `git push` (hoặc upload đè) → tự cập nhật ~1 phút. Ctrl/Cmd+Shift+R để xóa cache.
- GitHub Pages chỉ host **tĩnh**; FocusOS là single-file client-side nên hợp hoàn toàn.
