# Phòng thí nghiệm: FHSS-DWT MITM Key-Recovery AES

Bài thực hành Labtainers – Mô phỏng tấn công MITM key-recovery trên hệ thống giấu tin âm thanh FHSS-DWT-QIM và cải tiến phòng thủ bằng mã hóa AES trước khi giấu tin.

---

## Mô tả

Bài lab mô phỏng hệ thống âm thanh ẩn tin sử dụng DWT Haar, FHSS và QIM kết hợp nhiều container trong môi trường Labtainers.

### Hệ thống gồm:

- Alice gửi bản ghi âm thanh
- Bob nhận và giải mã thông điệp
- Attacker đóng vai trò MITM relay
- Attacker ghi lại âm thanh trên đường truyền
- Tấn công known-plaintext với crib `HELLO`
- Brute-force khóa FHSS 16-bit
- Phòng thủ AES-CBC trước khi giấu tin

### Bài lab minh họa:

- Sự khác nhau giữa steganography và cryptography
- Rủi ro khi dùng khóa FHSS ngắn
- Tại sao chỉ giấu tin là chưa đủ an toàn
- Cách AES bảo vệ payload ngay cả khi attacker tách được bitstream

---

## Tải về và chạy

```bash
imodule https://github.com/BachTG1505/fhss-dwt-mitm-key-recovery-aes/raw/main/fhss-dwt-key-recovery.tar

cd ~/labtainer/trunk/scripts/labtainer-student

labtainer fhss-dwt-key-recovery
```

---

## Các container

| Container | Vai trò | IP |
|---|---|---|
| alice | Sender | 192.168.100.10 |
| bob | Receiver | 192.168.100.20 |
| attacker | MITM attacker | 192.168.100.30 |

---

## Nhiệm vụ 1 – Nhúng thông điệp cơ sở

```bash
python3 generate_cover.py --out cover.wav

python3 alice_publish.py \
--cover cover.wav \
--message message.txt \
--key 4242 \
--out stego_audio.wav
```

### Kết quả mong đợi

```text
Alice created stego_audio.wav
PSNR: ~76 dB
```

---

## Nhiệm vụ 2 – MITM relay bắt file thường

### Trong Bob

```bash
python3 bob_receive.py --port 9001
```

### Trong Attacker

```bash
python3 mitm_relay.py \
--listen-port 9000 \
--bob-host 192.168.100.20 \
--bob-port 9001
```

### Trong Alice

```bash
python3 send_to_relay.py \
--file stego_audio.wav \
--relay 192.168.100.30 \
--port 9000
```

---

## Nhiệm vụ 3 – Bob tách tin hợp lệ

```bash
python3 bob_extract.py \
--audio stego_audio.wav \
--key 4242 \
--length 36
```

### Kết quả mong đợi

```text
Bob extracted: HELLO BOB, SECRET CODE IS B22DCAT024
```

---

## Nhiệm vụ 4 – Attacker key-recovery

```bash
python3 eve_bruteforce.py \
--audio stego_audio.wav \
--crib HELLO \
--length 36 \
--min-key 0 \
--max-key 65535
```

### Kết quả mong đợi

```text
FOUND KEY: 4242
MESSAGE: HELLO BOB, SECRET CODE IS B22DCAT024
```

---

## Nhiệm vụ 5 – Nhúng thông điệp có AES

```bash
python3 alice_publish_aes.py \
--cover cover.wav \
--message message.txt \
--fhss-key 4242 \
--aes-password ptit-secret \
--out aes_stego_audio.wav
```

---

## Nhiệm vụ 6 – MITM relay bắt file AES

### Trong Bob

```bash
python3 bob_receive.py --port 9001
```

### Trong Attacker

```bash
python3 mitm_relay.py \
--listen-port 9000 \
--bob-host 192.168.100.20 \
--bob-port 9001
```

### Trong Alice

```bash
python3 send_to_relay.py \
--file aes_stego_audio.wav \
--relay 192.168.100.30 \
--port 9000
```

---

## Nhiệm vụ 7 – Bob tách và giải mã AES

```bash
python3 bob_extract_aes.py \
--audio aes_stego_audio.wav \
--fhss-key 4242 \
--aes-password ptit-secret \
--hex-len 96
```

### Kết quả mong đợi

```text
Bob AES decrypted: HELLO BOB, SECRET CODE IS B22DCAT024
```

---

## Nhiệm vụ 8 – Attacker thử lại crib attack trên AES

```bash
python3 attacker_try_aes.py \
--audio aes_stego_audio.wav
```

### Kết quả mong đợi

```text
AES DEFENSE SUCCESS: HELLO crib attack failed
```

---

## Kiểm tra kết quả

```bash
stoplab

checkwork fhss-dwt-key-recovery
```

---

## Công cụ sử dụng

- Python 3
- NumPy
- PyCryptodome
- Docker
- Labtainers
- DWT Haar
- FHSS
- QIM
- AES-CBC
- TCP socket relay

---

## Kiến thức đạt được

- Audio steganography
- FHSS communication
- DWT + QIM embedding
- MITM relay simulation
- Known-plaintext attack
- Crib attack
- Brute-force key recovery
- AES defense
- Steganography vs cryptography
- Signal security analysis

---

## Tác giả

- Họ và tên: Trương Gia Bách
- Mã sinh viên: B22DCAT024
- Lớp: D22CQAT04-B
- Học phần: INT14102 – Kỹ thuật giấu tin
- Giảng viên hướng dẫn: PGS.TS. Đỗ Xuân Chợ
