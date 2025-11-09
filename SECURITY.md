# Security Policy

## Supported Versions

Use this section to tell people about which versions of your project are
currently being supported with security updates.

| Version | Supported          |
| ------- | ------------------ |
| 5.1.x   | :white_check_mark: |
| 5.0.x   | :x:                |
| 4.0.x   | :white_check_mark: |
| < 4.0   | :x:                |

## Reporting a Vulnerability

Use this section to tell people how to report a vulnerability.

Tell them where to go, how often they can expect to get an update on a
reported vulnerability, what to expect if the vulnerability is accepted or
declined, etc.
// pages/api/analyze.js
import fetch from 'node-fetch';

export default async function handler(req, res) {
  // Hämta ticker från query
  const ticker = (req.query.ticker || 'AAPL').toUpperCase();

  // Finnhub API-key (för test lokalt kan du hårdkoda, annars använd process.env.FINNHUB_API_KEY)
  const finnhubKey = 'd48fue9r01qnpsnnloggd48fue9r01qnpsnnloh0';

  // Hämta marknadsdata från Finnhub
  const fbResp = await fetch(`https://finnhub.io/api/v1/quote?symbol=${ticker}&token=${finnhubKey}`);
  const fbJson = await fbResp.json();

  // Skapa prompt till OpenAI
  const prompt = `
Du är en svensk aktieanalytiker. Här är data för ${ticker}:
- Aktuellt pris: ${fbJson.c}
- Öppning: ${fbJson.o}
- Hög: ${fbJson.h}
- Låg: ${fbJson.l}
- Volym: ${fbJson.v}

Skriv en kort analys på svenska: trend, kort rekommendation (Köp/Håll/Sälj) och varför.
  `;

  // OpenAI API-key (för test lokalt kan du hårdkoda, annars process.env.OPENAI_API_KEY)
  const openaiKey = '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';

  // Skicka prompt till OpenAI
  const openaiResp = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${openaiKey}`,
    },
    body: JSON.stringify({
      model: 'gpt-4o-mini',
      messages: [
        { role: 'system', content: 'Du är en hjälpsam aktieanalytiker.' },
        { role: 'user', content: prompt }
      ],
      max_tokens: 300,
    }),
  });

  const openaiJson = await openaiResp.json();
  const analysis = openaiJson.choices?.[0]?.message?.content || 'Ingen analys returned';

  // Returnera JSON till frontend
  res.status(200).json({
    ticker,
    marketData: fbJson,
    analysis
  });
}
