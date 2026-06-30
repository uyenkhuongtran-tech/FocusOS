# Nâng cấp Robot lên AI thật (Claude) — an toàn cho trẻ em

App hiện chạy **Robot offline** (an toàn, miễn phí, không cần key). Nếu muốn robot trò chuyện tự nhiên bằng **AI thật**, KHÔNG được đặt API key thẳng trong `index.html` (vì repo public → lộ key → bị lạm dụng, tốn tiền). Phải đi qua một **proxy backend** giữ key bí mật.

## Sơ đồ
```
index.html (GitHub Pages)  →  Proxy (giữ API key)  →  Claude API
        AI_ENDPOINT                Cloudflare Worker / Vercel
```

---

## Cách 1 — Cloudflare Worker (miễn phí, ~10 phút)

1. Tạo tài khoản Cloudflare → **Workers & Pages → Create Worker**.
2. Dán code dưới đây vào Worker:

```js
export default {
  async fetch(request, env) {
    // CORS cho phép GitHub Pages gọi
    const cors = {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "POST, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type",
    };
    if (request.method === "OPTIONS") return new Response(null, { headers: cors });
    if (request.method !== "POST") return new Response("OK", { headers: cors });

    const { message, context } = await request.json();

    const SYSTEM = `Bạn là "Robot Tích Tắc", người bạn robot thân thiện của một em bé khoảng 6 tuổi người Việt.
QUY TẮC BẮT BUỘC:
- Luôn trả lời bằng TIẾNG VIỆT, từ ngữ thật đơn giản, ấm áp, vui vẻ, 1-3 câu ngắn, có 1 emoji.
- Chỉ nói về: việc cần làm trong ngày, tập trung, lên kế hoạch ngày mai, thói quen tốt, lời khen và khích lệ.
- TUYỆT ĐỐI KHÔNG hỏi thông tin cá nhân (họ tên đầy đủ, địa chỉ, trường, số điện thoại, hình ảnh).
- KHÔNG nói về chủ đề người lớn, bạo lực, đáng sợ.
- Nếu bé buồn/sợ/bị đau/có chuyện nghiêm trọng: nhẹ nhàng an ủi và khuyên bé nói với ba mẹ hoặc thầy cô. Không đưa lời khuyên vượt quá việc an ủi.
- Luôn tích cực, không bao giờ chê bai hay làm bé buồn.
- Dùng thông tin tiến độ của bé để khen/nhắc đúng lúc.`;

    const userMsg = `Bé tên ${context?.name || "bé"}. Hôm nay làm xong ${context?.done||0}/${context?.total||0} việc, mục tiêu ${context?.goal||0}, điểm chăm chỉ ${context?.score||0}. Ngày mai đã lên ${context?.tomorrowCount||0} việc.
Bé nói: "${message}"`;

    const r = await fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "content-type": "application/json",
        "x-api-key": env.ANTHROPIC_API_KEY,
        "anthropic-version": "2023-06-01",
      },
      body: JSON.stringify({
        model: "claude-haiku-4-5-20251001",
        max_tokens: 200,
        system: SYSTEM,
        messages: [{ role: "user", content: userMsg }],
      }),
    });
    const data = await r.json();
    const reply = (data.content || []).filter(b => b.type === "text").map(b => b.text).join(" ").trim();
    return new Response(JSON.stringify({ reply: reply || "Mình ở đây với con nè! 🤖" }),
      { headers: { ...cors, "content-type": "application/json" } });
  },
};
```

3. **Settings → Variables and Secrets** → thêm secret `ANTHROPIC_API_KEY` = key từ https://console.anthropic.com (Encrypt).
4. Deploy → copy URL Worker (vd `https://kid-robot.<bạn>.workers.dev`).
5. Mở `index.html`, sửa dòng đầu phần script:
   ```js
   var AI_ENDPOINT = "https://kid-robot.<bạn>.workers.dev";
   ```
6. Push lại GitHub. Xong — robot giờ trả lời bằng AI thật; nếu mạng/AI lỗi sẽ tự rớt về robot offline (không bao giờ đứng hình).

---

## Lưu ý quan trọng
- **Chi phí:** mỗi câu trả lời tốn rất ít token (model Haiku rẻ), nhưng vẫn theo dõi usage ở Anthropic Console; có thể đặt **spend limit** để an toàn.
- **Bảo vệ key:** key chỉ nằm trong Worker (secret), không bao giờ ở client.
- **Quyền riêng tư của trẻ:** chỉ gửi dữ liệu tối thiểu (tên gọi + số liệu việc). Cân nhắc dùng biệt danh thay tên thật. Có thể thêm rate-limit/đổi `Access-Control-Allow-Origin` thành đúng domain GitHub Pages để hạn chế lạm dụng.
- **Giám sát của phụ huynh:** AI thật là hội thoại mở — nên có người lớn ở bên khi bé dùng. System prompt đã ràng buộc an toàn, nhưng giám sát vẫn là tốt nhất.

## Cách 2 — Vercel / Netlify Function
Tương tự: tạo một serverless function nhận `{message,context}`, gọi Claude API với key đặt trong Environment Variables, trả `{reply}`. Rồi gán URL đó vào `AI_ENDPOINT`.
