# Menu AR — Hướng dẫn triển khai

## 1. Cấu trúc thư mục
```
ar-menu/
├── index.html          ← trang menu + AR viewer (không cần sửa nếu không đổi giao diện)
├── data/cakes.json      ← DANH SÁCH BÁNH — sửa file này khi thêm/bớt/sửa bánh
└── assets/
    ├── models/           ← đặt file .glb và .usdz vào đây
    └── thumbs/           ← ảnh thumbnail (jpg/png) cho từng bánh
```

Hiện `cakes.json` có 6 bánh mẫu, đường dẫn model đang trỏ tới file **chưa tồn tại**
(`assets/models/tiramisu.glb`...). Bạn chỉ cần:
1. Đổi tên file GLB của bạn đúng theo tên trong `cakes.json` (hoặc sửa lại đường dẫn trong json).
2. Bỏ vào `assets/models/`.
3. Thêm ảnh thumbnail vào `assets/thumbs/` (nếu không có, trang tự hiện icon 🍰 thay thế).

Muốn thêm bánh mới: copy 1 khối `{ ... }` trong `cakes.json`, đổi `id`, `name`, `price`,
`ingredients`, `glb`, `usdz`, `poster`. Không cần đụng vào code.

## 2. Tối ưu file GLB (rất quan trọng)
File GLB gốc từ phần mềm 3D thường rất nặng (50-200MB) → sẽ load chậm/treo trên
điện thoại khách. Cần nén xuống dưới ~10-15MB/bánh.

Dùng công cụ miễn phí **gltf-transform** (chạy trên máy tính, cần Node.js):
```bash
npm install -g @gltf-transform/cli

gltf-transform optimize banh-goc.glb banh-toi-uu.glb \
  --texture-compress webp \
  --texture-resize 1024
```
Nếu bánh vẫn nặng, hạ `--texture-resize` xuống 512, hoặc thêm `--simplify` để giảm số
lượng polygon (dùng được cho model không cần chi tiết quá cao).

Kiểm tra nhanh: mở https://gltf.report và kéo thả file vào để xem báo cáo dung lượng,
số triangle, texture — công cụ sẽ gợi ý chỗ cần nén thêm.

## 3. Bắt buộc: tạo file USDZ cho iPhone
`<model-viewer>` dùng GLB cho Android (Scene Viewer), nhưng **iPhone/iPad chỉ đọc
được USDZ** (AR Quick Look của Apple), không đọc GLB. Nếu thiếu USDZ, khách dùng
iPhone sẽ không bấm được nút AR.

Cách tạo USDZ từ GLB (chọn 1):
- **Có máy Mac**: dùng app "Reality Converter" (miễn phí, Apple) — kéo GLB vào, xuất USDZ.
- **Không có Mac**: dùng công cụ online của Apple `usdzconvert`, hoặc trang
  https://products.aspose.app/3d/conversion/glb-to-usdz (kiểm tra kỹ output vì
  chất lượng convert online không phải lúc nào cũng hoàn hảo, nên tự soi lại bằng
  Quick Look trên 1 chiếc iPhone thật trước khi in QR hàng loạt).

## 4. Test thử trên máy trước khi đưa lên GitHub
Không mở trực tiếp `index.html` bằng cách double-click (trình duyệt sẽ chặn do
CORS khi đọc `cakes.json`). Chạy 1 server tĩnh đơn giản trong thư mục `ar-menu/`:
```bash
python3 -m http.server 8080
```
rồi mở `http://localhost:8080` trên máy tính, hoặc `http://<IP-máy-tính>:8080`
trên điện thoại (cùng mạng wifi) để test AR thật.

## 5. Deploy lên GitHub Pages
1. Tạo repo mới trên GitHub (ví dụ `ar-menu`), đẩy toàn bộ thư mục `ar-menu/` lên.
2. Vào **Settings → Pages** của repo → chọn branch `main`, thư mục `/root` → Save.
3. Sau vài phút, trang sẽ có địa chỉ dạng:
   `https://<ten-user>.github.io/ar-menu/`
4. Link riêng cho từng bánh (dùng để tạo QR):
   `https://<ten-user>.github.io/ar-menu/?id=tiramisu`
   `https://<ten-user>.github.io/ar-menu/?id=red-velvet`
   ... (đổi `id` theo từng bánh trong `cakes.json`)

Lưu ý: GitHub Pages tự có HTTPS — **bắt buộc phải có HTTPS** thì trình duyệt mới
cho phép mở camera để chạy AR.

## 6. Tạo mã QR
Vào một trang tạo QR miễn phí bất kỳ (vd. qr-code-generator.com), dán link riêng
của từng bánh ở bước 5 vào, tải ảnh QR về, in lên menu giấy cạnh tên/giá bánh đó.

## 7. Cách hoạt động khi khách quét
1. Khách quét QR cạnh tên bánh → mở thẳng trang AR của bánh đó (không cần chọn lại).
2. Bánh hiện lên xoay tự động, khách kéo để xoay tay, chụm 2 ngón để phóng to/thu nhỏ.
3. Chạm vào bánh (hoặc bấm nút "Xem thành phần") → bảng nguyên liệu trượt lên.
4. Bấm nút vàng "Xem bánh trên bàn (AR)" → điện thoại quét mặt bàn (plane detection)
   và đặt mô hình bánh thật kích thước lên bàn.

## 8. Nếu muốn khách xem cả menu tổng thay vì quét từng bánh
Vẫn dùng đúng file này — chỉ cần tạo thêm 1 mã QR trỏ tới link gốc không có `?id=`
(vd `https://<ten-user>.github.io/ar-menu/`), khách quét sẽ thấy lưới toàn bộ bánh
và tự chọn.
