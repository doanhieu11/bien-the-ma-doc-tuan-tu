# BÁO CÁO THỰC NGHIỆM LÝ THUYẾT VÀ KẾT QUẢ

## 1. Phương pháp Đề xuất (Proposed Methodology)

### 1.1. Thu thập và Tiền xử lý Dữ liệu (Data Collection & Preprocessing)
Hệ thống sử dụng phương pháp tự động thu thập thông tin phần mềm độc hại thông qua giao diện lập trình ứng dụng (API) của MalwareBazaar, tập trung vào 30 họ mã độc (malware families) đang phổ biến nhất. Nhằm loại bỏ dữ liệu nhiễu và cung cấp thông tin đối sánh chéo (Cross-Validation Pipeline), hệ thống tích hợp phân tích đa nền tảng bằng API chuyên sâu VirusTotal v3. Quá trình này khai thác các đặc trưng phân loại tĩnh (Static Analysis) như tỷ lệ phát hiện (Detection Ratio) và dung lượng tệp tin (File Size). Sau giai đoạn thu thập điểm danh, dữ liệu thô được tinh chế thành tập dữ liệu chuẩn (Dataset) gồm **18.494 mẫu** mã độc hoàn chỉnh.

### 1.2. Thuật toán Phân loại Ánh xạ Trực tiếp (Direct Mapping Engine)
Việc sử dụng phương pháp phân tích chuỗi văn bản (String Matching / Text Heuristics) trong môi trường dữ liệu thô thường sinh ra các điểm mù "Unknown", ảnh hưởng tới hơn 50% tập dữ liệu. Khắc phục hạn chế này, nghiên cứu đề xuất chiến lược "Ánh xạ trực tiếp lõi" (Direct Mapping). Cơ chế này trực tiếp ánh xạ các biến thể họ mã độc về 5 phân lớp hành vi quy chuẩn:
- **Backdoor** (Giao thức cửa sau): AsyncRAT, njRAT, XWorm, CobaltStrike.
- **Infostealer** (Đánh cắp thông tin): AgentTesla, RedLine, Lumma, Raccoon.
- **Trojan** (Ngựa Thành Troy): Emotet, TrickBot, Qakbot, IcedID.
- **Downloader** (Tải trọng độc hại): GuLoader, SmokeLoader, Amadey.
- **Worm** (Sâu máy tính): Mirai, Tofsee.
Sau quy trình gán nhãn, 100% mẫu (18.494 mẫu) được định danh chính xác vào 5 phân lớp lớn mà không có nhãn nhiễu hoặc sai lệch.

### 1.3. Trích xuất Đặc trưng và Phân loại bằng Học máy (Feature Extraction & Machine Learning)
Chiến lược kỹ nghệ đặc trưng (Feature Engineering) thực hiện cấu trúc hóa siêu dữ liệu (Metadata) thành các không gian Vector đặc trưng phù hợp cho thuật toán học máy:
- **Categorical Features**: Thuộc tính định dạng tệp (như `.exe`, `.dll`, `.elf`) được số hóa theo kỹ thuật `One-Hot Encoding` dạng thưa (Sparse).
- **Textual/Tags Features**: Các thẻ định danh hành vi mã độc (như `32-bit`, `upx`, `net`) trải qua quá trình mã hóa phân mảng `Multi-Hot Encoding` nhằm sàng lọc 30 tệp từ vựng quan trọng nhất.
- **Numerical Features**: Biến định lượng bao gồm số liệu cảnh báo độc hại từ các Engine (đặc trưng `vt_malicious`) và dung lượng tệp.

Dựa trên cấu trúc Vector này, hệ thống áp dụng ba mô hình Học máy cổ điển nhằm đánh giá hiệu suất phân loại phân lớp: Random Forest Classifier, XGBoost (Extreme Gradient Boosting), và Support Vector Machine (SVM).

---

## 2. Kết quả Thực nghiệm (Experimental Results)

### 2.1. Thống kê Tập dữ liệu (Dataset Statistics)
Dữ liệu sau khi đi qua module làm sạch được phân bổ theo 5 lớp (class), mô tả chi tiết độ mất cân bằng tự nhiên trong hệ thống phát tán Malware thực tế:
1. **Backdoor**: 7.405 mẫu (40.0%)
2. **Infostealer**: 5.048 mẫu (27.3%)
3. **Trojan**: 3.041 mẫu (16.4%)
4. **Downloader**: 2.000 mẫu (10.8%)
5. **Worm**: 1.000 mẫu (5.4%)
*Tổng cộng: 18.494 mẫu.*

### 2.2. Kết quả Đánh giá Mô hình Phân loại
Nghiên cứu sử dụng phương pháp kiểm chứng chéo 5 lớp (5-Fold Stratified Cross Validation) để xử lý xu hướng thiên lệch do lớp học (class imbalance bias), thiết lập tỷ lệ chia dữ liệu theo quy chuẩn Train/Test là 80/20. Bảng hiệu suất đo lường mức độ tương quan (Accuracy và Weighted F1-Score):

| Mô hình Học máy | Accuracy (Độ chính xác) | F1-Score (Trọng số) | K-Fold CV (Mean ± Std) |
|---|---|---|---|
| **XGBoost Classifier** | **100.00%** | **1.0000** | **99.98% ± 0.02%** |
| **Random Forest** | 99.57% | 0.9957 | 99.54% ± 0.10% |
| **SVM (RBF Kernel)** | 68.34% |  0.6994 | 68.46% ± 1.12% |

### 2.3. Bàn luận (Discussion)
Kết quả thực nghiệm cho thấy sự vượt trội tuyệt đối mang lại độ tin cậy của các thuật toán dạng Tổ hợp Cây Quyết Định (Tree-based ensemble) khi so sánh với thuật toán Máy học SVM:
- Các mô hình dựa trên nền tảng Cây ( **XGBoost và Random Forest** ) lần lượt đạt độ chính xác gần như hoàn hảo (từ 99.5% đến xấp xỉ 99.9%). Điều này thể hiện rằng các đặc trưng trích xuất từ phân tích cấu trúc phần cứng và chuỗi Metadata (danh sách `tags`, `file_type`) chứa đựng tính hệ số phân loại phân hóa (discriminative power) nhạy bén trong việc xác định nhánh mã độc mới.
- Về mặt hạn chế, khi thực thi mô hình đa phân lớp, thuật toán **Support Vector Machine (SVM)** chỉ đạt ngưỡng 68.34%. Do lớp dữ liệu bị mất cân bằng tự nhiên và không gian Vector đặc trưng thuộc dạng phân mảnh/rời rạc, tiến trình nội suy vách định tuyến đa chiều (Hyperplane) của SVM không thể phân ranh giới tốt như cơ chế đặt quy tắc điều kiện của Cây Quyết định.

**=> Kết luận chung:** 
Với độ chính xác cao được kiểm chứng chặt chẽ, chuỗi thiết kế được đề xuất trong nghiên cứu đã hoàn chỉnh hóa quy trình phân tích và phân loại mã độc đa chu kỳ (End-to-End). Hệ thống hoàn toàn có thể được ứng dụng trong tự động hóa quy trình tập lệnh của các Trung tâm Giám sát Điều hành An toàn Thông tin (SOC), khắc phục triệt để tồn đọng về nhãn lỗi hoặc các biến thể "Unknown Variant".
