# Odzyskiwanie dostępu SSH do Raspberry Pi — raport

## Cel
Odzyskanie dostępu SSH do Raspberry Pi Zero, gdy zapomniano hasła użytkownika.

---

## Kroki które wykonaliśmy

### 1. Instalacja WSL
- WSL nie był zainstalowany
- Zainstalowano przez PowerShell: `wsl --install`
- Utworzono użytkownika WSL (`soswi`)

### 2. Podłączenie karty SD do WSL (usbipd)
- WSL2 domyślnie **nie widzi dysków USB** — karta SD była niewidoczna
- Zainstalowano narzędzie **usbipd-win**
- Zidentyfikowano kartę SD: `1-1 Urządzenie pamięci masowej USB`
- Przekazano kartę do WSL:
  ```
  usbipd bind --busid 1-1
  usbipd attach --wsl --busid 1-1
  ```
- Karta pojawiła się jako `/dev/sde` z partycjami `sde1` (boot) i `sde2` (rootfs)

### 3. Zamontowanie rootfs i usunięcie hasła
- Zamontowano partycję z systemem: `sudo mount /dev/sde2 /mnt/rpi`
- Zidentyfikowano użytkownika: `user` (brak użytkownika `pi`)
- Usunięto hash hasła w `/etc/shadow`:
  ```
  # przed:
  user:$5$KrkL...$QtKK...:20416:0:99999:7:::
  # po:
  user::20416:0:99999:7:::
  ```

### 4. Włączenie SSH na partycji boot
- Plik `ssh` nie istniał na partycji boot — SSH był wyłączony
- Utworzono plik: `sudo touch /mnt/boot/ssh`
- Uwaga: Raspberry Pi OS **automatycznie usuwa plik `ssh`** po pierwszym uruchomieniu
- SSH był już włączony przez systemd (`ssh.service` istniał w `multi-user.target.wants`)

---

## Napotkane problemy

| Problem | Przyczyna | Rozwiązanie |
|---|---|---|
| WSL nie widzi karty SD | WSL2 nie obsługuje USB natywnie | Instalacja `usbipd-win` |
| Logowanie nadal wymaga hasła | `UsePAM yes` blokuje puste hasła przez SSH | Ustawienie nowego hasła zamiast pustego |
| `chroot` nie działa | Pi Zero to ARM, WSL to x86_64 | Generowanie hasła przez `openssl` |
| `python3 -c "import crypt"` nie działa | Moduł `crypt` usunięty w Python 3.13 | Użycie `openssl passwd -6` |

---

## Ostateczne rozwiązanie

Ponieważ puste hasło było blokowane przez SSH/PAM, wygenerowano nowy hash i wpisano go ręcznie do `/etc/shadow`:

```bash
# Generowanie hasha nowego hasła
openssl passwd -6 nowehaslo

# Wklejenie hasha do /etc/shadow
user:$6$...:20416:0:99999:7:::
```

---

## Wnioski — bezpieczeństwo

> **Posiadając fizyczny dostęp do karty SD lub dysku urządzenia, można przejąć pełną kontrolę nad systemem — bez znajomości jakiegokolwiek hasła.**

- Hasła w systemie Linux są przechowywane jako **hashe w pliku `/etc/shadow`**
- Mając dostęp do nośnika można ten hash **usunąć lub zastąpić własnym**
- SSH można włączyć przez **zwykły plik na partycji FAT32** widocznej na każdym systemie
- **Szyfrowanie dysku** (np. LUKS) skutecznie chroni przed takim atakiem — bez klucza dane są nieczytelne
- Raspberry Pi domyślnie **nie szyfruje karty SD** — jest podatne na ten rodzaj ataku

### Jak się zabezpieczyć?
- Włączyć szyfrowanie nośnika (LUKS)
- Fizycznie zabezpieczyć urządzenie przed dostępem osób trzecich
- Wyłączyć logowanie SSH hasłem — używać wyłącznie kluczy SSH
