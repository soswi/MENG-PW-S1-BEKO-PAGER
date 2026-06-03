# Uruchomienie projektu BEKO — raport

## Stos technologiczny
- **Frontend**: React + Vite + TypeScript + Tailwind + shadcn/ui
- **Backend gateway**: FastAPI + uvicorn (port 8000) + LoRa SX1276
- **Backend auth**: FastAPI + uvicorn (port 8001)
- **Urządzenie**: Raspberry Pi Zero (ARM, Debian Bookworm)
- **Sieć**: iPhone hotspot (`BEKO prezentacja`)

---

## Kolejność uruchamiania

### 1. Raspberry Pi — backend gateway
```bash
cd ~/backend_2/BEKO/gateway
source venv/bin/activate
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### 2. Raspberry Pi — backend auth (drugie okno SSH)
```bash
cd ~/frontend/BEKO/backend
source .venv/bin/activate
python -m uvicorn main:app --host 0.0.0.0 --port 8001
```

### 3. Komputer — frontend
```bash
cd MENG-PW-S1-BEKO-PAGER
npm install
npm run dev -- --host 0.0.0.0
```

### 4. W przeglądarce
- Otworzyć `http://localhost:5173`
- W panelu logowania zmienić IP na `10.218.118.11`
- Zalogować się: operator `0000` / administrator `9999`

---

## Napotkane problemy i rozwiązania

### Problem 1 — brak virtualenv dla backendu auth
Backend auth (`~/frontend/BEKO/backend`) nie miał folderu `venv`.

**Rozwiązanie:**
```bash
python3 -m venv .venv
python3 -m pip install -r requirements.txt
```
Uwaga: `pip` w nowym venv był uszkodzony — trzeba było użyć `python3 -m pip` zamiast `pip`.

### Problem 2 — brakujący plik `LogsPage.tsx`
Vite zgłaszał błąd przy starcie:
```
Failed to resolve import "./features/logs/LogsPage" from "src/App.tsx"
```
Plik `src/features/logs/LogsPage.tsx` nie istniał w repozytorium, mimo że `App.tsx` go importował.

**Rozwiązanie:**  
Stworzono brakujący plik ręcznie na podstawie:
- hooka `useLogs.ts`
- typów z `api.ts` (`LogEntry`, `LogsSummary`, `LogLevel`)
- API klienta `logs.ts`
- stylu reszty projektu (Tailwind + shadcn/ui)

Plik zapisano do `src/features/logs/LogsPage.tsx`.

---

### Problem 3 — CORS blokuje zapytania z frontendu
Frontend działał na `http://10.218.118.235:5173` (komputer), a backend auth na `http://10.218.118.11:8001` (Pi). Przeglądarka blokowała zapytania z powodu braku nagłówka `Access-Control-Allow-Origin`.

Błąd w konsoli:
```
Access to fetch at 'http://10.218.118.11:8001/auth/login' from origin
'http://10.218.118.235:5173' has been blocked by CORS policy
```

**Przyczyna:**  
W `config.py` lista `CORS_ORIGINS` zawierała tylko stare adresy IP (`10.69.72.11`) z poprzedniego środowiska. Aktualny adres komputera (`10.218.118.235`) nie był na liście.

**Rozwiązanie:**  
Zmieniono `CORS_ORIGINS` w `~/frontend/BEKO/backend/config.py` na:
```python
CORS_ORIGINS: list[str] = ["*"]
```
Następnie zrestartowano backend auth. `"*"` musi być jedynym elementem listy — Starlette/FastAPI nie obsługuje mieszania konkretnych adresów z `"*"`.

---

### Problem 4 — nieprawidłowe hasło logowania
Po naprawieniu CORS logowanie zwracało `401 Unauthorized`. Instrukcja podawała hasła `0000` i `9999`, ale baza zawierała innych użytkowników.

**Diagnoza:**
```bash
python3 -c "from database import SessionLocal; from models import User; \
db=SessionLocal(); print([(u.username, u.role) for u in db.query(User).all()])"
# Wynik: [('admin', 'admin'), ('user1', 'user'), ('kuchnia', 'user')]
```

Sprawdzono `seed.py` — domyślne hasło to `admin123`.

**Rozwiązanie:**  
Zalogowano się jako `admin` / `admin123`.

---

## Wnioski

- Repozytorium na branchu `frontend` było niekompletne — brakowało jednego pliku strony
- Backendy działają niezależnie na osobnych portach i muszą być uruchomione przed frontendem
- IP Raspberry Pi (`10.218.118.11`) jest konfigurowane z poziomu UI — nie wymaga zmian w kodzie
- Virtualenv trzeba tworzyć ręcznie jeśli nie jest zapisany w repozytorium (`.gitignore`)
- CORS musi być skonfigurowany pod aktualne środowisko sieciowe — przy zmianie sieci (inny hotspot = inne IP) trzeba zaktualizować listę origins lub użyć `"*"` na czas developmentu
- Hasła użytkowników nie były zgodne z instrukcją — zawsze warto sprawdzić `seed.py` jako źródło prawdy o domyślnych danych
