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
## 2. SỬ DỤNG DOCKER TRÊN UBUNTU
<img width="700" height="484" alt="image" src="https://github.com/user-attachments/assets/a2f3f166-9580-4d7d-8b13-ced4bbe47d70" />

### - Tạo file `nano django_app/requirements.txt`
<img width="974" height="509" alt="image" src="https://github.com/user-attachments/assets/528c586d-6897-4525-98de-5d4e91bf469c" />

```
# Django: framework web chinh, tao models, views, admin
Django==4.2

# mysqlclient: driver de Django ket noi voi MariaDB
mysqlclient==2.2.4

# Pillow: xu ly anh upload
Pillow==10.3.0
```

### - Tạo file `nano django_app/Dockerfile` 
<img width="977" height="504" alt="image" src="https://github.com/user-attachments/assets/c3378787-52cf-4612-8545-7b10df64ee35" />

```
# Dung image Python 3.11 phien ban slim (nhe, du dung)
FROM python:3.11-slim

# Cai cac thu vien he thong can thiet de build mysqlclient
RUN apt-get update && apt-get install -y \
    pkg-config \
    default-libmysqlclient-dev \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Dat thu muc lam viec ben trong container la /app
WORKDIR /app

# Copy file requirements.txt vao container truoc
COPY requirements.txt .

# Cai tat ca thu vien Python tu requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Mo cong 8000 de ben ngoai truy cap Django
EXPOSE 8000

# Lenh mac dinh khi container khoi dong
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### - Tạo file `nano docker-compose.yml`
```
version: '3.8'

services:

  # SERVICE 1: MariaDB - Co so du lieu
  db:
    image: mariadb:10.11
    container_name: camdo_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: camdo_db
      MYSQL_USER: camdo_user
      MYSQL_PASSWORD: camdo_pass
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - camdo_net

  # SERVICE 2: PhpMyAdmin - Giao dien xem CSDL
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: camdo_pma
    restart: always
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      PMA_USER: camdo_user
      PMA_PASSWORD: camdo_pass
    depends_on:
      - db
    networks:
      - camdo_net

  # SERVICE 3: Django - Ung dung web
  django:
    build: ./django_app
    container_name: camdo_django
    restart: always
    ports:
      - "8000:8000"
    volumes:
      - ./django_app:/app
    depends_on:
      - db
    networks:
      - camdo_net

networks:
  camdo_net:
    driver: bridge

volumes:
  db_data:
```

<img width="1048" height="103" alt="image" src="https://github.com/user-attachments/assets/ba4b56e3-28b4-48a7-98ef-41a801eb8551" />

### - Tạo file `models.py` bằng `nano django_app/quanlycamdo/models.py`
```
from django.db import models

class KhachHang(models.Model):
    ho_ten = models.CharField(max_length=100, verbose_name="Ho ten")
    so_dien_thoai = models.CharField(max_length=15, verbose_name="So dien thoai")
    dia_chi = models.TextField(verbose_name="Dia chi", blank=True)
    so_cmnd = models.CharField(max_length=20, verbose_name="So CMND/CCCD")

    def __str__(self):
        return self.ho_ten

    class Meta:
        verbose_name = "Khach hang"
        verbose_name_plural = "Khach hang"


class TaiSan(models.Model):
    ten_tai_san = models.CharField(max_length=200, verbose_name="Ten tai san")
    mo_ta = models.TextField(verbose_name="Mo ta", blank=True)
    tinh_trang = models.CharField(max_length=100, verbose_name="Tinh trang", blank=True)

    def __str__(self):
        return self.ten_tai_san

    class Meta:
        verbose_name = "Tai san"
        verbose_name_plural = "Tai san"


class HopDongCam(models.Model):
    TRANG_THAI_CHOICES = [
        ('dang_cam', 'Dang cam'),
        ('da_chuoc', 'Da chuoc'),
        ('qua_han', 'Qua han'),
    ]

    khach_hang = models.ForeignKey(
        KhachHang, on_delete=models.PROTECT, verbose_name="Khach hang"
    )
    tai_san = models.ForeignKey(
        TaiSan, on_delete=models.PROTECT, verbose_name="Tai san"
    )
    ngay_cam = models.DateField(verbose_name="Ngay cam")
    ngay_den_han = models.DateField(verbose_name="Ngay den han")
    so_tien_cam = models.DecimalField(
        max_digits=15, decimal_places=0, verbose_name="So tien cam (VND)"
    )
    lai_suat = models.DecimalField(
        max_digits=5, decimal_places=2, verbose_name="Lai suat (%/thang)"
    )
    trang_thai = models.CharField(
        max_length=20, choices=TRANG_THAI_CHOICES,
        default='dang_cam', verbose_name="Trang thai"
    )

    def __str__(self):
        return f"HD#{self.id} - {self.khach_hang} - {self.ngay_den_han}"

    class Meta:
        verbose_name = "Hop dong cam"
        verbose_name_plural = "Hop dong cam"


class ThanhToan(models.Model):
    hop_dong = models.ForeignKey(
        HopDongCam, on_delete=models.CASCADE, verbose_name="Hop dong"
    )
    ngay_thanh_toan = models.DateField(verbose_name="Ngay thanh toan")
    so_tien = models.DecimalField(
        max_digits=15, decimal_places=0, verbose_name="So tien (VND)"
    )

    def __str__(self):
        return f"TT {self.ngay_thanh_toan} - {self.so_tien}"

    class Meta:
        verbose_name = "Thanh toan"
        verbose_name_plural = "Thanh toan"
```

### - Tạo file `admin.py` bằng `nano django_app/quanlycamdo/admin.py`
```
from django.contrib import admin
from .models import KhachHang, TaiSan, HopDongCam, ThanhToan

@admin.register(KhachHang)
class KhachHangAdmin(admin.ModelAdmin):
    list_display = ['id', 'ho_ten', 'so_dien_thoai', 'so_cmnd']
    search_fields = ['ho_ten', 'so_cmnd', 'so_dien_thoai']

@admin.register(TaiSan)
class TaiSanAdmin(admin.ModelAdmin):
    list_display = ['id', 'ten_tai_san', 'tinh_trang']
    search_fields = ['ten_tai_san']

@admin.register(HopDongCam)
class HopDongCamAdmin(admin.ModelAdmin):
    list_display = ['id', 'khach_hang', 'tai_san', 'ngay_cam', 'ngay_den_han', 'so_tien_cam', 'trang_thai']
    list_filter = ['trang_thai']
    search_fields = ['khach_hang__ho_ten']

@admin.register(ThanhToan)
class ThanhToanAdmin(admin.ModelAdmin):
    list_display = ['id', 'hop_dong', 'ngay_thanh_toan', 'so_tien']
```

### - Truy cập vào `http://localhost:8000/admin`
<img width="1919" height="904" alt="image" src="https://github.com/user-attachments/assets/58719d22-7d12-4071-9510-a8779a424e35" />

<img width="1919" height="906" alt="image" src="https://github.com/user-attachments/assets/a3adf6cd-fb15-4dc0-8272-05ff73aa12e4" />
  
  + Thêm thử 1 khách hàng bằng cách bấm "Thêm vào" bên cạnh Khach hang:
<img width="1918" height="617" alt="image" src="https://github.com/user-attachments/assets/d5f088d7-5059-4442-8ebb-bb37d6b049a1" />
 
  + Thêm thử 1 tài sản bằng cách bấm "Thêm vào" bên cạnh Tai san:
<img width="1919" height="444" alt="image" src="https://github.com/user-attachments/assets/d316ee4e-4cc2-43c7-b84b-45e9991e34df" />

### - Tiếp tục truy cập vào `http://localhost:8080`
<img width="1919" height="908" alt="image" src="https://github.com/user-attachments/assets/22faeaed-cdc9-4d14-af7b-529869fdf239" />

### - Bấm vào `camdo_db` → sẽ thấy danh sách các bảng → tiếp tục bấm vào bảng `quanlycamdo_hopdongcam` để thấy cột FK lưu ID số
     + khach_hang_id
     + tai_san_id
<img width="1057" height="569" alt="image" src="https://github.com/user-attachments/assets/5b29518d-5651-45b2-b5ad-8c306aa65583" />

### - Quay lại `http://localhost:8000/admin` để thêm dữ liệu test
     + Thêm 1 Khach hang
<img width="1317" height="636" alt="image" src="https://github.com/user-attachments/assets/a5b30268-6c37-4a4d-8c22-71e52b23164b" />

     + Thêm 1 Tai san
<img width="1192" height="581" alt="image" src="https://github.com/user-attachments/assets/c26221b3-a5c2-4c0e-84d7-42e112e59e49" />

     + Thêm 1 Hop dong cam
<img width="880" height="626" alt="image" src="https://github.com/user-attachments/assets/ebfbd460-e8d1-492f-a926-df87ca61828e" />

### - Sau đó quay lại PhpMyAdmin xem bảng `quanlycamdo_hopdongcam` sẽ thấy `khach_hang_id = 1`, `tai_san_id = 1`
<img width="1154" height="556" alt="image" src="https://github.com/user-attachments/assets/c96afad3-3086-41e2-850c-aca4ec16060d" />

### - Tạo file `home.html`
```
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Tiệm Cầm Đồ - Con Nợ Đến Hạn</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background: #f5f5f5; }
        h1 { color: #c0392b; }
        table { width: 100%; border-collapse: collapse; background: white; }
        th { background: #c0392b; color: white; padding: 10px; }
        td { padding: 10px; border-bottom: 1px solid #ddd; }
        tr:hover { background: #ffeaea; }
        .badge { padding: 3px 8px; border-radius: 4px; background: #e74c3c; color: white; font-size: 12px; }
        .empty { color: green; font-size: 18px; margin-top: 20px; }
    </style>
</head>
<body>
    <h1>🏦 Tiệm Cầm Đồ - Danh sách con nợ đến hạn chưa trả</h1>
    <p>Hôm nay: <strong>{{ hom_nay }}</strong></p>
    <p>Tổng số: <strong>{{ tong_so }}</strong> hợp đồng đến hạn</p>

    {% if hop_dong_list %}
    <table>
        <thead>
            <tr>
                <th>Số HĐ</th>
                <th>Khách hàng</th>
                <th>Số điện thoại</th>
                <th>Tài sản cầm</th>
                <th>Tiền cầm (VNĐ)</th>
                <th>Lãi suất</th>
                <th>Ngày cầm</th>
                <th>Ngày đến hạn</th>
                <th>Trạng thái</th>
            </tr>
        </thead>
        <tbody>
            {% for hd in hop_dong_list %}
            <tr>
                <td>#{{ hd.id }}</td>
                <td>{{ hd.khach_hang.ho_ten }}</td>
                <td>{{ hd.khach_hang.so_dien_thoai }}</td>
                <td>{{ hd.tai_san.ten_tai_san }}</td>
                <td>{{ hd.so_tien_cam }}</td>
                <td>{{ hd.lai_suat }}%/tháng</td>
                <td>{{ hd.ngay_cam }}</td>
                <td style="color:red; font-weight:bold;">{{ hd.ngay_den_han }}</td>
                <td><span class="badge">Đến hạn / Quá hạn</span></td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
    {% else %}
        <p class="empty">✅ Không có hợp đồng nào đến hạn!</p>
    {% endif %}
</body>
</html>
```

### - Vào lại `http://localhost:8000`
<img width="1919" height="323" alt="image" src="https://github.com/user-attachments/assets/fc6ae6e8-91f4-49e4-9f0f-f0fe1954842c" />

### - Chạy lệnh `wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb`






