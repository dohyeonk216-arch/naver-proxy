// Vercel Serverless Function
// GET /api/price?codes=005930,360750
// 네이버페이 증권의 비공식 공개 시세 JSON을 대신 호출해주는 프록시입니다.
// 브라우저에서 직접 호출하면 CORS로 막히기 때문에, 서버에서 대신 요청하고
// CORS 허용 헤더를 붙여서 돌려줍니다. 키나 계좌 정보는 전혀 필요 없습니다.

export default async function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, OPTIONS');
  if (req.method === 'OPTIONS') {
    res.status(200).end();
    return;
  }

  const codes = req.query.codes;
  if (!codes || !/^[0-9A-Za-z,]+$/.test(codes)) {
    res.status(400).json({ error: '유효한 codes 파라미터가 필요합니다. 예: ?codes=005930,360750' });
    return;
  }

  try {
    const naverRes = await fetch(
      `https://polling.finance.naver.com/api/realtime/domestic/stock/${codes}`,
      { headers: { 'User-Agent': 'Mozilla/5.0' } }
    );

    if (!naverRes.ok) {
      res.status(naverRes.status).json({ error: '네이버 응답 오류', status: naverRes.status });
      return;
    }

    const data = await naverRes.json();
    const list = data?.datas || [];

    const out = {};
    list.forEach((d) => {
      if (d && d.itemCode) {
        const priceStr = d.closePriceRaw ?? d.closePrice;
        const rateStr = d.fluctuationsRatioRaw ?? d.fluctuationsRatio;
        out[d.itemCode] = {
          name: d.stockName || null,
          price: Number(String(priceStr).replace(/,/g, '')),
          changePct: Number(String(rateStr).replace(/,/g, '')),
          marketStatus: d.marketStatus || null,
          tradedAt: d.localTradedAt || null,
        };
      }
    });

    res.status(200).json({ prices: out, fetchedAt: new Date().toISOString() });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
}
