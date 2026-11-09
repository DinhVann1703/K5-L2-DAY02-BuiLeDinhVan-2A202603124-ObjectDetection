# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` |Xe con 4–7 chỗ, sedan, hatchback, SUV, taxi, xe bán tải chở người thông thường. |Phân biệt với `van` qua dáng xe có đuôi cốp riêng hoặc khoang hành lý tách biệt; phân biệt với `truck` vì bán tải cabin kép tính là `car`. |
| `truck` |Xe tải chở hàng thùng hở/thùng kín, xe ben, xe bồn, container kéo. |Có kết cấu chassis rời chịu lực, cabin lật hoặc khoang chở hàng riêng biệt tách rời buồng lái; không gán cho xe bán tải dân dụng. |
| `bus` |Xe khách chở người từ 16 chỗ trở lên, xe buýt công cộng nội đô.|Kích thước thân xe dài và cao vượt trội, nhiều hàng ghế chở khách; không gán cho xe 7–9 chỗ thuộc nhóm `van`.|
| `van` |Xe minivan (7–9 chỗ), xe van giao hàng nhỏ/cabin liền thùng (ví dụ Blind Van). |Thân xe hình hộp thuôn dài, cửa trượt ngang hông hoặc cốp mở đứng, không có cốp sedan riêng và không có chassis thùng rời như `truck`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là:Bounding box 2D dạng chữ nhật phải ôm sát các điểm cực viễn ngoài cùng của vỏ xe (bánh xe chạm mặt đường, gương chiếu hậu, mui xe, cản trước/sau). Không để lọt khoảng trống nền quá 2–3 pixel và không cắt lẹm vào thân vỏ xe.
- Vật thể bị che khuất:Chỉ vẽ box ôm trọn **phần nhìn thấy thực tế (visible boundary)** nếu nhận diện được rõ ràng $\ge 30\%$ diện tích thân xe và xác định chắc chắn được class. Không vẽ phỏng đoán phần diện tích bị che khuất bởi xe khác, dải phân cách hoặc cột đèn.
- Vật thể bị cắt bởi rìa ảnh: Nếu xe nằm ở mép khung hình và bị cắt cụt, bắt đầu vẽ cạnh box áp sát ngay mép ảnh và bao trọn phần thân xe còn xuất hiện trong khung hình; chỉ gán nhãn nếu phần nhìn thấy trong ảnh $\ge 20\%$.
- Vật thể rất nhỏ/mơ hồ: Bỏ qua không gán nhãn đối với các đốm pixel ở quá xa hậu cảnh có kích thước nhỏ hơn $15 \times 15$ pixel hoặc bị nhiễu mờ do nén ảnh CCTV mà mắt thường không thể phân biệt rõ cấu trúc hình học của xe.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_003.jpg` |Cụm phương tiện ùn tắc kéo dài ở phía xa dưới gầm cầu vượt đi bộ (hậu cảnh), kích thước cực nhỏ (< 15x15 px) và bị che khuất liên hoàn > 70%. |**Không gán nhãn** (bỏ qua) các xe quá mờ/quá xa ở cụm tắc đường phía xa; chỉ gán nhãn cho các xe ở hàng trước có hình khối nhận diện được rõ ràng. |Tuân thủ quy tắc ngưỡng kích thước tối thiểu và độ che khuất; việc cố gán nhãn các đốm pixel mờ ở xa sẽ tạo nhãn nhiễu (noise) làm giảm độ chính xác của mô hình phát hiện vật thể. |
| `drive_008.jpg` |Phương tiện giao thông hỗn hợp: Có xe 2 bánh (mô tô/xe máy/xe đạp điện) di chuyển ở giữa ngã tư; và xe tải ben màu đỏ chở đống đất/đá nhô cao hơn thành thùng. |- Xe 2 bánh: **Không gán nhãn** (bỏ qua).<br>- Xe tải ben: Gán nhãn **`truck` (Class 1)**, vẽ bounding box bao trọn cả cabin, thùng xe và phần đất đá nhô cao phía trên. |Schema 4 lớp chỉ định nghĩa phương tiện 4 bánh trở lên (`car`, `truck`, `bus`, `van`), không có lớp cho xe 2 bánh.<br>- Phần tải/hàng hóa trên thùng gắn liền với kích thước chiếm dụng thực tế của phương tiện khi xe tham gia giao thông. |
| `drive_043.jpg` |Xe con màu đen ở làn giữa phía dưới di chuyển tốc độ cao bị nhòe chuyển động (motion blur) và có bóng đổ đen (shadow) kéo dài xuống mặt đường. |Gán nhãn **`car` (Class 0)**. Vẽ bounding box ôm sát thân vỏ xe thực tế, **loại trừ phần bóng đổ trên mặt đường** và vệt mờ chuyển động loang ra ngoài. |Bounding box 2D trong ADAS phải định vị chính xác khối vật thể cơ học; bóng đổ trên mặt đường không thuộc cấu trúc xe, nếu bao cả bóng sẽ làm sai lệch tâm tọa độ và kích thước hộp. |
