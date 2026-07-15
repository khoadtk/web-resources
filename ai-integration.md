# AI integration

Nhúng AI vào web: chatbot, tóm tắt, search ngữ nghĩa (RAG), gợi ý.

## Nhà cung cấp

- **[Claude / Anthropic](https://docs.anthropic.com)** — model mạnh, hợp tác vụ dài, tool use
- **[OpenAI](https://platform.openai.com)** — phổ biến, nhiều model
- **[Google Gemini](https://ai.google.dev)** — đa phương thức
- **[OpenRouter](https://openrouter.ai)** — 1 API dùng nhiều model
- **[Ollama](https://ollama.com)** — chạy model local (dev/offline)

## Thư viện / công cụ

- **[Vercel AI SDK](https://sdk.vercel.ai)** — chuẩn cho web: streaming, chat UI, tool call (khuyên dùng)
- **[LangChain](https://js.langchain.com)** — orchestrate chain/agent phức tạp
- **Vector DB** cho RAG: [Pinecone](https://pinecone.io), [Supabase pgvector](https://supabase.com/docs/guides/ai), [Qdrant](https://qdrant.tech)

## Khái niệm

- **Chat completion** — gửi tin nhắn, nhận trả lời (thường **stream** từng chữ).
- **Embeddings** — biến text thành vector để so độ giống → nền của search ngữ nghĩa.
- **RAG** — tìm đoạn liên quan trong dữ liệu của bạn → nhét vào prompt → model trả lời có căn cứ.
- **Tool/function calling** — cho model gọi hàm của bạn (query DB, gọi API).

## Cách dùng

### HTML tĩnh

**KHÔNG gọi API AI trực tiếp từ client** (lộ API key). Phải qua backend/proxy. Trang tĩnh → gọi tới 1 endpoint server của bạn:

```js
const res = await fetch('/api/chat', {
  method: 'POST', body: JSON.stringify({ message: 'Xin chào' }),
});
```

### React

Dùng **Vercel AI SDK** (`useChat`) — lo sẵn streaming + state:

```bash
npm install ai @ai-sdk/react
```

```jsx
import { useChat } from '@ai-sdk/react';
const { messages, input, handleInputChange, handleSubmit } = useChat();
// render messages + form; backend là endpoint /api/chat
```

### Next.js

Backend chạy trên **Route Handler**, key ở server, trả về stream:

```jsx
// app/api/chat/route.ts
import { anthropic } from '@ai-sdk/anthropic';
import { streamText } from 'ai';
export async function POST(req) {
  const { messages } = await req.json();
  const result = streamText({ model: anthropic('claude-...'), messages });
  return result.toDataStreamResponse();
}
```

> Model id, giá, tham số Claude → tra qua skill **claude-api** trước khi code, đừng nhớ mò.

## Mẹo

- **API key chỉ ở server** — tuyệt đối không nhét vào client (xem [security.md](security.md)).
- **Stream** câu trả lời → UX tốt hơn hẳn chờ nguyên đoạn.
- Đặt **giới hạn/timeout + rate limit** → tránh cháy chi phí.
- RAG: chia nhỏ tài liệu (chunk) hợp lý, lưu embedding vào vector DB, lấy top-k khi hỏi.
- Cân nhắc **cache** câu trả lời hay lặp để tiết kiệm token.
