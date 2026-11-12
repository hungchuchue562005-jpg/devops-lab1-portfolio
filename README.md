# devops-lab1-portfolio

Lab 1 — HTML/CSS/JavaScript, GitHub Actions và GitHub Pages.

[Website portfolio](https://hungchuchue562005-jpg.github.io/devops-lab1-portfolio/)
· [GitHub Actions](https://github.com/hungchuchue562005-jpg/devops-lab1-portfolio/actions)

## Cấu trúc dự án

```text
devops-lab1-portfolio/
├── .github/workflows/
│   ├── deploy.yml       # Giữ nguyên bản có sẵn khi bắt đầu sửa
│   └── lighthouse.yml   # Audit độc lập với deploy
├── assets/
│   ├── avatar.png      # Bản sao ảnh đã có, tên không dấu
│   ├── favicon.svg     # Icon nhỏ để tránh lỗi favicon.ico 404
│   └── ảnh test.png    # Giữ lại ảnh gốc
├── index.html
├── about.html
├── contact.html
├── .gitignore
└── README.md
```

Ba trang dùng navigation tương đối `./index.html`, `./about.html`,
`./contact.html`; mỗi trang có một liên kết `aria-current="page"`.
Avatar ở Home dùng `./assets/avatar.png`, giữ kích thước 120 × 120,
hình tròn và viền tím. Cả ba trang dùng favicon SVG nhỏ, tránh tải ảnh avatar
lớn chỉ để hiển thị icon trên tab trình duyệt.

## Nguyên nhân avatar và trang phụ bị 404

Kiểm tra online ngày 14/10/2026 cho thấy:

- Ảnh gốc đã được Git theo dõi trên `main`, không bị `.gitignore` loại trừ.
- [Workflow trên main](https://github.com/hungchuchue562005-jpg/devops-lab1-portfolio/blob/main/.github/workflows/deploy.yml)
  chỉ chép `index.html` vào `_site/`, rồi xuất bản `_site/`.
- [Nhánh gh-pages](https://github.com/hungchuchue562005-jpg/devops-lab1-portfolio/tree/gh-pages)
  chỉ có `index.html` và `.nojekyll`.
- Home trả HTTP 200; ảnh gốc, `about.html`, `contact.html` trả HTTP 404.

Nguyên nhân đã xác minh là thiếu tài nguyên trong đầu ra deploy. Tên ảnh có
dấu và khoảng trắng không phải nguyên nhân chính được chứng minh.

Khi bắt đầu công việc, `deploy.yml` trong máy **đã có thay đổi chưa commit**
so với `HEAD`, trong đó `publish_dir` là `./`. Bản này được giữ nguyên từng byte;
không sửa hoặc khôi phục nó. SHA-256 trước và sau công việc:

```text
b4fa086c15b810274a1a4d228f35fe00e8b78be5ffb56c52be5f741072f118ef
```

Với bản deploy hiện tại trong máy, thư mục gốc chứa đủ ba trang và `assets/`,
nên lần deploy tiếp theo sẽ có avatar và các trang phụ. Chỉ sửa HTML hoặc thêm
workflow Lighthouse rồi giữ bản deploy cũ trên GitHub sẽ chưa giải quyết lỗi
thiếu tài nguyên. Cần đưa **bản deploy hiện tại đã có sẵn trong máy** lên `main`.

## Lighthouse CI

Mã đầy đủ nằm trong [lighthouse.yml](.github/workflows/lighthouse.yml).
Workflow chạy khi push lên `main` và hỗ trợ **Run workflow** thủ công.
Nó chạy độc lập trên Ubuntu, chỉ yêu cầu quyền đọc source code:

1. Kiểm tra ba file HTML và `assets/avatar.png` tồn tại, không rỗng.
2. Chép ba trang và toàn bộ `assets/` vào `_site/` riêng của job audit.
3. Cài Node.js 22 và `@lhci/cli@0.15.1`.
4. Dùng `staticDistDir` để mở server tạm trên runner; audit Home, About,
   Contact theo cấu hình mobile, mỗi trang ba lần.
5. Đo Performance, Accessibility, Best Practices và SEO.
6. Đọc báo cáo thực tế để hiển thị điểm 0–100 trong logs và **Job Summary**.
7. Lưu chín báo cáo HTML, chín báo cáo JSON và manifest vào artifact
   `lighthouse-reports`, giữ 14 ngày.

| Tiêu chí | Ngưỡng cảnh báo |
| --- | ---: |
| Performance | 70/100 |
| Accessibility | 80/100 |
| Best Practices | 80/100 |
| SEO | 80/100 |

Ngưỡng là cảnh báo chất lượng, không phải điểm cố định. Điểm trong Summary
lấy từ lần chạy đại diện do LHCI chọn trong ba lần audit mỗi trang. Lỗi chạy
audit, lỗi xuất báo cáo hoặc thiếu điểm/trang vẫn làm job thất bại. Không cần
LHCI Server, tài khoản, token riêng, hosting mới hay database.

Audit chạy trên runner, nên không xác nhận website online đã deploy đúng.
Điểm local và GitHub Actions có thể khác nhau theo môi trường đo.
Tham khảo [tài liệu cấu hình LHCI](https://googlechrome.github.io/lighthouse-ci/docs/configuration.html).

## Chạy và kiểm tra local

```sh
python3 -m http.server 8000
```

Mở `http://localhost:8000/`, bấm Home → About → Contact → Home;
kiểm tra avatar và `http://localhost:8000/assets/avatar.png`.

```sh
git status --short
git ls-files assets
git check-ignore -v assets/avatar.png
git diff -- .github/workflows/deploy.yml
shasum -a 256 .github/workflows/deploy.yml
git diff --check
```

`git check-ignore` không in gì, mã thoát 1 nghĩa là ảnh **không bị ignore**.
`avatar.png` mới chỉ xuất hiện trong `git ls-files` sau khi `git add`.
Diff của `deploy.yml` vẫn chứa thay đổi có sẵn trước công việc; không thể dùng
một diff rỗng để chứng minh file này được giữ nguyên trong phiên sửa.

## Tự commit và push

Chưa có commit, push hoặc thay đổi cấu hình repository nào được thực hiện.
Trên macOS, Git có thể còn ghi tên workflow cũ `Lighthouse.yml`. Nếu
`git ls-files .github/workflows` vẫn hiển thị tên này, ghi nhận việc đổi tên:

```sh
git mv .github/workflows/Lighthouse.yml .github/workflows/lighthouse.rename
git mv .github/workflows/lighthouse.rename .github/workflows/lighthouse.yml
```

Xem lại các thay đổi trước khi commit. Lệnh `git add deploy.yml` dưới đây
bao gồm **thay đổi deploy đã có sẵn của bạn**, cần thiết để xuất bản đủ tài nguyên:

```sh
git diff -- .github/workflows/deploy.yml
git add .github/workflows/lighthouse.yml assets/avatar.png assets/favicon.svg index.html about.html contact.html README.md
git add .github/workflows/deploy.yml
git diff --cached --stat
git commit -m "feat: add avatar and multi-page Lighthouse CI"
git push origin main
```

## Xem pipeline và báo cáo trên GitHub

1. Vào **Actions**, mở lần chạy **Deploy to GitHub Pages** mới nhất, đợi thành công.
2. Đợi **pages build and deployment** hoàn tất; kiểm tra nhánh `gh-pages` có
   `index.html`, `about.html`, `contact.html`, `assets/avatar.png`.
3. Mở website, tải lại không dùng cache, bấm navigation và mở trực tiếp
   [avatar](https://hungchuchue562005-jpg.github.io/devops-lab1-portfolio/assets/avatar.png),
   [About](https://hungchuchue562005-jpg.github.io/devops-lab1-portfolio/about.html),
   [Contact](https://hungchuchue562005-jpg.github.io/devops-lab1-portfolio/contact.html).
   Các URL này phải trả HTTP 200.
4. Mở **Lighthouse CI - Audit Portfolio** → lần chạy tương ứng → **Summary**
   để xem bảng điểm của ba trang.
5. Trong **Artifacts**, tải `lighthouse-reports`, giải nén và mở các file
   `.report.html` để xem chi tiết; JSON phục vụ đối chiếu và xử lý tự động.
6. Để chạy lại audit, chọn workflow Lighthouse → **Run workflow** → `main`.

Nếu deploy báo 403, kiểm tra **Settings → Actions → General → Workflow
permissions** có cho phép **Read and write permissions**. Bản deploy hiện tại
trong máy dùng quyền mặc định của repository. GitHub Pages cần chọn
**Deploy from a branch**, nhánh **gh-pages**, thư mục **/ (root)**.
Chỉ thay đổi các thiết lập này khi bạn chủ động quyết định thực hiện.

“Ngày truy cập” trên Home lấy từ đồng hồ trình duyệt, không phải thời điểm deploy.
