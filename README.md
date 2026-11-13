# **Version 1.0.25**

# I. MANUAL PAIR

App mở cổng của chính mình (`local_port`) là **5001**, listen ở địa chỉ: **192.168.4.1**, cổng **5000**

## APP (local_port: 5001) → DEVICE (local_port: 5000)

```json
{
  "name": "CmdSetManualConfig", // Lệnh cấu hình thủ công
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Phải đúng địa chỉ MAC
  "broker": "broker.hivemq.com:8883", // Broker và port của thiết bị
  "username": "mqtt", // Tên user mqtt
  "password": "mqtt@1883", // Mật khẩu user mqtt
  "mqttsub": "/SFDSmart/device/1039/cmd/d421b0d4", // Topic sub
  "mqttpub": "/SFDSmart/device/1039/data/81286cd5", // Topic pub
  "mqttalert": "/SFDSmart/device/1039/cloud/24c6db3e", // Topic alert
  "mqttstream": "/SFDSmart/device/1039/stream/925fd64f", // Không quan tâm
  "wifissid": "SFD.JSC", // Tên wifi cần truy cập
  "wifipassword": "123456789@" // Mật khẩu wifi cần truy cập
}
```

##### TEST

```json
{
  "name": "CmdSetManualConfig",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
  "broker": "broker.hivemq.com:8883",
  "username": "mqtt",
  "password": "mqtt@1883",
  "mqttsub": "/SFDSmart/device/1039/cmd/d421b0d4",
  "mqttpub": "/SFDSmart/device/1039/data/81286cd5",
  "mqttalert": "/SFDSmart/device/1039/cloud/24c6db3e",
  "mqttstream": "/SFDSmart/device/1039/stream/925fd64f",
  "wifissid": "SFD.JSC",
  "wifipassword": "123456789@"
}
```

## DEVICE → APP (Phản hồi)

```json
{
  "name": "CmdSetManualConfig", // Phản hồi lệnh cấu hình
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Địa chỉ MAC của thiết bị
  "status": 1, // 0: Đang thử kết nối, 1: thành công, 2: thất bại
  "software_version": "1.0.25" // phiên bản của thiết bị
}
```

---

# II. DATA


App cần kết nối đến mqtt broker với các thông số ở trên.
App cần subscribe topic `"mqttpub"` đã pair trước đó, ví dụ: `/SFDSmart/device/1039/data/81286cd5`
Thiết bị chỉ phản hồi khi người dùng tiến hành lấy nước (tức cảm biến lưu lượng hoạt động)

## DEVICE → APP (Tự động gửi/phản hồi)

```json
{
  "name": "CmdGetData", // Lệnh lấy dữ liệu
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Địa chỉ MAC của thiết bị
  "data":
  {
    "state": 1, // Đang hoạt động
    "mute": 0, // Trạng thái còi báo trên thiết bị
    "flowRate": 1.12, // Lưu lượng
    "tds": 1, // Giá trị TDS
    "filter_amount": 1, // Số lượng lõi lọc
    "filter_name": ["PP1"], // Tên lõi là mảng
    "capacitance": [7200], // Dung tích lõi lọc tối đa (lít)
    "filter_value": [6], // Dung tích đã sử dụng (lít)
    "filter_date": [0], // Thời gian lõi lọc đã sử dụng (ngày)
    "expiration": [360] // Thời gian lõi lọc hết hạn (ngày)
  }
}
```

## DEVICE → APP [Tắt tiếng còi báo] (Gửi thủ công)

Để ngắt tiếng còi

```json
{
  "name": "CmdSetData", // Lệnh lấy dữ liệu
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Địa chỉ MAC của thiết bị
  "data":
  {
    "mute": 0 // 1: ngắt tiếng còi cảnh báo, 0: vẫn để còi cảnh báo trên thiết bị
  }
}
```

##### TEST

```json
{
  "name": "CmdSetData",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
  "data":
  {
    "mute": 0
  }
}
```

## APP → DEVICE [RESET Lõi lọc] (Gửi thủ công)

Để reset lõi lọc từ app, gửi lệnh có cấu trúc như ở dưới:

```json
{
  "name": "CmdResetFilter", // Lệnh đặt lại lõi lọc
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Phải đúng địa chỉ MAC
  "data":
  {
    "unix_time": 1794303810, // Thời gian unix timestamp hiện hành tại thời điểm đặt lại. lấy từ backend hoặc app. cần thiết vì thiết bị dựa theo nó để tính thời gian hết lõi lọc
    "reset_idx": 0 // Lõi cần reset [0..9]
  }
}
```

##### TEST

```json
{
  "name": "CmdResetFilter",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
  "data":
  {
    "unix_time": 1794303810,
    "reset_idx": 0
  }
}
```

Lưu ý: Các lệnh ở trên thì thiết bị sẽ gửi bản tin `CmdGetData` phản hồi.

---

# III. DISTRIBUTOR

Gửi lệnh sau để lấy thông tin nhà phân phối

### APP → DEVICE [Lấy thông tin nhà phân phối] (Gửi thủ công)

```json
{
  "name": "CmdGetDistributor", // Lệnh lấy thông tin NPP
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Phải đúng địa chỉ MAC
}
```

##### TEST

```json
{
  "name": "CmdGetDistributor",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
}
```

### APP -> DEVICE (Phản hồi)

```json
{
  "name": "CmdGetDistributor", // Lệnh lấy thông tin NPP
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Địa chỉ MAC của thiết bị
  "data":
  {
    "distributor": "test", // Tên NPP
    "phone": "0909090909", // SĐT NPP
    "address": "test", // Địa chỉ NPP
    "email": "test", // Email NPP
    "userPhone": "0909090909", // SĐT người dùng
    "userAddress": "test", // Địa chỉ người dùng
    "userName": "test" // Tên người dùng
  }
}
```

---

# IV. ALERT [THÔNG BÁO APP]

Khi có cảnh báo hết lõi lọc, thiết bị tự động gửi bản tin `CmdGetAlert` vào topic `mqttalert` (app cần subcribre topic này) lập tức (chỉ 1 lần đầu tiên) hoặc vào các thời điểm 08:05, 12:00, 17:05, 22:00 mỗi ngày. với trạng thái đã thay lõi `"replaceFilter": 0` thời gian trễ thông báo có thể lên đến vài phút:

```json
{
  "name": "CmdGetAlert", // Lệnh cảnh báo
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Địa chỉ MAC của thiết bị
  "data":
  {
    "replaceFilter": 1, // 1: Cần thay thế lõi lọc, 0: Có lõi đã đc thay thế. Giúp thông báo trên app
    "distributor": "test", // Tên NPP
    "phone": "0909090909", // SĐT NPP
    "address": "test", // Địa chỉ NPP
    "email": "test",  // Email NPP
    "userPhone": "0909090909", // SĐT người dùng
    "userAddress": "test", // Địa chỉ người dùng
    "userName": "test", // Tên người dùng
    "nameFilter": "PP2,PP2,PP3," // Tên các lõi cần thay,
    // Gửi kèm thông tin nhà PP để hiển thị dialog cảnh báo trên App
  }
}
```

---

# V. OTA

Thiết bị cần kết nối Wifi có truy cập internet để thực hiện việc này

Backend là nơi lưu trữ tệp firmware và quản lý phiên bản, danh sách địa chỉ MAC hỗ trợ cập nhật. Thiết bị chỉ cập nhật hoặc lên lịch cập nhật khi có bản tin gửi đến. Do đó nên gửi lại mỗi ngày 1 lần với các thiết bị cập nhật thất bại.

## 1. Cập nhật thiết bị ngay lập tức

Cập nhật ngay lập tức nên dùng để update thủ công cho từng thiết bị, có thể dùng cho hàng loạt nhưng sẽ gây rắc rối nếu người dùng đang sử dụng thiết bị.

### APP → DEVICE (Gửi thủ công)

```json
{
  "name": "CmdSetOTA", // Lệnh đặt OTA
  "dpCode": "GMH2071", // Mã sản phẩm
  "addr": "08F9E06E6AD0", // Phải đúng địa chỉ MAC
  "data":
  {
    "otaID": 1, // ID của OTA (Id do backend quản lý)
    "scheduleID": 1, // ID của lịch (Id do backend quản lý)
    "forceUpdate": 1, // Cập nhật bắt buộc ngay lập tức
    "firmwareUrl": "https://github.com/phatvu1294/smart-ro-ota-host/raw/refs/heads/main/firmware_v1.0.25.bin", // Đường dẫn trỏ đến file bin
    "firmwareVersion": "1.0.25", // Phiên bản update phải lớn hơn hoặc bằng phiên bản cũ (điều kiện bằng chỉ để test)
    "firmwareChecksum": "" // Không cần thiết với thiết bị, nếu làm app, có thể check để cho phép nút cập nhật bật hay không
  }
}
```

##### TEST

```json
{
  "name": "CmdSetOTA",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
  "data":
  {
    "otaID": 1,
    "scheduleID": 1,
    "forceUpdate": 1,
    "firmwareUrl": "https://github.com/phatvu1294/smart-ro-ota-host/raw/refs/heads/main/firmware_v1.0.25.bin",
    "firmwareVersion": "1.0.25",
    "firmwareChecksum": ""
  }
}
```

Thiết bị gửi lại bản tin phản hồi `ReportProcessOTA`

### DEVICE -> APP (Phản hồi)

```json
{
  "name":"ReportProcessOTA",
  "dpCode":"GMH2071",
  "addr":"08F9E06E6AD0",
  "data":
  {
    "otaID": 1, // Id OTA
    "scheduleID": 1, // ID schedule
    "status": 2 // Trạng thái cập nhật
    // status - statusCode
    // 2  - scheduled: Đã nhận cấu hình từ bản tin CmdSetOTA
    // 10 - updated: Đã cập nhật thành công
    // 12 - failed: Đã có lỗi xảy ra
  },
}
```

---

## 2. Cập nhật thiết bị theo lịch

Cập nhật thiết bị theo lịch đặt sẵn.

**Điều kiện cập nhật:** `(giờ thiết bị >= (startTime + startTimeOfDay)) && (giờ thiết bị <= (endTime + endTimeOfDay))`

Có thể đặt `startTimeOfDay = 0` và `endTimeOfDay = 0`, lúc này chỉ cần đặt `startTime` và `endTime` là đc, vì chúng là unix timestamp nên có chứa cả giờ (không khuyến nghị).

Với danh sách thiết bị được phép cập nhật, có thể có thiết bị bị tắt, cập nhật thất bại hoặc không phản hồi, lúc này backend cần chủ động gửi lại bản tin mqtt vào danh sách thất bại mỗi ngày trong thời gian đó.

**startTimeOfDay, endTimeOfDay:**
- Giá trị khả dụng trong khoảng `[0, 86399]`
- Quy đổi: `Giờ*3600 + Phút*60 + Giây`, vd: `10:00:00 => 10*3600+0*60+0=36000` (giây)

### APP → DEVICE (Gửi thủ công)

```json
{
  "name": "CmdSetOTA",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
  "data":
  {
    "otaID": 1, // ID của OTA (Id do backend quản lý)
    "scheduleID": 1, // ID của lịch (Id do backend quản lý)
    "forceUpdate": 0, // Cập nhật không bắt buộc theo lịch lên sẵn
    "firmwareUrl": "https://github.com/phatvu1294/smart-ro-ota-host/raw/refs/heads/main/firmware_v1.0.25.bin", // Đường dẫn trỏ đến file bin
    "firmwareVersion": "1.0.25", // Phiên bản update phải lớn hơn hoặc bằng phiên bản cũ (điều kiện bằng chỉ để test)
    "firmwareChecksum": "", // Không cần thiết với thiết bị, nếu làm app, có thể check để cho phép nút cập nhật bật hay không
    "startTime": 1794502800, // [2026/11/13, 00:00:00], thời gian bắt đầu, unix timestamp, tính bằng giây [Dùng cho UI chọn ngày]
    "endTime": 1794502800, // [2026/11/13, 00:00:00] thời gian kết thúc, unix timestamp, tính bằng giây [Dùng cho UI chọn ngày]
    "startTimeOfDay": 0, // [00:00:00] thời gian bắt đầu trong ngày, tính bằng giây, [Dùng cho UI chọn giờ]
    "endTimeOfDay": 36000 // [10:00:00] thời gian kết thúc trong ngày, tính bằng giây, [Dùng cho UI chọn giờ]
  }
}
```

##### TEST

```json
{
  "name": "CmdSetOTA",
  "dpCode": "GMH2071",
  "addr": "08F9E06E6AD0",
  "data":
  {
    "otaID": 1,
    "forceUpdate": 0,
    "firmwareUrl": "https://github.com/phatvu1294/smart-ro-ota-host/raw/refs/heads/main/firmware_v1.0.25.bin",
    "firmwareVersion": "1.0.25",
    "firmwareChecksum": "",
    "scheduleID": 1,
    "startTime": 1794547620,
    "endTime": 1794589140,
    "startTimeOfDay": 0,
    "endTimeOfDay": 0
  }
}
```

### DEVICE -> APP (Phản hồi)

```json
{
  "name":"ReportProcessOTA",
  "dpCode":"GMH2071",
  "addr":"08F9E06E6AD0",
  "data":
  {
    "otaID": 1, // Id OTA
    "scheduleID": 1, // ID schedule
    "status": 2 // Trạng thái cập nhật
    // status - statusCode
    // 2  - scheduled: Đã nhận cấu hình từ bản tin CmdSetOTA
    // 10 - updated: Đã cập nhật thành công
    // 12 - failed: Đã có lỗi xảy ra
  },
}
```

---

# VI. GET ID

Lấy id thiết bị

### APP → DEVICE (Gửi thủ công)

```json
{
  "name": "CmdGetDeviceID",
  "dpCode": "GMH2071"
}
```

### DEVICE -> APP (Phản hồi)

```json
{
  "name": "CmdGetDeviceID",
  "dpCode": "GMH2071",
  "addrIp": "192.168.1.33", // Địa chỉ ip trong mạng wifi
  "addr": "08F9E06E6AD0" // Địa chỉ MAC của thiết bị
}
```
