# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con cá nhân chở người, sedan, hatchback, SUV, taxi, hoặc xe bán tải (pickup cabin nhỏ). | Gồm: sedan, hatchback, SUV, crossover, coupe, wagon, taxi, xe bán tải (pickup). Phân biệt: chiều cao thấp hơn van/bus, có đầu capo rõ, không có thùng tải lớn chuyên dụng như `truck` và không có thân hộp liền khối như `van`. |
| `truck` | Xe chuyên chở hàng hóa hoặc xe chuyên dụng tải nặng, có cabin tách biệt với thùng hàng/thiết bị phía sau. | Gồm: xe tải thùng (kín, bạt, lửng), xe ben, xe bồn, xe đầu kéo (kèm rơ-moóc), xe cứu hộ/cẩu. Phân biệt: nhìn thấy rõ cabin đầu bằng và thùng chở hàng phía sau (không tính xe bán tải/cabin nhỏ thuộc `car`). |
| `bus` | Xe chở khách công cộng hoặc đường dài cỡ lớn, thân dài, nhiều ô cửa sổ dọc hai bên. | Gồm: xe buýt thành phố (1 tầng, 2 tầng, buýt khớp nối), xe khách, minibus chở khách. Phân biệt: dài và cao hơn `van` rõ rệt, dãy cửa sổ hành khách liên tục, có cửa lên xuống lớn. |
| `van` | Xe van, minivan có thân liền khối dạng hộp (cabin và khoang sau liền nhau), dùng chở hàng hoặc chở người. | Gồm: xe van chở hàng, minivan chở người, Ford Transit, Toyota Hiace, MPV/minivan dáng hộp. Phân biệt: đầu xe ngắn hoặc bằng, mui cao dạng hộp vuông vắn, nhỏ hơn `bus` và cao/vuông hơn `car`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh bounding box chạm sát mép ngoài cùng phần nhìn thấy được của thân/bánh/gương xe, không lấy dư mặt đường, bóng đổ hay nền trời, mỗi xe 1 box riêng biệt.
- Vật thể bị che khuất: Vẽ 1 box bao quanh phần diện tích nhìn thấy được của xe nếu phần đó vẫn đủ đặc trưng để xác định chắc chắn lớp; nếu bị cột/dải phân cách mỏng cắt ngang nhưng vẫn là 1 xe thì giữ 1 box thống nhất.
- Vật thể bị cắt bởi rìa ảnh: Vẽ box bao phần xe nằm trong ảnh, cạnh box dừng đúng tại biên ảnh (`[0, 1]`), tuyệt đối không suy đoán phần nằm ngoài khung hình.
- Vật thể rất nhỏ/mơ hồ: Nếu xe ở quá xa hoặc bị che/cắt chỉ còn mảnh rất nhỏ không thể phân biệt chắc chắn thuộc lớp nào trong 4 lớp thì bỏ qua (không tự đoán) và ghi nhận vào guideline.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe SUV màu trắng ở làn bên phải bị cột đèn giao thông che cắt dọc chính giữa thân xe; đồng thời có hàng xe đang đỗ dọc lề đường bên trái. | Vẽ 1 box duy nhất (`0: car`) bao phần thân xe trắng nhìn thấy ở hai bên cột đèn; vẫn gán `0: car` riêng cho từng xe đang đỗ bên lề trái. | Cột đèn chỉ che hẹp ở giữa, vẫn nhận diện rõ 1 chiếc `car` thống nhất (không tách thành 2 box). Với xe đỗ: schema phân lớp theo loại vật thể, không phân biệt đang chạy hay đang đỗ. |
| `drive_005.jpg` | Xe cứu hộ/tải nhỏ màu trắng ở giữa đường và dòng phương tiện xếp hàng rất xa, kích thước rất nhỏ dưới gầm cầu vượt phía trên. | Gán `1: truck` cho xe tải cứu hộ màu trắng; ở khu vực xa phía trên chỉ gán các xe còn nhận diện rõ hình khối (`car`, `bus`, `van`), bỏ qua các chấm xe quá nhỏ/mờ ở cuối hàng. | Xe trắng có cabin và sàn/thùng chở hàng chuyên dụng nên thuộc `truck`. Các xe ở quá xa bị nhòe chỉ còn mảnh vài pixel không đủ căn cứ xác định lớp thì không tự đoán. |
| `drive_015.jpg` | Xe tải nhỏ chở hàng (chở các bao trắng) ở làn trên bên phải và một phần nóc xe tối màu bị cắt sát mép dưới cùng của ảnh (chỉ lộ một dải rất mỏng). | Gán `1: truck` cho xe chở hàng; không gán phần nóc xe bị cắt sát rìa dưới cùng. | Xe chở hàng có thùng hở chở hàng hóa rõ rệt nên thuộc `truck` (không phải `van` hay `car`). Phần xe ở mép dưới chỉ lộ mảnh quá nhỏ của nóc, không thể xác định chắc chắn lớp. |

