# EV Charging Open Data – Thành phố X (OLP 2025)

Bộ dữ liệu này mô tả các trạm sạc xe điện và các lần sạc được mô phỏng tại Thành phố X (khu vực TP.HCM), phục vụ bài thi Phần mềm Nguồn mở – OLP 2025. Nhóm tập trung vào lĩnh vực hạ tầng kỹ thuật – năng lượng (EV charging), tuân thủ các chuẩn NGSI-LD, FIWARE Smart Data Models và SOSA/SSN, đồng thời tiếp cận theo hướng Linked Open Data.

## 1. Cấu trúc dự án

Mã nguồn hiện chứa thư mục `data/` với ba tệp chính:

```text
.
├─ data/
│  ├─ stations.jsonld       # Dataset trạm sạc (EVChargingStation)
│  ├─ observations.jsonld   # Dataset phiên sạc & cảm biến (EVChargingSession/Sensor, có SOSA)
│  ├─ sessions.jsonld       # Dataset lịch sử sạc gắn với công dân (Person + EVChargingSession)
│  └─ realtime_sample.json  # Mẫu sự kiện NGSI-LD thời gian thực (update + session mới)
├─ LICENSE                  # Giấy phép dữ liệu mở
├─ CHANGELOG.md             # Lịch sử thay đổi phiên bản
└─ README.md
```

Trong đó `stations.jsonld`, `observations.jsonld` và `sessions.jsonld` là các dataset NGSI-LD hoàn chỉnh, còn `realtime_sample.json` dùng để mô phỏng dòng sự kiện khi demo.

## 2. NGSI-LD và Smart Data Models

Tất cả entity trong thư mục `data/` đều tuân theo mô hình NGSI-LD normalized: mỗi entity có trường `id` là IRI/URN (ví dụ `urn:ngsi-ld:EVChargingStation:001`) và trường `type` là `EVChargingStation` hoặc `EVChargingSession`. Các thuộc tính được biểu diễn dưới dạng NGSI-LD attribute, có `type` là `Property`, `GeoProperty` hoặc `Relationship` và mang giá trị trong trường `value` (hoặc `object` khi là quan hệ). Vị trí địa lý của trạm sạc được mô tả bằng thuộc tính `location` kiểu `GeoProperty` với giá trị GeoJSON `Point` theo thứ tự `[longitude, latitude]`.

Mỗi tệp dữ liệu đều khai báo `@context` gồm NGSI-LD core (`https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld`) và context của Smart Data Models cho EVChargingStation. Nhờ vậy, dữ liệu có thể được nạp trực tiếp vào các NGSI-LD broker như Orion-LD hoặc Scorpio mà không cần chuyển đổi thêm.

Về phía FIWARE Smart Data Models, dự án kế thừa data model `EVChargingStation` thuộc domain Transportation. Các thuộc tính đặc trưng như `capacity`, `socketNumber`, `availableCapacity`, `allowedVehicleType`, `socketType`, `openingHours`, `status`, `network`, `operator`, `amperage`, `voltage`, `chargeType`, `acceptedPaymentMethod`, `location`, `address`, `name` được sử dụng nhất quán giữa các trạm. Các giá trị có kiểu liệt kê (chẳng hạn `status`, `socketType`, `allowedVehicleType`, `chargeType`, `acceptedPaymentMethod`) đều được chọn trong danh sách enum do Smart Data Models định nghĩa.

Ngoài ra, một số trường mở rộng dành cho quan trắc và giao dịch cũng được sử dụng trong dữ liệu phiên sạc như `powerConsumption`, `amountCollected`, `taxAmountCollected`, `startDateTime`, `endDateTime`, `transactionId`, `transactionType`, `vehicleType` hay `chargingUnitId` giúp dữ liệu dễ dàng được phân tích về mặt năng lượng và doanh thu.

## 3. Vai trò của SOSA/SSN trong dữ liệu

Đối với các lần sạc, dự án mô hình hóa chúng như các quan trắc IoT theo ontology SOSA/SSN. Trong `@context`, kiểu `EVChargingSession` được ánh xạ sang lớp `sosa:Observation` và kiểu `Sensor` được ánh xạ sang `sosa:Sensor`. Mỗi phiên sạc có các quan hệ `refFeatureOfInterest` (tương đương `sosa:hasFeatureOfInterest`) trỏ về trạm sạc `EVChargingStation`, `refSensor` (tương đương `sosa:madeBySensor`) trỏ về cảm biến đo năng lượng, cùng với `observedProperty` mô tả thuộc tính được quan sát.

Về thời gian, mỗi quan trắc mang cả cặp thuộc tính `phenomenonTime` và `resultTime` theo SOSA, lần lượt phản ánh thời điểm bắt đầu và thời điểm kết thúc/ghi nhận kết quả của phiên sạc. Nhờ đó, dữ liệu có thể được phân tích dưới góc nhìn dòng thời gian quan trắc, không chỉ dưới góc nhìn giao dịch.

Tệp `observations.jsonld` ngoài các phiên sạc còn chứa các entity `Sensor` riêng (Ví dụ: cảm biến đo năng lượng cho trạm 001, 003, 007, 008), đóng vai trò nguồn phát sinh quan trắc trong mô hình SOSA/SSN.

## 4. Dữ liệu trạm sạc – `data/stations.jsonld`

Tệp `stations.jsonld` là một Dataset JSON-LD mô tả mười trạm sạc xe điện phân bố quanh TP.HCM. Mỗi trạm là một entity `EVChargingStation` với định danh dạng `urn:ngsi-ld:EVChargingStation:00X`. Dữ liệu bao gồm các trạm tại ĐH Sư Phạm, NowZone, Công viên 23/9, Landmark 81, Crescent Mall, Vincom Thủ Đức, Sân bay Tân Sơn Nhất, Bến xe Miền Đông mới, Công viên Gia Định và Chợ Bến Thành.

Đối với từng trạm, dữ liệu lưu các thông tin chung như tên, địa chỉ, giờ hoạt động, trạng thái; vị trí địa lý dưới dạng `location` GeoJSON; các thông số năng lực như `capacity`, `socketNumber`, `availableCapacity`, `amperage`, `voltage`; khả năng phục vụ phương tiện qua `allowedVehicleType` và `socketType`; thông tin vận hành như `network`, `operator`; cùng cấu hình thanh toán thông qua `chargeType` và `acceptedPaymentMethod`. Các trường `amperage` và `voltage` được gắn thêm mã đơn vị `unitCode` theo UN/CEFACT (lần lượt là `AMP` và `VLT`) để thể hiện rõ đơn vị đo cường độ dòng điện và điện áp.

## 5. Dữ liệu phiên sạc – `data/observations.jsonld`

Tệp `observations.jsonld` là Dataset JSON-LD chứa nhiều entity `EVChargingSession` (đồng thời là các `sosa:Observation`). Mỗi entity mô tả một lần sạc cụ thể, liên kết với trạm tương ứng qua `refFeatureOfInterest` và với cảm biến đo qua `refSensor`. Dữ liệu ghi lại thời điểm bắt đầu và kết thúc (`startDateTime`, `endDateTime`), kèm theo `phenomenonTime` và `resultTime` theo SOSA; loại phương tiện tham gia (`vehicleType` như `electricMotorcycle`, `electricCar`, `electricBus` – tương ứng xe máy điện, ô tô điện, bus điện), mã ổ sạc (`chargingUnitId`), thông tin giao dịch (`transactionId`, `transactionType`) và các số liệu đo lường như năng lượng tiêu thụ (`powerConsumption`), số tiền thu được (`amountCollected`) và thuế (`taxAmountCollected`).

Các trường định lượng quan trọng đều có `unitCode` rõ ràng: `powerConsumption` tính theo kWh (`KWH`), `amountCollected` và `taxAmountCollected` tính bằng VND (`VND`), giúp bộ dữ liệu dễ tích hợp với các pipeline phân tích hoặc hệ thống báo cáo tự động, đồng thời bám sát khuyến nghị của Smart Data Models về việc khai báo đơn vị đo.

Nhờ cấu trúc này, bộ dữ liệu phiên sạc rất phù hợp cho việc xây dựng dashboard phân tích: có thể vẽ biểu đồ năng lượng theo trạm và theo thời gian, thống kê doanh thu, so sánh hành vi giữa các loại phương tiện hoặc khu vực khác nhau trong thành phố, và đồng thời minh họa rõ ràng cách áp dụng SOSA/SSN trong bối cảnh NGSI-LD.

## 6. Dữ liệu lịch sử sạc công dân – `data/sessions.jsonld`

- Tệp JSON-LD chứa cả `Person` (schema.org) và các entity `EVChargingSession` gắn `refUser` tới từng công dân.
- Các phiên sử dụng chung context NGSI-LD/SOSA/Smart Data Models với `observations.jsonld`, bổ sung các thuộc tính phục vụ thống kê: `sessionStatus`, `durationMinutes`, `powerConsumption`, `amountCollected`, `taxAmountCollected`.
- Định danh `EVChargingSession` được chuẩn hóa dạng `citizen_user_x:station_id:timestamp` giúp backend ánh xạ nhanh tới tài khoản người dùng.
- Dataset này phục vụ trực tiếp cho tính năng “Lịch sử sạc” và thống kê cá nhân trong frontend, đồng thời là nguồn nhập cho ETL (collection `citizens`, `sessions`).

## 7. Mẫu dữ liệu thời gian thực – `data/realtime_sample.json`

Tệp `realtime_sample.json` không phải là một Dataset NGSI-LD đầy đủ mà là tập các sự kiện mẫu, dùng để viết script mô phỏng dòng dữ liệu thời gian thực. Tệp vẫn khai báo `@context` tương tự các tệp khác (kể cả mapping sang SOSA/SSN) và chứa một mảng `events`. Mỗi phần tử trong mảng này có `id`, trường `operation` (ví dụ `update` hoặc `upsert`) và một đối tượng `entity` là payload NGSI-LD hoàn chỉnh.

Một nhóm sự kiện dùng để cập nhật trạng thái trạm (thay đổi `availableCapacity`, `status`, `instantaneousPower`, `queueLength` vào các thời điểm khác nhau trong ngày). Thuộc tính `instantaneousPower` mang đơn vị kW thông qua `unitCode` `KWT`, còn các thời điểm đo được gắn bằng `observedAt` đúng khuyến nghị NGSI-LD. Nhóm sự kiện còn lại dùng để thêm mới các phiên sạc `EVChargingSession` với cùng mô hình SOSA như trong `observations.jsonld` (bao gồm `refFeatureOfInterest`, `refSensor`, `observedProperty`, `phenomenonTime`, `resultTime`) và kèm theo các trường `powerConsumption` (kWh), `amountCollected` (VND) và `taxAmountCollected` (VND) có đơn vị đo rõ ràng. Khi chạy lần lượt các sự kiện này, có thể mô phỏng được kịch bản giờ cao điểm, giờ thấp điểm, trạng thái có sự cố và các dòng giao dịch mới.

## 8. Giấy phép và bối cảnh sử dụng

Trừ khi được quy định khác trong phần mã nguồn của ứng dụng, dữ liệu trong thư mục `data/` được phát hành theo giấy phép Creative Commons Attribution 4.0 International (CC BY 4.0). Khi trích dẫn hoặc tái sử dụng, có thể ghi nguồn theo dạng: *"EV charging open data for Smart City X (OLP 2025), UE Core – Đại học Sư phạm TP.HCM"*.

Dữ liệu sử dụng vocabulary schema.org (https://schema.org), được cấp phép theo Creative Commons Attribution-ShareAlike 3.0 (CC BY-SA 3.0).

Giấy phép cho phần mềm (API, dịch vụ backend, frontend web hoặc mobile) nên là một giấy phép nguồn mở được OSI phê duyệt như MIT, Apache-2.0 hoặc GPL, và sẽ được mô tả riêng trong kho mã nguồn của ứng dụng.

Mục tiêu của bộ dữ liệu là cung cấp một tập ví dụ hoàn chỉnh, có thể nạp trực tiếp vào NGSI-LD broker, dễ dàng tích hợp với các thành phần FIWARE và có thể dùng lại trong nhiều bối cảnh nghiên cứu hoặc trình diễn liên quan đến thành phố thông minh, hạ tầng sạc xe điện và quản lý dữ liệu IoT theo các tiêu chuẩn mở hiện đại.

## 9. Hướng dẫn nạp dữ liệu vào NGSI-LD broker

Các tệp `data/stations.jsonld` và `data/observations.jsonld` được tổ chức dưới dạng một đối tượng Dataset JSON-LD có trường `mainEntity` chứa mảng các entity NGSI-LD. Để nạp dữ liệu này vào NGSI-LD broker (Orion-LD, Scorpio,…), thực hiện theo các bước sau:

1. Parse file JSON-LD và trích xuất:
   - Giá trị `@context` ở cấp Dataset.
   - Mảng entity trong trường `mainEntity`.

2. Khi gọi API tới broker:
   - Gửi từng entity riêng lẻ qua endpoint `POST /ngsi-ld/v1/entities`; hoặc
   - Gửi nhiều entity một lần qua endpoint `POST /ngsi-ld/v1/entityOperations/upsert` với body là mảng các entity.

3. Cung cấp context NGSI-LD cho broker:
   - Hoặc giữ `@context` ngay trong body JSON-LD;
   - Hoặc truyền qua HTTP header `Link`, ví dụ:

     `<https://uri.etsi.org/ngsi-ld/v1/ngsi-ld-core-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"`

Tóm tắt: khi nạp dữ liệu, luôn lấy mảng `mainEntity` trong các tệp Dataset rồi gửi các entity đó lên NGSI-LD broker qua `POST /ngsi-ld/v1/entities` hoặc `POST /ngsi-ld/v1/entityOperations/upsert`, kèm theo context NGSI-LD bằng `@context` trong body hoặc qua header `Link`.
