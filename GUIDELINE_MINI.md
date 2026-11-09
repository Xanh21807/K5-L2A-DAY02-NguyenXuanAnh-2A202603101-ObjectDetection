# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con 4-7 chỗ, sedan, SUV, hatchback, taxi, xe bán tải (pickup) | Xe cá nhân/gia đình, cabin nhỏ; xe bán tải vẫn xếp vào `car` theo schema |
| `truck` | Xe tải chở hàng, xe ben, xe đầu kéo container, xe bồn | Thùng hàng lớn, cabin rời hoặc xe tải hạng nặng chuyên chở hàng hóa |
| `bus` | Xe buýt công cộng, xe khách lớn, xe du lịch cỡ lớn | Thân xe dài, nhiều hàng ghế chở khách số lượng lớn |
| `van` | Xe tải nhỏ/xe van kín thùng, minivan chở khách/chở hàng (Ford Transit, Solati...) | Thùng liền thân cỡ trung bình, lớn hơn `car` nhưng nhỏ hơn `bus` lớn |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bao trọn mép ngoài phần thân xe nhìn thấy được, không lấn mặt đường hay bầu trời, không gộp 2 xe vào cùng một box.
- Vật thể bị che khuất: Vẽ box quanh phần thân xe còn nhìn thấy được nếu nhận diện rõ loại xe; nếu bị che quá nhiều không nhận diện được lớp thì bỏ qua.
- Vật thể bị cắt bởi rìa ảnh: Box chạm sát đúng rìa ảnh, không tự suy đoán phần thân xe nằm ngoài khung hình.
- Vật thể rất nhỏ/mơ hồ: Nếu không đủ chi tiết nhận dạng thì bỏ qua, không tự đoán; tuyệt đối không gán xe máy, xe đạp, người đi bộ hay đèn đường.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_002` | Xe khách du lịch (coach bus) kích thước lớn chạy ngược chiều ở làn giữa | `bus` | Xe chở khách thân dài nhiều chỗ ngồi, phân biệt với xe van cỡ nhỏ |
| `drive_004` | Xe minivan/minibus cỡ nhỏ màu vàng ở góc xa ngã tư | `van` | Xe khoang liền chở hàng/người cỡ trung, theo schema phân loại vào `van` thay vì `car` hay `bus` |
| `drive_024` | Xe bán tải (pickup) có thùng hở nhỏ phía sau chạy cùng chiều | `car` | Schema quy định rõ xe bán tải/cabin nhỏ thuộc lớp `car`, không xếp vào `truck` |