# Phòng thí nghiệm: FHSS-DWT MITM Key-Recovery AES

Bài thực hành Labtainers – Mô phỏng tấn công MITM key-recovery trên hệ thống giấu tin âm thanh FHSS-DWT-QIM và cải tiến phòng thủ bằng mã hóa AES trước khi giấu tin.

---

## Mô tả

Bài lab mô phỏng hệ thống giấu tin âm thanh sử dụng DWT Haar, FHSS và QIM kết hợp mô hình truyền tin nhiều container trong môi trường Labtainers.

Hệ thống gồm:

- Alice gửi audio steganography
- Bob nhận và giải mã thông điệp
- Attacker đóng vai trò MITM relay
- Attacker bắt file âm thanh trên đường truyền
- Known-plaintext attack với crib `HELLO`
- Brute-force key FHSS 16-bit
- AES-CBC defense trước khi giấu tin

Bài lab minh họa:

- Sự khác nhau giữa steganography và cryptography
- Rủi ro khi dùng key FHSS ngắn
- Tại sao chỉ giấu tin là chưa đủ an toàn
- Cách AES bảo vệ payload ngay cả khi attacker tách được bitstream

---

## Tải về và chạy

```bash
imodule https://github.com/BachTG1505/fhss-dwt-mitm-key-recovery-aes/raw/main/fhss-dwt-key-recovery.tar

cd ~/labtainer/trunk/scripts/labtainer-student

labtainer fhss-dwt-key-recovery
Các container
ContainerVai tròIP
aliceSender192.168.100.10
bobReceiver192.168.100.20
attackerMITM attacker192.168.100.30
Các nhiệm vụ
Nhiệm vụ 1 – Nhúng thông điệp cơ sở
python3 generate_cover.py --out cover.wav

python3 alice_publish.py \
--cover cover.wav \
--message message.txt \
--key 4242 \
--out stego_audio.wav

Kết quả mong đợi:

Alice created stego_audio.wav
PSNR: ~76 dB
Nhiệm vụ 2 – MITM relay bắt file thường

Trong Bob:

python3 bob_receive.py --port 9001

Trong Attacker:

python3 mitm_relay.py \
--listen-port 9000 \
--bob-host 192.168.100.20 \
--bob-port 9001

Trong Alice:

python3 send_to_relay.py \
--file stego_audio.wav \
--relay 192.168.100.30 \
--port 9000
Nhiệm vụ 3 – Bob tách tin hợp lệ
python3 bob_extract.py \
--audio stego_audio.wav \
--key 4242 \
--length 36

Kết quả mong đợi:

Bob extracted: HELLO BOB, SECRET CODE IS B22DCAT024
Nhiệm vụ 4 – Attacker key-recovery
python3 eve_bruteforce.py \
--audio stego_audio.wav \
--crib HELLO \
--length 36 \
--min-key 0 \
--max-key 65535

Kết quả mong đợi:

FOUND KEY: 4242
MESSAGE: HELLO BOB, SECRET CODE IS B22DCAT024
Nhiệm vụ 5 – Nhúng thông điệp có AES
python3 alice_publish_aes.py \
--cover cover.wav \
--message message.txt \
--fhss-key 4242 \
--aes-password ptit-secret \
--out aes_stego_audio.wav
Nhiệm vụ 6 – MITM relay bắt file AES

Trong Bob:

python3 bob_receive.py --port 9001

Trong Attacker:

python3 mitm_relay.py \
--listen-port 9000 \
--bob-host 192.168.100.20 \
--bob-port 9001

Trong Alice:

python3 send_to_relay.py \
--file aes_stego_audio.wav \
--relay 192.168.100.30 \
--port 9000
Nhiệm vụ 7 – Bob tách và giải mã AES
python3 bob_extract_aes.py \
--audio aes_stego_audio.wav \
--fhss-key 4242 \
--aes-password ptit-secret \
--hex-len 96

Kết quả mong đợi:

Bob AES decrypted: HELLO BOB, SECRET CODE IS B22DCAT024
Nhiệm vụ 8 – Attacker thử lại crib attack trên AES
python3 attacker_try_aes.py \
--audio aes_stego_audio.wav

Kết quả mong đợi:

AES DEFENSE SUCCESS: HELLO crib attack failed
Kiểm tra kết quả
stoplab

checkwork fhss-dwt-key-recovery
Kết quả chính
Pha 1 – Không AES

Attacker brute-force thành công:

FOUND KEY: 4242
MESSAGE: HELLO BOB, SECRET CODE IS B22DCAT024
Pha 2 – Có AES

Attacker vẫn bắt được file stego nhưng không thể đọc plaintext:

AES DEFENSE SUCCESS: HELLO crib attack failed

Payload thu được chỉ là ciphertext.

Docker Hub
b22dcat024/fhss-dwt-key-recovery.alice
b22dcat024/fhss-dwt-key-recovery.bob
b22dcat024/fhss-dwt-key-recovery.attacker
Công cụ sử dụng
Python 3
NumPy
PyCryptodome
Docker
Labtainers
DWT Haar
FHSS
QIM
AES-CBC
TCP socket relay
Kiến thức đạt được
Audio steganography
FHSS communication
DWT + QIM embedding
MITM relay simulation
Known-plaintext attack
Crib attack
Brute-force key recovery
AES defense
Steganography vs cryptography
Signal security analysis
Tác giả
Họ và tên: Trương Gia Bách
Mã sinh viên: B22DCAT024
Lớp: D22CQAT04-B
Học phần: INT14102 – Kỹ thuật giấu tin
Giảng viên hướng dẫn: PGS.TS. Đỗ Xuân Chợ
