---
name: "speckit-clarify"
description: "Xác định các vũng kiến thức bị thiếu (blind spots) hoặc mơ hồ trong feature spec hiện tại bằng cách đặt tối đa 5 câu hỏi làm rõ (clarification questions) có độ chính xác cao và tích hợp (integrate) câu trả lời trở lại spec."
argument-hint: "Các vùng tùy chọn cần làm rõ trong spec"
compatibility: "Yêu cầu cấu trúc dự án spec-kit với thư mục .specify/"
metadata:
  author: "github-spec-kit"
  source: "templates/commands/clarify.md"
user-invocable: true
disable-model-invocation: true
---

## Đầu vào từ người dùng

```text
$ARGUMENTS
```

Bạn **PHẢI** xem xét đầu vào từ người dùng trước khi tiến hành (nếu không rỗng).

## Tóm tắt

Mục tiêu: Phát hiện và giảm thiểu sự mơ hồ (ambiguities) hoặc các technical decisions còn khuyết trong feature specification hiện tại, sau đó document lại các clarifications (sự làm rõ) trực tiếp vào file spec.

Lưu ý: Quy trình làm rõ này được kỳ vọng chạy (và hoàn thành) TRƯỚC KHI gọi `/speckit.plan`. Nếu người dùng nói rõ họ đang bỏ qua việc làm rõ (ví dụ: exploratory spike), bạn có thể tiến hành, nhưng phải cảnh báo rằng rủi ro làm lại ở downstream tăng lên.

Các bước thực thi:

1. Chạy `.specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly` từ thư mục gốc repo **một lần** (chế độ kết hợp `--json --paths-only` / `-Json -PathsOnly`). Parse các trường payload JSON tối thiểu:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Tùy chọn: capture `IMPL_PLAN`, `TASKS` cho các luồng chain tiếp theo.)
   - Nếu JSON parsing thất bại, dừng lại và hướng dẫn người dùng chạy lại `/speckit.specify` hoặc xác minh môi trường feature branch.
   - Đối với dấu nháy đơn trong args như "I'm Groot", sử dụng cú pháp escape: ví dụ 'I'\''m Groot' (hoặc dùng dấu nháy kép nếu có thể: "I'm Groot").

2. Tải file spec hiện tại. Thực hiện quét cấu trúc về sự mơ hồ và coverage sử dụng taxonomy này. Với mỗi danh mục, đánh dấu trạng thái: Clear (Rõ) / Partial (Một phần) / Missing (Thiếu). Tạo bản đồ coverage nội bộ dùng cho ưu tiên (không xuất map thô trừ khi không có câu hỏi nào sẽ được hỏi).

   Phạm vi & Hành vi chức năng (Functional Scope & Behavior):
   - Mục tiêu người dùng cốt lõi và tiêu chí thành công
   - Các khai báo out-of-scope rõ ràng
   - Phân biệt vai trò / personas người dùng

   Domain & Data Model:
   - Entities, thuộc tính, quan hệ
   - Quy tắc về Identity & uniqueness
   - Vòng đời/state transitions
   - Giả định về Data volume / scale

   Tương tác & Luồng UX:
   - Các user journey / sequences quan trọng
   - Trạng thái error/empty/loading
   - Ghi chú về Accessibility hoặc localization

   Các thuộc tính chất lượng phi chức năng (Non-Functional Quality Attributes):
   - Hiệu năng (latency, throughput targets)
   - Khả năng mở rộng (horizontal/vertical, giới hạn)
   - Độ tin cậy & khả dụng (uptime, kỳ vọng phục hồi)
   - Khả năng quan sát (logging, metrics, tracing signals)
   - Bảo mật & quyền riêng tư (authN/Z, bảo vệ dữ liệu, các giả định về mối đe dọa)
   - Các ràng buộc compliance / quy định (nếu có)

   Tích hợp & Phụ thuộc bên ngoài:
   - Các dịch vụ/API bên ngoài và các failure modes
   - Các định dạng import/export dữ liệu
   - Các giả định về protocol/versioning

   Edge Cases & Xử lý lỗi:
   - Các kịch bản tiêu cực
   - Rate limiting / throttling
   - Conflict resolution (ví dụ: các chỉnh sửa đồng thời)

   Ràng buộc & Tradeoffs:
   - Các ràng buộc kỹ thuật (ngôn ngữ, lưu trữ, hosting)
   - Các tradeoff hoặc lựa chọn thay thế bị từ chối rõ ràng

   Thuật ngữ & Tính nhất quán:
   - Các thuật ngữ glossary chuẩn
   - Các từ đồng nghĩa cần tránh / thuật ngữ đã bị loại bỏ

   Các tín hiệu hoàn thành (Completion Signals):
   - Khả năng kiểm thử của tiêu chí chấp nhận
   - Các chỉ báo dạng Definition of Done có thể đo lường được

   Misc / Placeholders:
   - Các marker TODO / các quyết định chưa được giải quyết
   - Các tính từ mơ hồ ("robust", "intuitive") thiếu định lượng

   Với mỗi danh mục có trạng thái Partial hoặc Missing, thêm một cơ hội câu hỏi ứng viên trừ khi:
   - Việc làm rõ sẽ không thay đổi đáng kể implementation hoặc chiến lược validation
   - Thông tin nên được hoãn lại sang giai đoạn planning (ghi chú nội bộ)

3. Tạo (nội bộ) một hàng đợi ưu tiên các câu hỏi làm rõ ứng viên (tối đa 5). KHÔNG xuất tất cả cùng lúc. Áp dụng các ràng buộc sau:
   - Tối đa 5 câu hỏi tổng cộng trong toàn bộ session.
   - Mỗi câu hỏi phải có thể trả lời bằng MỘT TRONG:
     - Một lựa chọn multiple-choice ngắn gọn (2–5 lựa chọn khác biệt, loại trừ lẫn nhau), HOẶC
     - Một câu trả lời một từ / cụm từ ngắn (ràng buộc rõ ràng: "Trả lời trong <=5 từ").
   - Chỉ bao gồm các câu hỏi mà câu trả lời có tác động đáng kể đến kiến trúc, data modeling, task decomposition, thiết kế test, hành vi UX, mức độ sẵn sàng vận hành, hoặc validation tuân thủ.
   - Đảm bảo cân bằng coverage danh mục: cố gắng cover các danh mục chưa được giải quyết có tác động cao nhất trước; tránh hỏi hai câu hỏi tác động thấp khi một vùng tác động cao (ví dụ: security posture) chưa được giải quyết.
   - Loại trừ các câu hỏi đã được trả lời, sở thích phong cách đơn giản, hoặc chi tiết thực thi ở cấp plan (trừ khi blocking tính đúng đắn).
   - Ưu tiên các làm rõ giảm rủi ro làm lại downstream hoặc ngăn ngừa các acceptance test bị lệch.
   - Nếu có hơn 5 danh mục chưa được giải quyết, chọn 5 hàng đầu bằng heuristic (Impact \* Uncertainty).
   - Nếu nhiều hơn 5 danh mục vẫn chưa được giải quyết, chọn 5 hàng đầu bằng heuristic (Impact \* Uncertainty).

4. Vòng lặp hỏi tuần tự (tương tác):
   - Trình bày CHÍNH XÁC MỘT câu hỏi tại một thời điểm.
   - Với các câu hỏi multiple-choice:
     - **Phân tích tất cả các lựa chọn** và xác định **lựa chọn phù hợp nhất** dựa trên:
       - Best practices cho loại dự án
       - Các pattern phổ biến trong các implementation tương tự
       - Giảm rủi ro (bảo mật, hiệu năng, khả năng bảo trì)
       - Căn chỉnh với bất kỳ mục tiêu hoặc ràng buộc dự án rõ ràng nào thấy được trong spec
     - Trình bày **lựa chọn được khuyến nghị của bạn một cách nổi bật** ở đầu kèm lý do rõ ràng (1-2 câu giải thích tại sao đây là lựa chọn tốt nhất).
     - Định dạng: `**Khuyến nghị:** Lựa chọn [X] - <lý do>`
     - Sau đó hiển thị tất cả các lựa chọn dưới dạng bảng Markdown:

     | Lựa chọn | Mô tả                                                                                           |
     | -------- | ----------------------------------------------------------------------------------------------- |
     | A        | <Mô tả Lựa chọn A>                                                                              |
     | B        | <Mô tả Lựa chọn B>                                                                              |
     | C        | <Mô tả Lựa chọn C> (thêm D/E nếu cần, tối đa 5)                                                 |
     | Ngắn     | Cung cấp một câu trả lời ngắn khác (<=5 từ) (Chỉ bao gồm nếu alternative dạng tự do là phù hợp) |
     - Sau bảng, thêm: `Bạn có thể trả lời bằng chữ cái lựa chọn (ví dụ: "A"), chấp nhận khuyến nghị bằng cách nói "có" hoặc "khuyến nghị", hoặc cung cấp câu trả lời ngắn của riêng bạn.`

   - Với dạng trả lời ngắn (không có các lựa chọn rời rạc có ý nghĩa):
     - Cung cấp **câu trả lời đề xuất** của bạn dựa trên best practices và ngữ cảnh.
     - Định dạng: `**Đề xuất:** <câu trả lời đề xuất của bạn> - <lý do ngắn gọn>`
     - Sau đó xuất: `Định dạng: Trả lời ngắn (<=5 từ). Bạn có thể chấp nhận đề xuất bằng cách nói "có" hoặc "đề xuất", hoặc cung cấp câu trả lời của riêng bạn.`
   - Sau khi người dùng trả lời:
     - Nếu người dùng trả lời "có", "khuyến nghị", hoặc "đề xuất", sử dụng khuyến nghị/đề xuất đã nêu trước đó làm câu trả lời.
     - Nếu không, xác thực câu trả lời ánh xạ tới một lựa chọn hoặc phù hợp ràng buộc <=5 từ.
     - Nếu mơ hồ, yêu cầu disambiguation nhanh (số lượng vẫn thuộc cùng câu hỏi; không chuyển tiếp).
     - Khi đã hài lòng, ghi lại vào bộ nhớ làm việc (chưa ghi vào đĩa) và chuyển sang câu hỏi tiếp theo trong hàng đợi.
   - Dừng hỏi thêm câu hỏi khi:
     - Tất cả các sự mơ hồ quan trọng đã được giải quyết sớm (các mục còn lại trong hàng đợi trở nên không cần thiết), HOẶC
     - Người dùng báo hiệu hoàn thành ("done", "tốt", "không còn nữa"), HOẶC
     - Đạt 5 câu hỏi đã hỏi.
   - Không bao giờ tiết lộ trước các câu hỏi tiếp theo trong hàng đợi.
   - Nếu không có câu hỏi hợp lệ ngay từ đầu, báo cáo ngay không có sự mơ hồ quan trọng nào.

5. Tích hợp sau MỖI câu trả lời được chấp nhận (cách tiếp cận cập nhật tăng dần):
   - Duy trì biểu diễn spec trong bộ nhớ (đã tải một lần ở đầu) cộng với nội dung file thô.
   - Với câu trả lời tích hợp đầu tiên trong session này:
     - Đảm bảo tồn tại phần `## Clarifications` (tạo nó ngay sau phần ngữ cảnh/tổng quan cấp cao nhất theo spec template nếu thiếu).
     - Dưới nó, tạo (nếu chưa có) tiêu đề con `### Session YYYY-MM-DD` cho ngày hôm nay.
   - Nối một dòng bullet ngay sau khi chấp nhận: `- Q: <câu hỏi> → A: <câu trả lời cuối cùng>`.
   - Sau đó áp dụng ngay clarification vào các phần phù hợp nhất:
     - Sự mơ hồ về chức năng → Cập nhật hoặc thêm bullet trong Functional Requirements.
     - Sự mơ hồ về tương tác người dùng / phân biệt actor → Cập nhật User Stories hoặc phần Actors (nếu có) với vai trò, ràng buộc hoặc kịch bản đã được làm rõ.
     - Hình dạng dữ liệu / entities → Cập nhật Data Model (thêm fields, types, relationships) giữ nguyên thứ tự; ghi chú các ràng buộc đã thêm một cách ngắn gọn.
     - Ràng buộc phi chức năng → Thêm/sửa tiêu chí đo lường được trong Success Criteria > Measurable Outcomes (chuyển tính từ mơ hồ thành metric hoặc target rõ ràng).
     - Edge case / luồng tiêu cực → Thêm bullet mới trong Edge Cases / Error Handling (hoặc tạo phần con đó nếu template có placeholder cho nó).
     - Xung đột thuật ngữ → Chuẩn hóa thuật ngữ trên toàn spec; chỉ giữ lại bản gốc nếu cần thiết bằng cách thêm `(trước đây được gọi là "X")` một lần.
   - Nếu clarification làm mất hiệu lực một câu mơ hồ trước đó, thay thế câu đó thay vì trùng lặp; không để lại văn bản mâu thuẫn cũ.
   - Lưu file spec SAU MỖI lần tích hợp để giảm thiểu rủi ro mất ngữ cảnh (ghi đè nguyên tử).
   - Giữ nguyên định dạng: không sắp xếp lại các phần không liên quan; giữ nguyên hierarchy của heading.
   - Giữ mỗi clarification được chèn tối thiểu và có thể kiểm thử (tránh dịch chuyển narrative).

6. Xác thực (thực hiện SAU MỖI lần ghi + lần pass cuối):
   - Session clarifications chứa chính xác một bullet cho mỗi câu trả lời được chấp nhận (không trùng lặp).
   - Tổng số câu hỏi đã hỏi (được chấp nhận) ≤ 5.
   - Các phần đã cập nhật không còn các placeholder mơ hồ còn sót lại mà câu trả lời mới được định giải quyết.
   - Không có câu mâu thuẫn trước đó còn lại (quét các lựa chọn thay thế bây giờ không hợp lệ đã bị loại bỏ).
   - Cấu trúc Markdown hợp lệ; chỉ các heading mới được phép: `## Clarifications`, `### Session YYYY-MM-DD`.
   - Tính nhất quán thuật ngữ: cùng thuật ngữ chuẩn được sử dụng trên tất cả các phần đã cập nhật.

7. Ghi spec đã cập nhật trở lại `FEATURE_SPEC`.

8. Báo cáo hoàn thành (sau khi vòng hỏi kết thúc hoặc kết thúc sớm):
   - Số câu hỏi đã hỏi & đã trả lời.
   - Đường dẫn tới spec đã cập nhật.
   - Các phần đã chạm vào (liệt kê tên).
   - Bảng tóm tắt coverage liệt kê mỗi danh mục taxonomy với Status: Resolved (trước đó là Partial/Missing và đã được giải quyết), Deferred (vượt quá quota câu hỏi hoặc phù hợp hơn cho planning), Clear (đã đủ), Outstanding (vẫn Partial/Missing nhưng tác động thấp).
   - Nếu có bất kỳ Outstanding hoặc Deferred nào còn lại, khuyến nghị có nên tiến hành `/speckit.plan` hay chạy `/speckit.clarify` lại sau plan.
   - Lệnh tiếp theo được đề xuất.

Các quy tắc hành vi:

- Nếu không tìm thấy sự mơ hồ quan trọng nào (hoặc tất cả các câu hỏi tiềm năng đều có tác động thấp), trả lời: "Không phát hiện sự mơ hồ quan trọng nào cần làm rõ chính thức." và đề xuất tiến hành.
- Nếu file spec bị thiếu, hướng dẫn người dùng chạy `/speckit.specify` trước (không tạo spec mới ở đây).
- Không bao giờ vượt quá 5 câu hỏi tổng cộng (các lần thử lại làm rõ cho một câu hỏi không tính là câu hỏi mới).
- Tránh các câu hỏi về tech stack mang tính suy đoán trừ khi việc thiếu nó chặn sự rõ ràng về chức năng.
- Tôn trọng các tín hiệu kết thúc sớm của người dùng ("stop", "done", "proceed").
- Nếu không có câu hỏi nào được hỏi do coverage đầy đủ, xuất tóm tắt coverage nhỏ gọn (tất cả các danh mục Clear) rồi đề xuất tiến triển.
- Nếu quota đã đạt nhưng vẫn còn các danh mục tác động cao chưa được giải quyết, đánh dấu rõ ràng chúng dưới Deferred kèm theo lý do.

Ngữ cảnh cho việc ưu tiên: $ARGUMENTS
