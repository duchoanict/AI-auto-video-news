---
name: create-branded-news-video
description: Tạo hoặc dựng lại video tin tức dọc 9:16 trong project này từ URL, file văn bản, Markdown hoặc script.json, dùng branding try.ai.lov3, hình minh họa và một profile giọng nữ OmniVoice nhất quán. Dùng cho pipeline HyperFrames của repository; không dùng cho video ngoài project này.
---

# Create Branded News Video

## Cấu hình giọng hiện hành (ưu tiên cao nhất)

Phần này thay thế mọi chỉ dẫn giọng nam hoặc voice design cũ còn xuất hiện bên dưới.

- Mặc định dùng duy nhất profile OmniVoice `clone:try_ai_lov3_female_news`.
- Đây là profile giọng nữ trưởng thành, âm vực trung, sáng và rõ.
- Không dùng `voice: "auto"` hoặc chỉ dùng `instructions` cho video nhiều cảnh: mỗi request thiết kế giọng có thể tạo ra một danh tính người nói khác.
- Trong `src/tts/omnivoice-client.ts`, giữ `voice: "clone:try_ai_lov3_female_news"`, `language: "vi"`, `response_format: "mp3"`, `num_step: 16`, `seed: 20261010` và timeout ít nhất `300000` ms.
- Khởi động server bằng UTF-8 và một luồng xử lý: `$env:PYTHONUTF8='1'; py -m omnivoice_server --host 127.0.0.1 --port 8880 --max-concurrent 1`.
- Nếu profile chưa có, tạo một mẫu bằng `female, middle-aged, moderate pitch`, dùng audio tiếng Việt sạch dài khoảng 6–12 giây và `ref_text` khớp nguyên văn, rồi lưu qua `POST /v1/voices/profiles` với ID trên.
- Không dùng `ref_text` bỏ dấu. Nếu Windows báo lỗi `cp1252`, khởi động lại server với `PYTHONUTF8=1`.
- Khi đổi profile hoặc cấu hình giọng, xóa toàn bộ thư mục `voice`, `voice-raw.mp3` và `voice.mp3` trong đúng output directory rồi sinh lại cả tám cảnh. Không giữ lại bất kỳ scene audio cũ nào.
- Clone trên CPU có thể lâu hơn một phút. Không gửi lặp request khi server vẫn đang xử lý; sửa timeout client trước.
- Trước khi bàn giao, xác nhận đủ audio scene và log của cả video đều resolve thành `CLONE mode` với cùng profile.
- Nghe kiểm tra đầu, giữa và cuối video. Nếu môi trường không có khả năng nghe, nói rõ giới hạn thay vì khẳng định cảm nhận âm thanh.

## Hình ảnh trực quan phù hợp

Skill có thể tạo video dạng typography thuần hoặc bản có ảnh minh họa. Khi người dùng yêu cầu “thêm hình ảnh trực quan”, “thêm ảnh phù hợp”, “làm clip sinh động hơn” hoặc ý tương đương, thực hiện đầy đủ quy trình sau:

1. Đọc toàn bộ kịch bản và chọn 3–5 cảnh quan trọng nhất để minh họa. Ưu tiên hook, tình huống con người, nguyên nhân, giải pháp và chuyển biến cảm xúc. Giữ các cảnh số liệu/list dạng đồ họa khi chúng truyền đạt thông tin rõ hơn ảnh.
2. Dùng skill `imagegen` tạo từng ảnh riêng. Với nhiều nội dung khác nhau, gọi một lần cho mỗi ảnh thay vì yêu cầu một ảnh ghép.
3. Ảnh phải là khung dọc 9:16, không chữ, không logo, không watermark và không giao diện ứng dụng có thể đọc được. Với tin Việt Nam, ưu tiên nhân vật/bối cảnh Việt Nam khi phù hợp nội dung.
4. Giữ phong cách nhất quán trong cả bộ ảnh: ảnh biên tập chân thực, ánh sáng điện ảnh nhẹ, bảng màu navy–trắng–cyan của `try.ai.lov3`, chi tiết đời thường tự nhiên. Không làm quá kịch tính nếu nội dung không yêu cầu.
5. Chừa khoảng trống tối hoặc ít chi tiết ở vùng đặt tiêu đề. Không để chữ phủ lên mắt, mặt hoặc hành động chính của nhân vật.
6. Lưu mọi ảnh được dùng vào workspace; không để asset mà project tham chiếu chỉ nằm trong thư mục generated-images mặc định.

Prompt ảnh nên nêu rõ: chủ thể/cảnh, cảm xúc, phong cách ảnh biên tập chân thực, bố cục dọc 9:16, vùng trống cho chữ, màu navy–trắng–cyan và các ràng buộc `no text, no logo, no watermark`.

### Tích hợp ảnh vào video

- Dùng template `frame-photo-editorial` cho cảnh ảnh toàn màn hình.
- Chép ảnh vào `templates/frame-photo-editorial/assets/` bằng filename mô tả, ổn định và không có khoảng trắng.
- Input tối thiểu gồm `image`, `kicker_photo`, `headline_photo`, `caption` và `brand: "try.ai.lov3"`.
- Template phải dùng lớp phủ navy để chữ dễ đọc và chuyển động push-in/pan nhẹ; không kéo giãn ảnh.
- Phối hợp cảnh ảnh với template số liệu, comparison và list; không biến toàn bộ video thành slideshow ảnh giống nhau.
- Khi thêm hoặc đổi ảnh, xóa cache `clips/scene-XX.mp4` và `clips/scene-XX-fit.mp4` của đúng scene bị ảnh hưởng trước khi render lại.
- Xuất sang output directory phiên bản mới, ví dụ hậu tố `-visual-v2`; không ghi đè video cũ cho đến khi bản mới dựng và kiểm tra thành công.

### Kiểm tra ảnh sau render

- Trích preview từ từng cảnh có ảnh, không chỉ hook và outro.
- Xác nhận đúng ảnh/đúng cảnh, chữ Việt rõ và không bị cắt, chủ thể không bị che, ảnh không méo, màu phủ không quá tối.
- Kiểm tra ảnh không chứa chữ giả, logo, watermark, thương hiệu nguồn hoặc branding cũ.
- Nếu một ảnh không đạt, chỉ tạo lại hoặc chỉnh cảnh đó rồi xóa cache clip tương ứng; không cần dựng lại các ảnh đạt yêu cầu.
- Khi bàn giao, kèm liên kết preview của các cảnh ảnh chính và nói rõ ảnh được tạo bằng công cụ nào.

Tạo video tin tức tiếng Việt bằng pipeline HyperFrames của repository. Kết quả phải mang thương hiệu `try.ai.lov3` và không hiển thị thương hiệu mẫu hoặc tên miền nguồn bài viết.

## Đầu vào

Nhận URL bài viết, file `.txt`, hoặc `script.json` có sẵn nếu người dùng chỉ muốn dựng lại. Với URL, trích xuất tiêu đề, nội dung chính, ảnh đại diện và domain. Nếu trang không đọc được, thử phương thức HTTP/browser phù hợp trước khi yêu cầu người dùng cung cấp văn bản.

## Branding bắt buộc

- Thương hiệu/tài khoản: `try.ai.lov3`.
- Slogan, tagline và CTA: `Đam mê AI`.
- Logo nguồn hiện tại: `E:\logo2.png`.
- Logo runtime:
  - `templates/frame-liquid-bg-hero/assets/logo.png`
  - `templates/frame-logo-outro/assets/logo.png`
- Nếu người dùng cung cấp logo mới, sao chép vào cả hai đường dẫn runtime trước khi render.
- Màu chính: navy `#071A33`; màu phụ: trắng `#FFFFFF`; có thể dùng cyan/tím của logo làm màu nhấn.
- Giọng đọc OmniVoice: nữ trưởng thành, âm vực trung, dùng profile clone cố định theo phần cấu hình giọng hiện hành.
- Không để `AI Coding`, `aicodingvn`, `Senior AI Engineer`, biểu tượng `</>` mẫu hoặc branding cũ xuất hiện trong chữ, giọng đọc hay hình ảnh.

Tên `frame-aicoding-list` và `frame-aicoding-comparison` là ID kỹ thuật. Không đổi tên nếu chúng không được hiển thị trong video.

## Nguồn bài viết

Giữ URL, domain và ảnh trong `metadata.source` để truy vết nội bộ, nhưng không đưa chúng lên hình hoặc vào giọng đọc:

- Không hiển thị tên miền như `24h.com.vn`.
- Không hiển thị `Nguồn: ...` hoặc URL.
- Đặt slot `footer_right`/`source` thành chuỗi rỗng hoặc branding `try.ai.lov3`.
- `voiceText` không đọc tên miền hay URL nguồn.

## Sinh script.json

Đọc `templates/CATALOG.md` trước khi chọn template. Chỉ dùng `templateId` có trong catalog.

Khung bắt buộc:

```json
{
  "version": "1.0",
  "renderer": "hyperframes",
  "aspect": "9:16",
  "metadata": {
    "title": "...",
    "source": { "url": "...", "domain": "...", "image": null },
    "channel": "try.ai.lov3"
  },
  "voice": { "provider": "omnivoice", "speed": 1.0 },
  "scenes": []
}
```

Tạo 8–12 scene:

1. Một hook bằng `frame-liquid-bg-hero`.
2. Sáu đến mười body scene; mỗi scene chỉ truyền tải một ý và dùng template phù hợp, đa dạng.
3. Một outro bằng `frame-logo-outro` với:

```json
{
  "brand_name": "try.ai.lov3",
  "tagline": "Đam mê AI",
  "primary_url": "try.ai.lov3"
}
```

Quy ước input:

- `brand`, `footer_left`, `channel`, `side_left`: dùng `try.ai.lov3` khi phù hợp.
- `footer_right`, `source`: dùng `""` nếu vốn dành cho nguồn bài viết.
- CTA cuối trong `voiceText`: câu tự nhiên chứa `Theo dõi try.ai.lov3, đam mê AI.`

Mỗi body scene nên khoảng 25–40 từ và 6–10 giây. Tổng lời đọc thường 270–360 từ, nhưng ưu tiên nhịp tự nhiên và đủ nội dung.

## TTS tiếng Việt

- Viết số ra chữ nếu TTS dễ đọc sai: `5.5` → `năm chấm năm`, `30%` → `ba mươi phần trăm`, `2x` → `gấp đôi`.
- Không dùng emoji, URL hoặc `%`, `$`, `#`, `+`, `=` trong `voiceText`.
- Kết câu bằng dấu chấm hoặc dấu hỏi để tạo nhịp nghỉ.
- `inputs` hiển thị trên màn hình có thể giữ số và emoji vừa phải.

## Kiểm tra môi trường và render

Trước khi chạy, kiểm tra Node/npm, FFmpeg/ffprobe, Chrome, OmniVoice và `.env.local`. Xác nhận JSON parse được và chạy:

```powershell
npm.cmd run typecheck
npm.cmd run pipeline -- <outputDir>\script.json
```

Chạy pipeline foreground và theo dõi đến bước 8/8.

Pipeline tái sử dụng cache. Khi thay đổi:

- Sửa `voiceText`: xóa `voice/scene-XX.mp3` tương ứng.
- Đổi cấu hình giọng toàn cục: xóa thư mục `voice`, `voice-raw.mp3` và `voice.mp3`.
- Sửa input/template/logo: xóa `clips/scene-XX.mp4` của scene bị ảnh hưởng.
- Đổi logo thường ảnh hưởng hook và outro; bỏ nguồn/footer chỉ ảnh hưởng scene chứa các slot đó.
- Không xóa `script.json`. Giữ `video.mp4` cũ đến khi bản mới dựng thành công.

Chỉ xóa đường dẫn đã resolve và xác minh nằm trong đúng `outputDir`.

## Kiểm tra đầu ra

Sau khi render:

1. Dùng `ffprobe` xác nhận H.264, AAC, 1080×1920, 30 FPS và thời lượng hợp lệ.
2. Trích preview hook và outro; nếu vừa bỏ nguồn, trích thêm các scene từng chứa nguồn.
3. Kiểm tra logo không méo/cắt, chữ Việt đúng và footer không còn tên miền nguồn.
4. Tìm trong dữ liệu runtime để chắc chắn không còn branding cũ. Không coi ID kỹ thuật hoặc attribution trong NOTICE/README là lỗi runtime.

## Bàn giao

Báo ngắn gọn thay đổi, thông số video và cung cấp liên kết tới `video.mp4` cùng các preview. Kèm `voice.mp3` và `script.txt` nếu người dùng cần CapCut.

Không tuyên bố hoàn tất nếu pipeline chưa chạy hết hoặc chưa kiểm tra file đầu ra.
