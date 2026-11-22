---
marp: true
theme: gaia
_class: lead
paginate: true
backgroundColor: #f8fafc
color: #0f172a
style: |
  section {
    font-family: 'Segoe UI', Arial, sans-serif;
    padding: 38px 50px;
    font-size: 20px;
    line-height: 1.45;
  }
  h1 {
    color: #1e3a8a;
    font-size: 28px;
    margin-bottom: 12px;
  }
  h2 {
    color: #1d4ed8;
    font-size: 22px;
    margin-bottom: 10px;
  }
  h3 {
    color: #0f766e;
    font-size: 19px;
  }
  strong {
    color: #0369a1;
  }
  table {
    font-size: 14px;
    width: 100%;
    border-collapse: collapse;
    margin-top: 8px;
  }
  th, td {
    border: 1px solid #cbd5e1;
    padding: 6px 10px;
  }
  th {
    background-color: #e2e8f0;
    color: #1e293b;
    font-weight: bold;
  }
  .highlight-box {
    background-color: #eff6ff;
    border-left: 5px solid #2563eb;
    padding: 8px 14px;
    margin-top: 10px;
    border-radius: 4px;
    font-size: 15px;
  }
  .footer-custom {
    position: absolute;
    bottom: 15px;
    left: 50px;
    right: 50px;
    display: flex;
    justify-content: space-between;
    font-size: 12px;
    color: #64748b;
    border-top: 1px solid #e2e8f0;
    padding-top: 4px;
  }
---

<!-- ========================================================================================= -->
<!-- SLIDE 1: TITLE & EXECUTIVE OVERVIEW                                                       -->
<!-- ========================================================================================= -->

# Disagreement-Aware Vision and Weight Sensor Fusion for AIoT Smart Shelf Inventory Monitoring

## Báo Cáo Định Hướng Nghiên Cứu Khóa Luận Tốt Nghiệp — Cột Mốc M0

* **Sinh viên thực hiện:** Kỹ sư Kỹ thuật Điện tử & Tự động hóa
* **Cán bộ hướng dẫn:** TS. Huỳnh Võ Trung Dũng
* **Thời gian báo cáo:** Tháng 10/2026 | **Học kỳ:** HK261

---

### Bối Cảnh Thực Tiễn & Tuyên Bố Vấn Đề (Problem Statement)
* **Bối cảnh bán lẻ thông minh (Smart Retail):** Nhu cầu tự động hóa kiểm kê hàng tồn kho trên kệ trưng bày thời gian thực nhằm tránh đứt gãy nguồn hàng (Out-of-Stock - OOS), giảm chi phí kiểm kê thủ công.
* **Nút thắt công nghệ hiện nay:**
  - **Camera quang học (Vision):** Bị che khuất vật lý khi hàng xếp nhiều tầng (Occlusion), nhạy cảm với ánh sáng biến đổi $\rightarrow$ Đếm thiếu hàng.
  - **Cảm biến tải trọng (Load Cell):** Không có thông tin ngữ nghĩa (không phân biệt các SKU có cùng trọng lượng), nhạy cảm với lực tỳ tay và rung chấn cơ học $\rightarrow$ Đếm sai SKU.
* **Mục tiêu cốt lõi của đề tài:** Xây dựng hệ thống hợp nhất đa phương thức **nhận biết bất đồng (Disagreement-Aware)** trên phần cứng AIoT nhúng chi phí thấp (ESP32 + Camera), tự động phát hiện, phân loại và hóa giải xung đột giữa mắt nhìn và cân nặng để duy trì độ chính xác kiểm kê tối đa.

<div class="footer-custom">
  <span>Khóa Luận Tốt Nghiệp — Ngành Kỹ Thuật Điện Tử & Tự Động Hóa</span>
  <span>Milestone M0: Project Kickoff & Direction</span>
</div>

<!-- 
SPEAKER NOTES (SLIDE 1):
Kính thưa Thầy hướng dẫn TS. Huỳnh Võ Trung Dũng, hôm nay em xin phép trình bày báo cáo định hướng nghiên cứu cho khóa luận tốt nghiệp: "Disagreement-Aware Vision and Weight Sensor Fusion for AIoT Smart Shelf Inventory Monitoring".
Vấn đề xuất phát từ thực tế các kệ hàng siêu thị hiện đại: nếu chỉ dùng Camera thì khi chai nước phía trước che chai nước phía sau, camera vật lý hoàn toàn không nhìn thấy. Ngược lại, nếu chỉ dùng cân điện tử dưới đáy kệ thì cân chỉ biết tổng khối lượng chứ không biết khách hàng vừa lấy lon Coca hay lon Pepsi nếu chúng nặng bằng nhau (330g), chưa kể khách hàng tỳ tay lên kệ sẽ làm cân nhảy sai. Đề tài của em đề xuất giải pháp hợp nhất hai giác quan này nhưng có cơ chế "nhận thức bất đồng" (Disagreement-Aware) để xử lý mâu thuẫn một cách thông minh, chạy trên nền tảng AIoT chi phí thấp.
-->

---

<!-- ========================================================================================= -->
<!-- SLIDE 2: RESEARCH GAPS & DISAGREEMENT TAXONOMY                                            -->
<!-- ========================================================================================= -->

# Khoảng Trống Nghiên Cứu & Phân Loại Bất Đồng Cảm Biến

## Tại sao các phương pháp hợp nhất truyền thống thất bại trong thực tế?

* **Hạn chế của tài liệu hiện nay:** Các công trình đi trước (FAIM 2020, Chen EKF 2021, Zhang 2023) thường dùng **luật cứng (rule-based)** hoặc **cộng trọng số tuyến tính cố định ($\alpha V + \beta W$)**, coi sự lệch pha giữa camera và cân là "nhiễu ngẫu nhiên Gauss".
* **Thực tế:** Sự bất đồng giữa 2 cảm biến mang **thông tin chẩn đoán cốt lõi** về một biến cố vật lý đang diễn ra trên kệ hàng:

| Tình huống thực tế trên kệ | Trạng thái Vision (YOLOv8) | Trạng thái Cân (Load Cell) | Xung đột & Rủi ro | Cơ chế Disagreement-Aware của Đề tài |
| :--- | :--- | :--- | :--- | :--- |
| **Che khuất tầng sâu (Occlusion)** | $N_V = 2$ (Chỉ thấy hàng ngoài) | $N_W = 4$ (Đủ khối lượng 4 chai) | Vision đếm thiếu hàng | Xác định che khuất $\rightarrow$ **Ủy quyền cho Cân quyết định**. |
| **Khách tỳ tay / Va chạm cơ học** | $N_V = 3$ (Hàng không đổi) | $N_W = 7$ (Khối lượng tăng vọt) | Cân báo hàng giả mạo | Nhận diện xung lực đột biến $\rightarrow$ **Ủy quyền cho Vision**. |
| **Tráo đổi SKU cùng khối lượng** | Thấy lon Sprite màu xanh | Cân chỉ đo 330g vô danh | Cân không biết nhãn hàng | **Vision định danh SKU, Cân xác nhận số lượng**. |
| **Môi trường suy biến (Tối/Lóa)** | BBox confidence tụt sâu ($<0.3$) | $N_W = 3$ (Cân ổn định 990g) | Báo hết hàng ảo | Phân tích chất lượng ảnh $\rightarrow$ **Weight-Dominant mode**. |

<div class="highlight-box">
  <strong>Đóng góp học thuật:</strong> Chuyển trạng thái bất đồng từ "vấn đề gây lỗi" thành "tín hiệu chẩn đoán" để điều chỉnh độ tin cậy thích nghi (Adaptive Trust Weights) theo thời gian thực.
</div>

<div class="footer-custom">
  <span>Literature Review Matrix: 10 bài báo quốc tế (IEEE TPAMI, CVPR, IoT-J, MDPI Sensors, ACM TOSN)</span>
  <span>Trang 2 / 5</span>
</div>

<!-- 
SPEAKER NOTES (SLIDE 2):
Điểm mới mấu chốt của đề tài so với các nghiên cứu trước nằm ở slide này: Trước đây người ta thường giả định hoặc camera đúng, hoặc cân đúng, hoặc lấy trung bình cộng. Nhưng trong thực tế bán lẻ:
Khi bị che khuất: Camera chỉ thấy 2 lon, nhưng cân đo đủ 4 lon -> Nếu lấy trung bình sẽ ra 3 lon (sai cả hai!). Thuật toán của em nhận biết dấu hiệu che khuất (bounding box đè nhau) để chủ động tin tưởng cảm biến Cân.
Ngược lại khi khách hàng tỳ tay lên thành kệ: Cân đột ngột nhảy vọt lên 7 lon, nhưng camera thấy vẫn nguyên 3 lon và độ ổn định cao -> Thuật toán lọc bỏ xung lực và tin tưởng Camera.
Đây chính là "Disagreement Taxonomy" mà em đã xây dựng trong ma trận tài liệu tham khảo.
-->

---

<!-- ========================================================================================= -->
<!-- SLIDE 3: RESEARCH QUESTIONS & TECHNICAL CONTRIBUTIONS                                     -->
<!-- ========================================================================================= -->

# Câu Hỏi Nghiên Cứu (RQ1–RQ3) & Đóng Góp Kỹ Thuật

## Định hình phạm vi khoa học và mục tiêu định lượng của khóa luận

### Hệ thống 3 Câu hỏi Nghiên cứu Cốt lõi (Research Questions)
* **RQ1 — Visual Inventory Estimation Baseline:**
  Mô hình thị giác nhẹ (YOLOv8n) đạt độ chính xác phát hiện và đếm vật thể trên kệ hàng siêu thị ở mức nào dưới các điều kiện mật độ dày, che khuất tầng sâu và ánh sáng biến động?
* **RQ2 — Disagreement Characterization & Modeling:**
  Làm thế nào để định lượng độ bất đồng giữa số lượng thị giác $N_V$ và khối lượng vi phân $N_W$ thông qua chỉ số $\delta_t = |N_{V,t} - N_{W,t}|$ và phân loại chính xác các chế độ xung đột vật lý?
* **RQ3 — Disagreement-Aware Multimodal Fusion Evaluation:**
  Thuật toán Hợp nhất Nhận thức Bất đồng Hai tầng (Two-Tier Adaptive Fusion) cải thiện sai số đếm hàng (giảm MAE, RMSE) và triệt tiêu cảnh báo sai (False Restock Alerts) ở mức độ nào so với các phương pháp đơn cảm biến và hợp nhất thông thường?

---

### Các Đóng Góp Kỹ Thuật Dự Kiến (Technical Contributions)
1. **Thuật toán Hai tầng (Two-Tier Engine):** Tầng 1 (Fast-Path) xác nhận nhanh khi 2 cảm biến đồng thuận; Tầng 2 (Diagnostic Tier) kích hoạt khi phát hiện bất đồng để tái phân bổ trọng số thích nghi.
2. **Bộ tiêu chí đánh giá chuẩn hóa:** Đã hoàn thành module `fusion/metrics.py` đo lường toàn diện: MAE, RMSE, Exact Match %, Disagreement Rate, False Alarm Rate, Latency.
3. **Hiện thực hóa AIoT thực tế:** Tối ưu hóa pipeline chạy trên phần cứng nhúng biên với tổng độ trễ dưới 50 ms.

<div class="footer-custom">
  <span>Đặc tả chi tiết tại: thesis/problem_statement_and_research_questions.md</span>
  <span>Trang 3 / 5</span>
</div>

<!-- 
SPEAKER NOTES (SLIDE 3):
Về mặt học thuật, đề tài được xây dựng xoay quanh 3 câu hỏi nghiên cứu chặt chẽ:
RQ1: Thiết lập đường cơ sở (Baseline) để chứng minh thị giác YOLOv8n đơn lẻ sẽ gãy đổ ở đâu khi bị che khuất và lóa sáng.
RQ2: Là trái tim giải thuật - làm sao toán học hóa độ bất đồng delta = |Nv - Nw| và kết hợp với ma trận phương sai để chẩn đoán đúng nguyên nhân xung đột.
RQ3: Là thực nghiệm đối sánh - chứng minh thuật toán Disagreement-Aware của em vượt trội hơn các mô hình Vision-only, Weight-only và Static-fusion về cả chỉ số đếm đúng MAE lẫn việc không phát cảnh báo ảo.
Cả 3 RQ này đều gắn liền với bộ kiểm thử 69 test tự động em đã viết sẵn trong repo.
-->

---

<!-- ========================================================================================= -->
<!-- SLIDE 4: SYSTEM ARCHITECTURE, BOM v1 & MVP SCOPE                                          -->
<!-- ========================================================================================= -->

# Kiến Trúc Hệ Thống AIoT, Thiết Kế Phần Cứng BOM v1 & MVP

## Hiện thực hóa mô hình Kệ hàng Thông minh chi phí thấp

```
  [ KỆ THỰC NGHIỆM ]                   [ TẦNG THU THẬP BIÊN ]               [ TẦNG XỬ LÝ & HỢP NHẤT ]               [ DASHBOARD & DIGITAL TWIN ]
 ┌──────────────────────┐             ┌─────────────────────────┐          ┌────────────────────────────┐          ┌───────────────────────────┐
 │ 3 Ngăn Kệ (Slots)    │             │ ESP32 Node              │  MQTT    │ Gateway / Edge Host        │  State   │ Streamlit Dashboard       │
 │ • Slot 1: Nước ngọt  │──(Lực)────> │ • 4x Cảm biến tải 5kg   │────────> │ • YOLOv8n Object Detector  │────────> │ • Bản sao số Digital Twin │
 │ • Slot 2: Sữa hộp    │             │ • Mạch ADC 24-bit HX711 │ (JSON)   │ • Disagreement Engine      │ Updates  │ • Đèn cảnh báo xung đột đỏ│
 │ • Slot 3: Bánh snack │──(Quang)──> │ Web Camera HD 1080p     │          │ • Two-Tier Arbiter (50ms)  │          │ • Biểu đồ telemetry live  │
 └──────────────────────┘             └─────────────────────────┘          └────────────────────────────┘          └───────────────────────────┘
```

* **Phần cứng BOM v1 (Bill of Materials):**
  - **Vi điều khiển & Cảm biến:** 1x ESP32 DevKit V1, 4x Load cell thanh 5kg, 2x Mạch khuếch đại HX711, 1x Camera USB góc rộng 1080p.
  - **Khung cơ khí & Kết nối:** Khung mica/nhôm định hình 3 ô chia ngăn, đệm cao su giảm chấn, dây bọc chống nhiễu.
  - **Tổng ngân sách dự toán:** **~1.500.000 VNĐ** (Đã lên danh mục 13 linh kiện chi tiết tại `docs/hardware/BOM_v1_Smart_Shelf_Components.xlsx`).
* **Phạm vi mô hình MVP (Minimum Viable Product):**
  - Kệ hàng 1 tầng chia làm 3 slot độc lập, kích thước mỗi slot $15 \times 20\text{ cm}$.
  - Giám sát 3 danh mục SKU chuẩn: Lon nước ngọt 330ml (355g), Hộp sữa 180ml (195g), Gói snack (50g).
  - Tần số cập nhật trọng lượng 10Hz, Camera xử lý 5–15 FPS, trễ phản hồi toàn hệ thống $< 200\text{ ms}$.

<div class="footer-custom">
  <span>Tài liệu thiết kế: docs/architecture.md | docs/hardware/BOM_v1_Smart_Shelf_Components.xlsx</span>
  <span>Trang 4 / 5</span>
</div>

<!-- 
SPEAKER NOTES (SLIDE 4):
Về mặt triển khai kỹ thuật, hệ thống chia làm 4 khối rõ ràng:
Khối 1: Kệ hàng vật lý gồm 3 slot độc lập, dưới mỗi slot là một module cân riêng biệt.
Khối 2: Node AIoT ESP32 đọc dữ liệu từ cảm biến tải qua chip ADC 24-bit HX711 và gửi dữ liệu telemetry qua MQTT JSON.
Khối 3: Tầng xử lý trung tâm chạy trên Edge PC/Gateway thực hiện đồng thời nhận diện YOLOv8n và chạy bộ lọc Disagreement-Aware Fusion Engine.
Khối 4: Dashboard Digital Twin viết bằng Streamlit hiển thị trực quan trạng thái kệ 3D, khi có bất đồng sẽ bật đèn cảnh báo đỏ và ghi nhận nguyên nhân.
Toàn bộ danh mục linh kiện BOM v1 em đã khảo sát thực tế và lập bảng Excel với tổng kinh phí chỉ khoảng 1.5 triệu đồng, rất khả thi cho cấp độ khóa luận tốt nghiệp.
-->

---

<!-- ========================================================================================= -->
<!-- SLIDE 5: 12-MONTH ROADMAP, MILESTONES (M0–M7) & SPRINT KICKOFF                           -->
<!-- ========================================================================================= -->

# Lộ Trình 12 Tháng, 7 Cột Mốc (M0–M7) & Kế Hoạch Nghiệm Thu

## Kế hoạch hành động chi tiết theo tiêu chuẩn Khóa luận Tốt nghiệp

```
  Giai đoạn 1: Nền tảng & Khảo sát         Giai đoạn 2: Thu thập & Huấn luyện       Giai đoạn 3: Hợp nhất & Đối sánh          Giai đoạn 4: Đóng gói & Bảo vệ
 [Tháng 10 - 11/2026]                    [Tháng 12/2026 - 02/2027]                 [Tháng 03 - 05/2027]                      [Tháng 06 - 08/2027]
  ├─ Sprint 01-02: Literature, BOM v1,    ├─ Sprint 03-05: Ráp kệ, Boot ESP32,      ├─ Sprint 13-15: Two-Tier Fusion Engine, ├─ Sprint 19-21: Chạy demo kệ thực tế,
  │  Metrics design, M0 Presentation      │  Dataset collection (300 ảnh + cân)     │  Benchmark MAE/RMSE vs Baselines        │  Tối ưu hóa độ trễ thời gian thực
  └─ [MỐC M0]: Cố vấn duyệt RQ & Scope ──> ├─ Sprint 06-08: Gán nhãn, Fine-tune     └─ [MỐC M3, M4]: Hoàn thiện thuật toán ──> ├─ Sprint 22-26: Viết toàn văn luận văn,
                                          │  YOLOv8n trên tập thực tế               và đo đạc đầy đủ các ablation test        │  Gửi bài báo khoa học, Slide bảo vệ
                                          └─ [MỐC M1, M2]: Dataset & Vision Model                                             └─ [MỐC M5, M6, M7]: BẢO VỆ TỐT NGHIỆP
```

* **Mục tiêu nghiệm thu buổi họp M0 (Definition of Done - DoD):**
  1. $\checkmark$ Thầy hướng dẫn thông qua **Định hướng nghiên cứu**, **Tuyên bố vấn đề** và **Hệ thống 3 câu hỏi RQ1–RQ3**.
  2. $\checkmark$ Thầy phê duyệt **Phạm vi MVP** (Kệ 3 ô, 3 SKU, Camera + 4 Load Cell) và danh mục linh kiện **BOM v1**.
  3. $\checkmark$ Nghiệm thu tài liệu **Literature Matrix** (10 bài báo quốc tế) và bản nháp **Related Work 1 trang** (S02-W1).
* **Kế hoạch hành động ngay sau M0 (Sprint 03):**
  - Tiến hành mua sắm linh kiện theo bảng BOM v1 và gia công khung cơ khí kệ 3 ô.
  - Viết firmware ESP32 đọc cảm biến HX711, thiết lập hàm hiệu chuẩn (Calibration) và trừ bì (Tare).

<div class="footer-custom">
  <span>Kế hoạch chi tiết 12 tháng: Plan/Thesis_12_Month_Plan.pdf | docs/related_work_draft.md</span>
  <span>Trang 5 / 5</span>
</div>

<!-- 
SPEAKER NOTES (SLIDE 5):
Để đảm bảo đề tài về đích đúng hạn với chất lượng học thuật cao nhất, em đã lập bản kế hoạch chi tiết 12 tháng chia làm 26 Sprint và 7 Cột mốc nghiệm thu lớn từ M0 đến M7:
Hôm nay là Cột mốc M0: Em kính trình Thầy duyệt qua Phạm vi MVP, 3 câu hỏi RQ và danh mục BOM v1.
Sau khi được Thầy góp ý và phê duyệt, trong Sprint 03 em sẽ giải ngân mua linh kiện, dựng khung kệ vật lý và lập trình firmware đọc cảm biến lực.
Em xin chân thành cảm ơn Thầy và rất mong nhận được những ý kiến đóng góp quý báu từ Thầy để hoàn thiện đề tài!
-->
