# Nhật ký học AI Engineer Agentic Track

Khóa học: Ed Donner — AI Engineer Agentic Track  
Repo: https://github.com/lehoangpl2020/agents-course  
Bắt đầu ghi: 2026-11-01

## Tiếp tục từ đây

- Đang học: Week 1 — Foundations, `1_foundations/1_lab1.ipynb`.
- Sếp đã xác nhận mở và chạy bài đầu tiên; chưa xác nhận hoàn thành toàn bộ lab và Exercise.
- Exercise đã có code cho ba bước: chọn lĩnh vực → tìm pain point → đề xuất giải pháp Agentic AI.
- Bước nhỏ tiếp theo: xác nhận đã chạy đến bước nào trong Exercise, xem kết quả và giải thích cách dữ liệu đi qua `business_area`, `pain_point`, `solution`.
- Cần bổ sung: tên video Udemy hiện tại để đối chiếu chính xác với repo.

## Quy ước đồng hành

- Giải thích bằng tiếng Việt, gọi người học là “sếp”, xưng “em”.
- Mỗi lượt một nhóm bước nhỏ; giải thích mục đích trước khi sửa hoặc chạy code.
- Bám thứ tự khóa học và code thực tế; giải thích Python và đối chiếu ngắn với C# khi hữu ích.
- Chỉ đánh dấu `[x]` khi sếp xác nhận hoàn thành mục tương ứng. Chạy được code và hiểu bài được ghi nhận riêng.
- Cập nhật file sau mỗi chặng học: đã làm gì, hiểu gì, vướng gì, bước tiếp theo.
- Không ghi API keys, token, hoặc output chứa thông tin bí mật vào nhật ký.

## Theo dõi tiến trình

| Chặng | Trạng thái ghi nhận | Hoàn thành do sếp xác nhận |
| --- | --- | --- |
| Setup môi trường | Đã kiểm tra công cụ và `.venv`; sếp đã mở và chạy bài đầu | [ ] |
| Week 1 — Lab 1 | Đã chạy bài đầu theo xác nhận của sếp; cần chốt phạm vi đã chạy và phần đã hiểu | [ ] |
| Week 1 — Lab 1 Exercise | Đã viết code, kiểm tra cú pháp và push; cần sếp xác nhận kết quả thực hành | [ ] |
| Week 1 — Lab 2 | Chưa ghi nhận bắt đầu | [ ] |
| Phần 2 — OpenAI (`2_openai`) | Chưa ghi nhận bắt đầu | [ ] |
| Phần 3 — CrewAI (`3_crewai`) | Chưa ghi nhận bắt đầu | [ ] |
| Phần 4 — LangChain / LangGraph (`4_langchain_langgraph`) | Chưa ghi nhận bắt đầu | [ ] |
| Phần 5 — Agent frameworks (`5_agent_frameworks`) | Chưa ghi nhận bắt đầu | [ ] |
| Phần 6 — MCP (`6_mcp`) | Chưa ghi nhận bắt đầu | [ ] |

## Note kỹ thuật

### Môi trường đã kiểm tra ngày 2026-11-01

- Windows 11 Pro, PowerShell 7.6.5; Git 2.53.0; uv 0.13.6.
- Python ngoài repo: `C:\Python314\python.exe`, phiên bản 3.14.4.
- Python riêng của repo: `.venv\Scripts\python.exe`, phiên bản 3.12.12, khớp `.python-version`.
- `.venv` tách thư viện của repo khỏi Python bên ngoài. Tại thời điểm kiểm tra, terminal chưa kích hoạt `.venv`, nên lệnh `python` trỏ tới Python 3.14.4.
- Chưa xác minh trực tiếp kernel đang chọn trong editor.

### Lab 1 — Luồng dữ liệu

```text
Tạo câu hỏi → question
Trả lời câu hỏi → answer
Ghép question + answer vào prompt → đánh giá câu trả lời
```

- Python quyết định thứ tự các bước và truyền kết quả sang prompt kế tiếp.
- `messages = [...]` gán một list mới. Nội dung cần dùng ở bước sau phải được đưa vào prompt bước đó.
- `[{"role": "user", "content": question}]`: list chứa một dictionary; `question` là giá trị biến, còn `"question"` sẽ là chuỗi chữ cố định.
- `f"...{question}..."`: nội suy biến vào chuỗi, gần với `$"...{question}..."` trong C#.
- `f"""..."""`: chuỗi nhiều dòng có nội suy biến.
- `response.choices[0].message.content`: lấy nội dung câu trả lời đầu tiên; chỉ số list bắt đầu từ 0 như C#.
- Notebook giữ biến trong kernel giữa các cell. Khi restart kernel, cần chạy lại các cell tạo biến trước khi dùng chúng.

### Exercise — Chuỗi ba bước

```text
Chọn lĩnh vực → business_area
business_area → tìm vấn đề → pain_point
business_area + pain_point → đề xuất giải pháp → solution
```

- Code được chia thành ba cell để xem kết quả từng bước; mỗi cell có một lệnh gọi API.
- Kết quả được yêu cầu trả lời bằng tiếng Việt.
- Câu cần tự giải thích: vì sao bước đề xuất giải pháp phải nhận cả lĩnh vực và pain point?

## Nhật ký các buổi

### 2026-11-01 — Setup và Lab 1

- Đã đọc README, setup Windows và cấu hình Python; không thấy `AGENTS.md` ở gốc repo hoặc các thư mục cha tại thời điểm kiểm tra.
- Đã kiểm tra Git, Python, uv và `.venv`; không cài thêm công cụ.
- Sếp xác nhận đã mở và chạy bài đầu tiên.
- Đã thay phần Exercise còn dang dở bằng ba cell; kiểm tra cú pháp Python và cấu trúc notebook. Trợ lý không thực thi API.
- Đã commit và push thay đổi Lab 1: [`af04c229`](https://github.com/lehoangpl2020/agents-course/commit/af04c229).
- Đã loại bỏ output chứa đoạn giống tiền tố API key trước khi push.
- Chưa chốt hoàn thành lab hoặc Exercise; cần sếp xác nhận.

## Câu hỏi / vấn đề đang mở

- [ ] Tên video đang học tại https://www.udemy.com/course/the-complete-agentic-ai-engineering-course/learn/lecture/49770895
- [ ] Exercise đã chạy đến cell nào? Kết quả có nối đúng lĩnh vực → vấn đề → giải pháp không?
- [ ] Sếp đã tự giải thích được cách truyền kết quả giữa ba lần gọi model chưa?

## Mẫu ghi buổi tiếp theo

```markdown
### YYYY-MM-DD — Tên bài / chủ đề

- Video / notebook:
- Mục tiêu nhỏ:
- Đã làm và kết quả:
- Điều đã hiểu (tự diễn đạt):
- Lỗi gặp / nguyên nhân / cách xử lý:
- Câu hỏi còn mở:
- Bước nhỏ tiếp theo:
- Mục hoàn thành được sếp xác nhận:
```
