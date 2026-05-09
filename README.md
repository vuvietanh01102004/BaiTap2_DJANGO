# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421
## Họ tên: Vũ Việt Anh
## Lớp: K58KTP.K01
## MSSV: K225480106082
# SỬ DỤNG DJANGO ĐỂ TẠO WEB QUẢN LÝ TIỆM CẦM ĐỒ

# BÀI LÀM
## 1. TỔ CHỨC CSDL CHO HỆ THỐNG QUẢN LÝ TIỆM CẦM ĐỒ
```
KHACH_HANG (khách hàng / con nợ)
├── id (PK)
├── ho_ten
├── so_dien_thoai
├── dia_chi
└── so_cmnd

TAI_SAN (tài sản cầm cố)
├── id (PK)
├── ten_tai_san
├── mo_ta
└── tinh_trang

HOP_DONG_CAM (hợp đồng cầm đồ)
├── id (PK)
├── khach_hang_id (FK → KHACH_HANG)
├── tai_san_id (FK → TAI_SAN)
├── ngay_cam
├── ngay_den_han
├── so_tien_cam
├── lai_suat (% / tháng)
└── trang_thai (đang cầm / đã chuộc / quá hạn)

THANH_TOAN (lịch sử thanh toán lãi)
├── id (PK)
├── hop_dong_id (FK → HOP_DONG_CAM)
├── ngay_thanh_toan
└── so_tien_thanh_toan
```

