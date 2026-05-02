# BÀI KIỂM TRA SỐ 2 – HỆ QUẢN TRỊ CSDL SQL SERVER

**Họ và tên:** Bùi Duy Linh

**Mã sinh viên:** K235480106041

**Lớp:** K59KMT.K01

**Đề tài:** Quản lý sinh viên

---

## PHẦN MỞ ĐẦU

### 1. Thông tin và định hướng bài làm

Bài làm này được thực hiện trên **SQL Server** với đề tài **quản lý sinh viên**. Mục tiêu là xây dựng một cơ sở dữ liệu phục vụ việc quản lý thông tin sinh viên, lớp học, môn học và kết quả học tập.

Báo cáo được trình bày theo đúng cấu trúc yêu cầu của đề bài, gồm 5 phần:

1. Thiết kế và khởi tạo cấu trúc dữ liệu.
2. Xây dựng Function.
3. Xây dựng Stored Procedure.
4. Trigger và xử lý logic nghiệp vụ.
5. Cursor và duyệt dữ liệu.

### 2. Tóm tắt yêu cầu đầu bài

- Thực hiện toàn bộ bài trên **SQL Server**.
- Mỗi bước cần có **ảnh chụp màn hình** gồm cả câu lệnh SQL và kết quả thực thi.
- Ảnh cần có **chú thích rõ ràng**: làm gì, giải quyết vấn đề gì, kết quả nói lên điều gì.
- Sản phẩm nộp gồm:
  - `README.md`: trình bày bài làm, code và ảnh minh họa.
  - `baikiemtra2.sql`: chứa toàn bộ mã nguồn SQL.
- Cần đặt tên đúng quy chuẩn, logic đúng và có lịch sử commit trên GitHub.

### 3. Giới thiệu ngắn về đề tài

Đề tài **quản lý sinh viên** mô phỏng hoạt động quản lý sinh viên trong một trường đại học. Hệ thống tập trung vào các nghiệp vụ chính sau:

- Quản lý thông tin lớp học.
- Quản lý thông tin sinh viên.
- Quản lý môn học.
- Quản lý điểm số và học lực.
- Theo dõi sinh viên được học bổng và cảnh báo học vụ.

### 4. Tên cơ sở dữ liệu sử dụng

Theo yêu cầu đề bài, tên database có gắn mã sinh viên cá nhân:
QuanLySinhVien_K235480106041

### 5. Phân tích bài toán quản lý sinh viên

Đề tài này giải quyết các vấn đề thực tế trong quản lý sinh viên:

- **Quản lý thông tin cơ bản**: Lưu trữ họ tên, ngày sinh, giới tính, địa chỉ của sinh viên.
- **Phân lớp quản lý**: Mỗi sinh viên thuộc một lớp, mỗi lớp có sĩ số và giáo viên chủ nhiệm.
- **Quản lý kết quả học tập**: Theo dõi điểm các môn học, tính GPA, xếp loại học lực.
- **Cảnh báo học vụ**: Tự động phát hiện sinh viên có điểm trung bình thấp hoặc học lại nhiều.
- **Học bổng**: Xét duyệt sinh viên đủ điều kiện nhận học bổng dựa trên GPA và rèn luyện.

---

# PHẦN 1: THIẾT KẾ VÀ KHỞI TẠO CẤU TRÚC DỮ LIỆU

## 1.1 Mô tả logic thiết kế

Em chọn bài toán **quản lý sinh viên**. Cơ sở dữ liệu gồm 4 bảng chính:

- `[LopHoc]`: lưu thông tin các lớp học.
- `[SinhVien]`: lưu thông tin sinh viên.
- `[MonHoc]`: lưu thông tin môn học.
- `[Diem]`: lưu điểm số của sinh viên theo từng môn.

### Mối quan hệ giữa các bảng

- Một **lớp học** có thể có nhiều **sinh viên** → quan hệ 1-N.
- Một **sinh viên** có thể có nhiều **điểm môn học** → quan hệ 1-N.
- Một **môn học** có thể có nhiều **điểm** từ các sinh viên → quan hệ 1-N.

### Sơ đồ quan hệ đơn giản

```
[LopHoc] 1 ---- N [SinhVien] 1 ---- N [Diem] N ---- 1 [MonHoc]
```

## 1.2 Mô tả các bảng dữ liệu

### Bảng `[LopHoc]`

| Tên cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaLop] | INT | PK, IDENTITY(1,1) | Mã lớp học |
| [TenLop] | NVARCHAR(100) | NOT NULL, UNIQUE | Tên lớp |
| [NienKhoa] | NVARCHAR(20) | NOT NULL | Niên khóa |
| [SiSo] | INT | DEFAULT 0, CHECK >= 0 | Sĩ số hiện tại |
| [GiaoVienChuNhiem] | NVARCHAR(150) | NOT NULL | Giáo viên chủ nhiệm |
| [NgayTao] | DATE | DEFAULT GETDATE() | Ngày tạo lớp |

### Bảng `[SinhVien]`

| Tên cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaSV] | INT | PK, IDENTITY(1,1) | Mã sinh viên |
| [HoTen] | NVARCHAR(150) | NOT NULL | Họ tên sinh viên |
| [NgaySinh] | DATE | NOT NULL | Ngày sinh |
| [GioiTinh] | NVARCHAR(10) | NOT NULL | Nam/Nữ/Khác |
| [DiaChi] | NVARCHAR(200) | NULL | Địa chỉ |
| [Email] | VARCHAR(100) | NULL | Email |
| [MaLop] | INT | FK | Mã lớp |
| [DiemRenLuyen] | INT | DEFAULT 0, CHECK (0-100) | Điểm rèn luyện |
| [TrangThai] | NVARCHAR(30) | DEFAULT N'DangHoc' | Trạng thái SV |

### Bảng `[MonHoc]`

| Tên cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaMH] | INT | PK, IDENTITY(1,1) | Mã môn học |
| [TenMH] | NVARCHAR(150) | NOT NULL | Tên môn học |
| [SoTinChi] | INT | NOT NULL, CHECK > 0 | Số tín chỉ |
| [HocKy] | NVARCHAR(20) | NOT NULL | Học kỳ |
| [NamHoc] | NVARCHAR(20) | NOT NULL | Năm học |

### Bảng `[Diem]`

| Tên cột | Kiểu dữ liệu | Ràng buộc | Ý nghĩa |
|---|---|---|---|
| [MaDiem] | INT | PK, IDENTITY(1,1) | Mã điểm |
| [MaSV] | INT | FK | Mã sinh viên |
| [MaMH] | INT | FK | Mã môn học |
| [DiemGK] | DECIMAL(4,2) | NULL, CHECK (0-10) | Điểm giữa kỳ |
| [DiemCK] | DECIMAL(4,2) | NULL, CHECK (0-10) | Điểm cuối kỳ |
| [DiemTongKet] | DECIMAL(4,2) | NULL, CHECK (0-10) | Điểm tổng kết |
| [LanHoc] | INT | NOT NULL DEFAULT 1 | Lần học (1, 2, 3...) |
| [TrangThai] | NVARCHAR(20) | DEFAULT N'ChuaHoc' | Chưa học/Đạt/Không đạt |

## 1.3 Tạo database

```sql
CREATE DATABASE [QuanLySinhVien_K235480106041];
GO

USE [QuanLySinhVien_K235480106041];
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/454f6e3c-3483-4093-b3b9-21f008a905e2" />

> **Chú thích ảnh:** Ảnh này cho thấy em đã tạo database đúng tên theo yêu cầu có gắn mã sinh viên K235480106020.

## 1.4 Tạo bảng

### Tạo bảng `[LopHoc]`

```sql
CREATE TABLE [LopHoc]
(
    [MaLop] INT IDENTITY(1,1) NOT NULL,
    [TenLop] NVARCHAR(100) NOT NULL,
    [NienKhoa] NVARCHAR(20) NOT NULL,
    [SiSo] INT NOT NULL DEFAULT 0,
    [GiaoVienChuNhiem] NVARCHAR(150) NOT NULL,
    [NgayTao] DATE NOT NULL DEFAULT GETDATE(),

    CONSTRAINT [PK_LopHoc] PRIMARY KEY ([MaLop]),
    CONSTRAINT [UQ_LopHoc_TenLop] UNIQUE ([TenLop]),
    CONSTRAINT [CK_LopHoc_SiSo] CHECK ([SiSo] >= 0)
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1d95dc76-9ac7-408d-bc4a-33acabf17212" />

> **Chú thích ảnh:** Ảnh này cho thấy bảng LopHoc đã được tạo với PK là MaLop, ràng buộc UNIQUE cho TenLop và CHECK cho SiSo.

### Tạo bảng `[SinhVien]`

```sql
CREATE TABLE [SinhVien]
(
    [MaSV] INT IDENTITY(1,1) NOT NULL,
    [HoTen] NVARCHAR(150) NOT NULL,
    [NgaySinh] DATE NOT NULL,
    [GioiTinh] NVARCHAR(10) NOT NULL,
    [DiaChi] NVARCHAR(200) NULL,
    [Email] VARCHAR(100) NULL,
    [MaLop] INT NOT NULL,
    [DiemRenLuyen] INT NOT NULL DEFAULT 0,
    [TrangThai] NVARCHAR(30) NOT NULL DEFAULT N'DangHoc',

    CONSTRAINT [PK_SinhVien] PRIMARY KEY ([MaSV]),
    CONSTRAINT [FK_SinhVien_LopHoc] FOREIGN KEY ([MaLop]) REFERENCES [LopHoc]([MaLop]),
    CONSTRAINT [CK_SinhVien_GioiTinh] CHECK ([GioiTinh] IN (N'Nam', N'Nữ', N'Khác')),
    CONSTRAINT [CK_SinhVien_DiemRenLuyen] CHECK ([DiemRenLuyen] >= 0 AND [DiemRenLuyen] <= 100),
    CONSTRAINT [CK_SinhVien_TrangThai] CHECK ([TrangThai] IN (N'DangHoc', N'DaTotNghiep', N'BaoLuu', N'DinhChi'))
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/f6134b4b-566d-4ef7-a033-f60a91b7fa2c" />

> **Chú thích ảnh:** Ảnh này cho thấy bảng SinhVien có PK, FK liên kết tới LopHoc, ràng buộc CHECK cho GioiTinh và DiemRenLuyen.

### Tạo bảng `[MonHoc]`

```sql
CREATE TABLE [MonHoc]
(
    [MaMH] INT IDENTITY(1,1) NOT NULL,
    [TenMH] NVARCHAR(150) NOT NULL,
    [SoTinChi] INT NOT NULL,
    [HocKy] NVARCHAR(20) NOT NULL,
    [NamHoc] NVARCHAR(20) NOT NULL,

    CONSTRAINT [PK_MonHoc] PRIMARY KEY ([MaMH]),
    CONSTRAINT [CK_MonHoc_SoTinChi] CHECK ([SoTinChi] > 0)
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/08635f10-090b-4704-9a14-08dbf885f019" />

> **Chú thích ảnh:** Ảnh này cho thấy bảng MonHoc đã có PK và ràng buộc CHECK cho SoTinChi.

### Tạo bảng `[Diem]`

```sql
CREATE TABLE [Diem]
(
    [MaDiem] INT IDENTITY(1,1) NOT NULL,
    [MaSV] INT NOT NULL,
    [MaMH] INT NOT NULL,
    [DiemGK] DECIMAL(4,2) NULL,
    [DiemCK] DECIMAL(4,2) NULL,
    [DiemTongKet] DECIMAL(4,2) NULL,
    [LanHoc] INT NOT NULL DEFAULT 1,
    [TrangThai] NVARCHAR(20) NOT NULL DEFAULT N'ChuaHoc',

    CONSTRAINT [PK_Diem] PRIMARY KEY ([MaDiem]),
    CONSTRAINT [FK_Diem_SinhVien] FOREIGN KEY ([MaSV]) REFERENCES [SinhVien]([MaSV]),
    CONSTRAINT [FK_Diem_MonHoc] FOREIGN KEY ([MaMH]) REFERENCES [MonHoc]([MaMH]),
    CONSTRAINT [CK_Diem_DiemGK] CHECK ([DiemGK] IS NULL OR ([DiemGK] >= 0 AND [DiemGK] <= 10)),
    CONSTRAINT [CK_Diem_DiemCK] CHECK ([DiemCK] IS NULL OR ([DiemCK] >= 0 AND [DiemCK] <= 10)),
    CONSTRAINT [CK_Diem_DiemTongKet] CHECK ([DiemTongKet] IS NULL OR ([DiemTongKet] >= 0 AND [DiemTongKet] <= 10)),
    CONSTRAINT [CK_Diem_LanHoc] CHECK ([LanHoc] > 0),
    CONSTRAINT [CK_Diem_TrangThai] CHECK ([TrangThai] IN (N'ChuaHoc', N'Dat', N'KhongDat'))
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/0e247580-6719-4d6d-8c26-83b9e4acac88" />

> **Chú thích ảnh:** Ảnh này cho thấy bảng Diem có 2 FK liên kết tới SinhVien và MonHoc, cùng các ràng buộc CHECK cho điểm số (0-10).

## 1.5 Giải thích PK, FK, CK

**Phân tích thiết kế:**
- **PK (Khóa chính)**: Đảm bảo mỗi bản ghi có định danh duy nhất.
  - `[LopHoc].[MaLop]`
  - `[SinhVien].[MaSV]`
  - `[MonHoc].[MaMH]`
  - `[Diem].[MaDiem]`

- **FK (Khóa ngoại)**: Đảm bảo tính toàn vẹn tham chiếu.
  - `[SinhVien].[MaLop]` tham chiếu `[LopHoc].[MaLop]` - Sinh viên phải thuộc một lớp tồn tại.
  - `[Diem].[MaSV]` tham chiếu `[SinhVien].[MaSV]` - Điểm phải thuộc về một sinh viên tồn tại.
  - `[Diem].[MaMH]` tham chiếu `[MonHoc].[MaMH]` - Điểm phải thuộc về một môn học tồn tại.

- **CK (Ràng buộc cứng) tiêu biểu**:
  - `[SiSo] >= 0`: Sĩ số không thể âm.
  - `[DiemRenLuyen]` từ 0-100: Điểm rèn luyện theo thang điểm chuẩn.
  - `[DiemGK]`, `[DiemCK]`, `[DiemTongKet]` từ 0-10: Điểm số theo thang 10.
  - `[GioiTinh]` chỉ nhận Nam/Nữ/Khác: Tránh dữ liệu rác.
  - `[TrangThai]` trong các giá trị quy định: Đảm bảo trạng thái hợp lệ.

## 1.6 Chèn dữ liệu mẫu

### Dữ liệu bảng `[LopHoc]`

```sql
INSERT INTO [LopHoc] ([TenLop], [NienKhoa], [SiSo], [GiaoVienChuNhiem])
VALUES
(N'K59KMT.K01', N'2023-2027', 0, N'TS. Nguyễn Văn A'),
(N'K59KMT.K02', N'2023-2027', 0, N'ThS. Trần Thị B'),
(N'K58CNT.K01', N'2022-2026', 0, N'TS. Lê Văn C'),
(N'K60KMT.K01', N'2024-2028', 0, N'ThS. Phạm Thị D');
GO
```

### Dữ liệu bảng `[SinhVien]`

```sql
INSERT INTO [SinhVien] ([HoTen], [NgaySinh], [GioiTinh], [DiaChi], [Email], [MaLop], [DiemRenLuyen])
VALUES
(N'Nguyễn Văn Hải', '2005-03-15', N'Nam', N'Hà Nội', 'hai@gmail.com', 1, 85),
(N'Trần Thị Mai', '2005-08-20', N'Nữ', N'Hải Phòng', 'mai@gmail.com', 1, 90),
(N'Lê Văn Nam', '2004-12-10', N'Nam', N'Nam Định', 'nam@gmail.com', 1, 75),
(N'Phạm Thị Hà', '2005-05-01', N'Nữ', N'Thái Bình', 'ha@gmail.com', 2, 95),
(N'Hoàng Văn Tú', '2004-11-25', N'Nam', N'Quảng Ninh', 'tu@gmail.com', 2, 60),
(N'Vũ Thị Lan', '2005-07-18', N'Nữ', N'Bắc Giang', 'lan@gmail.com', 3, 88);
GO
```

### Dữ liệu bảng `[MonHoc]`

```sql
INSERT INTO [MonHoc] ([TenMH], [SoTinChi], [HocKy], [NamHoc])
VALUES
(N'Cơ sở dữ liệu', 3, N'HK1', N'2023-2024'),
(N'Lập trình hướng đối tượng', 4, N'HK1', N'2023-2024'),
(N'Toán rời rạc', 3, N'HK1', N'2023-2024'),
(N'Mạng máy tính', 3, N'HK2', N'2023-2024'),
(N'Hệ điều hành', 4, N'HK2', N'2023-2024');
GO
```

### Dữ liệu bảng `[Diem]`

```sql
INSERT INTO [Diem] ([MaSV], [MaMH], [DiemGK], [DiemCK], [DiemTongKet], [LanHoc], [TrangThai])
VALUES
(1, 1, 7.5, 8.0, 7.8, 1, N'Dat'),
(1, 2, 8.0, 7.5, 7.7, 1, N'Dat'),
(1, 3, 6.5, 7.0, 6.8, 1, N'Dat'),
(2, 1, 9.0, 8.5, 8.7, 1, N'Dat'),
(2, 2, 8.5, 9.0, 8.8, 1, N'Dat'),
(3, 1, 5.0, 4.5, 4.7, 1, N'KhongDat'),
(3, 1, 6.0, 6.5, 6.3, 2, N'Dat'),
(4, 1, 9.5, 9.0, 9.2, 1, N'Dat'),
(4, 3, 8.0, 8.5, 8.3, 1, N'Dat'),
(5, 1, 4.0, 3.5, 3.7, 1, N'KhongDat'),
(5, 2, 5.5, 6.0, 5.8, 1, N'KhongDat'),
(6, 4, 8.0, 7.5, 7.7, 1, N'Dat'),
(6, 5, 7.5, 8.0, 7.8, 1, N'Dat');
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a59cba79-3f50-4fa2-9e0a-c0de755f64d8" />

> **Chú thích ảnh:** Ảnh này cho thấy dữ liệu mẫu đã được thêm thành công để phục vụ kiểm thử các function, procedure, trigger và cursor.

---

# PHẦN 2: XÂY DỰNG FUNCTION

## 2.1 SQL Server có những loại built-in function nào?

Trong SQL Server có nhiều nhóm hàm dựng sẵn, ví dụ:

- Hàm chuỗi: `LEN`, `UPPER`, `LOWER`, `CONCAT`, `LEFT`, `RIGHT`
- Hàm ngày giờ: `GETDATE`, `DATEDIFF`, `DATEADD`, `YEAR`, `MONTH`
- Hàm số học: `ROUND`, `ABS`, `CEILING`, `FLOOR`
- Hàm tổng hợp: `SUM`, `AVG`, `COUNT`, `MIN`, `MAX`
- Hàm chuyển kiểu: `CAST`, `CONVERT`, `TRY_CONVERT`
- Hàm hệ thống: `DB_NAME`, `SUSER_NAME`, `NEWID`, `@@VERSION`

## 2.2 Một vài system function em tìm hiểu được

```sql
SELECT DB_NAME() AS TenCoSoDuLieu;
SELECT SUSER_NAME() AS TenNguoiDangNhap;
SELECT GETDATE() AS ThoiGianHeThong;
SELECT @@VERSION AS ThongTinPhienBanSQLServer;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/a9e3a09f-3bf7-46a6-bec1-0cb0f7ba4fde" />

> **Chú thích ảnh:** Ảnh này cho thấy các hàm hệ thống dùng để lấy tên database, tài khoản đăng nhập và thông tin hệ thống.

## 2.3 Khai thác built-in function trong đề tài quản lý sinh viên

**Phân tích bài toán:**
Trong quản lý sinh viên, cần xử lý nhiều dữ liệu như tính tuổi sinh viên, chuẩn hóa tên, tính điểm trung bình. Các built-in function giúp xử lý nhanh chóng.

```sql
-- Tính tuổi sinh viên
SELECT
    [MaSV],
    [HoTen],
    DATEDIFF(YEAR, [NgaySinh], GETDATE()) AS Tuoi
FROM [SinhVien];

-- Chuẩn hóa họ tên
SELECT
    [MaSV],
    UPPER([HoTen]) AS HoTenVietHoa,
    LEN([HoTen]) AS DoDaiHoTen
FROM [SinhVien];

-- Tính điểm trung bình của sinh viên
SELECT
    [MaSV],
    AVG([DiemTongKet]) AS DiemTrungBinh
FROM [Diem]
WHERE [TrangThai] = N'Dat'
GROUP BY [MaSV];
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/b677ed68-0e6d-4603-bef8-4d3701934335" />

> **Chú thích ảnh:** Ảnh này minh họa cách dùng hàm DATEDIFF tính tuổi, UPPER chuẩn hóa tên và AVG tính điểm trung bình sinh viên.

## 2.4 Hàm do người dùng tự viết dùng để làm gì?

**Phân tích:**
Hàm do người dùng tự viết giúp:
- Đóng gói logic xử lý để tái sử dụng nhiều lần.
- Làm câu lệnh SQL dễ đọc hơn.
- Phù hợp với các nghiệp vụ riêng của hệ thống mà hàm có sẵn không mô tả trực tiếp.

### Các loại User-defined Function

- **Scalar Function**: trả về một giá trị.
- **Inline Table-Valued Function**: trả về một bảng từ một câu `SELECT`.
- **Multi-statement Table-Valued Function**: trả về một bảng sau nhiều bước xử lý.

**Tại sao cần tự viết function?**
Mặc dù SQL Server có nhiều system function, vẫn cần tự viết function riêng vì bài toán quản lý sinh viên có logic đặc thù: tính GPA theo thang 4, xếp loại học lực, thống kê cảnh báo học vụ - những thứ system function không có sẵn.

## 2.5 Scalar Function

### Bài toán đặt ra

**Yêu cầu:** Viết hàm tính **GPA theo thang 4** từ điểm tổng kết (thang 10). Quy đổi: >=8.5→4.0, >=7.0→3.0, >=5.5→2.0, >=4.0→1.0, <4.0→0.

**Phân tích logic:**
Hàm nhận vào điểm thang 10, dùng CASE để quy đổi sang thang 4. Đây là logic đặc thù của hệ thống giáo dục Việt Nam mà system function không có.

```sql
-- ============================================================
-- FUNCTION: fn_TinhGPA
-- Mô tả:
-- Hàm dùng để quy đổi điểm hệ 10 sang điểm GPA hệ 4.
-- Trả về giá trị GPA tương ứng theo từng khoảng điểm.
-- ============================================================

CREATE OR ALTER FUNCTION [dbo].[fn_TinhGPA]
(
    -- Tham số đầu vào: Điểm hệ 10 của sinh viên
    @DiemThang10 DECIMAL(4,2)
)
-- Giá trị trả về: GPA hệ 4
RETURNS DECIMAL(3,2)
AS
BEGIN

    -- Khai báo biến lưu GPA sau khi quy đổi
    DECLARE @GPA DECIMAL(3,2);

    -- Nếu điểm NULL thì GPA cũng NULL
    IF @DiemThang10 IS NULL
        SET @GPA = NULL;

    -- Điểm từ 8.5 trở lên => GPA 4.0
    ELSE IF @DiemThang10 >= 8.5
        SET @GPA = 4.0;

    -- Điểm từ 7.0 đến dưới 8.5 => GPA 3.0
    ELSE IF @DiemThang10 >= 7.0
        SET @GPA = 3.0;

    -- Điểm từ 5.5 đến dưới 7.0 => GPA 2.0
    ELSE IF @DiemThang10 >= 5.5
        SET @GPA = 2.0;

    -- Điểm từ 4.0 đến dưới 5.5 => GPA 1.0
    ELSE IF @DiemThang10 >= 4.0
        SET @GPA = 1.0;

    -- Điểm dưới 4.0 => GPA 0.0
    ELSE
        SET @GPA = 0.0;

    -- Trả về kết quả GPA
    RETURN @GPA;
END;
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/b4a1f2f7-2f67-4e80-bfcf-0aeb4eb66352" />
Tạo hàm

### Khai thác hàm

```sql
-- ============================================================
-- TRUY VẤN: Hiển thị GPA hệ 4 của sinh viên
-- Mô tả:
-- Truy vấn lấy danh sách điểm tổng kết của sinh viên
-- và sử dụng hàm fn_TinhGPA để quy đổi sang GPA hệ 4.
-- ============================================================

SELECT

    -- Mã điểm của bản ghi
    d.[MaDiem],

    -- Họ tên sinh viên
    sv.[HoTen],

    -- Tên môn học
    mh.[TenMH],

    -- Điểm tổng kết hệ 10
    d.[DiemTongKet],

    -- Gọi hàm fn_TinhGPA để quy đổi GPA hệ 4
    [dbo].[fn_TinhGPA](d.[DiemTongKet]) AS GPA_Thang4

FROM [Diem] d

-- Liên kết bảng SinhVien để lấy thông tin sinh viên
INNER JOIN [SinhVien] sv 
    ON d.[MaSV] = sv.[MaSV]

-- Liên kết bảng MonHoc để lấy thông tin môn học
INNER JOIN [MonHoc] mh 
    ON d.[MaMH] = mh.[MaMH]

-- Chỉ lấy các bản ghi đã có điểm tổng kết
WHERE d.[DiemTongKet] IS NOT NULL;
```

<img width="1916" height="1079" alt="image" src="https://github.com/user-attachments/assets/5f6feedf-ae6f-42d4-b958-93a8cf82b3a6" />

> **Chú thích ảnh:** Ảnh này cho thấy hàm fn_TinhGPA đã quy đổi điểm thang 10 sang thang 4 theo đúng quy chuẩn giáo dục.

## 2.6 Inline Table-Valued Function

### Bài toán đặt ra

**Yêu cầu:** Viết hàm trả về danh sách sinh viên theo lớp kèm theo GPA trung bình và xếp loại.

**Phân tích logic:**
Hàm nhận mã lớp, join 3 bảng SinhVien, Diem, MonHoc để tính GPA trung bình cho từng sinh viên trong lớp. Inline TVF phù hợp vì chỉ cần một câu SELECT và được tối ưu tốt.

```sql
-- ============================================================
-- FUNCTION: fn_DanhSachSinhVienTheoLop
-- Mô tả:
-- Hàm trả về danh sách sinh viên thuộc một lớp học,
-- kèm điểm trung bình, GPA trung bình và xếp loại học lực.
-- ============================================================

CREATE OR ALTER FUNCTION [dbo].[fn_DanhSachSinhVienTheoLop]
(
    -- Tham số đầu vào: Mã lớp cần thống kê
    @MaLop INT
)

-- Hàm trả về dạng bảng (Table-Valued Function)
RETURNS TABLE
AS

RETURN
(

    SELECT

        -- Mã sinh viên
        sv.[MaSV],

        -- Họ tên sinh viên
        sv.[HoTen],

        -- Ngày sinh
        sv.[NgaySinh],

        -- Giới tính sinh viên
        sv.[GioiTinh],

        -- Tên lớp học
        lh.[TenLop],

        -- Tính điểm trung bình hệ 10
        -- Nếu chưa có điểm thì mặc định bằng 0
        ISNULL(AVG(d.[DiemTongKet]), 0) AS DiemTrungBinh,

        -- Tính GPA trung bình hệ 4
        -- Sử dụng hàm fn_TinhGPA để quy đổi
        ISNULL(AVG([dbo].[fn_TinhGPA](d.[DiemTongKet])), 0) AS GPA_TrungBinh,

        -- Xếp loại học lực dựa trên điểm trung bình
        CASE

            -- Điểm TB từ 8.5 trở lên
            WHEN AVG(d.[DiemTongKet]) >= 8.5 
                THEN N'XuatSac'

            -- Điểm TB từ 7.0 đến dưới 8.5
            WHEN AVG(d.[DiemTongKet]) >= 7.0 
                THEN N'Gioi'

            -- Điểm TB từ 5.5 đến dưới 7.0
            WHEN AVG(d.[DiemTongKet]) >= 5.5 
                THEN N'Kha'

            -- Điểm TB từ 4.0 đến dưới 5.5
            WHEN AVG(d.[DiemTongKet]) >= 4.0 
                THEN N'TrungBinh'

            -- Điểm TB dưới 4.0
            ELSE N'Yeu'

        END AS XepLoaiHocLuc

    FROM [SinhVien] sv

    -- Liên kết với bảng lớp học
    INNER JOIN [LopHoc] lh 
        ON sv.[MaLop] = lh.[MaLop]

    -- Liên kết bảng điểm để lấy điểm sinh viên
    -- Chỉ lấy các môn có trạng thái "Đạt"
    LEFT JOIN [Diem] d 
        ON sv.[MaSV] = d.[MaSV] 
        AND d.[TrangThai] = N'Dat'

    -- Điều kiện lọc theo mã lớp truyền vào
    WHERE sv.[MaLop] = @MaLop

    -- Gom nhóm theo từng sinh viên
    GROUP BY 
        sv.[MaSV], 
        sv.[HoTen], 
        sv.[NgaySinh], 
        sv.[GioiTinh], 
        lh.[TenLop]
);
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/1ae4c001-5331-45da-8f6f-0d7d2732a922" />

Tạo hàm

### Khai thác hàm

```sql
SELECT *
FROM [dbo].[fn_DanhSachSinhVienTheoLop](1)
ORDER BY [DiemTrungBinh] DESC;
```

<img width="1918" height="1079" alt="image" src="https://github.com/user-attachments/assets/e7dffaa8-5f59-49fa-aafe-9101274eb3c6" />

> **Chú thích ảnh:** Ảnh này cho thấy hàm trả về danh sách sinh viên trong lớp kèm GPA và xếp loại học lực.

## 2.7 Multi-statement Table-Valued Function

### Bài toán đặt ra

**Yêu cầu:** Viết hàm thống kê tình hình học tập của một sinh viên: số môn đạt, không đạt, GPA, xếp loại và cảnh báo học vụ.

**Phân tích logic:**
Hàm cần nhiều bước xử lý: tính số môn đạt/không đạt, tính GPA, xếp loại, và đưa ra cảnh báo. Multi-statement TVF phù hợp vì cần dùng biến bảng và nhiều câu lệnh xử lý.

```sql
-- ============================================================
-- FUNCTION: fn_ThongKeHocTapSinhVien
-- Mô tả:
-- Hàm dùng để thống kê tình hình học tập của một sinh viên,
-- bao gồm số môn học, GPA trung bình, xếp loại học lực
-- và cảnh báo học vụ.
-- ============================================================

CREATE OR ALTER FUNCTION [dbo].[fn_ThongKeHocTapSinhVien]
(
    -- Tham số đầu vào: Mã sinh viên cần thống kê
    @MaSV INT
)

-- Hàm trả về bảng kết quả thống kê
RETURNS @KetQua TABLE
(
    -- Mã sinh viên
    [MaSV] INT,

    -- Họ tên sinh viên
    [HoTen] NVARCHAR(150),

    -- Tổng số môn đã học
    [TongSoMon] INT,

    -- Số môn đạt
    [SoMonDat] INT,

    -- Số môn không đạt
    [SoMonKhongDat] INT,

    -- GPA trung bình hệ 4
    [GPA_Thang4] DECIMAL(3,2),

    -- Xếp loại học lực
    [XepLoai] NVARCHAR(50),

    -- Thông báo cảnh báo học vụ
    [CanhBaoHocVu] NVARCHAR(100)
)

AS
BEGIN

    -- ========================================================
    -- Khai báo biến dùng để lưu kết quả thống kê
    -- ========================================================
    DECLARE 
        @TongMon INT,
        @MonDat INT,
        @MonKhongDat INT,
        @GPA DECIMAL(3,2);

    -- ========================================================
    -- Tính tổng số môn, số môn đạt và không đạt
    -- ========================================================
    SELECT
        -- Tổng số môn học
        @TongMon = COUNT(*),

        -- Đếm số môn đạt
        @MonDat = SUM(
            CASE 
                WHEN [TrangThai] = N'Dat' THEN 1 
                ELSE 0 
            END
        ),

        -- Đếm số môn không đạt
        @MonKhongDat = SUM(
            CASE 
                WHEN [TrangThai] = N'KhongDat' THEN 1 
                ELSE 0 
            END
        )

    FROM [Diem]

    -- Lọc theo sinh viên cần thống kê
    WHERE [MaSV] = @MaSV;

    -- ========================================================
    -- Tính GPA trung bình hệ 4
    -- Chỉ tính các môn đạt
    -- ========================================================
    SELECT 
        @GPA = AVG([dbo].[fn_TinhGPA]([DiemTongKet]))

    FROM [Diem]

    WHERE [MaSV] = @MaSV 
        AND [TrangThai] = N'Dat';

    -- Nếu GPA NULL thì gán mặc định bằng 0
    IF @GPA IS NULL 
        SET @GPA = 0;

    -- ========================================================
    -- Chèn kết quả thống kê vào bảng trả về
    -- ========================================================
    INSERT INTO @KetQua

    SELECT

        -- Mã sinh viên
        sv.[MaSV],

        -- Họ tên sinh viên
        sv.[HoTen],

        -- Tổng số môn học
        ISNULL(@TongMon, 0),

        -- Số môn đạt
        ISNULL(@MonDat, 0),

        -- Số môn không đạt
        ISNULL(@MonKhongDat, 0),

        -- GPA trung bình hệ 4
        @GPA,

        -- ====================================================
        -- Xếp loại học lực dựa trên GPA
        -- ====================================================
        CASE

            -- GPA từ 3.6 trở lên
            WHEN @GPA >= 3.6 
                THEN N'XuatSac'

            -- GPA từ 3.0 đến dưới 3.6
            WHEN @GPA >= 3.0 
                THEN N'Gioi'

            -- GPA từ 2.0 đến dưới 3.0
            WHEN @GPA >= 2.0 
                THEN N'Kha'

            -- GPA từ 1.0 đến dưới 2.0
            WHEN @GPA >= 1.0 
                THEN N'TrungBinh'

            -- GPA dưới 1.0
            ELSE N'Yeu'

        END,

        -- ====================================================
        -- Đưa ra cảnh báo học vụ
        -- ====================================================
        CASE

            -- Không đạt từ 3 môn trở lên
            WHEN @MonKhongDat >= 3 
                THEN N'CANH BAO: Hoc vu yeu, can cai thien!'

            -- GPA quá thấp
            WHEN @GPA < 1.0 
                THEN N'CANH BAO: GPA qua thap!'

            -- Có môn chưa đạt
            WHEN @MonKhongDat > 0 
                THEN N'CANH BAO: Con mon chua dat.'

            -- Trạng thái học tập bình thường
            ELSE N'Binh thuong'

        END

    FROM [SinhVien] sv

    -- Lấy đúng sinh viên cần thống kê
    WHERE sv.[MaSV] = @MaSV;

    -- Trả về kết quả
    RETURN;

END;
GO
```

<img width="1919" height="1077" alt="image" src="https://github.com/user-attachments/assets/f6c3a490-f244-4b7c-840b-777edd9aa86a" />

### Khai thác hàm

```sql
SELECT *
FROM [dbo].[fn_ThongKeHocTapSinhVien](1);
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4710821d-b2e5-4079-a3dc-a7b9e6c78ecc" />

> **Chú thích ảnh:** Ảnh này cho thấy hàm đã thống kê đầy đủ tình hình học tập và đưa ra cảnh báo học vụ cho sinh viên.

---

# PHẦN 3: XÂY DỰNG STORED PROCEDURE

## 3.1 Một vài system stored procedure trong SQL Server

Một số stored procedure hệ thống thường dùng:

- `sp_help`: xem cấu trúc đối tượng.
- `sp_helpconstraint`: xem ràng buộc của bảng.
- `sp_spaceused`: xem dung lượng bảng.

```sql
EXEC sp_help 'SinhVien';
EXEC sp_helpconstraint 'Diem';
EXEC sp_spaceused 'SinhVien';
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/d5dc7088-4296-414f-be6f-7512f586d3a2" />


> **Chú thích ảnh:** Ảnh này minh họa cách dùng stored procedure hệ thống để khảo sát cấu trúc bảng và ràng buộc.

## 3.2 Stored Procedure 1: Thêm sinh viên mới có kiểm tra logic

### Bài toán đặt ra

**Yêu cầu:** Khi thêm sinh viên mới cần kiểm tra:
- Lớp có tồn tại không.
- Email không được trùng với sinh viên khác.
- Tuổi sinh viên phải từ 16 đến 30.
- Điểm rèn luyện từ 0-100.

**Phân tích logic:**
Procedure kiểm tra tính hợp lệ trước khi insert. Nếu vi phạm sẽ RAISERROR và dừng. Sau khi thêm thành công, cập nhật lại sĩ số lớp.

```sql
-- ============================================================
-- PROCEDURE: sp_ThemSinhVien
-- Mô tả:
-- Thủ tục dùng để thêm mới sinh viên vào hệ thống.
-- Bao gồm các bước kiểm tra dữ liệu hợp lệ trước khi lưu.
-- ============================================================

CREATE OR ALTER PROCEDURE [dbo].[sp_ThemSinhVien]

    -- Họ tên sinh viên
    @HoTen NVARCHAR(150),

    -- Ngày sinh
    @NgaySinh DATE,

    -- Giới tính
    @GioiTinh NVARCHAR(10),

    -- Địa chỉ sinh viên
    @DiaChi NVARCHAR(200),

    -- Email sinh viên
    @Email VARCHAR(100),

    -- Mã lớp học
    @MaLop INT,

    -- Điểm rèn luyện
    @DiemRenLuyen INT

AS
BEGIN

    -- Không hiển thị số dòng ảnh hưởng sau mỗi câu lệnh
    SET NOCOUNT ON;

    -- ========================================================
    -- Kiểm tra lớp học có tồn tại hay không
    -- ========================================================
    IF NOT EXISTS (
        SELECT 1 
        FROM [LopHoc] 
        WHERE [MaLop] = @MaLop
    )
    BEGIN

        -- Báo lỗi nếu lớp không tồn tại
        RAISERROR(N'Lớp học không tồn tại!', 16, 1);

        RETURN;
    END;

    -- ========================================================
    -- Kiểm tra email đã tồn tại hay chưa
    -- ========================================================
    IF EXISTS (
        SELECT 1 
        FROM [SinhVien] 
        WHERE [Email] = @Email
    )
    BEGIN

        -- Báo lỗi nếu email bị trùng
        RAISERROR(N'Email đã tồn tại trong hệ thống!', 16, 1);

        RETURN;
    END;

    -- ========================================================
    -- Kiểm tra độ tuổi sinh viên
    -- Điều kiện: từ 16 đến 30 tuổi
    -- ========================================================
    IF DATEDIFF(YEAR, @NgaySinh, GETDATE()) < 16 
        OR DATEDIFF(YEAR, @NgaySinh, GETDATE()) > 30
    BEGIN

        -- Báo lỗi nếu tuổi không hợp lệ
        RAISERROR(N'Tuổi sinh viên phải từ 16 đến 30!', 16, 1);

        RETURN;
    END;

    -- ========================================================
    -- Kiểm tra điểm rèn luyện hợp lệ
    -- Giá trị cho phép: từ 0 đến 100
    -- ========================================================
    IF @DiemRenLuyen < 0 
        OR @DiemRenLuyen > 100
    BEGIN

        -- Báo lỗi nếu điểm rèn luyện không hợp lệ
        RAISERROR(N'Điểm rèn luyện phải từ 0 đến 100!', 16, 1);

        RETURN;
    END;

    -- ========================================================
    -- Thêm sinh viên mới vào bảng SinhVien
    -- ========================================================
    INSERT INTO [SinhVien] 
    (
        [HoTen], 
        [NgaySinh], 
        [GioiTinh], 
        [DiaChi], 
        [Email], 
        [MaLop], 
        [DiemRenLuyen]
    )

    VALUES 
    (
        @HoTen, 
        @NgaySinh, 
        @GioiTinh, 
        @DiaChi, 
        @Email, 
        @MaLop, 
        @DiemRenLuyen
    );

    -- ========================================================
    -- Cập nhật sĩ số lớp sau khi thêm sinh viên
    -- ========================================================
    UPDATE [LopHoc]

    SET [SiSo] = [SiSo] + 1

    WHERE [MaLop] = @MaLop;

    -- ========================================================
    -- Thông báo thêm sinh viên thành công
    -- ========================================================
    PRINT N'Thêm sinh viên thành công!';

END;
GO
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/257694bd-e63a-4d26-9af9-a4239e418b3b" />

Tạo sp

### Khai thác procedure

```sql
EXEC [dbo].[sp_ThemSinhVien]
    @HoTen = N'Nguyễn Văn Test',
    @NgaySinh = '2005-01-01',
    @GioiTinh = N'Nam',
    @DiaChi = N'Hà Nội',
    @Email = 'test@gmail.com',
    @MaLop = 1,
    @DiemRenLuyen = 80;
```

<img width="1909" height="1079" alt="image" src="https://github.com/user-attachments/assets/f41ca778-cd65-4801-bff9-7549b24d46f7" />

> **Chú thích ảnh:** Ảnh này cho thấy procedure đã kiểm tra logic và thêm sinh viên thành công, đồng thời cập nhật sĩ số lớp.

## 3.3 Stored Procedure 2: Dùng OUTPUT để trả về giá trị tính toán

### Bài toán đặt ra

**Yêu cầu:** Viết procedure tính **GPA trung bình** và **học bổng dự kiến** của một sinh viên.

**Phân tích logic:**
Procedure tính GPA từ bảng Diem, sau đó dựa trên GPA và điểm rèn luyện để tính học bổng. Trả về qua tham số OUTPUT để có thể sử dụng tiếp.

```sql
-- ============================================================
-- PROCEDURE: sp_TinhHocBongSinhVien
-- Mô tả:
-- Thủ tục dùng để tính GPA hệ 4 và xác định mức
-- học bổng dự kiến của sinh viên dựa trên:
-- + GPA trung bình
-- + Điểm rèn luyện
-- ============================================================

CREATE OR ALTER PROCEDURE [dbo].[sp_TinhHocBongSinhVien]

    -- Mã sinh viên cần tính học bổng
    @MaSV INT,

    -- Biến OUTPUT lưu GPA hệ 4
    @GPA_Thang4 DECIMAL(3,2) OUTPUT,

    -- Biến OUTPUT lưu học bổng dự kiến
    @HocBongDuKien MONEY OUTPUT

AS
BEGIN

    -- Không hiển thị số dòng ảnh hưởng
    SET NOCOUNT ON;

    -- ========================================================
    -- Tính GPA trung bình hệ 4 của sinh viên
    -- Chỉ tính các môn đạt
    -- ========================================================
    SELECT 
        @GPA_Thang4 = AVG([dbo].[fn_TinhGPA]([DiemTongKet]))

    FROM [Diem]

    WHERE [MaSV] = @MaSV 
        AND [TrangThai] = N'Dat';

    -- Nếu GPA NULL thì gán mặc định bằng 0
    IF @GPA_Thang4 IS NULL 
        SET @GPA_Thang4 = 0;

    -- ========================================================
    -- Lấy điểm rèn luyện của sinh viên
    -- ========================================================
    DECLARE @DiemRL INT;

    SELECT 
        @DiemRL = [DiemRenLuyen]

    FROM [SinhVien]

    WHERE [MaSV] = @MaSV;

    -- ========================================================
    -- Xét học bổng dựa trên GPA và điểm rèn luyện
    -- ========================================================

    -- Học bổng loại A
    -- Điều kiện:
    -- GPA >= 3.6 và điểm rèn luyện >= 90
    IF @GPA_Thang4 >= 3.6 
        AND @DiemRL >= 90

        SET @HocBongDuKien = 5000000;

    -- Học bổng loại B
    -- Điều kiện:
    -- GPA >= 3.0 và điểm rèn luyện >= 80
    ELSE IF @GPA_Thang4 >= 3.0 
        AND @DiemRL >= 80

        SET @HocBongDuKien = 3000000;

    -- Học bổng loại C
    -- Điều kiện:
    -- GPA >= 2.0 và điểm rèn luyện >= 70
    ELSE IF @GPA_Thang4 >= 2.0 
        AND @DiemRL >= 70

        SET @HocBongDuKien = 1500000;

    -- Không đủ điều kiện nhận học bổng
    ELSE
        SET @HocBongDuKien = 0;

END;
GO
```

<img width="1916" height="1079" alt="image" src="https://github.com/user-attachments/assets/82685a03-9c3d-4f9b-a58e-732f66716456" />

### Khai thác procedure

```sql
-- ============================================================
-- THỰC THI PROCEDURE: sp_TinhHocBongSinhVien
-- Mô tả:
-- Đoạn lệnh dùng để:
-- + Gọi thủ tục tính GPA và học bổng sinh viên
-- + Nhận dữ liệu OUTPUT
-- + Hiển thị kết quả học bổng
-- ============================================================

-- ============================================================
-- Khai báo biến lưu kết quả OUTPUT
-- ============================================================

-- Biến lưu GPA hệ 4
DECLARE @GPA DECIMAL(3,2),

        -- Biến lưu học bổng dự kiến
        @HocBong MONEY;

-- ============================================================
-- Gọi thủ tục tính học bổng sinh viên
-- ============================================================
EXEC [dbo].[sp_TinhHocBongSinhVien]

    -- Mã sinh viên cần kiểm tra
    @MaSV = 1,

    -- Nhận GPA từ OUTPUT
    @GPA_Thang4 = @GPA OUTPUT,

    -- Nhận học bổng từ OUTPUT
    @HocBongDuKien = @HocBong OUTPUT;

-- ============================================================
-- Hiển thị kết quả sau khi thực thi procedure
-- ============================================================
SELECT

    -- GPA hệ 4 của sinh viên
    @GPA AS GPA_Thang4,

    -- Học bổng dự kiến
    @HocBong AS HocBongDuKien,

    -- Đưa ra kết luận đủ hay không đủ điều kiện học bổng
    CASE

        -- Nếu học bổng lớn hơn 0
        WHEN @HocBong > 0 
            THEN N'Du dieu kien nhan hoc bong'

        -- Không đủ điều kiện
        ELSE N'Khong du dieu kien nhan hoc bong'

    END AS KetLuan;
```

<img width="1917" height="1079" alt="image" src="https://github.com/user-attachments/assets/24205a37-783d-48b7-8de9-0fba464583ee" />

> **Chú thích ảnh:** Ảnh này cho thấy procedure đã tính GPA và học bổng dự kiến qua tham số OUTPUT.

## 3.4 Stored Procedure 3: Trả về result set sau khi join nhiều bảng

### Bài toán đặt ra

**Yêu cầu:** Viết procedure lập báo cáo bảng điểm chi tiết của một sinh viên, join đầy đủ thông tin.

**Phân tích logic:**
Procedure join 4 bảng SinhVien, LopHoc, Diem, MonHoc để tạo báo cáo chi tiết điểm số, GPA từng môn và trạng thái đạt/không đạt.

```sql
-- ============================================================
-- PROCEDURE: sp_BaoCaoBangDiemSinhVien
-- Mô tả:
-- Thủ tục dùng để xuất báo cáo bảng điểm chi tiết
-- của một sinh viên, bao gồm:
-- + Thông tin sinh viên
-- + Thông tin môn học
-- + Điểm từng môn
-- + GPA từng môn
-- + Trạng thái học tập
-- ============================================================

CREATE OR ALTER PROCEDURE [dbo].[sp_BaoCaoBangDiemSinhVien]

    -- Mã sinh viên cần xem bảng điểm
    @MaSV INT

AS
BEGIN

    -- Không hiển thị số dòng ảnh hưởng
    SET NOCOUNT ON;

    -- ========================================================
    -- Truy vấn thông tin bảng điểm sinh viên
    -- ========================================================
    SELECT

        -- Mã sinh viên
        sv.[MaSV],

        -- Họ tên sinh viên
        sv.[HoTen],

        -- Tên lớp học
        lh.[TenLop],

        -- Tên môn học
        mh.[TenMH],

        -- Số tín chỉ môn học
        mh.[SoTinChi],

        -- Điểm giữa kỳ
        d.[DiemGK],

        -- Điểm cuối kỳ
        d.[DiemCK],

        -- Điểm tổng kết hệ 10
        d.[DiemTongKet],

        -- Quy đổi GPA hệ 4 cho từng môn học
        [dbo].[fn_TinhGPA](d.[DiemTongKet]) AS GPA_MonHoc,

        -- Lần học môn học
        d.[LanHoc],

        -- Trạng thái đạt / không đạt
        d.[TrangThai]

    FROM [SinhVien] sv

    -- Liên kết bảng lớp học
    INNER JOIN [LopHoc] lh 
        ON sv.[MaLop] = lh.[MaLop]

    -- Liên kết bảng điểm
    LEFT JOIN [Diem] d 
        ON sv.[MaSV] = d.[MaSV]

    -- Liên kết bảng môn học
    LEFT JOIN [MonHoc] mh 
        ON d.[MaMH] = mh.[MaMH]

    -- Lọc theo mã sinh viên cần xem
    WHERE sv.[MaSV] = @MaSV

    -- Sắp xếp theo lần học và tên môn học
    ORDER BY 
        d.[LanHoc], 
        mh.[TenMH];

END;
GO
```
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/931d4e1c-d6de-4399-9cd1-6f6f9b5214da" />


### Khai thác procedure

```sql
EXEC [dbo].[sp_BaoCaoBangDiemSinhVien] @MaSV = 1;
```

<img width="1893" height="1079" alt="image" src="https://github.com/user-attachments/assets/89109aa8-5a35-4aae-b440-23e474f1fc22" />

> **Chú thích ảnh:** Ảnh này cho thấy procedure đã join 4 bảng để tạo báo cáo bảng điểm chi tiết của sinh viên.

---

# PHẦN 4: TRIGGER VÀ XỬ LÝ LOGIC NGHIỆP VỤ

## 4.1 Trigger 1: Khi thêm điểm thì tự động tính điểm tổng kết

### Bài toán đặt ra

**Yêu cầu:** Khi insert hoặc update bảng Diem, tự động tính DiemTongKet = (DiemGK * 0.3 + DiemCK * 0.7). Nếu DiemTongKet >= 5 thì TrangThai = 'Dat', ngược lại là 'KhongDat'.

**Phân tích logic:**
Trigger này tự động hóa việc tính toán điểm số, đảm bảo tính nhất quán. Không cần ứng dụng tính toán, dữ liệu luôn được cập nhật tự động.

```sql
-- ============================================================
-- TRIGGER: trg_Diem_TinhDiemTongKet
-- Mô tả:
-- Trigger tự động tính:
-- + Điểm tổng kết
-- + Trạng thái đạt / không đạt
-- sau khi thêm mới hoặc cập nhật điểm.
-- ============================================================

CREATE OR ALTER TRIGGER [trg_Diem_TinhDiemTongKet]

-- Trigger áp dụng trên bảng Diem
ON [Diem]

-- Kích hoạt sau khi INSERT hoặc UPDATE
AFTER INSERT, UPDATE

AS
BEGIN

    -- Không hiển thị số dòng ảnh hưởng
    SET NOCOUNT ON;

    -- ========================================================
    -- Cập nhật điểm tổng kết và trạng thái môn học
    -- ========================================================
    UPDATE d

    SET

        -- ====================================================
        -- Tính điểm tổng kết:
        -- 30% điểm giữa kỳ + 70% điểm cuối kỳ
        -- ====================================================
        d.[DiemTongKet] = 
            ISNULL(i.[DiemGK], 0) * 0.3 
            + ISNULL(i.[DiemCK], 0) * 0.7,

        -- ====================================================
        -- Xác định trạng thái môn học
        -- >= 5 : Đạt
        -- < 5  : Không đạt
        -- ====================================================
        d.[TrangThai] = CASE

            WHEN 
                ISNULL(i.[DiemGK], 0) * 0.3 
                + ISNULL(i.[DiemCK], 0) * 0.7 >= 5

                THEN N'Dat'

            ELSE N'KhongDat'

        END

    FROM [Diem] d

    -- Liên kết với bảng inserted
    -- inserted chứa dữ liệu vừa thêm hoặc cập nhật
    INNER JOIN inserted i 
        ON d.[MaDiem] = i.[MaDiem];

END;
GO
```

<img width="1919" height="1078" alt="image" src="https://github.com/user-attachments/assets/b402d986-620e-411d-b6af-2b44fffd57c2" />

Tạo trigger

### Khai thác trigger

```sql
-- Kiểm tra trước khi thêm
SELECT * FROM [Diem] WHERE [MaSV] = 3 AND [MaMH] = 2;

-- Thêm điểm mới (không cần nhập DiemTongKet, trigger sẽ tính)
INSERT INTO [Diem] ([MaSV], [MaMH], [DiemGK], [DiemCK], [LanHoc])
VALUES (3, 2, 6.0, 7.0, 1);

-- Kiểm tra sau khi thêm
SELECT * FROM [Diem] WHERE [MaSV] = 3 AND [MaMH] = 2;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/392304df-bcb1-43dc-9f8d-34aa9314a982" />


> **Chú thích ảnh:** Ảnh này cho thấy khi thêm điểm, trigger tự động tính DiemTongKet và cập nhật TrangThai.

## 4.2 Mô phỏng trigger đệ quy giữa hai bảng

### Tình huống yêu cầu đề bài

Thử tạo trigger trên bảng A cập nhật bảng B, rồi trigger trên bảng B lại cập nhật bảng A để quan sát phản ứng của hệ thống.

Ở đây em mô phỏng:
- Bảng A: `[SinhVien]`
- Bảng B: `[LopHoc]`

```sql

-- ============================================================
-- MÔ PHỎNG TRIGGER ĐỆ QUY GIỮA HAI BẢNG
-- Bảng A: SinhVien
-- Bảng B: LopHoc
-- ============================================================

-- ============================================================
-- Bật cho phép trigger đệ quy
-- ============================================================
ALTER DATABASE CURRENT 
SET RECURSIVE_TRIGGERS ON;
GO

-- ============================================================
-- Xóa trigger cũ nếu đã tồn tại
-- ============================================================
IF OBJECT_ID('trg_SinhVien_UpdateLop', 'TR') IS NOT NULL
    DROP TRIGGER trg_SinhVien_UpdateLop;
GO

IF OBJECT_ID('trg_LopHoc_UpdateSinhVien', 'TR') IS NOT NULL
    DROP TRIGGER trg_LopHoc_UpdateSinhVien;
GO

-- ============================================================
-- TRIGGER 1
-- Khi cập nhật SinhVien thì cập nhật sĩ số lớp
-- ============================================================
CREATE TRIGGER trg_SinhVien_UpdateLop
ON [SinhVien]
AFTER UPDATE
AS
BEGIN

    PRINT N'Trigger 1 dang chay...';

    -- --------------------------------------------------------
    -- Giảm sĩ số lớp cũ
    -- --------------------------------------------------------
    UPDATE lh
    SET lh.[SiSo] = lh.[SiSo] - 1

    FROM [LopHoc] lh
    INNER JOIN deleted d 
        ON lh.[MaLop] = d.[MaLop];

    -- --------------------------------------------------------
    -- Tăng sĩ số lớp mới
    -- --------------------------------------------------------
    UPDATE lh
    SET lh.[SiSo] = lh.[SiSo] + 1

    FROM [LopHoc] lh
    INNER JOIN inserted i 
        ON lh.[MaLop] = i.[MaLop];

END;
GO

-- ============================================================
-- TRIGGER 2
-- Khi LopHoc thay đổi thì cập nhật ngược SinhVien
-- Gây ra đệ quy trigger
-- ============================================================
CREATE TRIGGER trg_LopHoc_UpdateSinhVien
ON [LopHoc]
AFTER UPDATE
AS
BEGIN

    PRINT N'Trigger 2 dang chay...';

    -- --------------------------------------------------------
    -- Cập nhật ngược bảng SinhVien
    -- Trigger này sẽ gọi lại Trigger 1
    -- --------------------------------------------------------
    UPDATE [SinhVien]

    SET [DiemRenLuyen] = ISNULL([DiemRenLuyen], 0) + 1

    WHERE [MaSV] = 1;

END;
GO

-- ============================================================
-- TEST GÂY ĐỆ QUY
-- ============================================================
UPDATE [SinhVien]

SET [MaLop] = 2

WHERE [MaSV] = 1;
GO
```

### Nhận xét

Khi thực hiện mô hình này (bỏ comment dòng test), SQL Server phát sinh lỗi:

```
Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32).
```

### Kết luận

- Trigger cập nhật chéo giữa 2 bảng rất dễ gây vòng lặp đệ quy.
- Hệ thống sẽ dừng khi vượt quá mức nesting cho phép (32 cấp).
- Trong thực tế cần tránh thiết kế trigger cập nhật qua lại thiếu điều kiện dừng.
- Nên dùng Stored Procedure thay vì trigger nếu logic quá phức tạp.

<img width="1919" height="1078" alt="image" src="https://github.com/user-attachments/assets/bb0258be-642a-4e15-8810-95bf1a5c0058" />

> **Chú thích ảnh:** Ảnh này cho thấy trigger giữa hai bảng có thể gây vòng lặp và phát sinh lỗi lồng nhau vượt mức cho phép.

---

# PHẦN 5: CURSOR VÀ DUYỆT DỮ LIỆU

## 5.1 Bài toán dùng CURSOR

**Yêu cầu:** Duyệt từng sinh viên, tính GPA và in cảnh báo học vụ riêng cho từng người.

**Phân tích bài toán:**
Đây là bài toán phù hợp để minh họa CURSOR vì cần xử lý từng bản ghi một cách tuần tự, in thông báo riêng biệt cho từng sinh viên dựa trên GPA và điểm rèn luyện.

```sql
-- ============================================================
-- CURSOR: Kiểm tra tình trạng học tập sinh viên
-- Mô tả:
-- Cursor dùng để duyệt lần lượt từng sinh viên,
-- tính GPA và đưa ra cảnh báo học vụ hoặc
-- thông báo học bổng tương ứng.
-- ============================================================

-- ============================================================
-- Khai báo các biến sử dụng trong Cursor
-- ============================================================

-- Mã sinh viên
DECLARE
    @MaSV INT,

    -- Họ tên sinh viên
    @HoTen NVARCHAR(150),

    -- GPA hệ 4
    @GPA DECIMAL(3,2),

    -- Điểm rèn luyện
    @DiemRenLuyen INT,

    -- Nội dung cảnh báo
    @CanhBao NVARCHAR(200);

-- ============================================================
-- Khai báo Cursor duyệt danh sách sinh viên
-- ============================================================
DECLARE cur_SinhVien CURSOR

FOR

SELECT 
    -- Mã sinh viên
    sv.[MaSV],

    -- Họ tên sinh viên
    sv.[HoTen],

    -- Điểm rèn luyện
    sv.[DiemRenLuyen]

FROM [SinhVien] sv;

-- ============================================================
-- Mở Cursor
-- ============================================================
OPEN cur_SinhVien;

-- ============================================================
-- Lấy dòng dữ liệu đầu tiên
-- ============================================================
FETCH NEXT FROM cur_SinhVien 
INTO @MaSV, @HoTen, @DiemRenLuyen;

-- ============================================================
-- Lặp đến khi duyệt hết dữ liệu
-- ============================================================
WHILE @@FETCH_STATUS = 0
BEGIN

    -- ========================================================
    -- Tính GPA trung bình hệ 4
    -- Chỉ tính các môn đạt
    -- ========================================================
    SELECT 
        @GPA = AVG([dbo].[fn_TinhGPA]([DiemTongKet]))

    FROM [Diem]

    WHERE [MaSV] = @MaSV 
        AND [TrangThai] = N'Dat';

    -- Nếu GPA NULL thì gán mặc định bằng 0
    IF @GPA IS NULL 
        SET @GPA = 0;

    -- ========================================================
    -- Xác định cảnh báo học vụ hoặc học bổng
    -- ========================================================

    -- GPA quá thấp
    IF @GPA < 1.0

        SET @CanhBao = 
            N'CANH BAO: GPA qua thap, can cai thien ngay!';

    -- Điểm rèn luyện thấp
    ELSE IF @DiemRenLuyen < 60

        SET @CanhBao = 
            N'CANH BAO: Diem ren luyen thap!';

    -- Đủ điều kiện học bổng loại A
    ELSE IF @GPA >= 3.6 
        AND @DiemRenLuyen >= 90

        SET @CanhBao = 
            N'CHUC MUNG: Du dieu kien hoc bong loai A!';

    -- Trạng thái bình thường
    ELSE

        SET @CanhBao = N'Binh thuong';

    -- ========================================================
    -- Hiển thị thông tin sinh viên và kết quả đánh giá
    -- ========================================================
    PRINT 
        N'Ma SV: ' + CAST(@MaSV AS NVARCHAR)

        + N' | Ho ten: ' + @HoTen

        + N' | GPA: ' + CAST(@GPA AS NVARCHAR)

        + N' | Diem RL: ' + CAST(@DiemRenLuyen AS NVARCHAR)

        + N' | ' + @CanhBao;

    -- ========================================================
    -- Lấy dòng dữ liệu tiếp theo
    -- ========================================================
    FETCH NEXT FROM cur_SinhVien 
    INTO @MaSV, @HoTen, @DiemRenLuyen;

END;

-- ============================================================
-- Đóng Cursor
-- ============================================================
CLOSE cur_SinhVien;

-- ============================================================
-- Giải phóng bộ nhớ Cursor
-- ============================================================
DEALLOCATE cur_SinhVien;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/eec16879-815e-4a1e-9588-cb3365bbc6f7" />

> **Chú thích ảnh:** Ảnh này cho thấy CURSOR duyệt từng sinh viên, tính GPA và in cảnh báo riêng cho từng người.

## 5.2 Giải bài toán không dùng CURSOR

Cùng bài toán trên, có thể giải bằng truy vấn set-based như sau:

```sql
-- ============================================================
-- TRUY VẤN: Đánh giá tình trạng học tập sinh viên
-- Mô tả:
-- Truy vấn dùng để:
-- + Tính GPA trung bình của sinh viên
-- + Kiểm tra điểm rèn luyện
-- + Đưa ra cảnh báo học vụ hoặc thông báo học bổng
-- ============================================================

SELECT

    -- Mã sinh viên
    sv.[MaSV],

    -- Họ tên sinh viên
    sv.[HoTen],

    -- Điểm rèn luyện
    sv.[DiemRenLuyen],

    -- ========================================================
    -- Tính GPA trung bình hệ 4
    -- Chỉ tính các môn đạt
    -- Nếu chưa có điểm thì mặc định bằng 0
    -- ========================================================
    ISNULL(
        AVG([dbo].[fn_TinhGPA](d.[DiemTongKet])), 
        0
    ) AS GPA,

    -- ========================================================
    -- Đưa ra cảnh báo học vụ hoặc thông báo học bổng
    -- ========================================================
    CASE

        -- GPA quá thấp
        WHEN ISNULL(
                AVG([dbo].[fn_TinhGPA](d.[DiemTongKet])), 
                0
             ) < 1.0

            THEN N'CANH BAO: GPA qua thap!'

        -- Điểm rèn luyện thấp
        WHEN sv.[DiemRenLuyen] < 60

            THEN N'CANH BAO: Diem ren luyen thap!'

        -- Đủ điều kiện học bổng loại A
        WHEN ISNULL(
                AVG([dbo].[fn_TinhGPA](d.[DiemTongKet])), 
                0
             ) >= 3.6

             AND sv.[DiemRenLuyen] >= 90

            THEN N'CHUC MUNG: Du dieu kien hoc bong loai A!'

        -- Trạng thái bình thường
        ELSE N'Binh thuong'

    END AS CanhBao

FROM [SinhVien] sv

-- ============================================================
-- Liên kết bảng điểm
-- Chỉ lấy các môn có trạng thái đạt
-- ============================================================
LEFT JOIN [Diem] d 
    ON sv.[MaSV] = d.[MaSV] 
    AND d.[TrangThai] = N'Dat'

-- ============================================================
-- Gom nhóm theo từng sinh viên
-- ============================================================
GROUP BY 
    sv.[MaSV], 
    sv.[HoTen], 
    sv.[DiemRenLuyen];
```


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/f0545276-78d3-456d-b93a-a8d12165a744" />

> **Chú thích ảnh:** Ảnh này cho thấy cùng bài toán có thể xử lý theo kiểu set-based mà không cần duyệt từng dòng.

## 5.3 So sánh tốc độ giữa CURSOR và không dùng CURSOR

Để so sánh, có thể bật thống kê thời gian:

```sql
SET STATISTICS TIME ON;

-- Chạy truy vấn dùng CURSOR (đã viết ở trên)
-- Chạy truy vấn không dùng CURSOR (đã viết ở trên)

SET STATISTICS TIME OFF;
```

### Nhận xét

| Tiêu chí | CURSOR | Set-Based |
|---|---|---|
| Cách xử lý | Duyệt từng dòng | Xử lý cả tập dữ liệu |
| Tốc độ | Chậm hơn với dữ liệu lớn | Nhanh hơn |
| Tài nguyên | Tốn nhiều bộ nhớ | Tối ưu hơn |
| Phù hợp | Logic tuần tự phức tạp, gửi mail | Thống kê, báo cáo |


<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/2e6c0327-d02a-42ae-93bd-b148a2a79678" />

> **Chú thích ảnh:** Ảnh này cho thấy phương pháp set-based có thời gian xử lý tốt hơn so với CURSOR trong cùng một bài toán thống kê.

## 5.4 Bài toán chỉ CURSOR mới giải quyết tốt

**Bài toán:** Duyệt từng sinh viên có GPA thấp để gửi thông báo riêng qua email hoặc sinh nội dung thông báo cá nhân hóa.

**Phân tích:**
Đây là tình huống CURSOR gần như bắt buộc vì:
- Mỗi sinh viên có nội dung email khác nhau.
- Cần gọi thủ tục gửi mail cho từng người (`sp_send_dbmail`).
- Cần xử lý tuần tự, dễ kiểm soát lỗi từng lần gửi.

```sql
-- ============================================================
-- CURSOR: Gửi email cảnh báo học vụ sinh viên
-- Mô tả:
-- Cursor dùng để:
-- + Duyệt danh sách sinh viên có môn không đạt
-- + Tính GPA trung bình
-- + Tạo nội dung email cảnh báo riêng
-- + Giả lập gửi email bằng Database Mail
-- ============================================================

-- ============================================================
-- Khai báo các biến sử dụng trong Cursor
-- ============================================================

DECLARE

    -- Mã sinh viên
    @MaSV INT,

    -- Họ tên sinh viên
    @HoTen NVARCHAR(150),

    -- Email sinh viên
    @Email VARCHAR(100),

    -- GPA trung bình hệ 4
    @GPA DECIMAL(3,2),

    -- Nội dung email cảnh báo
    @NoiDungEmail NVARCHAR(MAX);

-- ============================================================
-- Khai báo Cursor duyệt sinh viên có môn không đạt
-- ============================================================
DECLARE cur_GuiCanhBao CURSOR

FOR

SELECT

    -- Mã sinh viên
    sv.[MaSV],

    -- Họ tên sinh viên
    sv.[HoTen],

    -- Email sinh viên
    sv.[Email]

FROM [SinhVien] sv

-- ============================================================
-- Chỉ lấy sinh viên có ít nhất một môn không đạt
-- ============================================================
WHERE EXISTS (

    SELECT 1 

    FROM [Diem] d

    WHERE d.[MaSV] = sv.[MaSV]
        AND d.[TrangThai] = N'KhongDat'
);

-- ============================================================
-- Mở Cursor
-- ============================================================
OPEN cur_GuiCanhBao;

-- ============================================================
-- Lấy dòng dữ liệu đầu tiên
-- ============================================================
FETCH NEXT FROM cur_GuiCanhBao 
INTO @MaSV, @HoTen, @Email;

-- ============================================================
-- Lặp đến khi duyệt hết dữ liệu
-- ============================================================
WHILE @@FETCH_STATUS = 0
BEGIN

    -- ========================================================
    -- Tính GPA trung bình hệ 4
    -- Chỉ tính các môn đạt
    -- ========================================================
    SELECT 
        @GPA = AVG([dbo].[fn_TinhGPA]([DiemTongKet]))

    FROM [Diem]

    WHERE [MaSV] = @MaSV 
        AND [TrangThai] = N'Dat';

    -- Nếu GPA NULL thì gán mặc định bằng 0
    IF @GPA IS NULL 
        SET @GPA = 0;

    -- ========================================================
    -- Tạo nội dung email cảnh báo học vụ
    -- ========================================================
    SET @NoiDungEmail = 

        N'Kinh gui ' + @HoTen + N',' 

        + CHAR(13) + CHAR(10)

        + N'He thong ghi nhan ban co GPA: ' 
        + CAST(@GPA AS NVARCHAR)

        + CHAR(13) + CHAR(10)

        + N'Va co mon chua dat. Vui long lien he van phong khoa de duoc tu van.'

        + CHAR(13) + CHAR(10)

        + N'Tran trong.';

    -- ========================================================
    -- Giả lập gửi email bằng Database Mail
    -- Cần cấu hình Database Mail để sử dụng thực tế
    -- ========================================================

    -- EXEC msdb.dbo.sp_send_dbmail
    --     @profile_name = 'SinhVienMailProfile',
    --     @recipients = @Email,
    --     @subject = 'Canh bao hoc vu',
    --     @body = @NoiDungEmail;

    -- ========================================================
    -- Hiển thị thông báo gửi email thành công
    -- ========================================================
    PRINT 

        N'Da gui email canh bao den: ' 

        + @HoTen 

        + N' (' + @Email + N')';

    -- ========================================================
    -- Lấy dòng dữ liệu tiếp theo
    -- ========================================================
    FETCH NEXT FROM cur_GuiCanhBao 
    INTO @MaSV, @HoTen, @Email;

END;

-- ============================================================
-- Đóng Cursor
-- ============================================================
CLOSE cur_GuiCanhBao;

-- ============================================================
-- Giải phóng bộ nhớ Cursor
-- ============================================================
DEALLOCATE cur_GuiCanhBao;
```

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/5c6e5ea7-fcc0-4719-b072-59849558806a" />

> **Chú thích ảnh:** Ảnh này cho thấy CURSOR duyệt từng sinh viên có cảnh báo để gửi email riêng biệt - bài toán chỉ CURSOR giải quyết tốt.

