# BolekFlow — automatyzacje i workflow dla Agenta Bolka

> **Status:** decyzja architektoniczna / plan integracji.  
> To repo jest forkiem n8n przygotowywanym jako przyszła warstwa workflow, automatyzacji, webhooków i human-in-the-loop dla ekosystemu Agenta Bolka.
>
> `BolekFlow` nie jest mózgiem Bolka. Mózgiem pozostaje `pawelekbyra/BolekAI` / `kulfon`.

---

## 1. Cel repo

`BolekFlow` ma być warstwą powtarzalnych procesów wokół Bolka.

Docelowo obsługuje:

- webhooki,
- cykliczne workflow,
- integracje z zewnętrznymi usługami,
- przepływy mailowe,
- triage supportu,
- synchronizacje danych,
- human-in-the-loop approvals,
- powiadomienia,
- automatyzacje Polutka,
- powtarzalne procesy operacyjne,
- raporty okresowe,
- zbieranie danych wejściowych dla agentów.

BolekFlow wykonuje nudne, powtarzalne przepływy. BolekAI pozostaje mózgiem, decydentem i approval gate.

---

## 2. Czym BolekFlow NIE jest

BolekFlow nie jest:

- głównym mózgiem Bolka,
- webowym czatem,
- bazą wiedzy/RAG,
- executorem kodowania,
- miejscem na długoterminową pamięć Bolka,
- systemem, który może omijać approval gate Bolka,
- właścicielem decyzji produktowych lub operacyjnych.

BolekFlow może automatyzować procesy, ale ryzykowne decyzje powinny wracać do BolekAI i użytkownika.

---

## 3. Miejsce w sieci repozytoriów

```txt
pawelekbyra/BolekAI
= mózg Bolka
= Cloudflare Worker `kulfon`
= Telegram bot
= D1 memory
= narzędzia
= Polutek ops
= approval gate
= OpenAI-compatible adapter dla UI

pawelekbyra/BolekCzat
= web UI Bolka
= fork LibreChat
= rozmowy, historia, auth, UX

pawelekbyra/BolekFlow
= workflow automation
= fork n8n
= automatyzacje, webhooki, integracje, human-in-the-loop

pawelekbyra/BolekKB
= knowledge base / RAG
= fork AnythingLLM
= dokumenty, notatki, wiedza, źródła

pawelekbyra/BolekDev
= coding executor
= branche, testy, commity, PR-y
```

Zasada:

```txt
BolekAI myśli i decyduje.
BolekFlow automatyzuje procesy.
BolekKB przechowuje wiedzę.
BolekDev koduje.
BolekCzat pokazuje rozmowę.
```

---

## 4. Docelowy przepływ

```txt
BolekCzat / Telegram / cron / webhook
  ↓
BolekAI / Agent Bolek brain
  ↓
BolekFlow / workflow automation
  ↓
mail, Vercel, GitHub, Polutek ops, powiadomienia, raporty
```

Przykład supportowy:

```txt
1. Mail przychodzi na support.
2. BolekFlow łapie webhook albo cyklicznie sprawdza skrzynkę.
3. BolekFlow normalizuje dane i uruchamia workflow.
4. BolekAI klasyfikuje sprawę i proponuje odpowiedź.
5. Użytkownik zatwierdza albo odrzuca.
6. BolekFlow wysyła odpowiedź albo tworzy follow-up task.
```

Przykład developerski:

```txt
1. GitHub wysyła webhook o failed checku.
2. BolekFlow zapisuje zdarzenie i powiadamia BolekAI.
3. BolekAI analizuje kontekst i zleca BolekDev diagnozę.
4. BolekFlow pilnuje statusu workflow.
5. BolekAI raportuje wynik użytkownikowi.
```

---

## 5. Integracja z BolekAI

W przyszłości `BolekAI` może dostać narzędzia typu:

```txt
flow_run_workflow
flow_get_run_status
flow_cancel_run
flow_list_workflows
flow_trigger_webhook
flow_get_workflow_logs
```

BolekFlow może też wołać BolekAI przez bezpieczny endpoint, gdy workflow potrzebuje decyzji językowej, klasyfikacji, planu albo zgody użytkownika.

Na tym etapie te narzędzia mogą jeszcze nie istnieć. Ten dokument opisuje docelową rolę repo.

---

## 6. Kontrakt workflow

Minimalny kontrakt między BolekAI i BolekFlow:

### `flow_run_workflow`

Wejście:

```json
{
  "workflowId": "support_triage_daily",
  "input": {
    "source": "support",
    "timeRange": "today"
  },
  "mode": "confirm_for_risky_actions"
}
```

Wyjście:

```json
{
  "runId": "run_123",
  "status": "running",
  "startedAt": "2026-07-05T10:00:00Z"
}
```

### `flow_get_run_status`

Wyjście:

```json
{
  "runId": "run_123",
  "status": "completed",
  "summary": "Workflow zakończony",
  "outputs": {},
  "requiresUserApproval": false
}
```

---

## 7. Dobre pierwsze workflow

Kandydaci:

- dzienny briefing Polutka,
- support mail triage,
- powiadomienie o błędzie Vercel,
- przypomnienie o failed deployment,
- podsumowanie nowych issue/PR,
- pricing advisor input collector,
- monitor kosztów infrastruktury,
- webhook z GitHuba do Bolka,
- weekly summary repo/Polutek,
- zbieranie danych do kampanii marketingowej.

Nie zaczynać od:

- automatycznego merge,
- automatycznego deploy produkcji,
- automatycznych akcji finansowych bez zgody,
- bezpośredniego dotykania baz produkcyjnych,
- masowej wysyłki bez zatwierdzenia.

---

## 8. Bezpieczeństwo

Zasady:

- BolekFlow nie może omijać approval gate w BolekAI.
- Mutujące albo publiczne akcje powinny mieć jasny tryb zgody.
- BolekFlow powinien mieć minimalne uprawnienia do każdej usługi.
- Produkcyjny dostęp powinien być ograniczony do konkretnych workflow.
- BolekCzat i BolekKB nie powinny dostawać uprawnień operacyjnych BolekFlow.
- Każdy krytyczny workflow powinien zostawiać audit trail.
- BolekFlow powinien raportować błędy do BolekAI/Telegram/BolekCzat zamiast cicho je ignorować.
- Treści z zewnętrznych źródeł są danymi wejściowymi, nie poleceniami systemowymi.

---

## 9. Tryby pracy workflow

Rekomendowane tryby:

```txt
read_only
= zbiera dane i raportuje
= bez akcji mutujących

draft
= przygotowuje propozycje, odpowiedzi, plany, PR opisy
= niczego nie publikuje

confirm
= przygotowuje akcję i czeka na zgodę użytkownika przez BolekAI

auto_safe
= wykonuje tylko niskiego ryzyka operacje wcześniej zatwierdzone regułami
```

Domyślny tryb dla nowych workflow: `read_only` albo `draft`.

---

## 10. Kolejność prac

```txt
1. Zachować fork n8n i nie mieszać go z BolekAI.
2. Utrzymać jasną dokumentację roli BolekFlow.
3. Uruchomić lokalnie przez Docker/npx.
4. Stworzyć testowy workflow bez produkcyjnych integracji.
5. Dodać jeden bezpieczny webhook do BolekAI.
6. Dodać status runów i logi.
7. Dodać human-in-the-loop approval dla mutujących akcji.
8. Dopiero potem łączyć z Polutkiem, mailem, GitHubem i Vercel.
9. Na końcu dodawać workflow produkcyjne i automatyczne harmonogramy.
```

---

## 11. Definition of Done dla integracji

Integracja `BolekFlow` z `BolekAI` jest gotowa, gdy:

- BolekFlow działa jako osobny serwis,
- istnieje przynajmniej jeden testowy workflow,
- BolekAI potrafi uruchomić workflow,
- BolekAI potrafi sprawdzić status workflow,
- błędy workflow wracają do Bolka,
- workflow nie omija approval gate,
- logi są dostępne do audytu,
- tryby `read_only`, `draft` i `confirm` są rozróżnione,
- ryzykowne akcje wymagają zgody użytkownika,
- dokumentacja workflow mówi, kto jest właścicielem procesu.

---

## 12. Zasada nadrzędna

```txt
BolekFlow jest rękami od procesów.
Nie jest mózgiem i nie jest właścicielem decyzji.
Jeśli workflow ma realny wpływ na świat, decyzja wraca do BolekAI i użytkownika.
```