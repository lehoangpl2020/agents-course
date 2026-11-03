# Thực hành Ollama local trên Windows

Guide này dành riêng cho máy hiện tại:

- Windows, Ollama `0.40.13` đã được cài và đang chạy.
- NVIDIA RTX 3060 Laptop, 6 GB VRAM.
- RAM khoảng 16 GB.
- Ổ C còn khoảng 46.5 GB tại thời điểm kiểm tra.
- Project đã có OpenAI Python SDK, nên có thể gọi Ollama qua endpoint tương thích OpenAI mà không cần API key thật.

Mục tiêu là hiểu Ollama bằng cách tự chạy model, gọi API, đo tốc độ, điều chỉnh tham số, tạo model tùy biến và làm một coding assistant nhỏ. Tất cả bài tập bên dưới chạy local; không cần đăng nhập, cloud hay API key.

## 0. Chọn model phù hợp với máy

| Model | Dung lượng model | Dùng để làm gì | Khuyến nghị |
|---|---:|---|---|
| `llama3.2:3b` | khoảng 2.0 GB | Chat, tóm tắt, tool calling cơ bản | Model học chính |
| `qwen2.5-coder:3b` | khoảng 1.9 GB | Sinh code, giải thích và sửa code | Model coding chính |
| `gemma3:4b` | khoảng 3.3 GB | Chat và thử vision | Chỉ tải khi cần lab vision |

Không nên bắt đầu bằng model 14B, 32B hoặc lớn hơn. Trọng số model không phải phần duy nhất dùng bộ nhớ: context và cache cũng cần VRAM/RAM. Model lớn có thể chạy một phần bằng RAM nhưng sẽ chậm, và máy chỉ có 16 GB RAM.

Nguyên tắc thực hành:

1. Bắt đầu với `llama3.2:3b`.
2. Dùng context nhỏ, khoảng 2K–8K token, trong các bài đầu.
3. Đóng bớt trình duyệt hoặc ứng dụng nặng trước khi benchmark.
4. Chỉ tải thêm model khi biết mình cần nó.

## 1. Lab 1 — Kiểm tra hệ thống và chạy chat đầu tiên

Mở một PowerShell mới để chắc chắn PATH mới được nạp.

```powershell
ollama --version
ollama list
Invoke-RestMethod http://localhost:11434/api/version
```

Kết quả mong đợi:

- `ollama --version` in ra phiên bản.
- `ollama list` ban đầu có thể chưa có model nào.
- API trả về JSON chứa version.

Tải model đầu tiên:

```powershell
ollama pull llama3.2:3b
```

Chạy chat:

```powershell
ollama run llama3.2:3b
```

Thử lần lượt các prompt sau:

```text
Giải thích sự khác nhau giữa RAM và VRAM cho người mới học AI.
```

```text
Viết một hàm Python kiểm tra số nguyên tố, sau đó giải thích từng bước.
```

```text
Tóm tắt câu trả lời vừa rồi trong đúng 3 gạch đầu dòng.
```

Trong phiên chat:

- Gõ `/help` để xem lệnh.
- Gõ `/show info` để xem thông tin model.
- Gõ `/bye` để thoát.

Checklist:

- [ ] Model tải thành công.
- [ ] Chat được bằng tiếng Việt.
- [ ] Hiểu rằng lần đầu model load sẽ chậm hơn các lần sau.

## 2. Lab 2 — Học các lệnh quản lý model

```powershell
ollama list
ollama show llama3.2:3b
ollama ps
```

Ý nghĩa:

- `ollama list`: các model đã tải về.
- `ollama show`: metadata, template, capability và tham số của model.
- `ollama ps`: model nào đang nằm trong RAM/VRAM, context hiện tại và processor.

Sau khi chat, chạy:

```powershell
ollama ps
nvidia-smi
```

Quan sát:

- VRAM tăng bao nhiêu sau khi model được load?
- `PROCESSOR` trong `ollama ps` là GPU, CPU hay kết hợp?
- Model còn được giữ trong bộ nhớ bao lâu?

Khi cần xóa model:

```powershell
ollama rm llama3.2:3b
```

Đừng chạy lệnh xóa lúc này nếu muốn làm các lab tiếp theo.

## 3. Lab 3 — Gọi Ollama Native API bằng PowerShell

Ollama local phục vụ API tại `http://localhost:11434/api`. Local API không cần authentication.

```powershell
$body = @{
    model = "llama3.2:3b"
    messages = @(
        @{
            role = "system"
            content = "Bạn là gia sư Python. Trả lời ngắn gọn bằng tiếng Việt."
        },
        @{
            role = "user"
            content = "List comprehension là gì? Cho một ví dụ."
        }
    )
    stream = $false
    options = @{
        temperature = 0.2
        num_ctx = 4096
    }
} | ConvertTo-Json -Depth 6

$response = Invoke-RestMethod `
    -Method Post `
    -Uri http://localhost:11434/api/chat `
    -ContentType "application/json" `
    -Body $body

$response.message.content
```

Xem metadata hiệu năng:

```powershell
$response | Select-Object model, done_reason, total_duration, load_duration, eval_count, eval_duration

$tokensPerSecond = $response.eval_count / ($response.eval_duration / 1e9)
"{0:N2} tokens/second" -f $tokensPerSecond
```

Bài tập:

1. Chạy cùng prompt hai lần và so sánh `load_duration`.
2. Đổi `temperature` từ `0.0` sang `1.0`; ghi lại khác biệt.
3. Đổi `num_ctx` từ `2048` sang `8192`; quan sát VRAM bằng `nvidia-smi`.

Kết luận cần tự rút ra:

- `temperature` điều khiển mức ngẫu nhiên, không làm model thông minh hơn.
- Context lớn hơn tốn nhiều bộ nhớ hơn.
- `load_duration` thường giảm mạnh khi model đã nằm trong bộ nhớ.

## 4. Lab 4 — Gọi endpoint tương thích OpenAI trong notebook

Course này đã cài `openai`, nên không cần cài thêm package. Ollama expose endpoint tương thích OpenAI tại `http://localhost:11434/v1`.

Tạo một notebook mới hoặc thêm cell vào bản notebook có thể chỉnh sửa. Không sửa snapshot read-only.

```python
from openai import OpenAI

MODEL = "llama3.2:3b"

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama",  # SDK bắt buộc có chuỗi; Ollama local bỏ qua giá trị này
)

response = client.chat.completions.create(
    model=MODEL,
    messages=[
        {
            "role": "system",
            "content": "Bạn là gia sư AI, trả lời rõ ràng bằng tiếng Việt.",
        },
        {
            "role": "user",
            "content": "Giải thích token, context window và quantization.",
        },
    ],
    temperature=0.2,
)

print(response.choices[0].message.content)
```

Thử streaming:

```python
stream = client.chat.completions.create(
    model=MODEL,
    messages=[
        {"role": "user", "content": "Viết 5 mẹo học Python hiệu quả."}
    ],
    stream=True,
)

for chunk in stream:
    text = chunk.choices[0].delta.content
    if text:
        print(text, end="", flush=True)
```

Checklist:

- [ ] Không dùng OpenAI API key thật.
- [ ] Request đi tới `localhost`, không phải cloud.
- [ ] Đổi model chỉ bằng cách pull model và đổi biến `MODEL`.

## 5. Lab 5 — So sánh model general và model coding

Tải model coding:

```powershell
ollama pull qwen2.5-coder:3b
ollama list
```

Dùng cùng một prompt cho cả hai model:

```text
Viết hàm Python parse_duration(text) nhận "1h 30m", "45s", "2h" và trả về tổng số giây. Bao gồm type hints, validation và 6 test case.
```

Chạy lần lượt:

```powershell
ollama run llama3.2:3b
ollama run qwen2.5-coder:3b
```

Chấm mỗi câu trả lời theo thang 0–2:

| Tiêu chí | 0 | 1 | 2 |
|---|---|---|---|
| Đúng yêu cầu | Sai nhiều | Đúng một phần | Đúng đầy đủ |
| Chất lượng code | Khó dùng | Dùng được | Sạch, rõ |
| Validation | Không có | Thiếu case | Hợp lý |
| Test | Không chạy | Thiếu | Đủ và hữu ích |
| Giải thích | Mơ hồ | Tạm ổn | Rõ ràng |

Đừng chỉ hỏi model nào “hay hơn”. Hãy đo trên đúng loại task mình sẽ dùng.

## 6. Lab 6 — Output JSON có cấu trúc

Native API hỗ trợ `format = "json"` hoặc JSON Schema. Bài đầu dùng chế độ JSON đơn giản:

```powershell
$body = @{
    model = "llama3.2:3b"
    messages = @(
        @{
            role = "user"
            content = "Phân tích câu: 'Hôm nay tôi học Ollama rất vui'. Trả JSON với các field language, sentiment, summary. Không thêm markdown."
        }
    )
    format = "json"
    stream = $false
    options = @{
        temperature = 0
    }
} | ConvertTo-Json -Depth 6

$response = Invoke-RestMethod `
    -Method Post `
    -Uri http://localhost:11434/api/chat `
    -ContentType "application/json" `
    -Body $body

$data = $response.message.content | ConvertFrom-Json
$data | Format-List
```

Bài tập:

1. Thêm field `keywords` là array string.
2. Cố tình viết prompt mơ hồ và xem JSON có còn ổn định không.
3. Giữ prompt, chạy 5 lần với `temperature = 0`.

## 7. Lab 7 — Tạo model tùy biến bằng Modelfile

Tạo file tên `Modelfile` trong một thư mục thực hành:

```text
FROM llama3.2:3b

PARAMETER temperature 0.2
PARAMETER num_ctx 4096

SYSTEM Bạn là gia sư Ollama cho người Việt. Luôn giải thích ngắn gọn, đưa ví dụ thực tế, và kết thúc bằng một bài tập nhỏ. Không bịa thông tin khi không chắc.
```

Tại thư mục chứa file, chạy:

```powershell
ollama create ollama-tutor-vi -f .\Modelfile
ollama run ollama-tutor-vi
```

Prompt kiểm tra:

```text
Giải thích quantization trong LLM.
```

Kiểm tra model được tạo từ đâu:

```powershell
ollama show --modelfile ollama-tutor-vi
```

Bài tập:

1. Đổi system prompt sang code reviewer.
2. Tạo model mới tên `local-code-reviewer` dựa trên `qwen2.5-coder:3b`.
3. Yêu cầu model luôn trả về các mục: lỗi, rủi ro, đề xuất, test.

## 8. Lab 8 — Tool calling: nền móng của agent

Model không tự chạy function. Model chỉ đề xuất tool call; chương trình của mình phải xác thực input, chạy function, rồi gửi kết quả lại cho model.

Thử tool calling với OpenAI-compatible endpoint:

```python
import json
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama",
)

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_course_progress",
            "description": "Lấy tiến độ học của một học viên",
            "parameters": {
                "type": "object",
                "properties": {
                    "student": {"type": "string"},
                },
                "required": ["student"],
            },
        },
    }
]

response = client.chat.completions.create(
    model="llama3.2:3b",
    messages=[
        {"role": "user", "content": "Kiểm tra tiến độ của Huy."}
    ],
    tools=tools,
    temperature=0,
)

message = response.choices[0].message
print("Text:", message.content)

for call in message.tool_calls or []:
    print("Tool:", call.function.name)
    print("Arguments:", json.loads(call.function.arguments))
```

Mục tiêu lab này là thấy model tạo ra tên function và arguments. Chưa cho model quyền đọc file, chạy shell hay sửa code. Khi làm agent thật, luôn giới hạn tool và validate arguments.

## 9. Mini project — Local coding assistant

Yêu cầu:

1. Dùng `qwen2.5-coder:3b`.
2. Nhận một đoạn code do mình paste vào.
3. Yêu cầu model trả về bốn phần:
   - Tóm tắt code.
   - Bug hoặc edge case.
   - Bản sửa đề xuất.
   - Test cần thêm.
4. Stream câu trả lời ra terminal.
5. Không tự động ghi đè file.

Khung code:

````python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama",
)

code = """
def divide(a, b):
    return a / b
"""

prompt = f"""
Review đoạn code sau.

Trả lời với đúng 4 heading:
1. Tóm tắt
2. Bug và edge case
3. Code đề xuất
4. Test đề xuất

Code:
```python
{code}
```
"""

stream = client.chat.completions.create(
    model="qwen2.5-coder:3b",
    messages=[
        {
            "role": "system",
            "content": "Bạn là code reviewer cẩn thận. Không khẳng định code đã chạy nếu chưa thực sự chạy.",
        },
        {"role": "user", "content": prompt},
    ],
    temperature=0.1,
    stream=True,
)

for chunk in stream:
    text = chunk.choices[0].delta.content
    if text:
        print(text, end="", flush=True)
````

Nâng cấp sau khi bản cơ bản chạy:

- Cho người dùng chọn model.
- Ghi thời gian phản hồi.
- Lưu prompt và kết quả vào file log.
- Thêm JSON Schema để kết quả dễ parse.
- Chỉ sau cùng mới thêm tool đọc file, và phải giới hạn trong một thư mục an toàn.

## 10. Cách benchmark có ý nghĩa

Tạo một bộ 10 prompt cố định thuộc đúng nhu cầu của mình:

- 2 prompt giải thích khái niệm.
- 2 prompt tóm tắt.
- 2 prompt sinh code.
- 2 prompt sửa bug.
- 2 prompt structured output hoặc tool calling.

Với mỗi model, ghi:

| Model | Task | Đúng yêu cầu | Chất lượng | Tokens/s | VRAM | Ghi chú |
|---|---|---:|---:|---:|---:|---|
| `llama3.2:3b` | | | | | | |
| `qwen2.5-coder:3b` | | | | | | |

Giữ nguyên prompt, temperature và context khi so sánh. Chạy ít nhất ba lần rồi lấy trung bình tokens/s.

## 11. Lỗi thường gặp

### `ollama` không được nhận diện

Đóng terminal cũ, mở PowerShell mới rồi chạy lại:

```powershell
ollama --version
```

Binary mặc định nằm ở `%LOCALAPPDATA%\Programs\Ollama`.

### API không phản hồi

Mở Ollama từ Start Menu. Sau đó kiểm tra:

```powershell
Invoke-RestMethod http://localhost:11434/api/version
```

Chỉ chạy `ollama serve` thủ công nếu app nền chưa chạy. Nếu port đã được dùng bởi Ollama app, không cần mở server thứ hai.

### Model chạy bằng CPU hoặc rất chậm

```powershell
ollama ps
nvidia-smi
```

Nếu model quá lớn, chuyển về model 3B. Đóng ứng dụng đang dùng nhiều RAM/VRAM và giảm `num_ctx`.

### Hết dung lượng ổ đĩa

```powershell
ollama list
ollama rm <model-name>
```

Chỉ xóa đúng model đã kiểm tra trong danh sách.

### Output không ổn định

- Giảm `temperature` về `0`–`0.2`.
- Viết rõ format cần trả về.
- Với dữ liệu cho chương trình xử lý, dùng structured output/JSON Schema.
- Model nhỏ có giới hạn; prompt tốt không thể bù hoàn toàn năng lực model.

## 12. Lộ trình 5 buổi

### Buổi 1 — CLI và model lifecycle

- Làm Lab 1–2.
- Hiểu pull, run, list, show, ps, rm.

### Buổi 2 — API và hiệu năng

- Làm Lab 3–4.
- Đo tokens/s, load time, VRAM và context.

### Buổi 3 — Chọn model bằng benchmark

- Làm Lab 5 và phần benchmark.
- So sánh general model với coding model.

### Buổi 4 — Output đáng tin cậy

- Làm Lab 6–7.
- Học JSON output và Modelfile.

### Buổi 5 — Agent cơ bản

- Làm Lab 8 và mini project.
- Hiểu ranh giới giữa “model đề xuất tool call” và “code thực sự chạy tool”.

## Tài liệu chính thức

- [Quickstart](https://docs.ollama.com/quickstart)
- [Windows](https://docs.ollama.com/windows)
- [CLI reference](https://docs.ollama.com/cli)
- [API introduction](https://docs.ollama.com/api/introduction)
- [Chat API](https://docs.ollama.com/api/chat)
- [Structured outputs](https://docs.ollama.com/capabilities/structured-outputs)
- [Tool calling](https://docs.ollama.com/capabilities/tool-calling)
- [Modelfile](https://docs.ollama.com/modelfile)
- [Hardware support](https://docs.ollama.com/gpu)
- [Llama 3.2 3B](https://ollama.com/library/llama3.2:3b)
- [Qwen 2.5 Coder 3B](https://ollama.com/library/qwen2.5-coder:3b)

## Điểm bắt đầu ngay bây giờ

Chạy đúng ba lệnh này:

```powershell
ollama pull llama3.2:3b
ollama run llama3.2:3b
ollama ps
```

Sau đó hoàn thành Lab 1 trước khi tải model thứ hai.
