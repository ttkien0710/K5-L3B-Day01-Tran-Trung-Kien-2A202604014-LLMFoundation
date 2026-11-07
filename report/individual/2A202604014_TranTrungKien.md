# Báo cáo cá nhân — Day 01: Lọc SMS spam/ham

**SEED:** 2475. Đã bổ sung thí nghiệm Gemini sau khi key mới xác thực thành công. Năm hàng ML/DL/embedding giữ nguyên số đo từ lần chạy local trước trên Windows/CPU, PyTorch 2.15.0+cpu, 2 luồng. 2 hàng LLM hoàn thành và hai lượt stress test LLM là đợt chạy Gemini bổ sung qua API thật, provider `gemini`, model nhỏ `gemini-3.5-flash-lite`, model lớn `gemini-3.5-flash`. Không chạy lại local để thay số đo hoặc trộn kết quả API mock vào thực nghiệm. Dữ liệu chia theo cùng SEED và cùng SUB40; thời điểm, điều kiện mạng và cách loại thời gian chờ của LLM khác phép đo local. Model lớn `gemini-3.5-flash` mới hoàn thành 18/40 tin trước khi chạm quota ngày Free Tier; chưa có F1 SUB40, không chấm điểm tập chưa đủ 40 tin như thể là toàn bộ SUB và không kết luận model lớn hơn/kém model nhỏ. Phần còn lại cần quota ngày mới hoặc người học chủ động bật billing; không tự thay sang model khác.

## 1. Bằng chứng

| Tệp | Trạng thái |
|---|---|
| [day01_lab.ipynb](day01_lab.ipynb) | Có code, câu trả lời và output; 15/15 mục kiểm tra đạt, 10 dự đoán được ghi |
| [lab1_report.json](lab1_report.json) | Do ô báo cáo cuối sinh ra; giữ nguyên dữ liệu và dấu thời gian |
| [hand_calc.jpg](hand_calc.jpg) | Ảnh giấy thật do người học cung cấp, đủ 5 phép tính và bước làm |

Dữ liệu UCI tải được có 5.574 dòng; sau bỏ trùng tuyệt đối còn 5.171 tin (4.518 ham, 653 spam). Theo split của SEED: train 3.621, val 775, test 775. SUB40 lấy từ test, gồm 20 spam và 20 ham. Con số dòng gốc khác mô tả 5.572 trong đề; báo cáo giữ đúng số đo khi đọc archive, không tự chỉnh dữ liệu cho khớp mô tả.

## 2. Kết quả chính

| Phương án | Precision | Recall | F1 test 775 | F1 SUB40 |
| --- | --- | --- | --- | --- |
| 1. BoW + LogReg (tự code) | 0.988 | 0.823 | 0.898 | 0.889 |
| 2. BoW + MLP (deep learning) | 0.956 | 0.906 | 0.930 | 0.923 |
| 3a. Encoder (MiniLM) + kNN | 0.862 | 0.979 | 0.917 | 0.923 |
| 3b. Encoder (MiniLM) + LogReg | 0.979 | 0.969 | 0.974 | 0.947 |
| 3c. Encoder (MiniLM) + MLP head | 0.959 | 0.969 | 0.964 | 0.947 |
| 4a. LLM nhỏ zero-shot | 0.905 | 0.950 | — | 0.927 |
| 4c. LLM nhỏ few-shot | 1.000 | 0.900 | — | 0.947 |

| Phương án | Train (s) | Tin/s | p50 ms | p95 ms | USD ước tính/1k tin |
| --- | --- | --- | --- | --- | --- |
| 1. BoW + LogReg (tự code) | 3.21 | 26091.73 | 0.06 | 0.13 | 0.00000 |
| 2. BoW + MLP (deep learning) | 7.66 | 33829.50 | 0.33 | 0.62 | 0.00000 |
| 3a. Encoder (MiniLM) + kNN | 77.15 | 13.92 | 60.09 | 80.61 | 0.00000 |
| 3b. Encoder (MiniLM) + LogReg | 78.04 | 27.01 | 30.96 | 45.21 | 0.00000 |
| 3c. Encoder (MiniLM) + MLP head | 79.29 | 48.37 | 14.35 | 22.97 | 0.00000 |
| 4a. LLM nhỏ zero-shot | 0.00 | 0.94 | 1023.49 | 1408.44 | 0.02426 |
| 4c. LLM nhỏ few-shot | 0.00 | 0.45 | 1278.99 | 4896.29 | 0.07437 |

Precision/recall của local đo trên test775; precision/recall của LLM đo trên SUB40. LLM không chạy test775 nên cột F1 test775 là —, chỉ dùng F1 SUB40 để so 7 phương án đã hoàn thành. Inference local tính từ văn bản thô theo lần đo trước. Đợt Gemini tính latency từ RTT của request API thành công; loại thời gian pacing chủ động, chờ quota và chờ/thử lại. Vì vậy throughput trong bảng là tổng số tin chia tổng RTT thành công, không phải tốc độ hoàn thành thực tế theo đồng hồ trong Free Tier. p95 không bao gồm thời gian đợi quota. Throughput hiệu dụng khi chịu giới hạn Free Tier thấp hơn. Ghi chú của lượt chạy: Original successful API request latency; excludes intentional pacing and quota/retry waits. Free-tier effective throughput is lower.

Năm hàng ML/DL/embedding giữ nguyên số đo từ lần chạy local trước trên Windows/CPU, PyTorch 2.15.0+cpu, 2 luồng. 2 hàng LLM hoàn thành và hai lượt stress test LLM là đợt chạy Gemini bổ sung qua API thật, provider `gemini`, model nhỏ `gemini-3.5-flash-lite`, model lớn `gemini-3.5-flash`. Không chạy lại local để thay số đo hoặc trộn kết quả API mock vào thực nghiệm. Dữ liệu chia theo cùng SEED và cùng SUB40; thời điểm, điều kiện mạng và cách loại thời gian chờ của LLM khác phép đo local. Model lớn `gemini-3.5-flash` mới hoàn thành 18/40 tin trước khi chạm quota ngày Free Tier; chưa có F1 SUB40, không chấm điểm tập chưa đủ 40 tin như thể là toàn bộ SUB và không kết luận model lớn hơn/kém model nhỏ. Phần còn lại cần quota ngày mới hoặc người học chủ động bật billing; không tự thay sang model khác.

Trên SUB40 và p95 đo được, đường Pareto gồm: **1. BoW + LogReg (tự code); 2. BoW + MLP (deep learning); 3c. Encoder (MiniLM) + MLP head**. Ví dụ: 3a. Encoder (MiniLM) + kNN bị 2. BoW + MLP (deep learning) lấn át. Các cách đạt p95 < 50 ms trong phép đo: 1. BoW + LogReg (tự code); 2. BoW + MLP (deep learning); 3b. Encoder (MiniLM) + LogReg; 3c. Encoder (MiniLM) + MLP head. Các cách không đạt: 3a. Encoder (MiniLM) + kNN; 4a. LLM nhỏ zero-shot; 4c. LLM nhỏ few-shot. Kết luận dựa vào số đo thực tế; chưa chứng minh p95 toàn tuyến ở tải sản phẩm.

| Model | Ví dụ | Prompt | n | Nhãn lỗi | USD ước tính | Input token | Output token | Cached input |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| gemini-3.5-flash-lite | 0 | build_classification_prompt | 40 | 0 | 0.000971 | 2902 | 40 | 0 |
| gemini-3.5-flash-lite | 4 | build_classification_prompt | 40 | 0 | 0.002975 | 9582 | 40 | 0 |
| gemini-3.5-flash-lite | 0 | build_classification_prompt | 50 | 0 | 0.001147 | 3406 | 50 | 0 |
| gemini-3.5-flash-lite | 0 | build_defended_prompt | 50 | 0 | 0.002495 | 7901 | 50 | 0 |

Đợt Gemini ghi 199 request; tổng chi phí theo bảng giá/usage ghi nhận ước tính 0.009706 USD. Tổng này gồm các lời gọi kiểm tra/demo và phần Flash chưa hoàn thành nếu có. Riêng các batch phân loại được lưu bằng chứng ước tính 0.007587 USD. Standard list-price estimates from API usage, including thought tokens; not an invoice or a free-tier charge. Token trong bảng batch là usage trả về nếu đã ghi nhận; dấu — nghĩa là thiếu dữ liệu, không được hiểu là 0 token. Chi phí ước tính không tự chứng minh số tiền bị trừ; xem billing của project để xác nhận.

Giá cấu hình theo triệu token: {"gemini-3.5-flash": [1.5, 9.0], "gemini-3.5-flash-lite": [0.3, 2.5]}. Nguồn công khai: [Gemini pricing](https://ai.google.dev/gemini-api/docs/pricing), [models](https://ai.google.dev/gemini-api/docs/models), [OpenAI compatibility](https://ai.google.dev/gemini-api/docs/openai).

Kết quả MLP local giữ nguyên: 128.129 tham số so với 2.001 của LogReg; val loss thấp nhất tại epoch6 (index5), tăng từ epoch7. F1 bảng là mô hình sau30epoch, không phải checkpoint6. Bỏ ReLU vẫn đạt F1≈0,930; cách tham số hóa/tối ưu khác LogReg NumPy nên không bắt buộc cùng F1.

Ảnh tính tay thật giữ đủ5mục: 87,37%; 0,50; 0,73; 0,60; 10,50USD/ngày theo giá đề. Hash baseline định dạng tỷ lệ0,8737 thành0,87; nên giữ phân số2/3 đến bước tính F1 cuối.

## 3. Dự đoán và thực tế

| Dự đoán | Thực tế | Giải thích |
|---|---|---|
| Người học: F1≈0,80; recall cao hơn precision | BoW F1=0,898; precision=0,988 > recall=0,823 | Mất cân bằng không tự suy ra FP>FN. Trọng số và ngưỡng 0,5 làm mô hình khá bảo thủ khi gán spam |
| Người học: lr=50 biến động mạnh; phần giải thích dự kiến dễ không hội tụ | Loss cực đại 2,561 nhưng cuối giảm còn 0,0052 | Bước lớn gây dao động ban đầu; không có nghĩa chắc chắn không hội tụ trên dữ liệu này. lr=0,001 giảm rất chậm, cuối còn 0,625 |
| AI: Encoder+LogReg tốt hơn BoW trên 6 tin ngụy trang | Cùng đúng 5/6, cùng bỏ lọt tin thúc giục giữ phần thưởng tài khoản | Encoder pre-train không tự bảo đảm hiểu ý định lừa đảo. Phải kiểm tra bằng case thật thay vì suy từ tên công nghệ |

Đối chiếu thêm: dự đoán người học về overfitting khớp đường loss; MLP nhỉnh hơn LogReg khoảng 3,3 điểm phần trăm F1. Dự đoán của AI rằng bỏ ReLU giảm F1 không được xác nhận trong lần chạy này.

**Bổ sung Gemini:** Zero-shot nhỏ F1 SUB=0.927. Model lớn `gemini-3.5-flash` mới hoàn thành 18/40 tin trước khi chạm quota ngày Free Tier; chưa có F1 SUB40, không chấm điểm tập chưa đủ 40 tin như thể là toàn bộ SUB và không kết luận model lớn hơn/kém model nhỏ. Phần còn lại cần quota ngày mới hoặc người học chủ động bật billing; không tự thay sang model khác. Few-shot nhỏ F1=0.947, thay đổi +0.021 so với zero-shot. few-shot/zero-shot nhỏ=3.06 lần. Các dự đoán ban đầu do AI ghi được lưu cùng timestamp trong prediction_history; dự đoán Gemini mới được ghi trước khi chạy từng thí nghiệm. Phần đoán model lớn đắt khoảng 10 lần được viết theo cấu hình OpenAI cũ, không phải dự báo giá Gemini; đối chiếu Gemini bằng chi phí và token thực ghi nhận ở trên. Chênh F1 dưới khoảng 0,05 trên SUB40 nên diễn giải thận trọng.

## 4. Phân tích 50 case khó

| Phương án | F1 SUB40 thường | F1 hard50 | Accuracy hard50 |
| --- | --- | --- | --- |
| 1. BoW + LogReg | 0.889 | 0.378 | 54.0% |
| 2. BoW + MLP | 0.923 | 0.378 | 54.0% |
| 3a. Emb + kNN | 0.923 | 0.737 | 70.0% |
| 3b. Emb + LogReg | 0.947 | 0.583 | 60.0% |
| 3c. Emb + MLP head | 0.947 | 0.596 | 62.0% |
| 4a. LLM nhỏ zero-shot | 0.927 | 1.000 | 100.0% |
| 4d. LLM nhỏ + prompt phòng thủ | — (không chạy SUB phòng thủ) | 1.000 | 100.0% |

| Nhóm (accuracy) | 1. BoW + LogReg | 2. BoW + MLP | 3a. Emb + kNN | 3b. Emb + LogReg | 3c. Emb + MLP head | 4a. LLM nhỏ zero-shot | 4d. LLM nhỏ + prompt phòng thủ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Ham chứa từ khóa spam | 100.0% | 90.0% | 70.0% | 80.0% | 90.0% | 100.0% | 100.0% |
| Spam giọng tự nhiên | 20.0% | 0.0% | 50.0% | 20.0% | 20.0% | 100.0% | 100.0% |
| Thao túng/lừa đảo | 12.5% | 37.5% | 87.5% | 25.0% | 25.0% | 100.0% | 100.0% |
| Né từ khóa | 16.7% | 0.0% | 83.3% | 83.3% | 83.3% | 100.0% | 100.0% |
| Prompt injection | 66.7% | 83.3% | 83.3% | 100.0% | 100.0% | 100.0% | 100.0% |
| Giao dịch hợp lệ | 83.3% | 100.0% | 83.3% | 83.3% | 83.3% | 100.0% | 100.0% |
| Nói về spam | 100.0% | 100.0% | 25.0% | 50.0% | 50.0% | 100.0% | 100.0% |

Tụt F1 nhiều nhất trong những phương án có cả SUB40 và hard50: **2. BoW + MLP**, giảm 0.545. Đây là so F1 với F1, không lấy F1 trừ accuracy. 4d chỉ chạy trên hard50, không có phép đo SUB40 phòng thủ riêng; không gán F1 SUB của 4a thành kết quả đo của 4d. Test cùng nguồn có thể lạc quan vì các mẫu gần trùng và phong cách quen thuộc vẫn còn sau bỏ trùng tuyệt đối.

Nhóm prompt injection: 4a đúng 100.0%, 4d phòng thủ đúng 100.0%, thay đổi +0.0 điểm phần trăm. F1 hard50: 1.000 → 1.000; accuracy toàn hard50: 100.0% → 100.0%. Ngoài nhóm injection, nhóm cải thiện: không có; nhóm giảm accuracy: không có trong lần đo này. Có 6 case injection, gồm cả ham; tỷ lệ đúng nhóm không tự chứng minh LLM đã nghe theo lệnh độc hại. Muốn xác nhận cần đọc case và phản hồi cụ thể. Prompt phòng thủ không bảo đảm an toàn tuyệt đối. Ngoài prompt nên thêm kiểm tra schema nhãn, chuẩn hóa đầu vào có kiểm soát, kiểm tra URL/nguồn gửi, bộ lọc độc lập và đường chuyển case rủi ro sang kiểm tra bổ sung.

Hai case cả năm model local đều sai ở lần chạy trước:

- **#12, spam giọng tự nhiên:** mở đầu “hi there, sorry to bother you, my friend gave me your number”; tiếp cận hẹn hò không yêu cầu có giọng trò chuyện cá nhân.
- **#16, spam giọng tự nhiên:** mở đầu “Long time no see! I've started a little side business with skincare”; giả làm bạn cũ để chào bán hàng.

Case #12: 4a đúng, 4d đúng. Case #16: 4a đúng, 4d đúng. Không mở rộng kết luận ‘tất cả model sai’ từ năm model local sang LLM khi chưa kiểm tra.

BoW nhìn token rời nên dễ bị né chữ; embedding và LLM có thể nhận nghĩa nhưng vẫn phụ thuộc phong cách, ngữ cảnh và dữ liệu. Chuẩn hóa Unicode/leetspeak có thể giúp nhưng phải tránh làm hỏng OTP/mã đơn hàng. 50 case viết tay dùng tìm điểm yếu và kiểm tra hồi quy, không đại diện tần suất spam ngoài đời.

Bonus giữ kết quả local cũ: 5 case do AI đề xuất chạy riêng trên Encoder+LogReg đều sai (bỏ lọt bốn spam và chặn nhầm một ham). Bẫy gồm xin mã thẻ quà tặng, mạo danh người thân, phí giao bưu kiện, né chữ free/cash và trích dẫn spam trong tin lớp học. Không gọi thêm LLM cho bonus và không trộn bonus vào hard50.

## 5. Thử thách cá nhân

SEED 2475 chọn **đề #4: kNN với k=1, 5, 25**.

| k | Precision | Recall | F1 test |
| --- | --- | --- | --- |
| 1 | 0.816 | 0.969 | 0.886 |
| 5 | 0.862 | 0.979 | 0.917 |
| 25 | 0.829 | 0.958 | 0.889 |

k=5 cao nhất trong phép đối chiếu này. k=1 nhạy với hàng xóm nhiễu; k=25 lấy vùng rộng có thể làm pha loãng tín hiệu và lệch về lớp phổ biến. Nếu triển khai, chọn k bằng validation, không dùng test để chọn rồi báo kết quả trên cùng test.

## 6. Phương án cho sản phẩm thật

Các cách đạt p95 < 50 ms theo số đo: 1. BoW + LogReg (tự code); 2. BoW + MLP (deep learning); 3b. Encoder (MiniLM) + LogReg; 3c. Encoder (MiniLM) + MLP head. F1 SUB cao nhất trong nhóm này là 3b. Encoder (MiniLM) + LogReg (0.947). Các hàng LLM được xét theo p95 RTT request thành công vừa đo, không được coi mặc nhiên đạt SLA vì F1 cao. p95 này loại thời gian pacing/quota/retry, nên chưa chứng minh SLA toàn tuyến hoặc năng lực phục vụ của Free Tier.

Lưu lượng 10 triệu SMS/ngày là **115,74 tin/s trung bình**. Throughput local giữ nguyên: Encoder+LogReg 27.01 tin/s, Encoder+MLP head 48.37 tin/s. Một cấu hình đo hiện tại chưa chứng minh đủ công suất encoder ở tải đó. Throughput LLM lấy từ RTT request thành công, không gồm pacing và chờ quota/thử lại. LLM đang gọi tuần tự từng tin nên throughput không phải tốc độ hoàn tất thực tế của Free Tier hay năng lực song song tối đa của provider, nhưng cũng không thể tự suy rằng tăng song song sẽ giữ nguyên p95 hoặc không chạm quota. Cần load test tải đỉnh, thời gian chờ, rate limit và độ trễ toàn tuyến.

Router đề xuất: BoW xử lý tin chắc chắn, embedding xử lý phần lưỡng lự, LLM là tầng bổ sung cho case cần ngữ cảnh sau khi cân nhắc p95/chi phí. Hiệu chỉnh xác suất và chọn ngưỡng bằng validation. Không đưa đường gọi LLM chậm vào tuyến bắt buộc 50 ms mà chưa thiết kế ngân sách thời gian, fallback hoặc xử lý bất đồng bộ. Kết hợp URL/nguồn gửi và kiểm tra giao dịch/OTP; F1 hard50 cho thấy vẫn cần dữ liệu lừa đảo đa dạng.

**Scale chi phí Gemini có điều kiện:** giả sử giữ nguyên prompt, usage và giá, 30 ngày × 10 triệu SMS/ngày tương đương 300.000 lô 1.000 tin. Khi đó:

- 4a. LLM nhỏ zero-shot: 0.02426 USD/1k × 300.000 ≈ **7,279.50 USD/tháng**.
- 4c. LLM nhỏ few-shot: 0.07437 USD/1k × 300.000 ≈ **22,309.50 USD/tháng**.

Đây là ngoại suy theo mẫu 40 tin, không phải hóa đơn; quota miễn phí, token suy luận, cache, độ dài SMS và retry có thể thay đổi số tiền thực trả. Router chỉ gửi một phần tin sẽ có chi phí khác.

**Bài tính tay theo đề vẫn giữ nguyên:** giá giả định và 100 input/1 output token cho 1 triệu SMS/ngày cho 10,50 USD/ngày; với 10 triệu/ngày và tháng 30 ngày là **3.150 USD/tháng**. Không thay con số bài tính tay này bằng giá Gemini. Model local có phí API 0 nhưng vẫn tốn máy chủ.

## 7. AI log và điều đã học

| Phần           | Yêu cầu hỗ trợ                        | AI thực hiện                                                  | Người học thực hiện/kiểm chứng                                 |
| -------------- | ------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------- |
| Code           | Hỗ trợ viết code                      | Viết code, chạy kiểm tra và thí nghiệm local                  | Đọc lại code và kiểm chứng kết quả                             |
| Câu trả lời    | Dừng để trả lời                       | Ghi câu trả lời đã cung cấp                                   | Cung cấp các câu trả lời và dự đoán; viết và chụp ảnh tính tay |
| LLM và báo cáo | Hỗ trợ cấu hình API, tổng hợp báo cáo | Viết hàm API, chạy Gemini Flash-Lite và tổng hợp số liệu thật | Lưu key mới, chọn giữ Free Tier và chờ quota Flash reset       |


Nhận xét tổng hợp từ kết quả: Accuracy cao có thể che việc bỏ lọt spam, nên cần xem precision, recall và F1 cùng case lỗi. F1 trên tập cùng nguồn không bảo đảm xử lý lừa đảo mới: model tốt nhất trên test thường không phải model tốt nhất ở hard50. Chọn hệ thống cần xét đồng thời chất lượng, công suất, p95 và chi phí, không chỉ độ phức tạp của model.
