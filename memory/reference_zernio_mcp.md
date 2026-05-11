---
name: Zernio MCP Reference
description: Konfiguracja i możliwości serwera MCP dla Zernio API — 280+ toolsów, obsługiwane platformy, kluczowe narzędzia
type: reference
originSessionId: 814f2f67-ac09-4c6a-a290-8e54c9e65a82
---
# Zernio MCP Server

**Hosted URL:** `https://mcp.zernio.com/mcp`
**API Base URL:** `https://zernio.com/api/v1`
**Docs:** https://docs.zernio.com
**API Keys:** https://zernio.com/dashboard/api-keys

## Konfiguracja (hosted, zalecana)

```json
{
  "mcpServers": {
    "zernio": {
      "url": "https://mcp.zernio.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

Dla Claude Code CLI (Windows): plik `%APPDATA%\Claude\claude_desktop_config.json`

## Obsługiwane platformy (14)

twitter, instagram, linkedin, tiktok, bluesky, facebook, youtube, pinterest, threads, googlebusiness, telegram, snapchat

## Kluczowe narzędzia (core tools)

| Tool | Opis |
|------|------|
| `accounts_list` | Lista podłączonych kont |
| `accounts_get` | Szczegóły konta dla platformy |
| `profiles_list/get/create/update/delete` | Zarządzanie profilami |
| `posts_list/get/create/update/delete` | Zarządzanie postami |
| `posts_publish_now` | Natychmiastowa publikacja |
| `posts_cross_post` | Publikacja na wiele platform jednocześnie |
| `posts_retry/list_failed/retry_all_failed` | Obsługa błędów publikacji |
| `media_generate_upload_link` | Generowanie linku do uploadu mediów |
| `media_check_upload_status` | Sprawdzenie statusu uploadu |
| `docs_search` | Przeszukiwanie dokumentacji API |

## Parametry posts_create

- `content` (wymagane) — treść posta
- `platform` (wymagane) — docelowa platforma
- `is_draft` — zapisz jako draft (nie publikuj)
- `publish_now` — opublikuj natychmiast
- `schedule_minutes` — minuty od teraz (domyślnie 60, tylko przy is_draft=false i publish_now=false)
- `media_urls` — oddzielone przecinkami URL-e mediów
- `title` — tytuł (wymagane dla YouTube, zalecane dla Pinterest)

## Kategorie auto-generowanych toolsów

- **Ads** — kampanie reklamowe, boostowanie postów, analityka reklam
- **WhatsApp** — broadcast, bulk send, szablony
- **Inbox** — konwersacje, wiadomości, recenzje
- **Contacts** — lista, tworzenie, pola
- **Sequences** — tworzenie sekwencji, enroll kontaktów
- **Comment Automations** — automatyzacje komentarzy
- **Broadcasts** — tworzenie i wysyłanie broadcastów
- **Analytics** — analityka, best time to post, demografika
- **Connect** — OAuth, Bluesky credentials
- **Webhooks** — ustawienia, logi, testy

## Upload mediów (flow)

1. `media_generate_upload_link` → otrzymaj URL
2. Użytkownik otwiera URL w przeglądarce i uploaduje plik
3. Użytkownik mówi "done"
4. `media_check_upload_status(token)` → pobierz URL pliku
5. Użyj URL w `posts_create(media_urls=...)`

Obsługiwane formaty: JPG, PNG, WebP, GIF, MP4, MOV, WebM, PDF. Maks. 5GB.

## Lokalna instalacja (alternatywna)

```bash
# uvx (zalecane lokalnie)
uvx --from zernio-sdk[mcp] zernio-mcp

# pip
pip install zernio-sdk[mcp]
python -m zernio.mcp
```

Env var: `ZERNIO_API_KEY=your_key`
