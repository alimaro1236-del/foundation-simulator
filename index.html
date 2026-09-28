// gemini-proxy.js
// وسيط آمن بين الموقع و Gemini API.
// الميزة الأساسية هنا: لو موديل زحمة (503) أو اتشال (404)، بيجرب موديل تاني
// تلقائي من غير ما يوقف الموقع، وبيعيد المحاولة بفاصل زمني بسيط قبل ما يفشل.

const MODELS_TO_TRY = [
  "gemini-3.8-flash",
  "gemini-flash-latest",
];

const MAX_RETRIES_PER_MODEL = 2;   // عدد المحاولات لكل موديل قبل ما ينتقل للي بعده
const RETRY_DELAY_MS = 800;        // فاصل زمني بسيط بين المحاولات

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function callGemini(model, apiKey, systemInstruction, contents) {
  const url = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;
  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      systemInstruction: { parts: [{ text: systemInstruction }] },
      contents: contents,
    }),
  });
  const data = await res.json().catch(() => ({}));
  return { status: res.status, data };
}

exports.handler = async function (event) {
  if (event.httpMethod !== "POST") {
    return { statusCode: 405, body: JSON.stringify({ error: "Method Not Allowed" }) };
  }

  let payload;
  try {
    payload = JSON.parse(event.body || "{}");
  } catch (e) {
    return { statusCode: 400, body: JSON.stringify({ error: "طلب غير صالح (JSON خطأ)" }) };
  }

  const { systemInstruction, contents } = payload;

  if (!contents) {
    return { statusCode: 400, body: JSON.stringify({ error: "الرسالة فارغة" }) };
  }

  const apiKey = process.env.GEMINI_API_KEY;
  if (!apiKey) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: "GEMINI_API_KEY غير موجود في إعدادات الموقع (Environment variables)" }),
    };
  }

  let lastError = null;

  // نجرب كل موديل بالترتيب، وكل موديل بيتعاد عليه المحاولة لو الخطأ مؤقت (503)
  for (const model of MODELS_TO_TRY) {
    for (let attempt = 1; attempt <= MAX_RETRIES_PER_MODEL; attempt++) {
      try {
        const { status, data } = await callGemini(model, apiKey, systemInstruction, contents);

        // نجاح
        if (status >= 200 && status < 300) {
          return {
            statusCode: 200,
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ ...data, _modelUsed: model }),
          };
        }

        // خطأ في المفتاح نفسه — مفيش فايدة نكرر أو نجرب موديل تاني
        if (status === 400 || status === 403) {
          return {
            statusCode: status,
            body: JSON.stringify({ error: "مشكلة في مفتاح الـ API (تأكد إنه صحيح ومفعّل)", details: data }),
          };
        }

        lastError = { status, data };

        // 503 = زحمة مؤقتة → نعيد المحاولة على نفس الموديل، وبعدين ننتقل للي بعده
        // 404 = الموديل مش متاح → ننتقل على طول للموديل اللي بعده
        if (status === 404) break;

        if (status === 503 && attempt < MAX_RETRIES_PER_MODEL) {
          await sleep(RETRY_DELAY_MS);
          continue;
        }
      } catch (err) {
        lastError = { status: 500, data: { error: err.message } };
      }
    }
  }

  // كل المحاولات فشلت — نرجّع رسالة واضحة للمستخدم
  return {
    statusCode: lastError ? lastError.status : 500,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      error: "الخدمة مزحومة حاليًا أو غير متاحة، جرّب تاني بعد لحظات.",
      details: lastError ? lastError.data : null,
    }),
  };
};
