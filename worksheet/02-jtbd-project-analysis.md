---
artifact: 02 — JTBD Project Analysis
bai-tap: Lab 2 — Dùng JTBD để soi lại dự án nhóm
format: Theo nhóm dự án → share trong bàn → chốt hypothesis cuối
nop-cuoi: Có — đây là file nộp cuối của Lab 2
companion-reference: Strategyn_JTBD_Playbook.pdf
---

# Lab 2 — JTBD Project Analysis / Dùng JTBD để soi lại dự án nhóm

**Tên dự án / sản phẩm:** C2-App-013 — AI20K-170 v2 Synthetic-to-Real YOLO Dataset Pipeline  
**Người làm:** Nguyễn Tiến Sỉ — 2A202600681

---

## Bước 0 — Khoanh đúng 1 lát cắt của dự án

### Khoanh đúng 1 lát cắt theo 4 điểm

- [x] **1 nhóm người dùng chính:** Robotics Perception Engineer tại robotics lab/startup nhỏ.
- [x] **1 hoàn cảnh / tình huống rõ:** Cần train/evaluate YOLO detector cho các vật thể tabletop đã biết nhưng không muốn lặp lại vòng chụp ảnh thật và gán nhãn thủ công mỗi lần đổi object/scene.
- [x] **1 job cốt lõi:** Tạo và kiểm chứng dataset object detection có bbox đáng tin cho camera thật.
- [x] **1 workflow đủ cụ thể để vẽ ra được:** chọn object/scene → sinh sim data → làm realism/QA → train YOLO → benchmark trên real holdout → quyết định dataset variant.

### Điền nhanh trước khi làm

- **Dự án của nhóm tôi là:** AI20K-170 v2 — pipeline tạo YOLO dataset synthetic-to-real cho tabletop robot object detection.
- **Lát cắt tôi chọn để phân tích hôm nay là:** Robotics Perception Engineer tạo dataset cho 7 YCB tabletop objects để train detector và chứng minh hiệu quả bằng benchmark.
- **Vì sao tôi chọn lát cắt này:**  
  > Đây là lát cắt rõ nhất trong repo `C2-App-013`: user, object set, pipeline, QA, benchmark và output đều đã được định nghĩa. Nếu phân tích rộng hơn thành “AI cho robotics” thì job sẽ mơ hồ và dễ nhảy vào feature list.

---

## Bước 1 — Project Snapshot

### Tóm tắt dự án trong 3 dòng

1. **Nhóm tôi đang nghĩ mình đang giải quyết vấn đề gì?**  
   > Giảm bottleneck tạo dataset cho robotics object detection bằng cách sinh dữ liệu từ simulation, tăng realism bằng GenAI, kiểm tra giữ nhãn và đo hiệu quả detector trên ảnh thật.

2. **Người dùng chính hiện nhóm đang nhắm tới là ai?**  
   > Robotics Perception Engineer tại lab/startup nhỏ, người phải build YOLO detector cho tabletop pick-and-place/sorting demo nhưng không có data team riêng.

3. **Hiện tại người dùng đó đang giải quyết vấn đề này bằng cách nào?**  
   > Chụp ảnh thật trong lab rồi gán nhãn thủ công bằng CVAT/Roboflow/Label Studio, dùng sim-only PyBullet/domain randomization, outsource labeling, hoặc tự viết script sinh data rời rạc không có QA/benchmark chuẩn.

---

## Bước 2 — Market Context

### Trả lời 4 câu ngắn

1. **Ai đang gặp vấn đề này?**  
   > Robotics Perception Engineer / CV engineer trong các nhóm robotics nhỏ cần detector nhận diện vật thể thật trên bàn.

2. **Vấn đề xuất hiện trong hoàn cảnh nào?**  
   > Khi chuyển từ prototype trong simulator sang camera thật: texture, lighting, background, camera noise, pose và occlusion khác nhau làm detector train từ sim hoặc lab-only data giảm hiệu năng.

3. **Hiện tại họ đang dùng giải pháp thay thế nào?**  
   > Manual capture + manual labeling, sim-only rendering, classical domain randomization, active learning, outsource labeling, hoặc dùng dataset public không đúng object/scene của họ.

4. **Vì sao đây là thời điểm đáng giải?**  
   > Simulation, diffusion models, VLM/zero-shot detectors như Grounding DINO và YOLO pipeline đã đủ trưởng thành để tạo vòng lặp mới: generate → QA → train → benchmark. Người dùng không chỉ cần thêm data; họ cần bằng chứng data đó có cải thiện model thật.

### Tóm tắt market context trong 3-4 dòng

> Robotics teams nhỏ bị kẹt giữa hai cực: ảnh thật thì đáng tin nhưng tốn công chụp/label; ảnh sim thì rẻ nhưng dễ gap với camera thật.  
> GenAI mở cơ hội biến sim frame thành ảnh realistic hơn, nhưng cũng tạo rủi ro label drift hoặc semantic corruption.  
> Sản phẩm đáng làm không phải “AI tạo ảnh”, mà là pipeline có kiểm chứng: sinh data, giữ nhãn, log metadata, benchmark trên real holdout.  
> Repo `C2-App-013` đang định vị đúng vào khoảng hẹp này.

---

## Bước 3 — Xác định `Job Executor`

- **Job executor của dự án này là:** Robotics Perception Engineer.
- **Vì sao tôi tin đây là người trực tiếp "thuê" giải pháp để làm job:**  
  > Người này trực tiếp cấu hình object/scene, chạy pipeline, kiểm tra preview/QA, train YOLO và đọc benchmark để quyết định có dùng dataset không. Founder/CTO/lab lead là stakeholder hoặc buyer, nhưng không phải người trực tiếp hoàn thành job hằng ngày.

---

## Bước 4 — Viết `Core JTBD`

### Bản nháp 1

**Core JTBD bản nháp:**  
> Dùng AI20K-170 để tạo synthetic-to-real YOLO dataset cho robot tabletop.

### Gạch bỏ từ solution nếu có

- Các từ solution tôi đang lỡ nhét vào câu: `AI20K-170`, `synthetic-to-real`, `YOLO`, `robot tabletop` nếu dùng như tên giải pháp thay vì job.

### Bản chốt

**Core JTBD cuối cùng:**  
> Tạo và kiểm chứng bộ dữ liệu nhận diện vật thể có nhãn đáng tin khi cần huấn luyện detector hoạt động tốt trên camera thật trong bối cảnh tabletop scene thay đổi.

Tự kiểm:

- [x] Nếu bỏ product hiện tại đi, job này vẫn còn tồn tại.
- [x] Trong câu không có tên sản phẩm/app cụ thể.
- [x] Câu mô tả điều user muốn hoàn thành, không phải danh sách feature.

---

## Bước 5 — Viết 3 `Job Stories`

| # | Trigger / When | Motivation / I want to | Outcome / so I can | Điều story này cho thấy |
|---|---|---|---|---|
| JS1 | When tôi thêm một object mới hoặc đổi background/camera setup cho demo robotics | I want to tạo nhanh dataset có bbox label và metadata tái lập | so I can train detector mà không phải chụp và label lại toàn bộ ảnh thật | Pain nằm ở vòng lặp data creation lặp đi lặp lại |
| JS2 | When detector train từ sim-only chạy kém trên ảnh camera thật | I want to so sánh sim_only, domain-randomized và gen_realism trên cùng benchmark | so I can chọn chiến lược data dựa trên mAP thay vì cảm tính | User cần evidence để ra quyết định, không chỉ ảnh đẹp |
| JS3 | When diffusion làm ảnh trông realistic hơn nhưng có thể làm lệch object/texture | I want to kiểm tra label preservation và failure cases trước khi dùng ảnh train | so I can tránh đưa data nhiễu vào YOLO model | AI leverage phải đi kèm QA/guardrail, không thể dùng GenAI mù |

Tự kiểm:

- [x] Mỗi story là một tình huống thật.
- [x] 3 story không trùng hệt nhau.
- [x] Sau khi đọc 3 story, có thể hình dung khi nào product đáng xuất hiện.

---

## Bước 6 — Liệt kê `Current Alternatives`

| Alternative hiện tại | User đang thuê nó để làm gì? | Nó làm tốt gì? | Nó fail ở đâu? | Switching cost hiện tại cao hay thấp? |
|---|---|---|---|---|
| Manual capture + CVAT/Label Studio/Roboflow | Chụp ảnh thật và vẽ bbox thủ công | Dễ tin vì ảnh là real-camera; reviewer kiểm soát được nhãn | Tốn thời gian, khó scale khi đổi object/scene, nhãn không nhất quán, chậm vòng train/eval | Trung bình: team quen workflow nhưng ghét chi phí thời gian |
| Sim-only PyBullet / BlenderProc | Tạo ảnh và nhãn tự động từ simulator | Rẻ, reproducible, label chính xác | Gap với camera thật: texture, light, background, noise, occlusion | Thấp-trung bình: dễ chạy nhưng user nghi ngờ real performance |
| Classical domain randomization | Tăng đa dạng texture/background/light để model generalize | Có research backing, không cần GenAI phức tạp | Không phải lúc nào đủ realistic; khó biết randomization nào giúp thật | Trung bình |
| Outsource labeling / data vendor | Mua công sức label ảnh thật | Giảm tải nội bộ | Chi phí cao, lead time dài, data privacy/quality review, không phù hợp vòng lặp nhanh | Cao nếu đang có budget/vendor |
| Public dataset có sẵn | Lấy dữ liệu nhanh để train thử | Nhanh, rẻ | Không đúng object/scene/camera của robot demo | Thấp |

### Kết luận nhanh

**Nếu project của tôi biến mất hôm nay, user nhiều khả năng sẽ quay về:**  
> Manual capture + CVAT/Roboflow cho golden/critical frames, kết hợp sim-only hoặc classical domain randomization cho data augmentation. Họ vẫn phải tự nối các bước QA và benchmark bằng script riêng.

---

## Bước 7 — JTBD Lite Map

| Step | Trong workflow này user đang cố làm gì? | Hôm nay họ đang dùng gì? | Friction / pain hiện tại | Mức đau |
|---|---|---|---|---|
| Define | Xác định object set, camera setup, scene type, metric cần đạt | README, notebook, trao đổi trong team | Dễ viết quá rộng: “AI cho robotics” thay vì một task detection cụ thể | Med |
| Locate | Tìm mesh/object model, ảnh thật holdout, background/material phù hợp | YCB dataset, ảnh tự chụp, dataset public | Nguồn dữ liệu phân tán; dữ liệu thật và mesh không luôn khớp | High |
| Prepare | Chuẩn bị simulator, URDF, texture, label format, train split | PyBullet/Blender scripts, CVAT, YAML thủ công | Nhiều bước kỹ thuật dễ lỗi; một lỗi COM/label làm hỏng cả benchmark | High |
| Confirm | Kiểm tra bbox, object count, nhãn có còn đúng sau realism không | Manual preview, spot check | Người dùng khó phát hiện label drift/semantic corruption ở quy mô lớn | High |
| Execute | Sinh dataset, chạy diffusion/QA, package output | Local scripts, RunPod, ComfyUI, training scripts | Pipeline dài, nhiều tool, dễ fail ở môi trường GPU/pod | Med-High |
| Monitor | Theo dõi reject rate, mean IoU, cost/image, runtime | Logs rời rạc, spreadsheet | Không có dashboard thống nhất thì khó biết batch nào đáng dùng | High |
| Modify | Điều chỉnh prompt, workflow, randomization, preserve_objects, object subset | Thử-sai thủ công | Cần hiểu failure theo class; ảnh đẹp có thể làm mAP tệ | High |
| Conclude | Train YOLO, benchmark, quyết định variant nào đáng dùng | Ultralytics logs, custom report | Nếu metric/narrative không nhất quán, stakeholder không tin kết luận | High |

### Chốt 2 bước đau nhất

**Bước đau nhất #1:** Confirm — kiểm chứng label preservation và quality sau GenAI.  
**Bước đau nhất #2:** Conclude — benchmark và diễn giải liệu dataset có thật sự cải thiện detector không.

**Vì sao đây là nơi đáng chú ý nhất:**  
> Nếu Confirm sai, ảnh generated đẹp nhưng label lệch sẽ làm model học nhiễu. Nếu Conclude sai, team có thể claim nhầm rằng GenAI giúp tốt hơn trong khi benchmark chỉ cho thấy domain randomization đang thắng. Hai bước này quyết định niềm tin của user và reviewer.

---

## Bước 8 — AI Leverage Point

| Step | AI nên giúp bằng cách nào? | Vì sao AI hợp ở đây? | Rủi ro chính nếu dùng AI |
|---|---|---|---|
| Prepare / Execute | Dùng generative realism để chuyển sim RGB thành ảnh gần real domain hơn, có conditioning từ depth/canny và optional preserve_objects | AI có thể bổ sung texture/light/background realism nhanh hơn việc chụp nhiều biến thể thật | Model có thể hallucinate texture, làm hỏng semantic fidelity hoặc label consistency |
| Confirm / Monitor | Dùng Grounding DINO / VLM-style re-detection để kiểm tra bbox IoU, object count và sinh quality report; LLM tóm tắt job quality | AI giúp kiểm tra hàng loạt và diễn giải failure cho người dùng nhanh hơn manual spot-check | QA gate hiện bắt geometry drift tốt hơn semantic corruption; cần thêm CLIP/reference-crop hoặc class-specific check |

### Kết luận nhanh

**AI leverage point quan trọng nhất của dự án tôi là:**  
> Không phải “tạo ảnh đẹp”, mà là **GenAI-assisted dataset generation có QA gate và benchmark loop** ở các bước Prepare → Execute → Confirm → Conclude.

**Vì sao không phải ở bước khác:**  
> Define và Locate vẫn cần quyết định domain của con người. AI tạo value lớn nhất khi biến config thành dataset có thể train được và khi kiểm chứng liệu output đó có đáng tin. Nếu đưa AI vào quá sớm để “nghĩ hộ use case”, dự án dễ over-scope; nếu đưa AI vào quá muộn chỉ để viết report, value không đủ lớn.

---

## Bước 9 — Product Hypothesis

### Bản hypothesis của tôi

> Nếu chúng ta giúp Robotics Perception Engineer tạo và kiểm chứng dataset object detection ở bước Prepare → Confirm → Conclude, bằng cách sinh synthetic/domain-randomized data, áp dụng generative realism có kiểm soát, lọc bằng QA gate và benchmark trên real holdout, thì họ sẽ chuyển từ manual lab-only capture hoặc sim-only scripts sang pipeline của nhóm, vì họ có thể giảm vòng lặp chụp–label thủ công và ra quyết định bằng mAP/QA evidence thay vì cảm tính.

### Tín hiệu sớm nếu hypothesis này đúng

1. User chạy được một job end-to-end và tải dataset ZIP kèm preview/metadata mà không cần sửa code.
2. User dùng quality report/mAP table để quyết định giữ hoặc loại một dataset variant.
3. Trong benchmark lặp lại, ít nhất một variant synthetic/domain-randomized hoặc gen-realism thắng baseline lab-only/sim-only trên real holdout theo metric đã định nghĩa trước.
4. User chấp nhận chạy pilot batch trước full run vì thấy cost/image và reject rate minh bạch.

---

## Bước 10 — Assumptions to Validate

| Assumption | Vì sao assumption này rủi ro? | Tôi đang có bằng chứng gì? | Cần validate bằng cách nào tiếp theo? |
|---|---|---|---|
| A1 — Job executor đúng là Robotics Perception Engineer | Nếu buyer/stakeholder chính là lab lead nhưng engineer không dùng trực tiếp, workflow có thể sai UX | Repo PRD định nghĩa primary user là Robotics Perception Engineer | Phỏng vấn 3-5 robotics/CV engineers về workflow dataset hiện tại |
| A2 — Pain manual capture/label đủ đau và xảy ra thường xuyên | Nếu team chỉ làm demo nhỏ một lần, họ có thể chấp nhận label thủ công | Evidence từ YCB-Video/PoseCNN và repo framing về data bottleneck | Hỏi user mỗi lần đổi object/scene mất bao lâu để tạo dataset mới |
| A3 — Gen-realism cải thiện detector đủ ổn định | Benchmark hiện cho thấy `realism` chưa thắng `sim` aggregate; chỉ thắng 4/7 class | Report YOLO26: realism = 0.819 mAP50-95, sim = 0.823; `cracker_box` giảm -0.180 | Chạy multi-seed, Qwen ablation, preserve_objects và benchmark riêng trên Golden real-only |
| A4 — QA gate bắt đủ lỗi quan trọng | Grounding DINO IoU bắt geometry drift nhưng chưa bắt texture/semantic corruption | Report chỉ ra `cracker_box` có thể pass geometry nhưng texture bị SD1.5 làm hỏng | Thêm CLIP/reference-crop similarity, class-specific visual QA, manual review sample |
| A5 — User chấp nhận độ phức tạp deploy/pod | Repo hiện cần local CPU + RunPod/ComfyUI + optional HF Space; không phải SaaS đơn giản | Docs ghi MVP single-process, diffusion/QA/training chạy pod | Test onboarding: user mới có chạy được trong 30 phút không; đo setup friction |

### Assumption nguy hiểm nhất nếu tôi đang sai

> A3 là nguy hiểm nhất: nếu gen-realism không cải thiện mAP ổn định so với sim/domain-randomization, product không nên claim “GenAI-enhanced data improves YOLO”. Khi đó định vị phải đổi thành “validation-first synthetic dataset pipeline” và coi GenAI realism là optional experiment, không phải core proof.

---

## Bước 11 — Share trong bàn

> Ghi chú: chưa có transcript thảo luận lớp trong repo. Phần này ghi lại các phản biện cần dùng khi share; nếu có ý kiến thật từ bàn, nên cập nhật lại trước khi nộp LMS.

### Mỗi người / mỗi nhóm chỉ nói 4 thứ

1. **Job executor:** Robotics Perception Engineer.
2. **Core JTBD:** Tạo và kiểm chứng bộ dữ liệu nhận diện vật thể có nhãn đáng tin khi cần huấn luyện detector hoạt động tốt trên camera thật trong bối cảnh tabletop scene thay đổi.
3. **Step đau nhất:** Confirm và Conclude.
4. **AI leverage point + assumption rủi ro nhất:** GenAI-assisted data generation + QA/benchmark; assumption rủi ro nhất là gen-realism chưa chắc thắng sim/domain randomization ổn định.

### Ghi nhanh sau khi nghe bàn phản biện

| Ý phản biện tôi nghe được | Nó chạm vào phần nào? | Tôi sẽ giữ / sửa gì? |
|---|---|---|
| “Đừng gọi ảnh generated là tốt hơn nếu benchmark chưa thắng sim.” | Product hypothesis / success metric | Sửa claim: value hiện tại là validation-first pipeline; GenAI realism là một variant cần chứng minh thêm |
| “Job executor không phải founder mà là engineer chạy pipeline.” | Job executor | Giữ Robotics Perception Engineer làm executor; founder/CTO là stakeholder |
| “QA gate đang bắt hình học, chưa bắt semantic texture.” | AI leverage / assumptions | Thêm assumption A4 và next step: CLIP/reference-crop similarity |

---

## Bước 12 — Chốt version cuối sau thảo luận

### Sau khi nghe phản biện, tôi thay đổi gì?

- [x] Giữ nguyên `job executor`
- [ ] Sửa `job executor`
- [x] Giữ nguyên `core JTBD`
- [ ] Sửa `core JTBD`
- [ ] Giữ nguyên `AI leverage point`
- [x] Sửa `AI leverage point`
- [ ] Giữ nguyên `product hypothesis`
- [x] Sửa `product hypothesis`

### Vì sao tôi giữ / sửa?

> Tôi giữ job executor vì robotics perception engineer là người trực tiếp chạy job và đọc benchmark. Tôi sửa AI leverage point và hypothesis để tránh overclaim “diffusion realism chắc chắn cải thiện mAP”. Bằng chứng hiện tại mạnh hơn cho thông điệp: pipeline tạo, kiểm soát và benchmark dataset có giá trị; GenAI realism cần thêm QA và ablation để thành lợi thế chắc chắn.

### Version cuối cùng tôi nộp

**Job executor:**  
> Robotics Perception Engineer tại robotics lab/startup nhỏ.

**Core JTBD:**  
> Tạo và kiểm chứng bộ dữ liệu nhận diện vật thể có nhãn đáng tin khi cần huấn luyện detector hoạt động tốt trên camera thật trong bối cảnh tabletop scene thay đổi.

**2 bước đau nhất trong workflow:**  
> Confirm — kiểm chứng label/quality sau generation; Conclude — benchmark và diễn giải dataset variant nào thật sự tốt hơn.

**AI leverage point chính:**  
> GenAI-assisted synthetic-to-real dataset generation kết hợp QA gate và benchmark loop, không phải chỉ tạo ảnh realistic.

**Product hypothesis:**  
> Nếu chúng ta giúp Robotics Perception Engineer tạo, lọc và benchmark dataset object detection bằng simulation, generative realism có kiểm soát, QA gate và mAP evidence, thì họ sẽ chuyển từ manual lab-only capture hoặc sim-only scripts sang pipeline của nhóm, vì họ giảm vòng lặp chụp–label thủ công và có bằng chứng định lượng để chọn dataset.

**Assumption cần validate đầu tiên:**  
> Gen-realism hoặc synthetic/domain-randomized variant phải cải thiện detector một cách ổn định trên real holdout theo multi-seed/multi-class benchmark; nếu không, product phải định vị lại là validation-first dataset engineering platform thay vì GenAI realism product.

---

## Checklist trước khi nộp

- [x] Tôi đã khoanh đúng 1 lát cắt cụ thể của dự án.
- [x] Tôi đã phân biệt được `job executor` với buyer / influencer.
- [x] `Core JTBD` của tôi không nhét solution vào câu.
- [x] Tôi đã viết đủ 3 `job stories`.
- [x] Tôi đã điền `JTBD lite map` và khoanh ra 2 bước đau nhất.
- [x] Tôi đã chỉ ra `AI leverage point` thay vì nhảy thẳng vào feature list.
- [x] Tôi đã ghi rõ `assumptions to validate`.
- [x] Tôi đã sửa version cuối sau khi share/phản biện.
