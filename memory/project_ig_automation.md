---
name: IG Automation — Zernio + n8n
description: Projekt automatyzacji IG (@mieczuuuzaw + @zdrowodosetki): sekwencja KURS — komentarz → odpowiedź publiczna + DM → follow-up
type: project
originSessionId: e7293c7e-81ce-4509-8ad9-70de4299abcf
---
Budujemy automatyczny responder na komentarze i wiadomości DM na Instagramie, wzorowany na sekwencji z pliku "Beti KURS- ManyChat .docx".

**Why:** Zernio obsługuje konta IG przez MCP; n8n dostarcza logikę (AI, routing, warunki, Google Sheets).

**How to apply:** Przy zadaniach związanych z automatyzacją IG — używać mcp__zernio__comments_* i mcp__zernio__messages_* do odczytu/odpowiedzi, narzędzia n8n MCP do wyzwalania i edycji workflow.

---

## Konta w Zernio (Instagram)
- mieczuuuzaw — ID: `692de4f6f43160a0bc999b6c` | profil FLOWTIC (`692de2f974099cf535b6f2b0`)
- wolnicadom — ID: `69312de3f43160a0bc999ff0`
- ksefexpert — ID: `697a173977637c5c857c9c72`
- pieniadz_w_grze — ID: `69a6c9a9dc8cab9432b4b672`
- **zdrowodosetki — ID: `69fdc7df92b3d8e85f9b0352` | profil Beata Janicka (`69fb541c49e027dde464e7e6`)**

---

## Sekwencja KURS — treści z pliku Beti (źródło prawdy)

### Krok 1 — Publiczna odpowiedź pod komentarzem (trigger: „KURS")
Rotacja losowa 3 wariantów:
- „Wysłałam Ci szczegóły na priv 📩 sprawdź wiadomości"
- „Masz już link na priv 📩 koniecznie daj mi znać jak wrażenia!"
- „Wysłałam dostęp 📩 sprawdź wiadomości!"

### Krok 2 — DM #1 (natychmiast)
Hej! 😊 Widzę, że zainteresował Cię mój Kurs 28 dni Całkowitej Przemiany...
[Quick reply: „Chcę dostęp"]

### Krok 3 — DM #2 (po kliknięciu „Chcę dostęp")
Super, że chcesz działać! 🙌 ... link: https://zdrowodosetki.pl/kurs/28-dni-calkowitej-przemiany/

### Krok 4 — DM #3 Follow-up (po 24h, TYLKO jeśli NIE kliknął)
Hej! 😊 Zastanawiam się, czy zapoznałeś się już z ofertą...

---

## Identyfikatory zasobów

| Zasób | ID / URL |
|---|---|
| Konto IG mieczuuuzaw | `692de4f6f43160a0bc999b6c` |
| Profil mieczuuuzaw | `692de2f974099cf535b6f2b0` |
| Zernio Webhook `comment.received` → `/webhook/ig-kurs-comment` | `69f264764d1f6b562459ccff` |
| Zernio Webhook `message.received` → `/webhook/ig-kurs-message` | `69f2692c7be7588dc23bcd7d` |
| Google Sheet (tracking) | `1ySpE9eJq3bf1y2e6a3Q9Rq5YQkC0N6fr31itNFnM8EM` |

---

## Workflow n8n — stan na 2026-05-09

### Aktywne (produkcja)
| Workflow | ID | URL | Stan |
|---|---|---|---|
| IG KURS - Auto Responder DM v2 (#1) | `YQSu2NUpydOhWlPm` | https://automatyyyyka.app.n8n.cloud/workflow/YQSu2NUpydOhWlPm | **AKTYWNY** ✅ |
| IG KURS - Obsługa Chcę dostęp (Sheets) (#2) | `ZBk4CedqCGZqjCBn` | https://automatyyyyka.app.n8n.cloud/workflow/ZBk4CedqCGZqjCBn | **AKTYWNY** ✅ |

### Stare (wyłączone)
| Workflow | ID | Stan |
|---|---|---|
| IG KURS - Auto Responder DM | `tf8oGl8LtVdrMKhM` | **WYŁĄCZONY** ✅ |
| IG KURS - Obsługa "Chcę dostęp" | `tFFwDmtYDbIBJw5D` | **WYŁĄCZONY** ✅ |

---

## Co zostało zrobione (2026-05-09)
- ✅ Google Sheet — nagłówki wiersz 1: `username | conversation_id | comment_id | started_at | status`
- ✅ Credentials Zernio API — przypisane do HTTP Request nodów w obu workflow
- ✅ Workflow #1 nowy (`YQSu2NUpydOhWlPm`) — aktywny
- ✅ Workflow #2 nowy (`ZBk4CedqCGZqjCBn`) — aktywny
- ✅ Stare workflow wyłączone
- ✅ Webhooks Zernio działają (failureCount=0, comment.received odpalił 09-05 07:03)

## Następne kroki
1. Testy end-to-end (skomentować „KURS" pod postem i sprawdzić czy sekwencja przechodzi poprawnie)
2. Weryfikacja ścieżek: conversationId z odpowiedzi DM API (`$json.id`) i w payloadzie `message.received` (`body.conversation.id`)
3. Ręczna konfiguracja columns w Sheets nodach (autoMap + matching column: username)

## Zweryfikowana struktura payloadu `comment.received` (2026-05-04)
- `body.event` = `"comment.received"`
- `body.comment.id` = ID komentarza (platformowy)
- `body.comment.text` = treść komentarza
- `body.comment.author.username` = username autora (**NIE** `from.username`)
- `body.comment.postId` = Zernio post ID
- `body.post.id` = Zernio post ID (identyczne jak `comment.postId`)
- `body.account.id` = ID konta Zernio

## Zweryfikowany endpoint odpowiedzi na komentarz
- **Poprawny:** `POST /v1/inbox/comments/{platformPostId}` z body `{accountId, message, commentId: comment.id}`
- **Private reply (DM do komentującego):** `POST /v1/inbox/comments/{postId}/{commentId}/private-reply` z body `{accountId, message}`

## Promowane rolki — kluczowe różnice
- `post.id` = `null` dla promowanych/boosted reeli (Zernio nie ma tego posta w bazie)
- `post.platformPostId` = natywny IG media ID — zawsze dostępny
- **Filtr w n8n:** używaj `platformPostId` zamiast `post.id`
- **Odpowiedź publiczna na komentarz pod promowaną rolką** → zwraca 400 przez API (brak uprawnień Meta dla boosted content przez standardowy token)
- **DM/private-reply** działa normalnie nawet dla promowanych rolek
- Webhook dla promowanej treści zawiera `comment.ad` z `ad.id` i `ad.title` (Instagram) lub `ad.promotionStatus` (Facebook)

## Zernio Comment Automation — zdrowodosetki (2026-05-11)
Rozwiązanie dla promowanej rolki @zdrowodosetki — zamiast HTTP Request przez n8n.

| Pole | Wartość |
|---|---|
| Automation ID | `6a018faae6e4a2a4ab077620` |
| Nazwa | Kurs - promowana rolka zdrowodosetki |
| Konto | zdrowodosetki (`69fdc7df92b3d8e85f9b0352`) |
| platformPostId | `17995268012923889` |
| Keyword | `kurs` (contains, case-insensitive) |
| comment_reply | "Wysłałam Ci szczegóły na priv 📩 sprawdź wiadomości" |
| dm_message | "Hej! 😊\nWidzę, że zainteresował Cię mój Kurs 28 dni Całkowitej Przemiany. Bardzo mi miło :)\nPrzygotowałam dla Ciebie dostęp 👇" |
| Status | **AKTYWNA** ✅ |

**Uwaga:** Jeśli n8n workflow dla zdrowodosetki też wysyła DM na słowo "kurs" pod tą rolką — osoba dostanie dwa DM-y. Węzeł DM w n8n dla tej rolki powinien być wyłączony lub przerobiony na follow-up.

## n8n — case-insensitive filtr słowa kluczowego
Zamiast `{{ $json.body.comment.text }} contains "KURS"` użyj:
- Lewe pole: `{{ $json.body.comment.text.toLowerCase() }}`
- Operator: `contains`
- Prawe pole: `kurs`

## Nadal do weryfikacji w testach
- Ścieżka conversationId z odpowiedzi DM API: zakładamy `$json.id`
- Ścieżka conversationId w payloadzie `message.received`: zakładamy `body.conversation.id`

---

## Konfiguracja MCP
- Zernio MCP: `https://mcp.zernio.com/mcp` (Bearer token w `.mcp.json`)
- n8n MCP: `https://automatyyyyka.app.n8n.cloud/mcp-server/http`
