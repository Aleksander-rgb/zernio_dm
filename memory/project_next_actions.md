---
name: Następne akcje — autoresponder KURS
description: Konkretne kroki do wykonania: filtr post ID w workflow mieczuuuzaw + obsługa promowanej rolki zdrowodosetki
type: project
originSessionId: e7293c7e-81ce-4509-8ad9-70de4299abcf
---
## Zadanie 1 — Ograniczenie workflow #1 do jednego posta (mieczuuuzaw)

**Workflow:** `IG KURS - Auto Responder DM v2` | ID: `YQSu2NUpydOhWlPm`

Dodaj węzeł **IF** po webhoku:
```
{{ $json.body.post.id }} == 698cc7da03b17f794bced079
```
- Post: `https://www.instagram.com/p/DUoKPwIEan3/` | Zernio post ID: `698cc7da03b17f794bced079`
- Konto: mieczuuuzaw (`692de4f6f43160a0bc999b6c`)

**Status: DO ZROBIENIA**

---

## Zadanie 2 — Obsługa promowanej rolki @zdrowodosetki

**Status: CZĘŚCIOWO ZROBIONE**

✅ Zernio Comment Automation aktywna (ID: `6a018faae6e4a2a4ab077620`)
- Keyword "kurs" → publiczny komentarz + DM #1 (natychmiastowy)
- platformPostId: `17995268012923889`

⬜ DO ZROBIENIA: Sprawdzić czy n8n workflow dla zdrowodosetki też wysyła DM pod tą rolką — jeśli tak, wyłączyć węzeł DM w n8n (żeby nie było duplikatów), zostawić tylko follow-up

⬜ DO ZROBIENIA: Przetestować — napisać komentarz z "kurs" pod rolką i sprawdzić:
1. Czy publiczny komentarz się pojawia (może nie działać dla boosted przez Meta permissions)
2. Czy DM dochodzi

⬜ DO ZROBIENIA: Skonfigurować follow-up DM po 24h dla zdrowodosetki (w n8n lub Zernio Sequences)

---

## Zadanie 3 — Weryfikacja ścieżek payloadu (mieczuuuzaw)

- conversationId z odpowiedzi DM API: zakładamy `$json.id`
- conversationId w payloadzie `message.received`: zakładamy `body.conversation.id`
