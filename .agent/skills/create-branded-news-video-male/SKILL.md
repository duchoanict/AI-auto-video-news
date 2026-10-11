---
name: create-branded-news-video-male
description: Tạo hoặc dựng lại video tin tức dọc 9:16 trong project này từ URL, văn bản, Markdown hoặc script.json, dùng branding try.ai.lov3, hình minh họa và một profile giọng nam OmniVoice trung, ấm, rõ chữ. Dùng cho pipeline HyperFrames của repository; không dùng cho video ngoài project này.
---

# Create Branded News Video — Male Voice

Tạo video tin tức tiếng Việt bằng pipeline HyperFrames của repository. Kết quả phải mang thương hiệu `try.ai.lov3`, không hiển thị thương hiệu mẫu hoặc tên miền nguồn bài viết, và phải dùng một danh tính giọng nam nhất quán trong toàn bộ video.

## Cấu hình giọng hiện hành — ưu tiên cao nhất

Phần này thay thế mọi chỉ dẫn giọng khác xuất hiện trong tài liệu, script hoặc cấu hình cũ.

- Chỉ dùng profile OmniVoice `clone:try_ai_lov3_male_warm_news`.
- Giọng mục tiêu: nam trưởng thành, âm vực trung, ấm, rõ chữ; phong thái điềm tĩnh, đáng tin cậy, phù hợp bản tin tiếng Việt.
- Không dùng `voice: "auto"` và không chỉ dựa vào `instructions` cho video nhiều cảnh, vì mỗi request thiết kế giọng có thể tạo ra một danh tính người nói khác.
- Trong `src/tts/omnivoice-client.ts`, giữ `voice: "clone:try_ai_lov3_male_warm_news"`, `language: "vi"`, `response_format: "mp3"`, `num_step: 16`, `seed: 20261010` và timeout ít nhất `300000` ms.
- Khởi động server bằng UTF-8 và một luồng xử lý: `$env:PYTHONUTF8='1'; py -m omnivoice_server --host 127.0.0.1 --port 8880 --max-concurrent 1`.
- Nếu profile chưa có, tạo mẫu bằng voice design `design:male,middle-aged,moderate pitch`. OmniVoice chỉ chấp nhận các thuộc tính có trong `GET /v1/voices`; không gửi `warm`, `clear articulation` hoặc chỉ dẫn tự do nếu server báo `Unsupported attributes`. Thể hiện chất giọng ấm, điềm tĩnh và rõ chữ bằng nội dung mẫu tiếng Việt, nhịp câu và cách đọc của audio tham chiếu.
- Audio tham chiếu phải sạch, dài 6–12 giây và có `ref_text` khớp nguyên văn. Dùng `ffprobe` kiểm tra thời lượng trước khi đăng ký.
- Kiểm tra profile bằng `GET /v1/voices/profiles/try_ai_lov3_male_warm_news`. Endpoint tập hợp `GET /v1/voices/profiles` có thể không được hỗ trợ.
- Tạo profile bằng multipart `POST /v1/voices/profiles` với `profile_id=try_ai_lov3_male_warm_news`, file `ref_audio` và `ref_text`. Sau đó gọi lại endpoint profile cụ thể để xác nhận.
- Không dùng `ref_text` bỏ dấu. Nếu Windows báo lỗi `cp1252`, khởi động lại server với `PYTHONUTF8=1`.
- Khi đổi profile hoặc cấu hình giọng, xóa toàn bộ thư mục `voice`, `voice-raw.mp3` và `voice.mp3` trong đúng output directory rồi sinh lại cả tám cảnh. Không giữ lại bất kỳ scene audio cũ nào.
- Clone trên CPU có thể lâu hơn một phút. Không gửi lặp request khi server vẫn đang xử lý; tăng timeout client trước.
- Trước khi bàn giao, xác nhận audio của mọi scene và log toàn video đều resolve thành `CLONE mode` với cùng profile `clone:try_ai_lov3_male_warm_news`.
- Nghe kiểm tra đầu, giữa và cuối video để xác nhận giọng nam trung, ấm, rõ chữ và nhất quán. Nếu môi trường không có khả năng nghe, nêu rõ giới hạn thay vì khẳng định cảm nhận âm thanh.

## Đầu vào

Nhận URL bài viết, file `.txt`, Markdown hoặc `script.json` có sẵn. Với URL, trích xuất tiêu đề, nội dung chính, ảnh đại diện và domain. Nếu trang không đọc được, thử phương thức HTTP hoặc browser phù hợp trước khi yêu cầu người dùng cung cấp văn bản.

## Branding bắt buộc

- Thương hiệu/tài khoản: `try.ai.lov3`.
- Slogan, tagline và CTA: `Đam mê AI`.
- Logo nguồn hiện tại: `E:\logo2.png`.
- Logo runtime:
  - `templates/frame-liquid-bg-hero/assets/logo.png`
  - `templates/frame-logo-outro/assets/logo.png`
- Nếu người dùng cung cấp logo mới, sao chép vào cả hai đường dẫn runtime trước khi render.
- Màu chính: navy `#071A33`; màu phụ: trắng `#FFFFFF`; có thể dùng cyan hoặc tím của logo làm màu nhấn.
- Không để `AI Coding`, `aicodingvn`, `Senior AI Engineer`, biểu tượng `</>` mẫu hoặc branding cũ xuất hiện trong chữ, giọng đọc hay hình ảnh.

Tên `frame-aicoding-list` và `frame-aicoding-comparison` là ID kỹ thuật. Không đổi tên nếu chúng không được hiển thị trong video.

## Nguồn bài viết

Giữ URL, domain và ảnh trong `metadata.source` để truy vết nội bộ, nhưng không đưa chúng lên hình hoặc vào giọng đọc:

- Không hiển thị tên miền nguồn.
- Không hiển thị `Nguồn: ...` hoặc URL.
- Đặt `footer_right` và `source` thành chuỗi rỗng hoặc branding `try.ai.lov3`.
- `voiceText` không được đọc tên miền hay URL nguồn.

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
3. Một outro bằng `frame-logo-outro` với `brand_name: "try.ai.lov3"`, `tagline: "Đam mê AI"` và `primary_url: "try.ai.lov3"`.

Quy ước input:

- `brand`, `footer_left`, `channel`, `side_left`: dùng `try.ai.lov3` khi phù hợp.
- `footer_right`, `source`: dùng `""` nếu vốn dành cho nguồn bài viết.
- CTA cuối trong `voiceText`: câu tự nhiên chứa `Theo dõi try.ai.lov3, đam mê AI.`

Mỗi body scene nên khoảng 25–40 từ và 6–10 giây. Tổng lời đọc thường 270–360 từ, nhưng ưu tiên nhịp tự nhiên và đủ nội dung.

## TTS tiếng Việt

- Viết số ra chữ nếu TTS dễ đọc sai: `5.5` thành `năm chấm năm`, `30%` thành `ba mươi phần trăm`, `2x` thành `gấp đôi`.
- Không dùng emoji, URL hoặc `%`, `$`, `#`, `+`, `=` trong `voiceText`.
- Kết câu bằng dấu chấm hoặc dấu hỏi để tạo nhịp nghỉ.
- `inputs` hiển thị trên màn hình có thể giữ số và emoji vừa phải.

## Hình ảnh trực quan

Video có thể dùng typography thuần hoặc ảnh minh họa. Khi người dùng yêu cầu thêm hình ảnh trực quan hoặc làm clip sinh động hơn:

1. Đọc toàn bộ kịch bản và chọn 3–5 cảnh quan trọng nhất để minh họa. Ưu tiên hook, tình huống con người, nguyên nhân, giải pháp và chuyển biến cảm xúc; giữ cảnh số liệu, comparison và list ở dạng đồ họa khi rõ hơn ảnh.
2. Dùng skill `imagegen` tạo từng ảnh riêng. Với nhiều nội dung khác nhau, gọi một lần cho mỗi ảnh thay vì tạo ảnh ghép.
3. Ảnh phải dọc 9:16, không chữ, logo, watermark hoặc giao diện ứng dụng có thể đọc được. Với tin Việt Nam, ưu tiên nhân vật và bối cảnh Việt Nam khi phù hợp.
4. Giữ phong cách nhất quán: ảnh biên tập chân thực, ánh sáng điện ảnh nhẹ, bảng màu navy–trắng–cyan, chi tiết đời thường tự nhiên.
5. Chừa khoảng trống tối hoặc ít chi tiết ở vùng đặt tiêu đề; không để chữ phủ lên mặt, mắt hoặc hành động chính.
6. Lưu mọi ảnh được dùng vào workspace; không để asset mà project tham chiếu chỉ nằm trong thư mục ảnh sinh mặc định.

Prompt ảnh phải nêu rõ chủ thể/cảnh, cảm xúc, phong cách ảnh biên tập chân thực, bố cục dọc 9:16, vùng trống cho chữ, màu navy–trắng–cyan và `no text, no logo, no watermark`.

Với tin thể thao, dùng hình biên tập mô tả đúng hành động nhưng tránh tạo logo, huy hiệu, nhà tài trợ, bảng quảng cáo có thể đọc được hoặc khuôn mặt cố sao chép chính xác người thật. Có thể dùng màu áo và bối cảnh để truyền đạt hai phía; giữ hình ảnh chân thực, không ghép nhiều diễn biến vào một ảnh.

### Tích hợp và kiểm tra ảnh

- Dùng `frame-photo-editorial` cho cảnh ảnh toàn màn hình.
- Chép ảnh vào `templates/frame-photo-editorial/assets/` bằng tên file mô tả, ổn định và không có khoảng trắng.
- Input tối thiểu theo catalog hiện hành gồm `image`, `kicker`, `headline`, `caption` và `brand: "try.ai.lov3"`. Không dùng tên slot `kicker_photo` hoặc `headline_photo` nếu template không khai báo chúng.
- Dùng lớp phủ navy để chữ dễ đọc và chuyển động push-in/pan nhẹ; không kéo giãn ảnh.
- Phối hợp cảnh ảnh với template số liệu, comparison và list; không biến toàn bộ video thành slideshow đồng dạng.
- Khi thêm hoặc đổi ảnh, xóa cache `clips/scene-XX.mp4` và `clips/scene-XX-fit.mp4` của đúng scene bị ảnh hưởng trước khi render lại.
- Xuất sang output directory phiên bản mới, ví dụ hậu tố `-visual-v2`; giữ video cũ đến khi bản mới dựng và kiểm tra thành công.
- Trích preview từng cảnh có ảnh. Kiểm tra đúng ảnh, đúng cảnh, chữ không bị cắt, chủ thể không bị che, ảnh không méo, lớp phủ không quá tối và không có chữ giả, logo, watermark hay branding nguồn.
- Nếu một ảnh không đạt, chỉ tạo lại hoặc chỉnh cảnh đó rồi xóa cache tương ứng.

### Nâng cấp một video đã render sang bản có ảnh

Khi người dùng yêu cầu thêm hình ảnh vào video đã hoàn tất:

1. Tạo output directory mới với hậu tố `-visual-v2`; không sửa hoặc ghi đè output cũ.
2. Chọn 3–5 scene cần ảnh. Tạo từng ảnh bằng một lần gọi `imagegen`, kiểm tra ảnh rồi chép vào `templates/frame-photo-editorial/assets/` bằng filename ổn định, mô tả nội dung và không có khoảng trắng.
3. Trong `script.json` của bản mới, chỉ đổi `templateId` và `inputs` của các scene được chọn sang `frame-photo-editorial`; giữ nguyên `voiceText` nếu nội dung lời đọc không đổi.
4. Nếu `voiceText`, profile và cấu hình TTS không đổi, được phép giữ và tái sử dụng `voice/`, `voice-raw.mp3` và `voice.mp3`. Không tái tạo audio chỉ vì đổi hình.
5. Xóa đúng `clips/scene-XX.mp4` và `clips/scene-XX-fit.mp4` của các scene đổi ảnh. Trước mọi thao tác xóa, resolve đường dẫn và xác minh chúng nằm trong output directory mới.
6. Chạy pipeline foreground đến bước 8/8. Xác nhận log cho thấy audio được `REUSE` và chỉ các clip bị thay đổi được render lại.
7. Trích preview từ từng `clips/scene-XX-fit.mp4` ở một frame giữa cảnh, thường khoảng giây thứ ba. Kiểm tra trực quan từng preview; không chỉ kiểm tra hook và outro.
8. Chỉ bàn giao bản `visual-v2` sau khi `ffprobe`, runtime branding scan và mọi preview đều đạt. Giữ nguyên bản video trước đó làm phương án quay lại.

## Kiểm tra môi trường và render

Trước khi chạy, kiểm tra Node/npm, FFmpeg/ffprobe, Chrome, OmniVoice và `.env.local`. Xác nhận JSON parse được rồi chạy:

```powershell
npm.cmd run typecheck
npm.cmd run pipeline -- <outputDir>\script.json
```

Chạy pipeline foreground và theo dõi đến bước 8/8.

Pipeline tái sử dụng cache:

- Sửa `voiceText`: xóa `voice/scene-XX.mp3` tương ứng.
- Đổi cấu hình giọng toàn cục: xóa thư mục `voice`, `voice-raw.mp3` và `voice.mp3`.
- Sửa input, template hoặc logo: xóa `clips/scene-XX.mp4` và clip fit tương ứng của scene bị ảnh hưởng.
- Không xóa `script.json`. Giữ `video.mp4` cũ đến khi bản mới dựng thành công.
- Chỉ xóa đường dẫn đã resolve và xác minh nằm trong đúng `outputDir`.

## Kiểm tra đầu ra

Sau khi render:

1. Dùng `ffprobe` xác nhận H.264, AAC, 1080x1920, 30 FPS và thời lượng hợp lý.
2. Trích preview hook, outro và các scene có ảnh hoặc từng chứa nguồn.
3. Kiểm tra logo không méo/cắt, chữ Việt đúng và footer không còn tên miền nguồn.
4. Tìm trong dữ liệu runtime để chắc chắn không còn branding cũ. Không coi ID kỹ thuật hoặc attribution trong NOTICE/README là lỗi runtime.
5. Xác nhận mọi cảnh dùng đúng profile nam cố định và nghe kiểm tra đầu, giữa, cuối video.

## Bàn giao

Báo ngắn gọn thay đổi, thông số video và cung cấp liên kết tới `video.mp4` cùng các preview. Kèm `voice.mp3` và `script.txt` nếu người dùng cần CapCut. Với video có ảnh sinh, nêu công cụ đã tạo ảnh và liên kết preview các cảnh ảnh chính.

Không tuyên bố hoàn tất nếu pipeline chưa chạy hết, chưa kiểm tra file đầu ra hoặc chưa xác nhận tính nhất quán của giọng nam.
