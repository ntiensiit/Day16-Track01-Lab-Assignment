---
artifact: 01 — Case Analysis
bai-tap: Lab 1 — Phân tích "tử huyệt" chiến lược
format: Cá nhân trước → share trong bàn → chốt verdict cuối
nop-cuoi: Có — đây là file nộp cuối của Lab 1
---

# Lab 1 — Case Analysis / Phân tích "tử huyệt" chiến lược

**Case đã chọn:** Manual real-image capture & labeling workflow cho robotics object detection  
**Người làm:** Nguyễn Tiến Sỉ — 2A202600681  
**Bàn / nhóm bàn:** Team 013 / repo review  
**Ngày:** 2026-06-18

## Bước 0 — Chọn case thật nhanh

- [x] Có một **AI shock** hoặc mốc đổi cục diện đủ rõ
- [x] Có thể tìm được ít nhất **3-5 bằng chứng công khai / repo evidence**
- [x] Có tác động đủ nhìn thấy được ở user / usage / vị thế cạnh tranh / workflow
- [x] Có thể trả lời câu hỏi: **"Điều gì đã thay đổi vĩnh viễn?"**

- **Case / sản phẩm / công ty:** Quy trình chụp ảnh thật + gán nhãn thủ công cho dataset robotics object detection, thường dùng CVAT/Label Studio/Roboflow/manual labeling hoặc tự chụp lab dataset.
- **AI / platform / sản phẩm mới tạo áp lực:** Synthetic data pipeline kết hợp PyBullet simulation, diffusion realism, Grounding DINO QA gate và YOLO benchmark trong repo `C2-App-013`.
- **Vì sao tôi chọn case này?**  
  > Đây là case sát nhất với dự án nhóm. Repo `C2-App-013` không chỉ build app mà còn tự chứng minh một workflow mới: thay vì bắt đầu bằng chụp ảnh thật và gán nhãn thủ công, người dùng có thể sinh dữ liệu từ simulation, làm realism bằng GenAI, kiểm tra giữ nhãn bằng QA gate, rồi đo mAP trên ảnh thật.

---

## Bước 1 — Gom 3-5 bằng chứng chốt

| # | Bằng chứng / số liệu chốt | Vì sao số này quan trọng? | Nguồn |
|---|---|---|---|
| E1 | YCB-Video/PoseCNN có 133,827 frame; repo dùng bằng chứng này để chỉ ra real-image collection + labeling là data bottleneck. | Cho thấy trước GenAI/synthetic pipeline, robotics perception phụ thuộc nhiều vào bộ dữ liệu thật rất tốn công thu thập và gán nhãn. | `C2-App-013/docs/AI20K-170_prd.md` mục 2.2; Xiang et al. 2018 PoseCNN/YCB-Video |
| E2 | Repo định nghĩa user need: robotics perception engineer cần tạo dataset object detection có bbox đáng tin để train/evaluate detector trên real-camera imagery mà không lặp lại quy trình chụp–label thủ công. | Xác định đúng JTBD của case: không phải “tạo ảnh đẹp”, mà là tạo dữ liệu huấn luyện đáng tin để model chạy tốt trên camera thật. | `C2-App-013/docs/AI20K-170_prd.md` mục 5 |
| E3 | Pipeline P1/P2 sinh RGB, 16-bit depth, canny, YOLO labels, metadata từ seed; P1 tạo 5 góc camera × 100 yaw = 500 frames/object. | Đây là điểm phá workflow cũ: dataset được tạo có cấu trúc, lặp lại được, có label sẵn, không cần chụp và label từng ảnh từ đầu. | `C2-App-013/README.md`; `docs/AI20K-170_prd.md` mục 7.1 |
| E4 | QA gate dùng blur/artifact checks + Grounding DINO IoU; accept nếu IoU ≥ 0.85, flag 0.70–0.84, reject < 0.70 hoặc lỗi count/blur. | AI shock chỉ đáng tin khi có kiểm soát label drift; nếu ảnh đẹp nhưng nhãn sai thì dataset vô dụng. QA gate biến GenAI từ “ảnh đẹp” thành “training data có kiểm chứng”. | `C2-App-013/qa/gate.py`; `README.md` quality thresholds |
| E5 | Benchmark YOLO26n: `sim` đạt mAP50-95 = 0.823, cao hơn `lab_raw` = 0.775; `realism` = 0.819, gần `sim` nhưng chưa vượt aggregate. Realism tốt nhất ở 4/7 class, nhưng giảm mạnh ở `cracker_box` do texture bị SD1.5 làm hỏng. | Đây là bằng chứng thật về cục diện mới: synthetic/domain-randomized data có thể cạnh tranh với lab-only real data, nhưng GenAI phải được benchmark theo từng class thay vì tin bằng mắt. | `C2-App-013/docs/YOLO26_SimToReal_Benchmark_Report.md` |

### 3 phát hiện ban đầu

1. **Case này từng thắng nhờ...**  
   > Dữ liệu ảnh thật và nhãn thủ công từng được xem là “source of truth” cho detector vì nó gần camera thật hơn simulation và dễ được tin hơn synthetic data.
2. **AI shock làm thay đổi...**  
   > GenAI + simulation + zero-shot detector làm thay đổi điểm bắt đầu của workflow: từ “chụp và label ảnh thật trước” sang “sinh dataset trước, QA nhãn, rồi benchmark trên ảnh thật”.
3. **Dấu hiệu mạnh nhất cho thấy luật chơi mới là...**  
   > `sim` synthetic data trong benchmark đã vượt `lab_raw` về mAP50-95, dù `realism` chưa thắng aggregate. Điều này cho thấy nguồn gốc ảnh không còn là tiêu chuẩn duy nhất; tiêu chuẩn mới là kết quả validation.

---

## Bước 2 — Viết 4 nhận định bắt buộc

### Nhận định 1 — Trước AI, case này thắng nhờ giả định gì?

**Trả lời của tôi:**  
> Trước AI shock, workflow chụp ảnh thật + gán nhãn thủ công thắng nhờ giả định rằng dữ liệu thật luôn đáng tin hơn dữ liệu synthetic, và nhãn do người kiểm soát là cách an toàn nhất để huấn luyện model. Robotics perception engineer “thuê” workflow này để hoàn thành job: có một bộ ảnh có bbox đủ tin để train detector chạy được trên camera thật. Switching cost nằm ở toàn bộ thói quen vận hành: setup chụp, label tool, reviewer, dataset versioning, train/test, rồi lặp lại khi đổi object hoặc scene.

**Bằng chứng đỡ nhận định này:** E1, E2

---

### Nhận định 2 — Kỳ vọng người dùng và luật chơi cạnh tranh đã đổi ở đâu?

**Shift kỳ vọng quan trọng nhất:** Làm xong giúp tôi + phản hồi ngay + có evidence để tôi tin.  
**Competitive dynamic quan trọng nhất:** Build-copy cycles tăng tốc; data advantage dịch từ “ai có nhiều ảnh thật hơn” sang “ai có pipeline tạo, lọc, đo và cải thiện data nhanh hơn”.

**Trả lời của tôi:**  
> User không chỉ muốn một tool label ảnh, họ muốn một pipeline tạo dataset có thể train được, có QA, có benchmark và có metadata tái lập. Luật chơi cạnh tranh chuyển từ “ai label nhiều ảnh thật hơn” sang “ai tạo được nhiều biến thể dữ liệu đủ tin cậy hơn với chi phí thấp hơn và chứng minh bằng metric trên ảnh thật”. Trong repo `C2-App-013`, sản phẩm không bán ảnh synthetic; sản phẩm bán một bằng chứng: synthetic/domain-randomized dataset có thể giúp detector đạt mAP tốt hơn lab-only real dataset trong benchmark.

**Bằng chứng đỡ nhận định này:** E3, E4, E5

---

### Nhận định 3 — Giả định nào không còn đúng nữa? Điều gì đã thay đổi vĩnh viễn?

**Điều đã thay đổi vĩnh viễn theo tôi là:**  
> Giả định “dữ liệu thật + nhãn thủ công luôn là default path tốt nhất” không còn chắc đúng. Chuẩn mới là validation-first dataset engineering: dataset có thể sinh ra từ simulation hoặc GenAI, nhưng phải có QA giữ nhãn, metadata tái lập và benchmark trên real-camera holdout. Hình ảnh realistic không còn đủ; ảnh synthetic cũng không bị loại chỉ vì là synthetic. Câu hỏi mới là: dataset đó có làm model tốt hơn trên môi trường thật không?

**Bằng chứng đỡ nhận định này:** E4, E5

---

### Nhận định 4 — Case này còn cứu được không? Nếu có, phải đổi bằng cách nào?

**Verdict ban đầu của tôi:** Có nhưng phải đổi rất mạnh.

**Trả lời của tôi:**  
> Manual labeling / lab-photo workflow vẫn còn sống, nhưng không nên đứng một mình như bước chính. Nó phải chuyển thành lớp review, golden benchmark và failure analysis. Các công cụ như CVAT/Roboflow/labeling workflow cần tích hợp synthetic generation, active learning, QA gate và model-eval loop. Nếu chỉ giữ vai trò “vẽ bbox thủ công trên ảnh thật”, sản phẩm sẽ bị ép xuống thành commodity hoặc chỉ dùng cho edge cases, vì phần tạo biến thể dữ liệu đã được automation và GenAI xử lý nhanh hơn.

**Bằng chứng đỡ nhận định này:** E2, E3, E4, E5

---

## Tóm tắt cá nhân trước khi share trong bàn

1. `Case này yếu đi vì...` workflow chụp ảnh thật + gán nhãn thủ công không còn là cách duy nhất để tạo dataset object detection đáng tin; synthetic + QA + benchmark có thể tạo vòng lặp nhanh hơn.
2. `Điều thay đổi vĩnh viễn là...` chất lượng dataset được quyết định bởi validation evidence trên real holdout, không phải bởi việc ảnh đến từ camera thật hay simulation.
3. `Verdict của tôi là...` case này còn cứu được nếu chuyển từ manual-labeling-first sang validation/review/evaluation layer trong pipeline synthetic-to-real.

---

## Bước 3 — Share trong bàn

> Ghi chú: chưa có transcript thảo luận lớp trong repo. Phần dưới là ghi chú phản biện dựa trên review dự án `C2-App-013`; nếu có share trực tiếp trên lớp, nên thay bằng ghi chú người thật.

### Ghi nhanh khi nghe các bạn cùng bàn / góc phản biện

| Người / góc nhìn | Case | Bằng chứng mạnh nhất họ nêu | Điều họ cho là "thay đổi vĩnh viễn" | Verdict của họ |
|---|---|---|---|---|
| Góc sản phẩm | Manual labeling workflow | User không cần ảnh đẹp; user cần dataset giúp mAP tốt hơn trên real camera | Product phải bán outcome/evidence, không bán pipeline steps | Cứu được nếu định vị lại thành eval layer |
| Góc kỹ thuật | Sim-only vs Gen-realism | Realism chưa thắng sim aggregate; `cracker_box` bị giảm mạnh | GenAI cần class-aware QA, không thể tin output bằng mắt | Chưa nên claim quá mạnh |
| Góc evaluation | Golden benchmark | Benchmark mixed test set có thể gây hiểu nhầm nếu gọi là Golden Dataset | Metric definition phải nhất quán với claim | Cần sửa narrative trước khi nộp |
| Góc platform | FastAPI app | App có web UI nhưng chưa auth/queue/admin | Demo pipeline khác production web app | Cần bổ sung minimum product requirements |

### Sau khi cả bàn share xong, chốt 3 ý chung

**1. Bàn tôi thấy case nào có bằng chứng mạnh nhất? Vì sao?**  
> Case manual labeling / lab-only dataset có bằng chứng mạnh nhất vì repo đã có số liệu benchmark cụ thể: `sim` vượt `lab_raw`, còn `realism` không thắng aggregate. Đây không phải suy đoán thị trường mà là evidence từ chính pipeline.

**2. Có pattern nào lặp lại giữa nhiều case không?**  
> Pattern lặp lại là AI không chỉ thay feature, mà thay chuẩn kỳ vọng: user muốn workflow tự tạo output, có kiểm chứng, có metric và có khả năng lặp lại. Moat cũ dựa trên manual effort giảm giá trị nếu AI có thể tự động hóa phần tạo và kiểm thử.

**3. Một cảnh báo cho chính dự án của nhóm tôi là gì?**  
> Không được claim “GenAI cải thiện dataset” chỉ vì ảnh trông realistic hơn. Dự án phải luôn bám metric: QA label preservation, per-class mAP, reject rate, cost/image và failure cases.

---

## Bước 4 — Chốt lại verdict cá nhân sau thảo luận

### Sau khi nghe bàn phản biện, verdict của tôi:

- [ ] Giữ nguyên
- [x] Đổi nhẹ
- [ ] Đổi mạnh

### Vì sao tôi giữ / đổi verdict?

> Tôi đổi nhẹ vì ban đầu có thể nói “GenAI realism thay thế manual data”. Sau khi nhìn benchmark, nhận định đúng hơn là: synthetic/domain-randomized pipeline đang làm lung lay lab-only/manual workflow, còn diffusion realism cần QA tốt hơn mới thành lợi thế ổn định. Điểm mạnh thật của dự án là validation-first data engineering, không phải bản thân diffusion.

### Verdict cuối cùng của tôi (phiên bản nộp)

**Case này tổn thương trước AI vì:**  
> Quy trình chụp ảnh thật + gán nhãn thủ công bị tổn thương vì AI/simulation pipeline có thể tạo nhanh dataset có nhãn, có metadata, có QA và có benchmark; manual workflow không còn là điểm bắt đầu mặc định.

**Điều thay đổi vĩnh viễn là:**  
> Dataset robotics perception sẽ được đánh giá bằng validation evidence trên real holdout và failure analysis theo class, không chỉ bằng nguồn gốc “ảnh thật” hay cảm giác ảnh realistic.

**Nếu phải rút 1 bài học cho dự án của nhóm mình, tôi rút ra:**  
> Dự án AI Product phải claim bằng metric chứ không bằng demo đẹp. Với `C2-App-013`, claim an toàn nhất là: pipeline giúp tạo, kiểm soát và benchmark dataset nhanh hơn; diffusion realism là một biến thể cần tiếp tục kiểm chứng, không phải chiến thắng đã chắc chắn.

---

## Checklist trước khi nộp

- [x] Tôi đã chọn ít nhất 3 bằng chứng chốt có nguồn.
- [x] Mỗi nhận định đều chỉ vào ít nhất 1 bằng chứng.
- [x] Tôi đã ghi lại phần share/phản biện trong bàn hoặc ghi rõ giới hạn dữ liệu share.
- [x] Tôi đã viết verdict cuối sau thảo luận/phản biện.
