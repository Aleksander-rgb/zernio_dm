# Analiza: Multi-step IG Responder — mieczuuuzaw

Data: 2026-04-29 (aktualizacja planu implementacji)

---

## Cel

Zbudować automatyczny, wieloetapowy responder na komentarze i wiadomości DM na Instagramie (@mieczuuuzaw, ID: `692de4f6f43160a0bc999b6c`), wzorowany na sekwencji ManyChat z pliku "Beti KURS- ManyChat .docx".

---

## Treści z pliku (źródło prawdy)

### Krok 1 — Publiczna odpowiedź pod komentarzem (trigger: „KURS")
Rotacja losowa 3 wariantów:
- „Wysłałam Ci szczegóły na priv 📩 sprawdź wiadomości"
- „Masz już link na priv 📩 koniecznie daj mi znać jak wrażenia!"
- „Wysłałam dostęp 📩 sprawdź wiadomości!"

### Krok 2 — DM #1 (natychmiast po komentarzu)
```
Hej! 😊
Widzę, że zainteresował Cię mój Kurs 28 dni Całkowitej Przemiany. Bardzo mi miło :)
Przygotowałam dla Ciebie dostęp 👇
[Quick reply: „Chcę dostęp"]
```

### Krok 3 — DM #2 (po kliknięciu „Chcę dostęp")
```
Super, że chcesz działać! 🙌
Ten kurs pokaże Ci krok po kroku w jaki sposób:
✔️ przyspieszyć metabolizm
✔️ spalić tłuszcz bez katowania się na siłowni
✔️ lepiej zarządzać energią i snem
Kliknij poniżej i sprawdź szczegóły:
➡️ https://zdrowodosetki.pl/kurs/28-dni-calkowitej-przemiany/
```

### Krok 4 — DM #3 Follow-up (po 24h, tylko jeśli NIE kliknął „Chcę dostęp")
```
Hej! 😊
Zastanawiam się, czy zapoznałeś się już z ofertą kursu, który Ci podesłałam?
Wiele osób odkłada to na później… a potem dalej stoi w tym samym miejscu.
Nie popełnij tego błędu i kliknij w link 👇
➡️ https://zdrowodosetki.pl/kurs/28-dni-calkowitej-przemiany/
```

---

## Uwaga: Przyciski — ManyChat vs nasze podejście

ManyChat jest oficjalnym Meta Business Partnerem i używa **postback buttons** — klik nie wysyła widocznej wiadomości, a cichy event do serwera ManyChat.

My używamy **quick replies** (publiczne Meta API) — gdy użytkownik tapnie „Chcę dostęp", wysyłana jest wiadomość o tej treści do inbox. Efekt końcowy identyczny. Jedyna różnica: widoczna wiadomość „Chcę dostęp" w konwersacji po stronie użytkownika.

---

## Architektura docelowa

```
Instagram: ktoś komentuje „KURS" na poście
        ↓
[Zernio Webhook: comment.received] → n8n Workflow #1
        ↓
[n8n] Filtr: czy treść zawiera „KURS"? (IF node)
  → NIE: stop
  → TAK:
        ↓
[n8n] Code node: losuj 1 z 3 tekstów publicznej odpowiedzi
        ↓
[n8n] HTTP Request → Zernio API: wyślij publiczną odpowiedź pod komentarzem
        ↓
[n8n] HTTP Request → Zernio API: wyślij DM #1 z quick reply „Chcę dostęp"
        ↓
[n8n] HTTP Request → Zernio API: utwórz konwersację i pobierz conversationId
        ↓
[n8n] Data Table: zapisz { username, conversationId, commentId, timestamp, status: "waiting_click" }
        ↓
[n8n] Wait 24h
        ↓
[n8n] HTTP Request → Zernio API: pobierz wpis z Data Table dla username
  → status == "clicked": stop (nie wysyłaj follow-up)
  → status == "waiting_click": wyślij DM #3 Follow-up


Instagram: użytkownik tapie „Chcę dostęp" w DM
        ↓
[Zernio Webhook: message.received] → n8n Workflow #2
        ↓
[n8n] Filtr: czy treść wiadomości == „Chcę dostęp"?
  → NIE: stop
  → TAK:
        ↓
[n8n] HTTP Request → Zernio API: wyślij DM #2 (szczegóły kursu + link)
        ↓
[n8n] Data Table: zaktualizuj status username na "clicked"
```

---

## Stan istniejący (co już zbudowano)

| Komponent | Stan | Uwagi |
|---|---|---|
| Zernio Comment Automation „KURS - Auto Responder" | ✅ aktywna | Wysyła 1 stały tekst publiczny + DM #1 bez quick reply — DO USUNIĘCIA |
| Zernio Webhook `comment.received` → n8n | ✅ aktywny | ID: `69f264764d1f6b562459ccff` |
| n8n Workflow #1 „IG KURS - Auto Responder DM" | ⚠️ częściowy | Ma filtr KURS + Wait 24h + DM #3, brakuje rotacji, odpowiedzi publicznej, DM #1 z quick reply, Data Table |
| Zernio Webhook `message.received` → n8n | ❌ brak | Potrzebny do obsługi kliknięcia „Chcę dostęp" |
| n8n Workflow #2 (obsługa kliknięcia) | ❌ brak | Cały do zbudowania |
| n8n Data Table (śledzenie statusu) | ❌ brak | Potrzebny do warunkowego follow-up |

---

## Szczegółowy plan implementacji

### Etap 1 — Sprzątanie (co usunąć)

**1a. Usuń Zernio Comment Automation** (`69f25dfb769a9295a6d23a7d`)
- Powód: wysyła DM #1 bez przycisku i stały tekst publiczny; całą tę logikę przejmuje n8n

**1b. Dodaj `message.received` do istniejącego webhooka Zernio**
- Webhook ID: `69f264764d1f6b562459ccff`
- Dodaj event `message.received` obok `comment.received`
- Oba trafiają na ten sam URL n8n → n8n rozróżnia po `event` w payloadzie

### Etap 2 — n8n Data Table

Utwórz tabelę `ig_kurs_contacts` z kolumnami:
- `username` (string) — Instagram username komentującego
- `conversation_id` (string) — ID konwersacji DM w Zernio
- `comment_id` (string) — ID komentarza
- `started_at` (string) — ISO timestamp startu sekwencji
- `status` (string) — `waiting_click` | `clicked` | `followup_sent`

### Etap 3 — Zernio API: endpointy do użycia w HTTP Request

**a) Publiczna odpowiedź pod komentarzem:**
```
POST https://zernio.com/api/v1/inbox/comments/{commentId}/reply
Body: { "accountId": "692de4f6f43160a0bc999b6c", "message": "<losowy tekst>" }
```
*Uwaga: sprawdzić dokładny endpoint przez docs_search przed implementacją*

**b) Wyślij DM #1 z quick reply:**
```
POST https://zernio.com/api/v1/inbox/conversations
Body: {
  "accountId": "692de4f6f43160a0bc999b6c",
  "participantUsername": "<username>",
  "message": "Hej! ...",
  "quickReplies": [{ "title": "Chcę dostęp", "payload": "chce_dostep" }]
}
```
*Uwaga: zweryfikować format quickReplies przez docs_search przed implementacją*

**c) Wyślij DM #2 (po kliknięciu):**
```
POST https://zernio.com/api/v1/inbox/conversations/{conversationId}/messages
Body: {
  "accountId": "692de4f6f43160a0bc999b6c",
  "message": "Super, że chcesz działać! ..."
}
```

**d) Wyślij DM #3 (follow-up):**
```
POST https://zernio.com/api/v1/inbox/conversations/{conversationId}/messages
Body: {
  "accountId": "692de4f6f43160a0bc999b6c",
  "message": "Hej! Zastanawiam się..."
}
```

### Etap 4 — Przebudowa n8n Workflow #1 (comment.received)

Nowa struktura nodów:
```
Webhook trigger (POST /ig-kurs-comment)
  ↓
IF: event == "comment.received"
  → NIE: stop
  → TAK:
      ↓
    IF: body.comment.text zawiera "KURS" (case insensitive)
      → NIE: stop
      → TAK:
          ↓
        Code node: losuj tekst (Math.random() * 3 → index 0/1/2)
          ↓
        HTTP Request: wyślij publiczną odpowiedź pod komentarzem
          ↓
        HTTP Request: wyślij DM #1 z quick reply „Chcę dostęp"
          ↓
        Data Table: INSERT { username, conversation_id, started_at, status: "waiting_click" }
          ↓
        Wait 24h
          ↓
        Data Table: GET status dla username
          ↓
        IF: status == "clicked"
          → TAK: stop
          → NIE: HTTP Request → wyślij DM #3 Follow-up
                  ↓
                Data Table: UPDATE status = "followup_sent"
```

### Etap 5 — Nowy n8n Workflow #2 (message.received / kliknięcie „Chcę dostęp")

Nowa struktura nodów:
```
Webhook trigger (POST /ig-kurs-message)  ← osobny URL
  ↓
IF: event == "message.received"
  → NIE: stop
  → TAK:
      ↓
    IF: body.message.text == "Chcę dostęp"
      → NIE: stop
      → TAK:
          ↓
        HTTP Request: wyślij DM #2 (szczegóły kursu + link)
          ↓
        Data Table: UPDATE status = "clicked" WHERE username = sender
```

*Uwaga: Webhook #2 może być osobnym workflow z własnym URL-em, lub ten sam URL co Workflow #1 z rozgałęzieniem po `event`.*

### Etap 6 — Aktualizacja Zernio webhooka

Dodaj `message.received` do webhooka `69f264764d1f6b562459ccff` (przez curl, bo MCP tool ma buga).

Jeśli Workflow #2 ma osobny URL:
```
POST https://zernio.com/api/v1/webhooks/settings
{ "name": "IG Message → n8n KURS", "url": "https://automatyyyyka.app.n8n.cloud/webhook/ig-kurs-message", "events": ["message.received"] }
```

Jeśli jeden URL — zaktualizuj istniejący webhook żeby obsługiwał oba eventy.

---

## Kolejność wykonania

1. Usuń Comment Automation (Etap 1a)
2. Utwórz Data Table w n8n (Etap 2)
3. Sprawdź przez docs_search formaty endpointów Zernio (Etap 3) — przed pisaniem kodu
4. Przebuduj Workflow #1 (Etap 4)
5. Zbuduj Workflow #2 (Etap 5)
6. Zaktualizuj webhook Zernio (Etap 6)
7. Testy end-to-end

---

## Identyfikatory (do użycia podczas implementacji)

| Zasób | ID |
|---|---|
| Konto IG mieczuuuzaw | `692de4f6f43160a0bc999b6c` |
| Profil mieczuuuzaw | `692de2f974099cf535b6f2b0` |
| Post testowy (dziś) | platforma: `18263158507293629`, Zernio: `69f23fc05e4acd2eb19f3d79` |
| Comment Automation (do usunięcia) | `69f25dfb769a9295a6d23a7d` |
| Webhook Zernio (comment.received) | `69f264764d1f6b562459ccff` |
| n8n Workflow #1 | `tf8oGl8LtVdrMKhM` |
| Zernio API base URL | `https://zernio.com/api/v1` |
| Bearer token Zernio | w `.mcp.json` i n8n credentials |
