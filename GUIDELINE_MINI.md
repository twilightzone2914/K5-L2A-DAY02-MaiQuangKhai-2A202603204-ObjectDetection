# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | xe con, sedan, SUV, taxi, xe bán tải | Ví dụ: xe gia đình, xe taxi có mào, xe bán tải cabin nhỏ. Điểm phân biệt: có phần mũi/nắp ca-pô nhô rõ phía trước và trần thấp hơn van/bus; xe bán tải vẫn gán là car (không gán truck); không gán mô tô/xe máy. |
| `truck` | xe tải chở hàng, xe đầu kéo | Ví dụ: xe tải thùng kín, xe tải thùng bạt, xe đầu kéo chở container. Điểm phân biệt: phần đầu cabin tách biệt rõ với thùng hàng phía sau, khung gầm và kích thước lớn; không gán xe bán tải cabin nhỏ (car) hay xe van thân liền khối (van). |
| `bus` | xe buýt, minibus chở khách | Ví dụ: xe buýt công cộng, xe khách, xe buýt cỡ trung chở khách. Điểm phân biệt: thân xe hình hộp chữ nhật dài và cao, hai bên hông có dãy nhiều ô cửa kính liên tiếp, đầu xe phẳng với kính chắn gió rộng; dài và lớn hơn hẳn van. |
| `van` | xe van, minivan chở hàng hoặc chở người | Ví dụ: xe van chở hàng nhẹ, xe minivan. Điểm phân biệt: thân xe hình hộp liền khối (không tách rời cabin và thùng hàng như truck), đầu xe ngắn/mũi tù hơn car/SUV, ngắn và ít khoang cửa sổ hông hơn bus (thường chỉ 2–3 khoang kính). Nửa thân trước là khoang cabin xe van thương mại còn nửa sau cải tạo thành sàn công tác hở gắn cần cẩu kéo cứu hộ |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bốn cạnh của bounding box phải tiếp xúc sát với các điểm cực trị nhìn thấy được của thân xe (cản trước, cản sau, nóc xe, mép dưới bánh xe và gương chiếu hậu nếu nhìn rõ). Không chừa khoảng trống nền thừa và **tuyệt đối không** bao gồm bóng đổ trên mặt đường khô hoặc bóng phản chiếu dưới mặt đường ướt khi trời mưa.
- Vật thể bị che khuất: Chỉ vẽ bounding box bao quanh **phần diện tích thực tế nhìn thấy được** của phương tiện (không tự ước lượng hay kéo rộng box ra phần bị khuất hoàn toàn sau trụ cầu vượt, nhà chờ xe buýt, tán cây hoặc xe phía trước). Nếu phương tiện bị che khuất quá nặng (chỉ lộ một góc nhỏ không đủ đặc trưng để xác định thuộc lớp nào) thì bỏ qua không gán nhãn.
- Vật thể bị cắt bởi rìa ảnh: Vẫn gán nhãn nếu phần thân xe nằm trong khung hình có kích thước tối thiểu từ $20\\text{px} \\times 20\\text{px}$ trở lên và còn đủ đặc trưng hình dáng để nhận diện chính xác lớp (`car`, `truck`, `bus`, `van`). Cạnh của bounding box tại phía bị cắt phải dừng chính xác tại mép khung hình (`0` hoặc `640`), và bỏ qua nếu chỉ lộ một mẩu mũi/đuôi xe quá nhỏ sát mép ảnh.
- Vật thể rất nhỏ/mơ hồ: **Không gán nhãn** các phương tiện ở quá xa phía chân trời của camera giám sát có chiều rộng hoặc chiều cao nhỏ hơn $20\\text{px}$ ($\\text{width} \< 20\\text{px}$ hoặc $\\text{height} \< 20\\text{px}$ trên ảnh $640 \\times 640$), hoặc bị nhòe/mờ nặng khiến mắt người không thể phân biệt rõ lớp phương tiện.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_050.jpg` | **Ca 1 — Vật thể bị che khuất / bị cắt bởi rìa khung hình**: Ở góc dưới ảnh, chiếc xe con màu đỏ bị cắt ngang phần thân dưới bởi rìa khung hình; đồng thời một số xe con ở làn giữa bị các xe đi trước che khuất một phần đầu hoặc thân dưới. | Vẫn gán nhãn s`car` với cạnh bounding box chỉ bao quanh **phần diện tích thực tế nhìn thấy được** và dừng chính xác tại mép ảnh (`y_max = 640`) hoặc tại ranh giới bị xe trước che khuất. | Phần thân xe lộ ra trong khung hình vẫn có kích thước lớn ($\\ge 20\\text{px} \\times 20\\text{px}$) và giữ đầy đủ đặc trưng nhận diện lớp; không tự ước lượng hay kéo box vượt qua phần bị che khuất/rìa ảnh để tránh đưa vùng nền nhiễu vào nhãn. |
| `drive_020.jpg` | **Ca 2 — Vật thể bị mờ nhòe (blurry)**: Hai chiếc xe ở làn đường giữa bị nhòe chuyển động (motion blur) do tốc độ cao, trong khi các phương tiện ở góc xa phía trên đường chân trời bị mờ và rất nhỏ ($\< 20\\text{px}$). | Vẫn gán nhãn `car` cho hai xe bị nhòe chuyển động ở làn giữa (bám sát biên nhìn thấy); **bỏ qua không gán nhãn** các phương tiện ở xa bị mờ nhòe có chiều rộng hoặc chiều cao $\< 20\\text{px}$. | Hai xe bị motion blur ở gần vẫn có kích thước lớn ($\\ge 20\\text{px}$) và phân biệt rõ hình dáng sedan/hatchback của lớp `car`; ngược lại các xe quá mờ và nhỏ $\< 20\\text{px}$ ở xa không còn đủ chi tiết thị giác để phân biệt chính xác giữa `car` và `van`. |
| `drive_006.jpg` | **Ca 3 — Môi trường đặc biệt (mặt đường ướt gây bóng phản chiếu)**: Trời mưa làm mặt đường nhựa ướt phản chiếu bóng tối và hình ảnh thân xe rất đậm ngay dưới gầm chiếc xe trắng và chiếc xe đỏ ở tiền cảnh, dễ gây nhầm lẫn vị trí đáy xe thật. | Gán nhãn `car` với cạnh dưới của bounding box dừng đúng tại điểm tiếp xúc thực tế của lốp xe và cản xe với mặt đường, **loại bỏ hoàn toàn** phần bóng phản chiếu trên mặt đường ướt. | Bóng nước phản chiếu không phải là bộ phận vật lý của phương tiện; nếu kéo box bao gồm cả vùng phản chiếu sẽ làm box bị giãn dài xuống dưới, làm lệch tâm `y_center` và giảm IoU khi huấn luyện mô hình. |
