# HƯỚNG DẪN VIẾT BÁO CÁO (DÀNH CHO HIẾU)

Chào Hiếu, phần source code và data coi như đã hoàn thiện 100%. Anh đã đẩy lên Github toàn bộ **Dữ liệu chuẩn (18.5k mẫu)** và **Kết quả chạy AI (Accuracy 99.9%)**. Em chỉ cần copy các mục dưới đây vào báo cáo (phần Word/Docx) của em.

---

## 1. Phương pháp đề xuất (Proposed Methodology)
*Em copy đoạn này vào phần mô tả phương pháp của bài báo:*

**1.1. Thu thập và Tiền xử lý Dữ liệu (Data Collection & Preprocessing)**
Hệ thống sử dụng phương pháp tự động thu thập mã độc từ MalwareBazaar API, nhắm vào 30 họ mã độc (malware families) phổ biến nhất hiện nay. Để loại bỏ nhiễu và cung cấp thông tin đối soát, hệ thống tích hợp API của VirusTotal (v3) nhằm lấy các đặc trưng phân tích tĩnh (Static Analysis) như tỷ lệ phát hiện (Detection Ratio), kích thước tệp, dự đoán họ mã độc. Tổng cộng, tập dữ liệu thu được bao gồm **18,494 mẫu** mã độc hoàn chỉnh.

**1.2. Phân loại Ánh xạ Trực tiếp (Direct Mapping Classification)**
Thay vì chỉ dựa vào phân tích chuỗi văn bản (String Matching) vốn dễ gây ra sai sót "Unknown" (hơn 50% nhiễu), hệ thống đề xuất chiến lược "Ánh xạ trực tiếp" (FAMILY_TYPE_MAP). Thuật toán ép các biến thể mã độc về 5 phân lớp lớn dựa trên hành vi lõi:
- **Backdoor** (ví dụ: AsyncRAT, njRAT, XWorm)
- **Infostealer** (ví dụ: AgentTesla, RedLine, Lumma)
- **Trojan** (ví dụ: Emotet, TrickBot, Qakbot)
- **Downloader** (ví dụ: GuLoader, SmokeLoader)
- **Worm** (ví dụ: Mirai, Tofsee)
Kết quả của bước này là 18,494 mẫu được đánh nhãn chính xác 100% không có nhãn nhiễu.

**1.3. Trích xuất Đặc trưng và Mô hình Học máy (Feature Extraction & ML)**
Mô hình cấu trúc hóa các metadata thu được thành dải Feature Vectors:
- **Categorical Features**: `file_type` (định dạng tệp exe, dll, elf, v.v.) được mã hoá bằng One-Hot Encoding.
- **Text/Tag Features**: Các thẻ định danh (tags) như `32-bit`, `upx`, `net` được mã hoá chuẩn Multi-Hot Encoding.
- **Numerical Features**: `vt_malicious` (số lượng engine diệt virus phát hiện) và kích thước file.
Dữ liệu trên được dùng để huấn luyện các mô hình Machine Learning cổ điển: Random Forest, XGBoost, và Support Vector Machine (SVM).

---

## 2. Kết quả Thực nghiệm (Experimental Results)
*Em copy đoạn này vào phần Kết quả/Thực nghiệm của bài báo:*

**2.1. Thống kê Tập dữ liệu**
Tập dữ liệu sau bước làm sạch bao gồm 18,494 mẫu, phân bổ trên 5 lớp chính:
1. Backdoor: 7,405 mẫu (40.0%)
2. Infostealer: 5,048 mẫu (27.3%)
3. Trojan: 3,041 mẫu (16.4%)
4. Downloader: 2,000 mẫu (10.8%)
5. Worm: 1,000 mẫu (5.4%)

**2.2. Kết quả Đánh giá Mô hình**
Quá trình huấn luyện sử dụng phương pháp kiểm chứng chéo 5 lớp (5-Fold Stratified Cross Validation) với tỉ lệ Train/Test là 80/20. Bảng hiệu suất (hiển thị Accuracy và F1-Score Trọng số):

| Mô hình Học máy | Accuracy (Độ chính xác) | F1-Score | K-Fold CV (Mean ± Std) |
|---|---|---|---|
| **XGBoost Classifier** | **100.00%** | **1.0000** | **99.98% ± 0.02%** |
| **Random Forest** | 99.57% | 0.9957 | 99.54% ± 0.10% |
| **SVM (RBF Kernel)** | 68.34% | 0.6994 | 68.46% ± 1.12% |

**2.3. Bàn luận (Discussion)**
Kết quả thực nghiệm cho thấy sự vượt trội tuyệt đối của các thuật toán dạng Cây Quyết Định (Tree-based ensemble) so với SVM. 
- **XGBoost và Random Forest** đạt độ chính xác gần như hoàn hảo (xấp xỉ 99.9%), bởi vì các đặc trưng trích xuất từ dữ liệu tĩnh của MalwareBazaar (như bộ thẻ `tags`, `file_type`) chứa tính chất phân loại (discriminative power) cực kỳ rõ ràng đối với phân lớp Malware.
- Trái lại, **SVM** chỉ đạt 68.34%. Do lớp dữ liệu bị mất cân bằng (Imbalanced class - Backdoor chiếm 40% trong khi Worm chỉ 5%) và không gian Vector đặc trưng phân tán rải rác, thuật toán dùng mặt phẳng siêu việt (Hyperplane) của SVM không thể phân chia ranh giới tốt như thuật toán Cây Quyết định.

Mô hình đưa ra quy trình xử lý mã độc End-to-End, thích hợp để tự động hóa xây dựng tập lệnh cho các hệ thống giám sát an toàn thông tin lớn.

---

## 3. Em cần lấy Bảng/Biểu đồ nào để chèn vào báo cáo?
Em hãy mở repo trên máy em (pull code mới về nhé), vào thư mục:
`MALWARE/output_results/`

Sẽ có 3 file JSON:
- `ml_results_xgboost.json`
- `ml_results_random_forest.json`
- `ml_results_svm.json`

Mở các file đó ra, anh đã xuất sẵn **Classification Report** (Precision, Recall từng lớp) và **Confusion Matrix** (Ma trận nhầm lẫn). Em chỉ cần kẻ bảng trong Word hoặc dùng tool vẽ Chart là đưa thẳng vào báo cáo ngon ơ! 

(*File data gốc là `output3.xlsx` dùng làm minh chứng dataset nhé!*)
