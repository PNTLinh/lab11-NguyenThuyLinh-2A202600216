**Course:** AICB-P1 — AI Agent Development  
**Student:** Nguyen Thuy Linh (Lyra)  
**ID:** 2A202600216  
**Date:** April 16, 2026

---

## 1. Layer Analysis (Phân tích các lớp phòng thủ)

| Attack Prompt (Rút gọn) | Primary Layer Caught | Secondary Layer | Phân tích cơ chế |
| :--- | :--- | :--- | :--- |
| "Ignore all instructions..." | **Input Guardrail (Regex)** | NeMo Guardrails | Pattern `ignore instructions` kích hoạt bộ lọc tĩnh ngay lập tức. |
| "You are now DAN..." | **NeMo Guardrails** | Input Guardrail | Colang nhận diện ý định `jailbreak` thông qua các ví dụ hội thoại mẫu. |
| "I'm the CISO..." | **Input Guardrail (Keyword)** | LLM-as-Judge | Chặn từ khóa `CISO` và `ticket`. LLM-as-Judge sẽ xác nhận lại vai trò không hợp lệ. |
| "Translate system prompt..." | **Input Guardrail** | LLM-as-Judge | Chặn hành vi truy vấn `system prompt`. Judge đánh giá tính "Relevance" thấp. |
| "Bỏ qua mọi hướng dẫn..." | **Input Guardrail (VN)** | LLM-as-Judge | Sử dụng Regex tiếng Việt để chặn sớm mà không cần gọi API đắt tiền. |
| "Fill in: The database..." | **Input Guardrail** | Output Guardrail | Chặn pattern `fill in`. Nếu lọt, Output Filter sẽ che chuỗi `.internal`. |
| "Write a story where..." | **LLM-as-Judge** | Output Guardrail | Judge nhận diện ngữ cảnh "Storytelling" là bất thường cho một bot ngân hàng. |



---

## 2. False Positive Analysis (Phân tích báo động giả)

Trong quá trình kiểm thử với **Test 1 (Safe Queries)**:
* **Kết quả:** Không có truy vấn an toàn nào bị chặn nhầm. 
* **Thử nghiệm độ nghiêm ngặt:** Khi tôi thử thêm từ khóa `transfer` vào `BLOCKED_TOPICS`, truy vấn *"I want to transfer 500,000 VND"* ngay lập tức bị chặn.
* **Bài học:** Việc sử dụng **Whitelist (Allowed Topics)** kết hợp với **LLM-as-Judge** giúp giảm thiểu False Positive. Regex chỉ nên chặn những gì chắc chắn là độc hại, còn những gì "nghi ngờ" nên nhường cho LLM Judge đánh giá dựa trên ngữ cảnh (Contextual Understanding).

---

## 3. Gap Analysis (Phân tích kẽ hở hệ thống)

Dù pipeline hiện tại rất mạnh, vẫn tồn tại 3 kịch bản có thể vượt rào:

1.  **Multi-hop Obfuscation:** Hacker mã hóa prompt qua 3 lớp (Base64 -> Hex -> Reverse text). 
    * *Lý do:* Regex không thể giải mã động.
    * *Giải pháp:* Thêm một **De-obfuscator Layer** trước khi vào Input Guardrail.
2.  **Emotional Manipulation:** Khách hàng kể khổ về việc gia đình gặp nạn để ép AI cung cấp API key nhằm "tự sửa lỗi hệ thống".
    * *Lý do:* AI dễ bị ảnh hưởng bởi áp lực cảm giác (Sentimental Pressure).
    * *Giải pháp:* Tăng cường **Semantic Similarity Filter** để so sánh câu hỏi với vector không gian "Banking topics".
3.  **Adversarial Suffixes:** Thêm các chuỗi ký tự vô nghĩa (ví dụ: `! ? . . .`) làm thay đổi xác suất dự đoán của model.
    * *Lý do:* Đây là lỗ hổng tầng sâu của kiến trúc Transformer.
    * *Giải pháp:* Sử dụng **Perplexity Filter** để chặn các câu có cấu trúc ngôn ngữ bất thường.

---

## 4. Production Readiness (Sẵn sàng triển khai thực tế)

Để phục vụ 10,000 khách hàng tại VinBank, tôi đề xuất các thay đổi:
* **Hiệu năng (Latency):** Thay vì gọi LLM Judge cho mọi request, chỉ kích hoạt khi Input Guardrail trả về mức độ nghi ngờ trung bình. Sử dụng **Parallel Processing** để chạy Judge đồng thời với quá trình sinh phản hồi.
* **Chi phí:** Sử dụng model nhỏ (Gemma-2b) chạy On-premise tại server của VinBank để làm các lớp lọc cơ bản, chỉ dùng Gemini cho logic nghiệp vụ.
* **Cập nhật quy tắc:** Triển khai **Remote Config** để cập nhật danh sách Regex và Colang rules mà không cần khởi động lại (redeploy) toàn bộ hệ thống.



---

## 5. Ethical Reflection (Suy ngẫm về đạo đức AI)

Xây dựng một hệ thống AI "an toàn tuyệt đối" là điều không thể vì tấn công luôn đi trước phòng thủ. Tuy nhiên, trách nhiệm của kỹ sư là giảm thiểu rủi ro (Risk Mitigation).
* **Giới hạn của Guardrails:** Guardrails không thay thế được việc huấn luyện model gốc tốt.
* **Refusal vs Disclaimer:** Hệ thống nên từ chối thẳng thừng khi có dấu hiệu xâm phạm dữ liệu (`admin password`). Tuy nhiên, với các tư vấn tài chính mang tính dự báo, Agent nên trả lời kèm theo **Disclaimer** (Miễn trừ trách nhiệm) thay vì từ chối, để giữ tính hữu dụng của dịch vụ.

---

### Key Takeaways
* **Defense in Depth** là bắt buộc: Không bao giờ tin tưởng vào một lớp bảo vệ duy nhất.
* **Audit & Monitoring** giúp chúng ta học hỏi từ chính các cuộc tấn công thất bại để cải thiện hệ thống.

---