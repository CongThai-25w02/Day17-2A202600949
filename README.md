# Day17-2A202600949

---
artifact: 03 — Cheaper Tests / 3 cách rẻ hơn để test cùng 1 hypothesis
bai-tap: Lab 3 — Tìm cách test rẻ hơn cho hypothesis lõi của dự án
format: Theo nhóm dự án → present trong bàn → chốt + ghi post-present
time: ~25 phút trên lớp
nop-cuoi: Có — đây là file nộp cuối của Lab 3
companion-reference: 02-jtbd-project-analysis.md (Lab 2, file nguồn của hypothesis)
---

# Lab 3 — 3 Cách Rẻ Hơn Để Test Cùng Một Hypothesis

**Tên dự án / sản phẩm:** RoboPlanner (AI20K‑162) — lớp thực thi agent **CÓ KIỂM CHỨNG** cho robot kho mô phỏng, ra lệnh bằng tiếng Việt.

> File này nối tiếp [`02-jtbd-project-analysis.md`](02-jtbd-project-analysis.md).
> Lab 2 đã chốt *job thật + hypothesis*. Lab 3 hỏi tiếp một câu rất khác: **làm sao kiểm chứng hypothesis đó mà KHÔNG phải build hết cả hệ thống.**

Mục tiêu của bài này **không** phải nghĩ thêm 3 tính năng.
Mục tiêu là:

1. **chốt lại đúng 1 hypothesis** đang đắt/chậm để test
2. **nói rõ current approach đang test nó tốn gì**
3. **nghĩ ra ít nhất 3 hướng test rẻ hơn cho CÙNG hypothesis đó**
4. **gắn mỗi hướng với 1 artifact cụ thể** đủ để đi hỏi người dùng

Quy tắc xuyên suốt: **3 cách = 3 hướng test cùng 1 hypothesis, KHÔNG phải 3 feature, KHÔNG phải 3 hypothesis.**

---

## Đầu ra bắt buộc

Người chấm cần thấy đủ 6 phần trong chính file này:

1. **JTBD checkpoint** (kế thừa từ file 02)
2. **Hypothesis card** (chọn & chốt 1 hypothesis)
3. **Current approach snapshot** (cách test hiện tại + chi phí)
4. **Ít nhất 3 hướng test rẻ hơn** (mỗi hướng 8 trường)
5. **Artifact cho cả 3 cách**
6. **Present prep + post‑present notes**

---

## Bước 0 — JTBD Checkpoint (kế thừa file 02)

> Không làm lại từ đầu. Chỉ kéo bản chốt cuối từ Lab 2 xuống để mọi người cùng đứng trên 1 nền.

| Câu hỏi | Chốt (từ file 02) |
|---|---|
| **Core JTBD hiện tại là gì?** | Đưa đúng hàng tới đúng vị trí trong kho khi bố trí **liên tục thay đổi**, mà **không phải lập trình lại từng bước**, và **vẫn kiểm soát được rủi ro** để dám đưa vào quy trình thật. |
| **Current workflow hiện tại là gì?** | Lập trình cứng từng bước / teach‑pendant + luật WMS cố định → mỗi lần bố trí đổi thì **gọi kỹ sư** lập lại → QA tin status "done" robot tự khai, **ít trace để audit**. |
| **Pain step đau nhất là gì?** | Cụm **Confirm / Monitor / Conclude** — tin & kiểm chứng được robot đang/đã làm đúng (hộp đen, không audit được). Đau thứ nhì: **Modify** (replan khi môi trường đổi mà không gọi kỹ sư). |
| **AI leverage point nằm ở bước nào?** | Đặt đúng ở cụm **Confirm/Monitor/Conclude**: lớp thực thi **CÓ KIỂM CHỨNG** — mọi hành động đọc trạng thái thật (grounded) + trace audit từng bước + oracle độc lập chấm + biết **"nói không"** khi bất khả thi. |

---

## Bước 1 — Hypothesis Card (chọn & chốt 1 hypothesis)

| Câu hỏi | Trả lời |
|---|---|
| **Hypothesis bạn chọn là gì?** | *Nếu* đội QA/an toàn (và vận hành viên) được xem hành động của agent ở bước Confirm/Monitor/Conclude dưới dạng **grounded + trace audit từng bước + oracle độc lập chấm + biết "nói không"**, *thì* họ sẽ **dám ký duyệt** cho agent vào quy trình thật và **chuyển khỏi hộp đen / lập trình cứng** — *vì* lần đầu họ có thứ vừa linh hoạt vừa **audit được khi có sự cố**. |
| **Hypothesis này thuộc phần nào của dự án?** | Phần **lõi / moat**: lớp thực thi CÓ KIỂM CHỨNG (Confirm/Monitor/Conclude). Đây là **North Star** của dự án, không phải tính năng phụ. Tương ứng assumption **A3** trong file 02 ("audit được + không bịa là điều kiện để duyệt, không phải nice‑to‑have"). |
| **Nếu sai thì chuyện gì sập / yếu đi?** | Nếu QA **không** thấy trace + oracle là đủ để ký duyệt → toàn bộ định vị *"audit được = điều kiện deploy"* sụp. Dự án rớt về **"một lời gọi LLM"** không bán được, và differentiator biến mất. Cả pitch lẫn pilot proposal đều dựa trên hypothesis này. |
| **Vì sao chọn hypothesis này để làm bài hôm nay?** | Vì đây là điều **khó & đắt nhất để kiểm thật** (cần người duyệt thật + agent chạy được), lại là chỗ **quyết định giá trị deploy**. Quan trọng hơn: agent lõi đang **success 0%** (A4) nên *không thể* test bằng pipeline đầy đủ ngay lúc này → càng cần cách test rẻ, tách rời khỏi việc agent đã chạy hay chưa. |

---

## Bước 2 — Current Approach Snapshot

| Câu hỏi | Trả lời |
|---|---|
| **Dự án đã từng làm / đang định làm gì để test hypothesis này?** | Build **full pipeline**: agent LangGraph 7 node (parse→perceive→plan→act→observe→replan→summarize) + sim "khóa" bằng bất biến S1–S6 + **oracle độc lập** (`check_object_moved`) + **eval 2 lớp** (Bảng A agent thật / Bảng B solver A\*) chạy đa seed → rồi đưa trace cho QA đọc & chấm theo rubric audit. |
| **Cách đó có cần build gì tương đối lớn không?** | **Có, rất lớn.** Agent phải tự sinh tool‑call hợp lệ end‑to‑end (đang hỏng: `valid_action_rate` 0.8%, `success` 0%), cộng harness eval đa seed, oracle, **141 test**. Phải vá xong bug function‑calling (P0‑1) thì mới có trace thật để đưa QA. |
| **Cách đó tốn gì?** | Thời gian build + debug; **42 LLM calls/task**, **~6.11s/bước** (mục tiêu <3s chưa đạt); cần nhiều seed để ổn định; và **vẫn cần người QA thật ngồi chấm** — hiện **chưa có phỏng vấn người duyệt thật** nào. |
| **Vì sao cách hiện tại đang đắt hoặc chậm?** | Vì nó **trộn 2 câu hỏi vào nhau**: (1) *agent có tự sinh được hành động/trace không?* và (2) *trace đó có làm QA dám ký duyệt không?* Cách hiện tại bắt phải giải xong (1) — cả khối kỹ thuật, đang 0% — rồi mới chạm tới (2). Test niềm tin bị **chặn sau** toàn bộ phần kỹ thuật. |
| **Câu hỏi bạn muốn tự ép mình hỏi lại?** | **"Mình có thể test 'QA có tin & dám ký duyệt không' MÀ KHÔNG cần agent chạy được không?"** → Có. Tách **câu hỏi niềm tin** (trace/oracle có đủ để duyệt?) ra khỏi **câu hỏi năng lực** (agent có tự sinh được trace?). Dùng trace **dựng tay (WoZ)** để hỏi niềm tin trước, song song với việc vá agent. |

> **Insight mở khóa Lab 3:** hypothesis lõi là về *niềm tin của người duyệt*, không phải về *năng lực sinh tool‑call*. Vậy có thể kiểm niềm tin bằng trace dựng tay — rẻ hơn nhiều và không bị kẹt sau bug A4.

---

## Bước 3 — 3 Cách Rẻ Hơn Test Cùng Hypothesis

> Nhắc lại: 3 cách dưới đây **không phải 3 feature, không phải 3 hypothesis**. Cả 3 cùng test **một** hypothesis ở Bước 1 (QA tin & dám ký duyệt nhờ lớp kiểm chứng), chỉ khác **góc** và **độ gần hành vi thật**.

### Cách test A — "Audit‑Card Review" (WoZ)

| Trường | Điền |
|---|---|
| **Tên hướng test** | Audit‑Card Review (Wizard‑of‑Oz) |
| **Loại artifact** | Hi‑fi mock 1 màn **"Audit Card"** + WoZ (trace dựng tay, không cần agent) |
| **Người dùng sẽ thấy gì?** | Một thẻ audit cho 1 nhiệm vụ "đã xong": mục tiêu tiếng Việt → **trace từng bước** (suy nghĩ → tool → tham số → trạng thái thật trả về) → **con dấu oracle PASS/FAIL** → nút **[Ký duyệt] / [Từ chối]**. Bộ 5 ca: 3 ca đúng, **1 ca "done giả" gài lỗi** (oracle phải bắt), 1 ca **abstain** (agent nói không). |
| **Phía sau bạn sẽ làm gì?** | Tự **viết tay 5 trace** sát thật. Ngồi 1‑1 ~20'/người với 2–3 người đóng vai QA/an toàn, quan sát họ duyệt/từ chối + hỏi "đủ tin chưa, thiếu trường gì". |
| **Nó đang test hypothesis nào?** | **Cùng hypothesis Bước 1**: grounded + trace + oracle có **đủ để QA dám ký duyệt** không, và họ có **bắt được ca "done giả"** không. |
| **Vì sao rẻ hơn current approach?** | Bỏ qua **toàn bộ agent đang 0%** + pipeline eval đa seed. Không LLM calls, không multi‑seed; chỉ cần mock tĩnh + vài buổi review 20'. |
| **Nó giúp học được gì?** | Trace format có "đủ để hiểu & tin" không; người duyệt có **bắt lỗi gài** không; thiếu trường nào trong audit view; ngôn ngữ/đơn vị nào gây mất tin. |
| **Nó chưa giúp học được gì?** | Agent **thật** có sinh được trace sạch như vậy ổn định không (vẫn cần vá A4); ca biên thực tế; niềm tin qua thời gian dài dùng. |

### Cách test B — "Hộp đen vs Hộp kính" (A/B cô lập nhân‑quả)

| Trường | Điền |
|---|---|
| **Tên hướng test** | A/B "Hộp đen vs Hộp kính" |
| **Loại artifact** | **Storyboard / 2 màn wireframe side‑by‑side** (đều là mock) |
| **Người dùng sẽ thấy gì?** | **Cùng 1 nhiệm vụ**, 2 cách trình bày kết quả: (1) **LLM trần** báo "đã xong" — không bằng chứng; (2) **Audit Card** grounded + trace + oracle. Câu hỏi: *"Bản nào bạn dám ký duyệt đưa vào dây chuyền thật?"* |
| **Phía sau bạn sẽ làm gì?** | Cả 2 bản đều **dựng tay/mock**. Đo lựa chọn + lý do; **xoay thứ tự A/B** giữa các người để chống thiên lệch. |
| **Nó đang test hypothesis nào?** | Cùng hypothesis, nhưng **cô lập nhân‑quả**: chính **lớp kiểm chứng** (chứ không phải yếu tố khác) có làm **đổi quyết định ký duyệt** không. |
| **Vì sao rẻ hơn current approach?** | 1 artifact A/B + phỏng vấn ngắn; **không agent, không eval**. Tách được đúng 1 biến ("có/không kiểm chứng") rẻ hơn nhiều so với build full rồi mới biết. |
| **Nó giúp học được gì?** | Lớp kiểm chứng có phải **yếu tố quyết định** không (vs họ vốn sẽ duyệt dù gì / không bao giờ duyệt dù gì); giá trị "kiểm chứng" có thật sự **lật được** quyết định. |
| **Nó chưa giúp học được gì?** | Độ lớn hiệu ứng trên **nhiều ca thật**; trace agent thật có sạch như mock không; chi phí vận hành thật. |

### Cách test C — Fake‑door "Bật ở dây chuyền của bạn"

| Trường | Điền |
|---|---|
| **Tên hướng test** | Fake‑door "Pilot / Bật ở dây chuyền thật" |
| **Loại artifact** | **Fake‑door**: 1 trang "RoboPlanner — Audit Mode" + 1 mẫu Audit Card thật + **CTA "Đăng ký pilot / Bật ở dây chuyền của tôi"**; kèm **script phỏng vấn** người duyệt |
| **Người dùng sẽ thấy gì?** | Pitch ngắn + 1 mẫu Audit Card → nút cam kết: *"Bạn có đưa cái này vào quy trình thật của mình không?"* / form đăng ký pilot + ô ghi điều kiện để ký. |
| **Phía sau bạn sẽ làm gì?** | **Chưa có sản phẩm chạy.** Đo tỉ lệ **bấm/cam kết** + lý do từ chối; với người duyệt thật, hỏi **điều kiện cần để ký duyệt** (hồ sơ, trách nhiệm, compliance). |
| **Nó đang test hypothesis nào?** | Cùng hypothesis, nhấn phần **"switch"**: "audit được + không bịa" có phải **ĐIỀU KIỆN để duyệt/mua** (A3) hay chỉ nice‑to‑have. |
| **Vì sao rẻ hơn current approach?** | Landing/phỏng vấn, **gần như không build**. Đo **ý định chuyển đổi thật** trước khi viết thêm dòng code agent nào. |
| **Nó giúp học được gì?** | Mức sẵn sàng chuyển đổi thật + **rào cản ký duyệt thật** (compliance, trách nhiệm pháp lý, định dạng hồ sơ audit). |
| **Nó chưa giúp học được gì?** | Sản phẩm có chạy được không; niềm tin **bền vững** sau khi dùng thật; con số hiệu năng. |

---

## Bước 4 — Artifact Cho Cả 3 Cách

| Cách | Đã có artifact chưa? | Artifact là gì? | Link / ảnh / frame |
|---|---|---|---|
| **A** | Một phần — `frontend/` đã có **bảng trace + canvas 2D**, cần đóng gói thành "Audit Card" 5 ca | Mock hi‑fi Audit Card (5 trace dựng tay: 3 đúng + 1 done‑giả + 1 abstain) | Tái dùng [`frontend/`](frontend) → export 5 frame; hoặc dựng nhanh trên Figma/giấy |
| **B** | Chưa — cần dựng 2 màn side‑by‑side | Storyboard / wireframe "hộp đen vs hộp kính" cho **cùng 1 task** | 2 frame mock (Figma/giấy); bản (2) tái dùng đúng Audit Card của Cách A |
| **C** | Chưa — cần 1 landing + script | Fake‑door landing + 1 mẫu Audit Card + script phỏng vấn người duyệt | 1 trang (có thể tái dùng [`index.html`](index.html)/[`frontend/`](frontend)) + form đăng ký |

### Bảng tra: muốn test kiểu gì → artifact có thể là gì

| Nếu muốn test kiểu... | Artifact có thể là... |
|---|---|
| **Value / desirability** | storyboard · fake‑door · landing page · script phỏng vấn có artifact |
| **Usability** | paper prototype · click‑through wireframe · before/after flow |
| **Feasibility** | WoZ setup · manual demo script · mocked output samples · throwaway screen |
| **Trust / comprehension** | 3 màn hi‑fi nhẹ · output examples · human‑in‑the‑loop flow |

> Map nhanh cho dự án này: **Cách A = Trust/comprehension (WoZ)** · **Cách B = Value/desirability (storyboard A/B)** · **Cách C = Desirability/adoption (fake‑door)**. Hypothesis lõi là về *niềm tin để chuyển đổi*, nên 3 artifact đều xoay quanh **trace audit** chứ không phải tính năng mới.

---

## Bước 5 — So Sánh Nhanh 3 Cách

| Tiêu chí | Cách A — Audit‑Card WoZ | Cách B — A/B hộp đen/kính | Cách C — Fake‑door pilot |
|---|---|---|---|
| **Nhanh hơn current approach ở đâu?** | Bỏ qua agent + eval; review tĩnh ~20'/người | 1 artifact A/B + phỏng vấn ngắn | Landing/phỏng vấn, không build |
| **Rẻ hơn current approach ở đâu?** | Không LLM calls, không multi‑seed, không cần vá A4 trước | Không agent/eval; chỉ tách đúng 1 biến | Gần như 0 build |
| **Gần với hành vi thật tới đâu?** | **Cao** — đúng hành vi "đọc trace → ký duyệt" | Trung bình — quyết định so sánh, hơi nhân tạo | Cao về **ý định**, nhưng là lời hứa chứ chưa dùng thật |
| **Học được điều gì rõ nhất?** | Trace có **đủ để tin & bắt lỗi** không | Kiểm chứng có phải **yếu tố quyết định** (nhân‑quả) | "Audit được" có phải **điều kiện deploy thật** không |
| **Giới hạn lớn nhất là gì?** | Trace dựng tay ≠ trace agent thật (A4) | Tình huống A/B nhân tạo | Lời hứa ≠ hành vi dùng thật; không chứng minh agent chạy |

---

## Bước 6 — Present Prep + Post‑Present

### Present prep (chuẩn bị nói ~3')

Nói đúng 4 ý, không sa vào tính năng:

1. **Hypothesis:** QA/vận hành dám ký duyệt cho agent **nhờ lớp kiểm chứng** ở Confirm/Monitor/Conclude.
2. **Current approach đang đắt vì** test niềm tin bị **chặn sau** agent lõi đang 0% + pipeline eval đa seed.
3. **3 cách rẻ hơn** tách *niềm tin* khỏi *năng lực*, dùng **trace dựng tay**: Audit‑Card WoZ · A/B hộp đen‑kính · Fake‑door pilot.
4. **Artifact** cho cả 3 + mình kỳ vọng học được gì từ mỗi cách.

Câu dễ bị hỏi, thủ sẵn:
- *"3 cách này có đang test cùng 1 hypothesis không?"* → Có: cả 3 đều hỏi "QA có tin & dám ký duyệt nhờ lớp kiểm chứng không", chỉ khác góc.
- *"Trace dựng tay có gian lận không?"* → Không kết luận về năng lực agent; chỉ kiểm *định dạng trace có đủ để duyệt không* — năng lực vẫn đo riêng ở Bảng A.
- *"Vì sao không build luôn cho chắc?"* → Vì build full đang kẹt ở bug A4 (0%); test niềm tin rẻ chạy song song, không phải chờ.

### Sau present — chốt lại theo 4 ý

> Điền sau khi present + sau khi chạy thử ít nhất 1 trong 3 cách.

1. **Mình đang test hypothesis nào:**
   > QA/vận hành viên dám ký duyệt cho agent vào quy trình thật nhờ lớp kiểm chứng (grounded + trace + oracle + abstain) ở bước Confirm/Monitor/Conclude.

2. **Current approach hiện đang đắt / chậm ở đâu:**
   > Bị chặn sau agent lõi 0% (valid_action_rate 0.8%) + eval đa seed (42 LLM calls/task, ~6.11s/bước); trộn câu hỏi năng lực với câu hỏi niềm tin.

3. **Ít nhất 3 cách rẻ hơn để test cùng hypothesis đó:**
   > A) Audit‑Card Review (WoZ) · B) A/B "hộp đen vs hộp kính" · C) Fake‑door "bật ở dây chuyền thật". Cả 3 dùng trace dựng tay, tách niềm tin khỏi năng lực.

4. **Artifact của 3 cách cho thấy điều gì:**
   > *(điền sau khi chạy)* — ví dụ cần trả lời: trace có đủ để duyệt không? người duyệt bắt được ca "done giả" không? lớp kiểm chứng có lật quyết định ở A/B không? có ai cam kết "bật ở dây chuyền thật" ở fake‑door không, và điều kiện họ đòi là gì?

| Ý phản biện nghe được khi present | Chạm vào phần nào? | Giữ / sửa gì? |
|---|---|---|
| | | |
| | | |

---

## Checklist trước khi nộp

- [x] Kéo **JTBD checkpoint** từ Lab 2, không bịa lại.
- [x] Chốt **đúng 1 hypothesis** (không phải 3 hypothesis).
- [x] Nói rõ **current approach + chi phí** (cái gì khiến nó đắt/chậm).
- [x] Có **≥3 hướng test rẻ hơn**, mỗi hướng đủ **8 trường**.
- [x] Cả 3 cách **cùng test 1 hypothesis** (không trượt thành 3 feature).
- [x] Mỗi cách gắn với **1 artifact cụ thể** + nơi artifact sẽ nằm.
- [x] Có **bảng so sánh nhanh** 3 cách.

