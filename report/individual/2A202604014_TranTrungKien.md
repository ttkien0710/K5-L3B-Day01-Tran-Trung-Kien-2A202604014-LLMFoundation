# Báo cáo cá nhân — Day 01: Lọc SMS spam/ham

**SEED:** 2475. Số liệu lấy từ lần chạy notebook thực tế trên Windows/CPU, PyTorch 2.15.0+cpu, 2 luồng. Phần LLM được bỏ qua theo lựa chọn người học sau khi kết nối Gemini bị từ chối key. Không có kết quả API phân loại giả lập trong bảng thực nghiệm.

## 1. Bằng chứng

| Tệp | Trạng thái |
|---|---|
| [day01_lab.ipynb](day01_lab.ipynb) | Có code, câu trả lời và output; 15/15 mục kiểm tra đạt, 10 dự đoán được ghi |
| [lab1_report.json](lab1_report.json) | Do ô báo cáo cuối sinh ra; giữ nguyên dữ liệu và dấu thời gian |
| [hand_calc.jpg](hand_calc.jpg) | Ảnh giấy thật do người học cung cấp, đủ 5 phép tính và bước làm |

Dữ liệu UCI tải được có 5.574 dòng; sau bỏ trùng tuyệt đối còn 5.171 tin (4.518 ham, 653 spam). Theo split của SEED: train 3.621, val 775, test 775. SUB40 lấy từ test, gồm 20 spam và 20 ham. Con số dòng gốc khác mô tả 5.572 trong đề; báo cáo giữ đúng số đo khi đọc archive, không tự chỉnh dữ liệu cho khớp mô tả.

## 2. Kết quả chính

| Phương án | Precision | Recall | F1 test | F1 SUB40 |
| --- | --- | --- | --- | --- |
| BoW + LogReg | 0.988 | 0.823 | 0.898 | 0.889 |
| BoW + MLP | 0.956 | 0.906 | 0.930 | 0.923 |
| MiniLM + kNN | 0.862 | 0.979 | 0.917 | 0.923 |
| MiniLM + LogReg | 0.979 | 0.969 | 0.974 | 0.947 |
| MiniLM + MLP head | 0.959 | 0.969 | 0.964 | 0.947 |

| Phương án | Train (s) | Tin/s cả lô | p50 (ms) | p95 (ms) | USD API/1k tin |
| --- | --- | --- | --- | --- | --- |
| BoW + LogReg | 3.21 | 26091.73 | 0.06 | 0.13 | 0,00 |
| BoW + MLP | 7.66 | 33829.50 | 0.33 | 0.62 | 0,00 |
| MiniLM + kNN | 77.15 | 13.92 | 60.09 | 80.61 | 0,00 |
| MiniLM + LogReg | 78.04 | 27.01 | 30.96 | 45.21 | 0,00 |
| MiniLM + MLP head | 79.29 | 48.37 | 14.35 | 22.97 | 0,00 |

Inference đo từ văn bản thô đến nhãn, bao gồm vectorize/encode. Train của embedding gồm encode tập train và train head nếu có. Các giá trị phụ thuộc máy, warm-up và tải hệ thống; chênh lệch tốc độ giữa các head không hoàn toàn do kiến trúc. Phần LLM zero/few-shot và phòng thủ chưa chạy; không thể xếp hạng chất lượng, độ trễ hay chi phí LLM.

Trên SUB40 và p95 đo được, đường Pareto gồm BoW+LogReg, BoW+MLP và MiniLM+MLP head. kNN bị BoW+MLP lấn át vì cùng F1 SUB nhưng chậm hơn; Emb+LogReg bị Emb+MLP head lấn át trên hai tiêu chí này. Tuy nhiên Emb+LogReg vẫn có F1 toàn bộ test cao hơn.

MLP có 128.129 tham số so với 2.001 của LogReg. Val loss thấp nhất tại epoch 6 (index 5), tăng từ epoch 7 trong khi train loss giảm: nên chọn checkpoint epoch 6 bằng validation. Số F1 trong bảng là mô hình sau 30 epoch theo cell gốc, không phải checkpoint epoch 6. Mạng bỏ ReLU cũng đạt F1≈0,930: hai Linear liên tiếp gộp thành một biến đổi affine, nhưng cách tối ưu/tham số hóa khác LogReg NumPy nên không buộc ra cùng trọng số hay cùng F1.

Tính tay: accuracy luôn đoán ham = 87,37%; σ(0)=0,50; F1 ví dụ =0,73; cosine=0,60; chi phí theo đề =10,50 USD/ngày. Trong hash kiểm tra, accuracy lưu theo tỷ lệ 0,8737 và được định dạng 2 chữ số thập phân thành 0,87. Ảnh giấy làm tròn recall ở bước trung gian nhưng kết quả F1 cuối vẫn đúng 0,73; khi tính chính xác nên giữ 2/3 đến bước cuối.

## 3. Dự đoán và thực tế

| Dự đoán | Thực tế | Giải thích |
|---|---|---|
| Người học: F1≈0,80; recall cao hơn precision | BoW F1=0,898; precision=0,988 > recall=0,823 | Mất cân bằng không tự suy ra FP>FN. Trọng số và ngưỡng 0,5 làm mô hình khá bảo thủ khi gán spam |
| Người học: lr=50 biến động mạnh; phần giải thích dự kiến dễ không hội tụ | Loss cực đại 2,561 nhưng cuối giảm còn 0,0052 | Bước lớn gây dao động ban đầu; không có nghĩa chắc chắn không hội tụ trên dữ liệu này. lr=0,001 giảm rất chậm, cuối còn 0,625 |
| AI: Encoder+LogReg tốt hơn BoW trên 6 tin ngụy trang | Cùng đúng 5/6, cùng bỏ lọt tin thúc giục giữ phần thưởng tài khoản | Encoder pre-train không tự bảo đảm hiểu ý định lừa đảo. Phải kiểm tra bằng case thật thay vì suy từ tên công nghệ |

Đối chiếu thêm: dự đoán người học về overfitting khớp đường loss; MLP nhỉnh hơn LogReg khoảng 3,3 điểm phần trăm F1. Dự đoán của AI rằng bỏ ReLU giảm F1 không được xác nhận trong lần chạy này.

## 4. Phân tích 50 case khó

| Phương án | F1 SUB40 | F1 hard50 | Accuracy hard50 |
| --- | --- | --- | --- |
| 1. BoW + LogReg | 0.889 | 0.378 | 54.0% |
| 2. BoW + MLP | 0.923 | 0.378 | 54.0% |
| 3a. Emb + kNN | 0.923 | 0.737 | 70.0% |
| 3b. Emb + LogReg | 0.947 | 0.583 | 60.0% |
| 3c. Emb + MLP head | 0.947 | 0.596 | 62.0% |

Tụt nhiều nhất từ SUB40 sang hard50 là **BoW+MLP**, từ 0,923 xuống 0,378 (giảm 0,545). Nhóm spam giọng tự nhiên và né từ khóa đều đạt 0% accuracy với model này. Không đồng nhất các nhóm hay lấy F1 SUB trừ accuracy hard. Test thông thường cùng nguồn với train và vẫn có mẫu spam gần trùng dù đã bỏ trùng tuyệt đối, nên dễ lạc quan về khả năng tổng quát hóa.

| Nhóm (accuracy) | BoW LR | BoW MLP | Emb kNN | Emb LR | Emb MLP |
| --- | --- | --- | --- | --- | --- |
| Ham chứa từ khóa spam | 100.0% | 90.0% | 70.0% | 80.0% | 90.0% |
| Spam giọng tự nhiên | 20.0% | 0.0% | 50.0% | 20.0% | 20.0% |
| Thao túng/lừa đảo | 12.5% | 37.5% | 87.5% | 25.0% | 25.0% |
| Né từ khóa | 16.7% | 0.0% | 83.3% | 83.3% | 83.3% |
| Prompt injection | 66.7% | 83.3% | 83.3% | 100.0% | 100.0% |
| Giao dịch hợp lệ | 83.3% | 100.0% | 83.3% | 83.3% | 83.3% |
| Nói về spam | 100.0% | 100.0% | 25.0% | 50.0% | 50.0% |

Hai case cụ thể tất cả 5 model local đều sai:

- **#12, spam giọng tự nhiên:** mở đầu “hi there, sorry to bother you, my friend gave me your number”; ngữ cảnh là tiếp cận hẹn hò không yêu cầu. Cách nói giống trò chuyện cá nhân khiến BoW thiếu từ khóa, vector nghĩa gần ham.
- **#16, spam giọng tự nhiên:** mở đầu “Long time no see! I've started a little side business with skincare”; giả làm bạn cũ để chào bán hàng. Giọng thân thiện che ý định quảng cáo; đặc trưng rút từ tập SMS thường chưa đủ phân biệt.

BoW nhìn token rời nên dễ bị né chữ; embedding nắm ý nghĩa tổng quát nhưng có thể nhầm giọng cá nhân với tin hợp lệ. Có thể thử chuẩn hóa Unicode, ký tự phân cách và leetspeak; phải kiểm tra nguy cơ làm hỏng OTP/mã đơn hàng và tăng chặn nhầm. Chưa có LLM nên chưa kết luận prompt injection có lừa LLM hay prompt phòng thủ có hiệu quả. Tỷ lệ đúng của các model local ở nhóm mang tên prompt injection không chứng minh an toàn của LLM.

50 case viết tay dùng để tìm điểm yếu và kiểm tra hồi quy, không đại diện tần suất spam ngoài đời hoặc dùng làm ước lượng chất lượng triển khai.

Bonus: 5 case do AI đề xuất được chạy riêng trên Emb+LogReg, sai **5/5** (bỏ lọt 4 spam, chặn nhầm 1 ham). Các bẫy lần lượt là mạo danh xin mã thẻ quà tặng, mạo danh người thân xin chuyển tiền, phí giao lại bưu kiện, né chữ “free/cash”, và tin lớp học trích dẫn câu spam để cảnh báo. Mô hình dựa quá nhiều vào giống nghĩa/phong cách với tập train, thiếu kiểm chứng ý định và ngữ cảnh. Giữ nguyên bộ hard50, không trộn bonus vào bảng chính.

## 5. Thử thách cá nhân

SEED 2475 chọn **đề #4: kNN với k=1, 5, 25**. AI đã viết code và chạy; đây là phần có AI hỗ trợ, không phải thử thách tự làm không dùng AI.

| k | Precision | Recall | F1 test |
| --- | --- | --- | --- |
| 1 | 0.816 | 0.969 | 0.886 |
| 5 | 0.862 | 0.979 | 0.917 |
| 25 | 0.829 | 0.958 | 0.889 |

k=5 cao nhất trong phép đối chiếu này. k=1 nhạy với hàng xóm nhiễu; k=25 lấy vùng rộng có thể làm pha loãng tín hiệu và lệch về lớp phổ biến. Nếu triển khai, chọn k bằng validation, không dùng test để chọn rồi báo kết quả trên cùng test.

## 6. Phương án cho sản phẩm thật

Bốn cách đạt **p95 < 50 ms/tin** trong phép đo hiện tại là BoW+LogReg (0,13 ms), BoW+MLP (0,62 ms), MiniLM+LogReg (45,21 ms), MiniLM+MLP head (22,97 ms). kNN không đạt (80,61 ms).

MiniLM+LogReg có F1 test cao nhất (0,974), nhưng F1 hard chỉ 0,583; MiniLM+MLP head có F1 test 0,964 và F1 hard 0,596. Trên SUB40, cả hai bằng 0,947. Nếu ưu tiên tradeoff đo được trên SUB, MLP head có cùng F1 SUB với độ trễ thấp hơn; nếu ưu tiên toàn bộ test thì LogReg cao hơn. Không coi chênh lệch nhỏ trên 40 tin là kết luận chắc chắn.

**Độ trễ và công suất là hai điều khác nhau.** Lưu lượng 10 triệu/ngày tương đương 115,74 tin/s trung bình, trong khi throughput cả lô đo được của Emb+LogReg chỉ 27,01 tin/s, Emb+MLP head 48,37 tin/s. Vì vậy một cấu hình đo hiện tại chưa chứng minh đủ công suất; cần tối ưu encoder/batching, thêm tài nguyên và load test cả tải đỉnh, thời gian chờ và p95 toàn tuyến. Không thể lấy p95 một request để kết luận đáp ứng tải sản phẩm.

Phương án đề xuất để thử nghiệm: BoW xử lý trường hợp chắc chắn, embedding xử lý trường hợp lưỡng lự; dùng thêm tín hiệu nguồn gửi, URL và kiểm tra rủi ro với OTP/giao dịch. Hiệu chỉnh xác suất và chọn ngưỡng trên validation. Chỉ đưa LLM vào tuyến bổ sung sau khi có số đo chất lượng, độ trễ và chi phí; hiện chưa triển khai LLM. Tập hard cho thấy cần dữ liệu lừa đảo/phishing đa dạng trước khi vận hành thật.

Theo **giá và số token giả định của câu tính tay**, 1 triệu SMS/ngày tốn 10,50 USD/ngày; 10 triệu/ngày × 30 ngày tốn **3.150 USD/tháng**. Không phải chi phí Gemini đã đo hoặc báo giá vận hành. Các model local có phí API bằng 0 nhưng vẫn cần chi phí máy chủ.

## 7. AI log và điều đã học

| Phần | Yêu cầu gửi AI | AI thực hiện | Người học kiểm chứng / sửa |
|---|---|---|---|
| Chuẩn bị | Đọc lab, dùng câu trả lời đã có | Đối chiếu yêu cầu, tạo notebook làm việc, giữ nội dung đã cung cấp | Người học cung cấp notebook cá nhân và câu trả lời trong chat |
| Code 1–3 | Viết code, chạy từng cell | Viết các hàm NumPy, PyTorch, cosine, kNN; chạy kiểm tra và thí nghiệm local | Chưa có xác nhận người học đã tự giải thích/kiểm chứng toàn bộ code; phần này có AI viết, khác yêu cầu tự code của đề |
| Trả lời từng bước | Dừng để người học trả lời | Điền nguyên văn câu 0.1, Checkpoint A, 2.A; giữ dự đoán ml_f1, ml_lr, dl_curve | Người học tự cung cấp các nội dung này; có xem gợi ý AI |
| Hoàn thiện phần còn lại | Tự làm tiếp, chuẩn bị lời giải tính tay | AI viết dự đoán, phân tích kết quả, thử thách kNN và 5 case bonus | Các nội dung ghi AI hỗ trợ; thử thách không phải phần tự làm không dùng AI |
| Tính tay | Chuẩn bị lời giải để viết giấy | AI đưa công thức và kết quả tham khảo | Người học viết ra giấy, chụp ảnh thật; đã đối chiếu đủ 5 mục |
| LLM | Lấy key Gemini và chạy API | Viết hàm gọi API, hướng dẫn lưu key; thử kết nối Gemini ngoài notebook | Key bị API từ chối; người học yêu cầu bỏ qua, không có kết quả phân loại LLM |
| Báo cáo và nộp | Hoàn thiện file report, chuẩn bị bài nộp | AI tổng hợp từ số liệu thật, kiểm tra file và hướng dẫn nộp | Người học cần đọc lại trước khi nộp; không coi kiểm tra tự động là bằng chứng tự code |


Nhận xét tổng hợp từ kết quả (AI hỗ trợ): Accuracy cao có thể che việc bỏ lọt spam, nên cần xem precision, recall và F1 cùng case lỗi. F1 trên tập cùng nguồn không bảo đảm xử lý lừa đảo mới: model tốt nhất trên test thường không phải model tốt nhất ở hard50. Chọn hệ thống cần xét đồng thời chất lượng, công suất, p95 và chi phí, không chỉ độ phức tạp của model.

Code Phần 1–3 và thử thách có AI hỗ trợ, đã khai báo đúng phạm vi thực tế; việc khai báo không thay thế yêu cầu tự code của đề.
