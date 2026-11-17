# PHÂN TÍCH TRADE-OFF: SO SÁNH GIẢI PHÁP PROMPTING CHO KỸ THUẬT NÂNG CAO (LOGIC & CHUỖI)



## BƯỚC 1: ĐỀ XUẤT 2 PROMPT MẪU

Để giải quyết bài toán tự động hóa quy trình xử lý khiếu nại và bảo hành phức tạp (áp dụng Chain-of-Thought, Prompt Chaining và xử lý đứt gãy dữ liệu) tại hệ thống thương mại điện tử **TechShop**, dưới đây là 2 phương án thiết kế Prompt với mức độ phức tạp khác nhau:

###  Giải pháp 1: Prompt Đơn giản & Ngắn gọn (Direct CoT)

*Phương án này sử dụng câu lệnh gọn nhẹ, yêu cầu AI gộp chung các bước suy luận và xử lý vào một lần gọi duy nhất để tiết kiệm ký tự.*

```text
Bạn là nhân viên CSKH TechShop. Hãy đọc phản hồi khiếu nại của khách hàng, tự suy luận nhanh nguyên nhân và trả về kết quả JSON gồm: mã đơn hàng (order_id), vấn đề (issue), và phương án giải quyết (solution). Nếu thiếu mã đơn hàng thì tự điền là "Không rõ".

Input cần xử lý:
"Giao hàng chán kinh khủng, mua cái máy ép trái cây mã đơn #TS-1122 mà nhận về vỡ nát phần cối xay, gọi tổng đài không ai bắt máy!"

```

###  Giải pháp 2: Prompt Chi tiết, Đa tầng & Có cơ chế Fallback (Advanced CoT & Error Handling)

*Phương án này áp dụng cấu trúc tường minh, bắt buộc AI tuân thủ tư duy từng bước (Chain-of-Thought), tích hợp quy tắc nghiệp vụ nghiêm ngặt và cơ chế bắt lỗi dị biệt (Fallback) khi dữ liệu trống hoặc sai lệch.*

```text
[Context / Character]
Bạn là Chuyên viên Phân tích dữ liệu khiếu nại & Vận hành cấp cao tại TechShop, chịu trách nhiệm xử lý các luồng dữ liệu tự động phức tạp.

[Request & Chain-of-Thought Rules]
Hãy xử lý đoạn phản hồi thô của khách hàng qua các bước tư duy sau:
Bước 1 (CoT): Đọc kỹ văn bản, phân tích các từ ngữ bề mặt và nhận diện sự cố kỹ thuật/vận chuyển cốt lõi.
Bước 2 (Entity Extraction): Trích xuất thực thể `order_id` (định dạng #TS-XXXX).
Bước 3 (Error Handling & Fallback): 
- Kiểm tra tính hợp lệ của `order_id`. Nếu trường này bị trống, `null`, không có trong văn bản hoặc sai định dạng, **bắt buộc** không được tự bịa đặt, phải gán giá trị `"FALLBACK_REQUIRED"` cho trạng thái hệ thống và yêu cầu xử lý thủ công.
- Nếu hợp lệ, đề xuất phương án bồi thường theo chính sách TechShop (đổi mới trong 24h).

[Type of Output & Format]
Kết quả trả về PHẢI là đối tượng JSON thuần túy theo cấu trúc:
{
  "system_status": "OK hoặc FALLBACK_REQUIRED",
  "cot_reasoning": "Chuỗi suy luận từng bước...",
  "extracted_data": {
    "order_id": "...",
    "issue_description": "..."
  },
  "action_plan": "..."
}

[Input Data cần xử lý]:
"Giao hàng chán kinh khủng, mua cái máy ép trái cây mã đơn #TS-1122 mà nhận về vỡ nát phần cối xay, gọi tổng đài không ai bắt máy!"

```

---

## BƯỚC 2: LẬP BẢNG SO SÁNH TRADE-OFF (ĐÁNH GIÁ ĐỔI LẤY)

| Tiêu chí đánh giá | Giải pháp 1 (Đơn giản) | Giải pháp 2 (Chi tiết & Logic ràng buộc) |
| --- | --- | --- |
| **Độ dễ hiểu / Viết nhanh** | **Cao** (Ngắn gọn, dễ viết, không tốn thời gian thiết kế tầng logic phức tạp). | **Trung bình** (Đòi hỏi tư duy cấu trúc chuỗi, lập luận từng bước và quy tắc điều kiện rõ ràng). |
| **Chi phí Token (Token Cost)** | **Ít** (Sử dụng ít token đầu vào, phản hồi nhanh chóng). | **Nhiều hơn** (Tốn token hơn cho các khối quy tắc, hướng dẫn CoT và cấu trúc JSON mẫu). |
| **Khả năng chặn lỗi biên (Error Handling)** | **Thấp** (Dễ gặp hiện tượng AI tự sinh ra dữ liệu giả mạo khi gặp trường hợp thiếu mã đơn hàng). | **Cực tốt** (Có cơ chế kiểm tra điều kiện Fallback, bảo vệ hệ thống không bị crash hoặc nhận dữ liệu sai lệch). |
| **Độ ổn định dữ liệu đầu ra** | **Trung bình** (Thỉnh thoảng thiếu trường dữ liệu hoặc sai lệch cấu trúc khi hệ thống tích hợp tự động). | **Rất cao** (Cấu trúc tường minh giúp định dạng JSON ổn định, sẵn sàng tích hợp vào Database/Dashboard). |

---

## BƯỚC 3: LỰA CHỌN PHƯƠNG ÁN TỐI ƯU VÀ KẾT QUẢ CHẠY THỬ

### 1. Lựa chọn phương án tối ưu

* **Giải pháp 2** là phương án tối ưu nhất và được chọn để triển khai trong hệ thống vận hành thực tế của TechShop.
* **Lý do lựa chọn:** Trong các hệ thống tự động hóa (Logic & Chaining), rủi ro lớn nhất là **dữ liệu dị biệt (Anomaly Data)** và hiện tượng ảo giác tự bịa số liệu khi gặp lỗi thiếu thông tin. Việc đầu tư thêm token cho Giải pháp 2 giúp đổi lấy sự an toàn tuyệt đối nhờ cơ chế **Fallback**, đảm bảo hệ thống không bị sai lệch dữ liệu và có thể bàn giao mượt mà sang quy trình xử lý thủ công khi cần thiết.

### 2. Kết quả chạy thử thực tế (Test Run Simulation với Giải pháp 2)

Khi nạp đoạn đầu vào chứa mã đơn hàng `#TS-1122` vào Giải pháp 2, AI sẽ thực thi logic từng bước và trả về kết quả chuẩn xác như sau:

```json
{
  "system_status": "OK",
  "cot_reasoning": "Bước 1: Khách hàng bức xúc vì vấn đề vận chuyển và chất lượng sản phẩm (vỡ nát cối xay máy ép). Bước 2: Trích xuất thành công mã đơn hàng #TS-1122 từ văn bản. Bước 3: Mã đơn hàng hoàn toàn hợp lệ, không cần kích hoạt trạng thái Fallback, tiến hành lên phương án đổi mới sản phẩm.",
  "extracted_data": {
    "order_id": "#TS-1122",
    "issue_description": "Vỡ nát phần cối xay của máy ép trái cây do vận chuyển"
  },
  "action_plan": "Thu hồi sản phẩm hỏng, đổi mới 100% cối xay/máy ép trong vòng 24h, miễn phí vận chuyển 2 chiều và gửi lời xin lỗi."
}

```