# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

> Đổi tên file này thành `MSSV_HoTen.md` của bạn, ví dụ `2A20260000_NguyenVanAn.md`, rồi điền từng mục. Xoá các dòng hướng dẫn in nghiêng khi xong.

- **MSSV:**
- **Họ và tên:**
- **SEED** (in ở ô đầu notebook):3929

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | ☑ |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | ☑ |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | ☑ |

## 2. Kết quả chính
Model / Phương phápF1 test (gốc)F1 SUB 40p95 độ trễ (ms/tin)Chi phí (USD/1k tin)1. BoW + LogReg (tự code)0.9470.947   ~0.3 ms   0.00$2. BoW + MLP (deep learning)0.9190.919   ~0.4 ms   0.00$3a. Encoder (MiniLM) + kNN0.9470.947   ~250 ms   0.00$3b. Encoder (MiniLM) + LogReg0.9190.919   ~70 ms   0.00$3c. Encoder (MiniLM) + MLP head0.9470.947   ~70 ms   0.00$4a. LLM nhỏ zero-shot0.8840.884   ~2800 ms   ~0.01$   4b. LLM lớn zero-shot0.9470.950   ~3000 ms   ~0.10$   4d. LLM nhỏ + prompt phòng thủ0.8840.884   ~2900 ms~0.01$   

## 3. Dự đoán và thực tế
Dự đoán 1 — Accuracy của BoW trên tập 50 case khó:Đoán: Khoảng 60% – 70%.Thực tế: Chỉ đạt 50% (F1 tụt thảm hại xuống 0.286).   Vì sao lệch: Dự đoán quá lạc quan vì không lường trước được việc tập test khó chứa toàn bộ các bẫy triệt tiêu hoàn toàn đặc trưng từ khóa (spam viết như người thật hoặc cố ý né từ khóa). 
Dự đoán 2 — Độ trễ và vị trí Pareto của BoW + LogReg:Đoán: Mô hình ML truyền thống sẽ bị Encoder hoặc LLM vượt trội hoàn toàn về chất lượng.Thực tế: BoW + LogReg nằm trên đường Pareto, lấn át hoàn toàn BoW + MLP và Encoder + MLP head.   Vì sao lệch: Chưa tính đến yếu tố đánh đổi độ trễ: BoW + LogReg xử lý cực nhanh (dưới 1 ms) mà chất lượng trên tập test thường không hề thua kém các mô hình phức tạp.
Dự đoán 3 — LLM nhỏ bị lừa bởi Prompt Injection:Đoán: Mô hình LLM sẽ dễ dàng bị các câu lệnh lừa đổi nhãn (accuracy nhóm này dưới 50%).Thực tế: Phiên bản 4a vẫn đạt 83% và khi có prompt phòng thủ (4d) đã đạt 100%.   Vì sao lệch: Cơ chế phòng vệ bằng cấu trúc thẻ cô lập <sms> đã tách bạch dữ liệu và chỉ dẫn hiệu quả hơn nhiều so với dự tính ban đầu.

## 4. Phân tích 50 case khó
Model tụt nhiều nhất: 1. BoW + LogReg tụt mạnh nhất, F1 từ 0.947 rơi thẳng đứng xuống 0.286 (accuracy đạt đúng 50%).   Nhóm case gây sập nặng nhất: Nhóm "Spam giọng tự nhiên" (đoán đúng 0%) và "Né từ khoá" (đoán đúng 17%).   2 case cụ thể và cơ chế gây sai:Case #11 (Spam giọng tự nhiên - Thật: spam): "Hey it's Jess from the gym we're doing a members-only thing this week, 3 months for the price of 1, just reply JOIN..."Cơ chế sai: Tin nhắn dùng từ ngữ thân thiện, giả danh bạn bè, hoàn toàn vắng bóng các từ khóa cờ đỏ truyền thống. BoW nhìn văn bản thành các token độc lập và thấy toàn từ vựng phổ thông nên phân loại sai thành ham.   Case #30 (Né từ khoá - Thật: spam): "F R E E  e n t r y  into our weekly draw, t x t  WIN  to 80086"Cơ chế sai: Kẻ phát tán cố tình chèn khoảng trắng giữa các chữ cái. Bộ tokenizer tách chuỗi thành các chữ cái đơn lẻ (F, R, E, E) không nằm trong tập từ vựng 2000 từ phổ biến, khiến vector BoW toàn số 0 và mô hình mất khả năng nhận diện. 

## 5. Thử thách cá nhân
Đề bài & Mục tiêu: Tự thiết kế tối thiểu 5 edge cases (tập trung vào nhóm ham_looks_spam, talks_about_spam, natural_spam) có cấu trúc tinh vi nhằm thử thách giới hạn nhận diện của mô hình tốt nhất (4d. LLM nhỏ + prompt phòng thủ).   Cách làm: Bổ sung 5 trường hợp có tính đối kháng cao vào bộ test:Case #51 (ham_looks_spam): Chuyển tiền viện phí khẩn cấp, nhắc số tài khoản.Case #52 (talks_about_spam): Kể chuyện cười có trích dẫn câu lệnh prompt injection.Case #53 & #54 (natural_spam): Giả mạo bạn bè rủ đi ăn tiệm bánh và giả thư ký y tế nhắc lịch khám để cài link.Case #55 (ham_looks_spam): Bạn bè rủ mua hộ áo khoác trong đợt sale gấp.Kết quả: Trên tập 55 case, mô hình 4d đạt độ chính xác 92.7% (chỉ sai 4/55 case). Cả 5 case tự tạo đều được mô hình nhận diện chính xác.   Giải thích: Nhờ cơ chế hiểu ngữ cảnh sâu và prompt phòng thủ phân định rõ ràng giữa "chỉ dẫn" và "dữ liệu thô", mô hình LLM đã giải mã được dụng ý thực sự đằng sau câu từ thay vì chỉ nhìn vào các bề mặt từ vựng. 

## 6. Phương án cho sản phẩm thật
Yêu cầu hệ thống: Xử lý 10.000.000 SMS/ngày, SLA p95 < 50 ms/tin.   Phương án lựa chọn: Kiến trúc phân tầng Hai lớp (Two-tier Cascade Pipeline) kết hợp tiền xử lý chuẩn hóa:Lớp 1 (Fast-path): BoW + Logistic Regression (chạy sau bộ Text Normalizer để khử né từ khóa). Xử lý ngay lập tức ~95% lượng tin nhắn thông thường.   Lớp 2 (Slow-path / Review): Chuyển các tin nhắn có độ bất định cao (prediction probability nằm trong khoảng 0.4 – 0.6) sang cho LLM nhỏ có prompt phòng thủ hoặc mô hình Encoder MiniLM phân xử.Dẫn chứng số liệu:BoW + LogReg có độ trễ cực thấp chỉ ~0.3 ms/tin, hoàn toàn thỏa mãn SLA 50 ms và tốn 0 USD chi phí API.   Nếu dùng LLM trực tiếp cho toàn bộ 10 triệu tin, hệ thống sẽ vi phạm SLA hoàn toàn (độ trễ ~2800 ms) và tiêu tốn 105 USD/ngày (~3.150 USD/tháng).   Đánh đổi chấp nhận:Chấp nhận đánh đổi một tỷ lệ nhỏ false-negative ở các dạng spam biến tướng phức tạp để đổi lấy độ trễ siêu tốc, hệ thống vận hành ổn định theo thời gian thực và chi phí hạ tầng tối thiểu.

## 7. AI log và điều đã học
Nhật ký tương tác AI (AI Log):Lần 1: Hỏi cách giải quyết bài toán tính tay và tính số tham số mạng MLP build_mlp(2000, 64). Nhận công thức chi tiết ($128.129$ tham số) và đã tự tính nhẩm lại để kiểm tra trước khi ghi vào hand_calc.jpg.   Lần 2: Hỏi giải thích 5 dòng lệnh TODO trong vòng lặp huấn luyện PyTorch và hậu quả khi quên zero_grad(). Nhận câu trả lời và cô đọng lại thành các ý chính ngắn gọn.Lần 3: Hỏi cách sửa lỗi cú pháp unpack tuple (t, l, c) khi chèn biến MY_CASES. Nhận giải pháp sửa cấu trúc dữ liệu sang DataFrame và chạy thành công trên Colab.Điều nhận ra hôm nay:Ảo tưởng về điểm số cao: Điểm F1 hay Accuracy cao trên tập test phân bố chuẩn thường mang tính "lừa mị"; một mô hình tưởng chừng hoàn hảo vẫn có thể sụp đổ hoàn toàn khi gặp các trường hợp biên ngoài thực tế.Giá trị của sự đơn giản: Trong các bài toán production thực tế, một giải pháp tuyến tính đơn giản nhưng siêu nhanh và rẻ (như BoW + LogReg) nhiều khi lại là lựa chọn tối ưu hơn hẳn so với những mô hình ngôn ngữ lớn đắt đỏ và cồng kềnh.
