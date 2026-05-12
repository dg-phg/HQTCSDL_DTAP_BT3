# HQTCSDL_DTAP_BT3
## Thông tin sinh viên:
+ **Họ và tên:** Dương Thị Anh Phương
+ **Mã sinh viên:** K235480106056
+ **Lớp:** K235480106056
+ **Trường:** Đại học Kỹ thuật Công nghiệp Thái Nguyên
---
## BÀI TẬP 3
### Nhiệm vụ 1: Thiết kế CSDL
Vẽ sơ đồ ERD: thể hiện rõ thực thể, thuộc tính, khóa chính, khóa ngoại

+ Tạo Database: `QuanLyCamDo_K235480106056`
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/754010f4-85ba-4267-aa6e-81380b52a840" />

+ Tạo bảng khách hàng
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/383e8e4f-76b9-4716-95e6-c0b60f97372c" />

+ Tạo bảng hợp đồng
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/228e5e1b-e88b-488a-8ec8-6b5b10ea8cd2" />

+ Tạo bảng tài sản
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7f226d89-56b5-427b-ac7e-456cce653cc7" />

+ Tạo bảng giao dịch
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/980761f1-eeaf-4841-8e02-c16097de0c29" />

+ Sơ đồ ERD
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e9264fea-d4b8-422a-a82d-197b6f6a9fb0" />

CODE 
```sql
-- 1. Tạo Database (Đổi tên theo đúng mã SV của bạn nếu cần)
CREATE DATABASE QuanLyCamDo_K235480106056;
GO
USE QuanLyCamDo_K235480106056;
GO

-- 2. Tạo bảng Khách Hàng
CREATE TABLE KhachHang (
    MaKhachHang INT IDENTITY(1,1) PRIMARY KEY,
    HoTen NVARCHAR(100) NOT NULL,
    SoDienThoai VARCHAR(15) NOT NULL,
    CCCD VARCHAR(20) UNIQUE NOT NULL,
    DiaChi NVARCHAR(200)
);
GO

-- 3. Tạo bảng Hợp Đồng
CREATE TABLE HopDong (
    MaHopDong INT IDENTITY(1,1) PRIMARY KEY,
    MaKhachHang INT NOT NULL,
    NgayLap DATE NOT NULL DEFAULT GETDATE(),
    SoTienGoc DECIMAL(18,2) NOT NULL CHECK (SoTienGoc > 0),
    Deadline1 DATE NOT NULL,
    Deadline2 DATE NOT NULL,
    TrangThai NVARCHAR(50) DEFAULT N'Đang vay' 
        CHECK (TrangThai IN (N'Đang vay', N'Quá hạn (nợ xấu)', N'Đã thanh toán', N'Đã thanh lý')),
    CONSTRAINT FK_HopDong_KhachHang FOREIGN KEY (MaKhachHang) REFERENCES KhachHang(MaKhachHang)
);
GO

-- 4. Tạo bảng Tài Sản
CREATE TABLE TaiSan (
    MaTaiSan INT IDENTITY(1,1) PRIMARY KEY,
    MaHopDong INT NOT NULL,
    TenTaiSan NVARCHAR(100) NOT NULL,
    GiaTriDinhGia DECIMAL(18,2) NOT NULL CHECK (GiaTriDinhGia > 0),
    TrangThai NVARCHAR(50) DEFAULT N'Đang cầm cố'
        CHECK (TrangThai IN (N'Đang cầm cố', N'Đã trả khách', N'Sẵn sàng thanh lý', N'Đã bán thanh lý')),
    CONSTRAINT FK_TaiSan_HopDong FOREIGN KEY (MaHopDong) REFERENCES HopDong(MaHopDong)
);
GO

-- 5. Tạo bảng Lịch sử giao dịch (Audit Log)
CREATE TABLE Log_GiaoDich (
    MaLog INT IDENTITY(1,1) PRIMARY KEY,
    MaHopDong INT NOT NULL,
    NgayTra DATETIME DEFAULT GETDATE(),
    SoTienTra DECIMAL(18,2) NOT NULL CHECK (SoTienTra > 0),
    NguoiThuTien NVARCHAR(100) NOT NULL,
    GhiChu NVARCHAR(200),
    CONSTRAINT FK_Log_HopDong FOREIGN KEY (MaHopDong) REFERENCES HopDong(MaHopDong)
);
GO
```

### Nhiệm vụ 2: Cài đặt SQL (Yêu cầu viết Scripts)

#### Event 1: Đăng ký hợp đồng mới _(Vay tiền)_
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f824fe12-e104-4ced-b781-1f57b7d1e3d3" />

```
CREATE PROCEDURE sp_DangKyHopDong
    @HoTen NVARCHAR(100),
    @SĐT VARCHAR(15),
    @CCCD VARCHAR(20),
    @SoTienVay DECIMAL(18,2),
    @NgayLap DATE,
    @TenTaiSan NVARCHAR(100),
    @GiaTriTaiSan DECIMAL(18,2)
AS
BEGIN
    DECLARE @MaKH INT;
    
    -- 1. Kiểm tra khách hàng đã tồn tại chưa, nếu chưa thì thêm mới
    IF NOT EXISTS (SELECT 1 FROM KhachHang WHERE CCCD = @CCCD)
    BEGIN
        INSERT INTO KhachHang (HoTen, SoDienThoai, CCCD) VALUES (@HoTen, @SĐT, @CCCD);
    END
    SELECT @MaKH = MaKhachHang FROM KhachHang WHERE CCCD = @CCCD;

    -- 2. Thêm hợp đồng và tính Deadline (Ví dụ D1 là 30 ngày, D2 là 60 ngày sau khi lập)
    INSERT INTO HopDong (MaKhachHang, NgayLap, SoTienGoc, Deadline1, Deadline2, TrangThai)
    VALUES (@MaKH, @NgayLap, @SoTienVay, DATEADD(day, 30, @NgayLap), DATEADD(day, 60, @NgayLap), N'Đang vay');

    -- 3. Thêm tài sản thế chấp
    DECLARE @MaHD INT = SCOPE_IDENTITY();
    INSERT INTO TaiSan (MaHopDong, TenTaiSan, GiaTriDinhGia, TrangThai)
    VALUES (@MaHD, @TenTaiSan, @GiaTriTaiSan, N'Đang cầm cố');

    PRINT N'Đã đăng ký hợp đồng thành công cho khách hàng: ' + @HoTen;
END;

```
#### Event 2: Tính toán công nợ thời gian thực

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/217e25b6-843a-43eb-ba75-bf62684a4a6d" />

```
CREATE FUNCTION fn_CalcMoneyContract (@MaHD INT, @TargetDate DATE)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @Goc DECIMAL(18,2), @D1 DATE, @NgayLap DATE, @TongNo DECIMAL(18,2);
    DECLARE @SoNgayDon INT, @SoNgayKep INT;

    SELECT @Goc = SoTienGoc, @D1 = Deadline1, @NgayLap = NgayLap 
    FROM HopDong WHERE MaHopDong = @MaHD;

    -- Trường hợp 1: Chưa quá Deadline 1 (Chỉ tính lãi đơn)
    IF @TargetDate <= @D1
    BEGIN
        SET @SoNgayDon = DATEDIFF(day, @NgayLap, @TargetDate);
        SET @TongNo = @Goc + (@Goc * 0.005 * @SoNgayDon);
    END
    -- Trường hợp 2: Đã qua Deadline 1 (Tính lãi đơn tới D1, sau đó tính lãi kép)
    ELSE
    BEGIN
        -- Tính tổng gốc + lãi đơn tại thời điểm D1
        SET @SoNgayDon = DATEDIFF(day, @NgayLap, @D1);
        DECLARE @NoTaiD1 DECIMAL(18,2) = @Goc + (@Goc * 0.005 * @SoNgayDon);
        
        -- Tính lãi kép từ sau D1 đến TargetDate (Lãi 0.5%/ngày trên tổng nợ cũ)
        SET @SoNgayKep = DATEDIFF(day, @D1, @TargetDate);
        SET @TongNo = @NoTaiD1 * POWER(1 + 0.005, @SoNgayKep);
    END

    RETURN @TongNo;
END;
```
TESS

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/adc55cf5-de1a-4ef4-80b0-f319c8d02c33" />

```
-- Test 1: Đăng ký hợp đồng (Vay 10 triệu ngày 01/01/2026)
EXEC sp_DangKyHopDong N'Lê Hoàng Cường', '0912345678', '123456789', 10000000, '2026-01-01', N'Xe Honda Vision', 25000000;

-- Test 2: Tính tiền nợ sau 10 ngày (Chỉ lãi đơn)
SELECT dbo.fn_CalcMoneyContract(1, '2026-01-11') AS TienNo_Sau10Ngay;

-- Test 3: Tính tiền nợ sau 40 ngày (Đã sang lãi kép - Vì D1 là 30 ngày)
SELECT dbo.fn_CalcMoneyContract(1, '2026-02-10') AS TienNo_Sau40Ngay;

```

#### Event 3: Xử lý trả nợ và hoàn trả tài sản
```sql
CREATE PROCEDURE sp_XuLyTraNo
    @MaHopDong INT,
    @SoTienKhachTra DECIMAL(18,2),
    @NguoiThuTien NVARCHAR(100)
AS
BEGIN
    SET NOCOUNT ON;

    -- 1. Kiểm tra nếu tài sản đã bị thanh lý
    IF EXISTS (SELECT 1 FROM TaiSan WHERE MaHopDong = @MaHopDong AND TrangThai = N'Đã bán thanh lý')
    BEGIN
        PRINT N'Thông báo: Tài sản đã bị thanh lý sau Deadline 2. Hệ thống không thu tiền và không trả đồ.';
        RETURN;
    END

    -- 2. Tính tổng nợ hiện tại (Gốc + Lãi) bằng Function đã tạo ở bước trước
    DECLARE @TongNoHienTai DECIMAL(18,2);
    SET @TongNoHienTai = dbo.fn_CalcMoneyContract(@MaHopDong, GETDATE());

    -- 3. Ghi nhận giao dịch vào bảng Log
    INSERT INTO Log_GiaoDich (MaHopDong, NgayTra, SoTienTra, NguoiThuTien, GhiChu)
    VALUES (@MaHopDong, GETDATE(), @SoTienKhachTra, @NguoiThuTien, 
            N'Khách trả tiền. Tổng nợ lúc trả: ' + CAST(@TongNoHienTai AS VARCHAR));

    -- 4. Xử lý logic thanh toán và cập nhật trạng thái
    DECLARE @DuNoConLai DECIMAL(18,2) = @TongNoHienTai - @SoTienKhachTra;

    IF @DuNoConLai <= 0
    BEGIN
        -- Trường hợp khách trả hết nợ hoặc trả dư
        UPDATE HopDong SET TrangThai = N'Đã thanh toán đủ' WHERE MaHopDong = @MaHopDong;
        UPDATE TaiSan SET TrangThai = N'Đã trả khách' WHERE MaHopDong = @MaHopDong;
        PRINT N'Kết quả: Khách đã thanh toán đủ. Đã trả toàn bộ tài sản.';
    END
    ELSE
    BEGIN
        -- Trường hợp khách mới trả được một phần
        UPDATE HopDong SET TrangThai = N'Đang trả góp' WHERE MaHopDong = @MaHopDong;
        PRINT N'Kết quả: Đã cập nhật trạng thái Đang trả góp. Số tiền còn nợ: ' + CAST(@DuNoConLai AS VARCHAR);

        -- 5. Đưa ra danh sách gợi ý trả lại tài sản (Quy tắc: Giá trị đồ còn lại >= Dư nợ)
        PRINT N'--- DANH SÁCH TÀI SẢN GỢI Ý CÓ THỂ TRẢ LẠI ---';
        
        DECLARE @TongGiaTriTaiSanDangGiu DECIMAL(18,2);
        SELECT @TongGiaTriTaiSanDangGiu = SUM(GiaTriDinhGia) 
        FROM TaiSan WHERE MaHopDong = @MaHopDong AND TrangThai = N'Đang cầm cố';

        SELECT TenTaiSan, GiaTriDinhGia, N'Có thể trả' AS GoiY
        FROM TaiSan
        WHERE MaHopDong = @MaHopDong 
          AND TrangThai = N'Đang cầm cố'
          AND (@TongGiaTriTaiSanDangGiu - GiaTriDinhGia) >= @DuNoConLai;
    END
END;
GO
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/466f319c-1102-4c4c-9279-4ff835f3e252" />

+ Test sp
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/35f282f0-54e0-4d32-b2fa-7b784d8b83ea" />

#### Event 4: Truy vấn danh sách nợ xấu (Nợ khó đòi)

```sql
-- Tạo View để quản lý danh sách nợ xấu
CREATE VIEW v_DanhSachNoXau AS
SELECT 
    kh.HoTen AS [Tên Khách Hàng], 
    kh.SoDienThoai AS [Số Điện Thoại], 
    hd.SoTienGoc AS [Tiền Gốc], 
    -- Tính số ngày quá hạn kể từ Deadline 1
    DATEDIFF(day, hd.Deadline1, GETDATE()) AS [Số Ngày Quá Hạn],
    
    -- Gọi hàm tính tiền nợ đến ngày hôm nay
    dbo.fn_CalcMoneyContract(hd.MaHopDong, GETDATE()) AS [Tổng Nợ Hiện Tại],
    
    -- Gọi hàm tính tiền nợ dự kiến sau 1 tháng (30 ngày) nữa
    dbo.fn_CalcMoneyContract(hd.MaHopDong, DATEADD(month, 1, GETDATE())) AS [Dự Kiến Nợ Sau 1 Tháng]

FROM KhachHang kh
JOIN HopDong hd ON kh.MaKhachHang = hd.MaKhachHang
WHERE 
    GETDATE() > hd.Deadline1                     -- Đã quá hạn Deadline 1
    AND hd.TrangThai <> N'Đã thanh toán đủ'      -- Và chưa trả hết tiền
    AND hd.TrangThai <> N'Đã thanh lý';          -- Và chưa bán đồ thanh lý
GO
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/607172df-3737-49a2-ab0c-70c681647488" />

+ Test
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7d417b29-7d8e-49d3-ba47-f39e7e8060bb" />

#### Event 5: Quản lý thanh lý tài sản
+ Viết một Trigger tự động chuyển trạng thái hợp đồng sang "Quá hạn (nợ xấu)" sau khi hợp
đồng đang ở trạng thái "Đang vay" mà ngày vượt quá Deadline 1
```sql
CREATE TRIGGER trg_KiemTraQuaHanD1
ON HopDong
AFTER UPDATE, INSERT
AS
BEGIN
    -- Nếu ngày hiện tại > Deadline1 và đang ở trạng thái 'Đang vay'
    UPDATE HopDong
    SET TrangThai = N'Quá hạn (nợ xấu)'
    FROM HopDong hd
    JOIN inserted i ON hd.MaHopDong = i.MaHopDong
    WHERE GETDATE() > hd.Deadline1 
      AND hd.TrangThai = N'Đang vay';
END;
GO
```
<img width="1920" height="1073" alt="image" src="https://github.com/user-attachments/assets/16520c80-56ff-48ae-b0e0-81af1c6dc5c3" />

+ Viết một Trigger tự động chuyển trạng thái tài sản sang "Sẵn sàng thanh lý" sau khi hợp
đồng đang ở trạng thái "Quá hạn (nợ xấu)" mà ngày vượt quá Deadline 2.
```sql
CREATE TRIGGER trg_SanSangThanhLy
ON HopDong
AFTER UPDATE
AS
BEGIN
    -- Nếu ngày hiện tại > Deadline2 và hợp đồng đã nợ xấu
    IF EXISTS (SELECT 1 FROM inserted WHERE TrangThai = N'Quá hạn (nợ xấu)' AND GETDATE() > Deadline2)
    BEGIN
        UPDATE TaiSan
        SET TrangThai = N'Sẵn sàng thanh lý'
        FROM TaiSan ts
        JOIN inserted i ON ts.MaHopDong = i.MaHopDong
        WHERE GETDATE() > i.Deadline2;
    END
END;
GO
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/78b39c11-9b44-4233-af05-a906dc88e168" />

+ Viết một Trigger tự động chuyển trạng thái tài sản thành “Đã bán thanh lý” sau khi trạng
thái của hợp đồng chuyển sang "Đã thanh lý".
```sql
CREATE TRIGGER trg_DaBanThanhLy
ON HopDong
AFTER UPDATE
AS
BEGIN
    -- Kiểm tra nếu cột TrangThai vừa được cập nhật thành 'Đã thanh lý'
    IF EXISTS (SELECT 1 FROM inserted WHERE TrangThai = N'Đã thanh lý')
    BEGIN
        UPDATE TaiSan
        SET TrangThai = N'Đã bán thanh lý'
        FROM TaiSan ts
        JOIN inserted i ON ts.MaHopDong = i.MaHopDong;
        
        PRINT N'Hệ thống: Toàn bộ tài sản của hợp đồng này đã được chuyển sang trạng thái Đã bán thanh lý.';
    END
END;
GO
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/386e5080-b6eb-4953-ae22-3e008e7dd5fa" />

+ Sự kiện Gia hạn hợp đồng: Khách đến trả toàn bộ tiền lãi tính đến thời điểm hiện tại để dời
Deadline 1 và Deadline 2 sang một kỳ hạn mới để tránh bị tính lãi kép.
```sql
CREATE PROCEDURE sp_GiaHanHopDong
    @MaHD INT,
    @NguoiThu NVARCHAR(100)
AS
BEGIN
    SET NOCOUNT ON;
    
    -- 1. Tính số tiền lãi khách phải trả để được gia hạn (Tổng nợ - Gốc)
    DECLARE @Goc DECIMAL(18,2), @TongNoHienTai DECIMAL(18,2), @TienLai DECIMAL(18,2);
    
    SELECT @Goc = SoTienGoc FROM HopDong WHERE MaHopDong = @MaHD;
    SET @TongNoHienTai = dbo.fn_CalcMoneyContract(@MaHD, GETDATE());
    SET @TienLai = @TongNoHienTai - @Goc;

    -- 2. Ghi nhận việc trả lãi vào Log
    INSERT INTO Log_GiaoDich (MaHopDong, NgayTra, SoTienTra, NguoiThuTien, GhiChu)
    VALUES (@MaHD, GETDATE(), @TienLai, @NguoiThu, N'Trả lãi để gia hạn hợp đồng');

    -- 3. Cập nhật lại mốc thời gian của hợp đồng
    -- Dời Deadline 1 và 2 sang kỳ hạn mới (ví dụ +30 và +60 ngày kể từ hôm nay)
    UPDATE HopDong
    SET NgayLap = GETDATE(),
        Deadline1 = DATEADD(day, 30, GETDATE()),
        Deadline2 = DATEADD(day, 60, GETDATE()),
        TrangThai = N'Đang vay'
    WHERE MaHopDong = @MaHD;

    PRINT N'Gia hạn thành công! Khách đã đóng lãi: ' + CAST(@TienLai AS VARCHAR);
    PRINT N'Deadline 1 mới: ' + CAST(DATEADD(day, 30, GETDATE()) AS VARCHAR);
END;
GO
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e76815b8-8619-48ea-b4a2-d8324e73fbf4" />

+ Lịch sử hợp đồng (Audit Log): CSDL phải có bảng Log để ghi lại mỗi lần khách trả một ít
tiền (Ngày trả, số tiền trả, người thu tiền). Tránh việc chỉ ghi đè số tổng nợ khiến mất dấu
vết dòng tiền.
```sql
-- Kiểm tra lịch sử trả tiền của hợp đồng số 1
SELECT 
    l.NgayTra, 
    l.SoTienTra, 
    l.NguoiThuTien, 
    l.GhiChu
FROM Log_GiaoDich l
WHERE l.MaHopDong = 1  -- Thay @MaHD bằng số 1 (hoặc số ID hợp đồng bạn đã tạo)
ORDER BY l.NgayTra DESC;
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/376fff81-7018-4b8b-a924-4a899554587b" />







