# Phát Hiện Bình Luận Ngôn Từ Thù Ghét Và Giám Sát, Phát Hiện Chủ Đề Bị Tấn Công 

## 1. Giới thiệu chung
Dự án này là đồ án môn học SE363, tập trung xây dựng một quy trình tự động phân tích bình luận tiếng Việt trên mạng xã hội nhằm phát hiện và phân loại chủ đề thảo luận. 
Mục tiêu cốt lõi là nhận diện các cuộc tấn công tập thể (collective attacks), đi sâu vào phân loại 19 loại hình tấn công chi tiết (như Regionalism, Body Shaming, Politics,...)

## 2. Thông tin nhóm thực hiện
* **Giảng viên hướng dẫn:** TS.Đỗ Trọng Hợp 
***Thành viên phát triển:**
  *Võ Anh Quân - 22521192
  * Võ Minh Quyền - 22521227

## 3. Kiến trúc hệ thống(img/pipeline.png)
Hệ thống sử dụng kiến trúc lai để tối ưu hóa tài nguyên phần cứng, đặc biệt là tránh lỗi tràn bộ nhớ (Out of Memory) trên các cụm Spark:
* **Hạ tầng Dữ liệu (Docker Environment):** Các thành phần như Kafka Broker, Apache Spark Streaming và MongoDB được đóng gói và chạy trên Docker Containers.
* **Model Server (FastAPI):** Mô hình Deep Learning được tách ra thành một API Service độc lập bằng FastAPI chạy trên máy chủ (Local), giúp chỉ load model một lần duy nhất.
* **Giao diện giám sát:** Sử dụng Streamlit để xây dựng Dashboard cảnh báo.

## 4. Tập dữ liệu & Tiền xử lý(img/type_attack.png)
* **Dataset:** Gồm 15.298 câu bình luận thực tế được thu thập từ Facebook, YouTube và Reddit.
* **Tiền xử lý:**
  * Khôi phục dấu tiếng Việt tự động bằng mô hình XLM-Roberta (vietnamese-accent-marker).
  * Chuẩn hóa từ viết tắt, teencode và loại bỏ nhiễu (URL, emoji).
  * Tách từ (Word Segmentation) bằng thư viện PyVi và gán nhãn từ loại bằng Underthesea.
(img/Phan_phoi.png)
## 5. Kiến trúc Mô hình
Hệ thống triển khai hai kiến trúc mô hình học sâu dựa trên nền tảng Transformers:
1. **Mô hình Lai Multi-task (PHOBERT-CNN-BIGRU):** Tận dụng khả năng hiểu ngữ nghĩa của PhoBERT, trích xuất đặc trưng cục bộ bằng CNN đa tỷ lệ (Multi-scale) và nắm bắt ngữ cảnh tuần tự bằng Bi-GRU. Mô hình phân loại đồng thời 3 khía cạnh: Cá nhân (Individual), Nhóm (Group), và Xã hội (Societal).
2. **Mô hình Phân loại Đơn nhiệm:** Sử dụng PhoBERT kết hợp mạng nơ-ron tuyến tính để thực hiện bài toán phân loại đa nhãn cho 19 loại hình tấn công. 

## 6. Kết quả thực nghiệm
* Mô hình Type Attack (Đa nhãn) đạt Hamming Loss rất thấp (0.0613) và F1-Micro tối ưu là 0.6011 tại ngưỡng 0.3.
* Kiến trúc lai cho thấy sự vượt trội so với các phương pháp Machine Learning truyền thống (như SVM kết hợp TF-IDF) trong việc xử lý ngôn ngữ nhiễu và hiểu ngữ cảnh tiếng Việt.

---

## 7. Hướng dẫn cài đặt & Khởi chạy (Installation & Usage)

### Yêu cầu hệ thống (Prerequisites)
* Docker & Docker Compose
* Python 3.8+

### Các bước khởi chạy
**Bước 1: Khởi động hạ tầng dữ liệu (Kafka, Spark, MongoDB)**
```bash
# Di chuyển vào thư mục chứa file docker-compose.yml
docker-compose up -d
```
**Bước 2: Khởi chạy Model Server (FastAPI)**

```bash
# Mở một terminal mới, kích hoạt môi trường ảo (virtual environment)
pip install -r requirements.txt
# Chạy server FastAPI
uvicorn main:app --host 0.0.0.0 --port 8000
```
**Bước 3: Khởi chạy Giao diện Dashboard (Streamlit)**

```bash
# Mở một terminal mới
streamlit run dashboard.py
