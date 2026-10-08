# devops-lab1-portfolio

Lab 1 — DevOps Course: My First CI/CD Pipeline.

Website: https://hungchuchue562005-jpg.github.io/devops-lab1-portfolio/

## Cấu trúc mã nguồn trên nhánh main

```text
devops-lab1-portfolio/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── .gitignore
├── index.html
└── README.md
```

`_site/` là thư mục tạm được pipeline tạo để xuất bản website. Pipeline
chỉ sao chép `index.html` vào đó; nếu thêm CSS, JavaScript hoặc ảnh riêng,
cần bổ sung lệnh sao chép các tài nguyên này trong bước `Prepare Website`.

## Chạy trên macOS

```sh
open index.html
```

Đường dẫn đúng của workflow là `.github/workflows/deploy.yml`.
Trên macOS/Linux, tạo thư mục bằng `mkdir -p .github/workflows`.
Dùng `mkdir .github\workflows` trong shell này sẽ tạo nhầm `.githubworkflows`.

## Pipeline và cách cập nhật

```text
Sửa code trên main → git push origin main → GitHub Actions
→ kiểm tra index.html → tạo _site/ → xuất bản sang gh-pages → GitHub Pages
```

Workflow chạy khi push lên `main` hoặc chạy thủ công từ tab **Actions**.
`permissions: contents: write` đặt ở cấp workflow để `GITHUB_TOKEN` có
quyền tạo/cập nhật nhánh `gh-pages`. Không cần tự tạo token hoặc secret.

## Cấu hình GitHub Pages lần đầu

1. Push mã nguồn lên `main` và đợi workflow **Deploy to GitHub Pages** thành công.
   Action tự tạo nhánh `gh-pages`; không cần tạo nhánh này bằng tay.
2. Vào repository → **Settings → Pages**.
3. Chọn **Source: Deploy from a branch**.
4. Chọn **Branch: gh-pages**, thư mục **/ (root)**, rồi **Save**.
5. Đợi job **pages build and deployment** thành công, sau đó mở URL website.

Workflow này dùng `peaceiris/actions-gh-pages`, nên nguồn Pages cần là
**Deploy from a branch**, với nhánh **gh-pages**.

Mỗi lần cập nhật website:

```sh
git switch main
# Sửa index.html rồi lưu file.
git add index.html
git commit -m "update: portfolio content"
git push origin main
```

Luôn sửa và push mã nguồn lên `main`. `gh-pages` là đầu ra do pipeline quản lý;
sửa trực tiếp nhánh đó có thể bị lần deploy tiếp theo ghi đè.

## Nguyên nhân lỗi và cách kiểm tra

- `permissions` nằm trong một phần tử `steps` làm workflow không hợp lệ
  (GitHub có thể báo `Unexpected value 'permissions'`). Khóa này phải nằm ở
  cấp workflow hoặc job. Workflow đã chuyển nó lên cấp workflow.
- Bản hướng dẫn ban đầu thiếu `contents: write`; token chỉ có quyền đọc
  có thể khiến bước push nhánh `gh-pages` báo lỗi 403. Workflow đã khai báo
  quyền ghi cần thiết.
- Thư mục `.githubworkflows` không phải nơi GitHub tìm workflow. File đúng
  phải nằm trong `.github/workflows/`.
- Push lên `gh-pages` không kích hoạt pipeline có trigger `branches: [main]`.
  Cập nhật bằng cách push lên `main`.
- Website báo 404 sau khi action xuất bản thành công: kiểm tra Pages đã chọn
  `gh-pages` và `/ (root)`, rồi đợi job Pages hoàn tất.

Theo dõi tại tab **Actions**:
https://github.com/hungchuchue562005-jpg/devops-lab1-portfolio/actions

Lưu ý: ngày hiển thị trong trang hiện lấy từ đồng hồ trình duyệt lúc mở trang,
không phải thời điểm GitHub triển khai.
