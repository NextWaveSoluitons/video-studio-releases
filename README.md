# Cài Video Studio trên máy Windows mới

Cài **một lần** (Claude Code tự làm theo file này, khoảng 15–25 phút). Sau đó chỉ cần mở **Video Studio** từ menu
Start / Desktop: app tự kiểm tra và cài bản mới mỗi lần mở, không phải cài gì thêm.

> Không dùng Claude Code? Tải [VideoStudio.exe](https://github.com/NextWaveSoluitons/video-studio-releases/releases/latest/download/VideoStudio.exe)
> rồi bấm đúp: trình cài đặt hiện ra → bấm **Cài đặt** → xong bấm **Mở Video Studio**. App được thêm vào menu Start,
> Desktop và Control Panel → Programs (gỡ cài đặt ở đó).

Yêu cầu: Windows 10/11 có mạng, có `winget` (App Installer — có sẵn trên Windows 11), tài khoản Claude Pro/Max hoặc
API key Anthropic.

---

## Bước 0 — Bạn tự làm (2 phút): cài Claude Code

Mở **PowerShell** (bấm Start, gõ "PowerShell") và chạy:

```powershell
winget install -e --id Git.Git --accept-package-agreements --accept-source-agreements
irm https://claude.ai/install.ps1 | iex
```

Đóng PowerShell, mở lại, rồi chạy `claude` và đăng nhập theo hướng dẫn trên màn hình.

## Bước 1 — Dán câu này cho Claude Code

```text
Đọc https://github.com/NextWaveSoluitons/video-studio-releases/blob/main/README.md (hoặc file README_CAI_DAT.md
trong thư mục video-studio nếu tôi đã chép sẵn) và cài Video Studio cho máy này theo đúng mục
"Lệnh cài cho Claude Code". Chạy lần lượt từng lệnh, đọc kết quả, lỗi thì xem mục "Xử lý lỗi" rồi sửa và chạy lại.
Cuối cùng báo cho tôi kết quả của doctor và test.
```

---

## Lệnh cài cho Claude Code

> Dành cho Claude Code: chạy trong **PowerShell** (`powershell -NoProfile -Command "..."` nếu đang ở Bash).
> Mỗi lệnh chạy lại nhiều lần vẫn an toàn. Sau mỗi bước cài phần mềm, PATH mới đã được nạp trong cửa sổ cài; nếu
> lệnh sau báo "not recognized", mở PowerShell mới rồi chạy lại.

### Cách A — Có file VideoStudio.exe (khuyến nghị)

```powershell
# 1. Lấy exe mới nhất (bỏ qua nếu người dùng đã chép exe vào máy — dùng đường dẫn exe đó ở bước 2)
$exe = "$env:USERPROFILE\Downloads\VideoStudio.exe"
Invoke-WebRequest "https://github.com/NextWaveSoluitons/video-studio-releases/releases/latest/download/VideoStudio.exe" -OutFile $exe

# 2. Cài: giải nén app vào %LOCALAPPDATA%\VideoStudio + cài Git, Python 3.12, ffmpeg, Node.js LTS, MiKTeX,
#    Claude Code, thư viện Python, thư viện 3D (Three.js), dữ liệu ban đầu, lối tắt Start + Desktop, mục gỡ cài đặt trong Control Panel. Không hỏi gì.
& $exe --install --yes | Out-Host      # "| Out-Host" để PowerShell chờ exe (ứng dụng cửa sổ) chạy xong

# 3. Kiểm tra máy + biên dịch thử LaTeX (MiKTeX tải sẵn gói, lần đầu vài phút)
& "$env:LOCALAPPDATA\VideoStudio\VideoStudio.exe" --doctor --fix --warmup | Out-Host

# 4. Chạy bộ test tự động (không tốn usage Claude) — phải ra "0 FAIL"
cd "$env:LOCALAPPDATA\VideoStudio"; python tests\smoke_test.py
```

### Cách B — Có thư mục mã nguồn `video-studio` (chép từ máy khác / git clone)

```powershell
cd D:\video-studio                      # đổi thành nơi đặt thư mục
powershell -NoProfile -ExecutionPolicy Bypass -File .\install.ps1 -SkipLogin
python doctor.py --fix --warmup
python tests\smoke_test.py
```

Cách B tạo lối tắt Desktop trỏ vào `run.bat` (cũng tự cập nhật khi mở: thư mục có `.git` thì `git pull`, không có thì
tải bản phát hành mới nhất).

### Bước cuối — đăng nhập Claude cho Video Studio (người dùng làm, nếu `doctor` báo chưa đăng nhập)

```powershell
claude auth login
```

---

## Dùng hằng ngày

| Việc | Lệnh / thao tác |
|---|---|
| Mở app | **Video Studio** trong menu Start hoặc trên Desktop: cửa sổ app riêng (không có cmd, không cần trình duyệt), tự cập nhật trước khi mở. Đóng cửa sổ là tắt app. Ghim thanh tác vụ: khi app đang mở, chuột phải biểu tượng → *Pin to taskbar* |
| Gỡ cài đặt | **Control Panel → Programs and Features → Video Studio → Uninstall** (hoặc Settings → Apps). Hỏi có giữ video/dữ liệu không; Python, Node.js, MiKTeX… được giữ lại |
| Cập nhật thủ công | Trong app: **Cài đặt → Phiên bản & cập nhật**, hoặc `python updater.py` (cách B: `update.bat`) |
| Kiểm tra máy | `VideoStudio.exe --doctor --fix` hoặc `doctor.bat` |
| Xem phiên bản | `VideoStudio.exe --version` |
| Cài lại video mẫu | `python seed.py --reset-samples` |
| Test tự động | `python tests\smoke_test.py` (`--fast` bỏ phần dựng video) |

Thư mục app (cách A): `%LOCALAPPDATA%\VideoStudio`. Dữ liệu của bạn — `projects\`, `settings.json`, `brand\` — không
bao giờ bị cập nhật ghi đè. Sao lưu: chép 3 mục đó.

## Dành cho người phát triển: đóng gói & phát hành

```powershell
winget install -e --id GitHub.cli; gh auth login          # một lần
python tools\release.py --create-repo                     # một lần: tạo repo công khai video-studio-releases
python tools\package.py                                   # thử đóng gói: dist\video-studio-<ver>.zip + dist\VideoStudio.exe
python tests\smoke_test.py                                # phải 0 FAIL trước khi phát hành
python tools\release.py --bump patch --notes "Nội dung thay đổi"   # tăng VERSION, tag, đóng gói, đăng Release
```

Mọi máy đang dùng sẽ tự nhận bản mới ở lần mở app kế tiếp.

## Xử lý lỗi

| Hiện tượng | Cách xử lý |
|---|---|
| `winget` không có | Mở Microsoft Store → cài **App Installer** → chạy lại |
| `python` / `node` / `ffmpeg` "not recognized" ngay sau khi cài | Mở PowerShell mới (Windows cần nạp PATH mới) rồi chạy lại lệnh |
| Cài MiKTeX xong nhưng `xelatex` không thấy | `$env:Path += ";$env:LOCALAPPDATA\Programs\MiKTeX\miktex\bin\x64"` rồi chạy lại |
| LaTeX lỗi thiếu gói khi dựng video | `initexmf --set-config-value=[MPM]AutoInstall=1` rồi `VideoStudio.exe --doctor --warmup` |
| Cảnh 3D báo không thấy Chrome/Edge | Cài Chrome: `winget install -e --id Google.Chrome`, hoặc đặt biến `CHROME_PATH` |
| Edge TTS lỗi 403 / No audio | `python -m pip install -U edge-tts` |
| App không mở / báo lỗi | Xem nhật ký `%LOCALAPPDATA%\VideoStudio\logs\server.log` |
| Cổng 8765 bị dùng | Đóng cửa sổ Video Studio cũ, hoặc `$env:PORT=9000; python server.py` |
| `doctor` báo chưa đăng nhập Claude | `claude auth login` |

---

**Video Studio** — phát triển bởi **CÔNG TY TNHH NEXTWAVE SOLUTIONS** (NEXTWAVE SOLUTIONS COMPANY LIMITED)
· Mã số thuế 0319297771 · 235/10 Lưu Hữu Phước, Phường Phú Định, Thành phố Hồ Chí Minh, Việt Nam.
