# Dokument wymagań produktu (PRD) - Book2Cook
## 1. Przegląd produktu
Book2Cook (MVP) to webowa aplikacja umożliwiająca użytkownikom konwersję e-booków z przepisami (PDF z warstwą tekstową) na ustrukturyzowane, edytowalne przepisy, tworzenie tygodniowych planów posiłków i generowanie z nich list zakupów. MVP skupia się na core flow: import PDF → automatyczna ekstrakcja przepisów (do JSON) → inline edycja → plan tygodniowy → lista zakupów. Aplikacja będzie dostępna w języku polskim i oferuje prosty system kont użytkowników.

Kluczowe założenia techniczne i operacyjne:
- Obsługiwane jedynie PDF-y z warstwą tekstową (bez OCR).  
- Asynchroniczny pipeline przetwarzania z kolejką, retry i powiadomieniami w UI.  
- Target latencji: < 2 min dla pliku ~20–30 MB.  
- Limit pliku ~100 MB; limit przechowywania per user 500 MB; retencja oryginału 6 miesięcy.  
- Brak eksportu w MVP; powiadomienia tylko w UI.  
- TOS wymaga oświadczenia użytkownika o posiadaniu praw do przesyłanego e-booka.

## 2. Problem użytkownika
Użytkownicy posiadają e-booki z przepisami (często w formacie PDF), które trudno przeszukiwać, edytować i przekształcać w praktyczne narzędzia do planowania posiłków. Trudności obejmują: wolne wyszukiwanie, brak struktury przepisów (składniki, kroki), brak automatycznego tworzenia list zakupów oraz brak możliwości szybkiego tworzenia tygodniowych planów. Book2Cook eliminuje te bolączki, automatyzując ekstrakcję przepisów i dając narzędzia do ich edycji i planowania.

## 3. Wymagania funkcjonalne
3.1 Import i analiza e-booka
- Użytkownik może wgrać plik PDF (formaty: PDF z warstwą tekstową).  
- System odrzuca pliki >100 MB z komunikatem wyjaśniającym.  
- Upload inicjuje asynchroniczne zadanie w kolejce.  
- Po przyjęciu pliku system parsuje PDF lokalnie na ustrukturyzowany JSON zawierający co najmniej: tytuł przepisu, listę składników (wraz z rozpoznanymi ilościami i jednostkami, jeśli dostępne), kroki przygotowania, makroskładniki (białko, węglowodany, tłuszcze) oraz metadane strony i fragmentów tekstu.  
- Wyodrębniony JSON jest zapisywany w bazie danych powiązanej z kontem użytkownika; oryginalny PDF jest przechowywany zgodnie z polityką retencji (retencja 6 miesięcy).  
- Dalsze przetwarzanie i klasyfikacja (np. automatyczna kategoryzacja, wzbogacanie metadanych) odbywa się na poziomie JSON — do modelu LLM wysyłany jest wyłącznie wygenerowany JSON (nie cały plik PDF), aby zmniejszyć transfer danych i przyspieszyć pipeline.  
- Każde wyodrębnione pole otrzymuje metrykę pewności (confidence score).  
- Wyniki przetwarzania zapisywane są powiązane z kontem użytkownika wraz z odwołaniem do oryginalnego PDF.

3.2 Zarządzanie przepisami
- Wyświetlanie listy przepisów z informacją o pewności ekstrakcji i linkiem do edycji.  
- Inline edycja pól: tytuł, makroskładniki, składniki, kroki (z możliwością zapisu zmian).  
- Możliwość kategoryzacji przepisu (np. śniadanie, obiad, deser).  
- Oznaczanie przepisu jako "niska pewność" gdy confidence < ustalonego progu.
 - Automatyczna kategoryzacja przepisów przez AI (kategorie predefiniowane + możliwość ręcznej korekty).  
 - Automatyczna kategoryzacja składników (np. nabiał, warzywa, mięso) przez AI z confidence score.

3.3 Tworzenie planu posiłków
- Interfejs do tworzenia tygodniowego planu posiłków (dni tygodnia × posiłki).  
- Dodawanie/usuwanie przepisów do konkretnego dnia/posiłku.  
- Zapis i edycja planów; wersjonowanie prostych zmian nieobowiązkowe.

3.4 Generowanie listy zakupów
- Generowanie listy zakupów z zapisanych planów tygodniowych.  
- Agregacja składników: sumowanie ilości tych samych składników.  
- Grupowanie składników wg kategorii (nabiał, warzywa, mięso itp.) przy użyciu prostych reguł mapowania.  
- Możliwość ręcznej edycji listy zakupów przed finalnym użyciem.

3.5 Konta użytkowników i bezpieczeństwo
- Rejestracja i logowanie (email + hasło).  
- Przechowywanie danych użytkownika i jego przesłanych plików/przepisów.  
- Ograniczenie dostępu: każdy użytkownik widzi jedynie swoje pliki i przepisy.  
- Szyfrowanie danych at-rest dla przechowywanych PDF-ów, JSON-ów z parsowania i wyodrębnionych treści.  
- Akceptacja TOS z oświadczeniem o prawach do e-booka przed uploadem.

3.6 Operacje i telemetria
- Rejestracja telemetrii: wskaźniki accuracy per pole, liczba wysłanych plików, latencja przetwarzania, liczba korekt użytkowników.  
- Automatyczne testy regresyjne na zbiorze kontrolnym raportujące accuracy per pole.  
- UI-only powiadomienia o statusie przetwarzania (queued, processing, failed, completed).

3.7 Polityka przechowywania i zarządzanie plikami
- Limit 500 MB per user; mechanizm odmowy uploadu po przekroczeniu limitu z jasnym komunikatem.  
- Retencja oryginalnych PDF-ów: 6 miesięcy; automatyczne usunięcie po tym okresie (z powiadomieniem przed usunięciem).  
- Możliwość ręcznego usunięcia pliku przez użytkownika (usuwa powiązane wyodrębnione przepisy lub archiwizuje zgodnie z regułami).

## 4. Granice produktu
- Nie obsługujemy OCR (pliki bez warstwy tekstowej są odrzucane).  
- Brak integracji z zewnętrznymi źródłami przepisów (strony WWW) w MVP.  
- Brak eksportu danych (CSV/JSON/Sync) w MVP.  
- Brak natywnej aplikacji mobilnej w MVP (responsywna web app).  
- Brak funkcji społecznościowych: udostępnianie, oceny, komentarze.  
- Brak zaawansowanej personalizacji diet (kcal, alergeny) w MVP.

## 5. Historyjki użytkowników
Lista wszystkich niezbędnych historyjek użytkownika (podstawowe, alternatywne i skrajne), każda z unikalnym ID i kryteriami akceptacji.

US-001
Tytuł: Rejestracja nowego użytkownika
Opis: Jako nowy użytkownik chcę zarejestrować konto przy użyciu email i hasła, aby zapisywać przesłane PDF-y i moje przepisy.
Kryteria akceptacji:
- GIVEN formularz rejestracji, WHEN użytkownik poda poprawny email i hasło spełniające politykę haseł, THEN konto zostaje utworzone i użytkownik zostaje zalogowany.
- Konto przechowuje metadane użytkownika i początkowe limity pamięci (500 MB limit).

US-002
Tytuł: Logowanie istniejącego użytkownika
Opis: Jako zarejestrowany użytkownik chcę się zalogować, aby uzyskać dostęp do moich plików i przepisów.
Kryteria akceptacji:
- GIVEN poprawne dane logowania, WHEN użytkownik się loguje, THEN zostaje uwierzytelniony i przekierowany do pulpitu użytkownika.
- GIVEN niepoprawne dane, WHEN próba logowania, THEN wyświetlany jest komunikat o błędnych danych bez ujawniania szczegółów bezpieczeństwa.

US-003
Tytuł: Akceptacja TOS przed uploadem
Opis: Jako użytkownik chcę potwierdzić, że posiadam prawa do przesyłanego e-booka przed uploadem.
Kryteria akceptacji:
- Przed możliwym przesłaniem pliku, użytkownik musi zaznaczyć zgodę na TOS zawierające oświadczenie o prawach do materiału.
- Upload bez zaakceptowania TOS jest zablokowany i pokazuje wyraźny komunikat wymagania zgody.

US-004
Tytuł: Upload pliku PDF (poprawny)
Opis: Jako użytkownik chcę przesłać PDF z warstwą tekstową, aby system mógł wyodrębnić przepisy.
Kryteria akceptacji:
- GIVEN zalogowany użytkownik i zaakceptowane TOS, WHEN wyśle PDF <=100MB z warstwą tekstową i w limicie konta, THEN plik zostaje przyjęty i zadanie przetwarzania trafi do kolejki (status queued).
- Po przyjęciu pliku system parsuje PDF do strukturalnego JSON, zapisuje JSON w bazie danych powiązanej z kontem użytkownika, a następnie — w razie potrzeby — wysyła jedynie ten JSON do modelu LLM w celu klasyfikacji/augmentacji. Oryginalny PDF jest przechowywany zgodnie z polityką retencji.
- UI pokazuje postęp statusu (queued → processing → completed/failed) oraz informuje, że dalsze kroki przetwarzania odbywają się na poziomie JSON.

US-005
Tytuł: Upload pliku przekraczającego limit
Opis: Jako użytkownik chcę otrzymać jasny komunikat, gdy przesyłam plik >100MB lub gdy przekroczę limit 500MB konta.
Kryteria akceptacji:
- GIVEN plik >100MB, WHEN próbuję uploadu, THEN upload jest zablokowany i pokazany jest komunikat o limicie pliku.
- GIVEN obecne użycie + rozmiar nowego pliku >500MB, WHEN próbuję uploadu, THEN upload jest zablokowany i pokazane są informacje o limicie konta.

US-006
Tytuł: Obsługa pliku bez warstwy tekstowej
Opis: Jako użytkownik chcę, aby system odrzucił pliki, które nie zawierają warstwy tekstowej i dał wskazówkę, że OCR nie jest obsługiwany.
Kryteria akceptacji:
- GIVEN plik bez warstwy tekstowej, WHEN użytkownik próbuje wysłać, THEN system odrzuca plik i pokazuje informację "plik nie zawiera warstwy tekstowej — OCR nieobsługiwany w MVP".

US-007
Tytuł: Powiadomienie UI o statusie przetwarzania
Opis: Jako użytkownik chcę widzieć status przetwarzania mojego pliku w UI.
Kryteria akceptacji:
- UI pokazuje status: queued, processing, failed, completed oraz timestampy dla istotnych zmian.
- W przypadku failed, UI wyświetla powód błędu i przycisk retry (o ile retry jest dopuszczalny).

US-008
Tytuł: Retry przetwarzania po błędzie
Opis: Jako użytkownik chcę ponowić przetwarzanie pliku, jeśli wystąpił błąd przetwarzania.
Kryteria akceptacji:
- GIVEN przetwarzanie zakończone failed, WHEN użytkownik kliknie retry, THEN zadanie zostaje ponownie umieszczone w kolejce i status zmienia się na queued.
- System stosuje retry z ograniczeniem (np. max 3 ponowień na plik) i rejestruje telemetrię błędów.

US-009
Tytuł: Wyświetlanie listy wyodrębnionych przepisów
Opis: Jako użytkownik chcę zobaczyć listę wszystkich przepisów wyodrębnionych z mojego PDF-a z informacją o pewności.
Kryteria akceptacji:
- Lista przepisów pokazuje tytuł, krótkie podsumowanie (np. pierwsza linia kroku), label pewności (high/medium/low) oraz link do edycji.
- Przepisy są powiązane z oryginalnym plikiem i datą przetworzenia.

US-010
Tytuł: Inline edycja przepisu
Opis: Jako użytkownik chcę móc edytować tytuł, składniki i kroki przepisu bez opuszczania widoku przepisu.
Kryteria akceptacji:
- GIVEN otwarty przepis, WHEN edytuję pole i zapisuję, THEN zmiany są trwałe i dostępne natychmiast.
- System rejestruje korektę (anonimowo, jeśli użytkownik zaakceptował rejestrowanie korekt) do późniejszego ETL.

US-011
Tytuł: Oznaczanie niskiej pewności i sugestia edycji
Opis: Jako użytkownik chcę, aby przepisy z niską pewnością były wyraźnie oznaczone, aby szybciej je poprawić.
Kryteria akceptacji:
- Przepisy z confidence < threshold są oznaczone jako "niska pewność" i wyświetlają sugestię "Proszę zweryfikować" oraz skrót do edycji.

US-012
Tytuł: Automatyczna kategoryzacja przepisu przez AI
Opis: Jako użytkownik chcę, aby system proponował kategorię przepisu automatycznie (z oceną pewności), aby oszczędzić czas przy kategoryzacji.
Kryteria akceptacji:
- GIVEN przepis wyekstrahowany, WHEN AI proponuje kategorię, THEN propozycja zawiera kategorię i confidence score.
- IF confidence >= threshold, THEN propozycja jest domyślnie zastosowana z możliwością ręcznej zmiany przez użytkownika.

US-012b
Tytuł: Automatyczna kategoryzacja składników
Opis: Jako użytkownik chcę, aby system automatycznie przypisywał kategorię do każdego składnika (np. nabiał, warzywa), aby listy zakupów były poprawnie pogrupowane.
Kryteria akceptacji:
- GIVEN lista składników wyodrębniona z przepisu, WHEN AI klasyfikuje składniki, THEN każdy składnik ma przypisaną kategorię i confidence score.
- Użytkownik może ręcznie poprawić kategorię składnika; poprawki są zapisywane i (opcjonalnie) anonimowo zbierane do ETL.

US-013
Tytuł: Wyszukiwanie i filtrowanie przepisów
Opis: Jako użytkownik chcę wyszukiwać przepisy po tytule, składnikach i filtrach kategorii/pewności.
Kryteria akceptacji:
- Wyszukiwanie zwraca trafne wyniki i może być filtrowane po: kategoriach, poziomie pewności, dacie przetworzenia.

US-014
Tytuł: Tworzenie tygodniowego planu posiłków (podstawowy)
Opis: Jako użytkownik chcę stworzyć tydzień planu posiłków wybierając przepisy do dni i posiłków.
Kryteria akceptacji:
- Użytkownik może przeciągać/dodawać przepisy do poszczególnych dni i posiłków (np. śniadanie/obiad/kolacja).
- Plan można zapisać i przywrócić później.

US-015
Tytuł: Edycja i usuwanie planu tygodniowego
Opis: Jako użytkownik chcę móc edytować istniejący plan lub go usunąć.
Kryteria akceptacji:
- Zapisane plany można edytować (zmiana przepisu w komórce), wersja zapisu jest aktualizowana.
- Użytkownik może usunąć cały plan i otrzymać potwierdzenie przed usunięciem.

US-016
Tytuł: Generowanie listy zakupów z planu
Opis: Jako użytkownik chcę wygenerować listę zakupów z zapisanego planu tygodniowego.
Kryteria akceptacji:
- System agreguje składniki i sumuje ilości tam, gdzie to możliwe.
- Składniki są pogrupowane według kategorii (np. nabiał, warzywa), a lista jest edytowalna.

US-017
Tytuł: Ręczna edycja listy zakupów
Opis: Jako użytkownik chcę poprawić lub usunąć pozycje na wygenerowanej liście zakupów.
Kryteria akceptacji:
- Użytkownik może zmieniać ilości i usuwać pozycje; zmiany są zapisywane lokalnie w planie zakupów.

US-018
Tytuł: Przegląd przesłanych plików i zarządzanie nimi
Opis: Jako użytkownik chcę przeglądać wszystkie przesłane PDF-y i usuwać je ręcznie.
Kryteria akceptacji:
- Widok plików pokazuje nazwę, rozmiar, datę uploadu i status przetwarzania.  
- Usunięcie pliku nie usuwa powiązane przepisy (JSON-owe wyniki parsowania pozostają zgodnie z polityką zachowania danych unless user requests deletion).

US-019
Tytuł: Zgoda na rejestrowanie korekt (anonimowo)
Opis: Jako użytkownik chcę wyrazić zgodę na anonimowe zbieranie moich korekt, aby poprawki zasilały przyszłe modele/reguły.
Kryteria akceptacji:
- Przy pierwszej edycji użytkownik widzi opcję opt-in; jeśli wyrazi zgodę, korekty są zapisywane anonimowo.  
- Użytkownik może cofnąć zgodę w ustawieniach; w takim przypadku nowe korekty nie będą zapisywane.

US-020
Tytuł: Bezpieczne wylogowanie i usunięcie konta
Opis: Jako użytkownik chcę możliwość wylogowania i trwałego usunięcia konta oraz danych.
Kryteria akceptacji:
- Wylogowanie usuwa sesję.  
- Usunięcie konta inicjuje procedurę usuwania danych: usunięcie plików i przepisów oraz anons o czasie realizacji; proces powinien być jednoznaczny i zgodny z polityką retencji.

US-021
Tytuł: Obsługa limitów pamięci konta
Opis: Jako użytkownik chcę, aby system blokował upload gdy przekroczę limit 500 MB i informował, jakie pliki mogę usunąć.
Kryteria akceptacji:
- System nie przyjmie uploadu, jeśli sumaryczne użycie + nowy plik >500MB i wskaże które pliki wpływają na limit.

US-022
Tytuł: Metryki ekstrakcji i raport accuracy (operacyjne)
Opis: Jako produktowiec chcę mieć dane accuracy per pole na zbiorze kontrolnym, aby monitorować jakość parsera.
Kryteria akceptacji:
- System produkuje raporty accuracy per pole (tytuł, składniki, kroki, makroskładniki) po każdej aktualizacji parsera lub na żądanie.
- Raport zawiera metryki: precision/recall oraz procent poprawnych pozycji zgodnie z progiem (np. 70% dla tytułów).

US-023
Tytuł: Maksymalny czas przetwarzania i eskalacja
Opis: Jako użytkownik chcę wiedzieć, jeśli przetwarzanie pliku przekroczy docelowy czas (2 min), aby móc podjąć działania.
Kryteria akceptacji:
- Jeśli latencja przekroczy 2 min, UI oznacza task jako "opóźnione" i zapewnia informację o przewidywanym czasie dłuższego przetwarzania.
- Po przekroczeniu progu, system rejestruje metrykę do monitoringu operacyjnego.

US-024
Tytuł: Audyt i logi błędów przetwarzania (operacyjne)
Opis: Jako operator chcę logi błędów i przyczyn failed, aby móc diagnozować problemy z parserem.
Kryteria akceptacji:
- Dla każdego failed job rejestrowany jest kod błędu, stack/trace i fragment wejścia (bez narażenia danych użytkownika), dostępny w panelu operacyjnym.

US-025
Tytuł: Odrzucenie uploadu z powodu limitów bezpieczeństwa lub naruszeń TOS
Opis: Jako użytkownik chcę otrzymać jasny komunikat, jeśli plik jest zablokowany ze względów prawnych lub bezpieczeństwa.
Kryteria akceptacji:
- Jeśli algorytm lub ręczna weryfikacja wykryje naruszenie, plik jest zablokowany, a użytkownik informowany o procedurze odwołania (bez ujawniania szczegółów moderacji).

US-026
Tytuł: Obsługa sieci i błędów klienta podczas uploadu
Opis: Jako użytkownik chcę, aby upload był odporowy na krótkie przerwy sieciowe i informował o konieczności ponownego wysłania w przypadku trwałych błędów.
Kryteria akceptacji:
- Upload z krótkim przerywaniem powinien być wznawialny (jeśli wsparcie przeglądarki pozwala) lub wyświetlać informację o konieczności ponowienia.
- Przy trwałym błędzie użytkownik otrzymuje jasny komunikat z instrukcją działania.

US-027
Tytuł: Filtrowanie i sortowanie planów oraz przepisów
Opis: Jako użytkownik chcę sortować i filtrować moje plany i przepisy po dacie, kategorii i poziomie pewności.
Kryteria akceptacji:
- UI pozwala na sortowanie malejąco/rosnąco i filtrację po wymienionych polach.

US-028
Tytuł: Widok szczegółowy przepisu z historią zmian (minimalny)
Opis: Jako użytkownik chcę zobaczyć historię ostatnich edycji przepisu (kto/ kiedy — anonimowość zachowana), aby móc cofnąć błędne poprawki.
Kryteria akceptacji:
- Widok pokazuje listę zmian z timestampem i typem zmiany; jeśli korekty są anonimowe, nie pokazuje identyfikatorów osobowych.
- Cofnięcie zmiany jest możliwe do ostatniego zapisu (undo pojedynczego kroku).

US-029
Tytuł: Widoczność powiadomień o usunięciu plików po retencji
Opis: Jako użytkownik chcę otrzymać powiadomienie w UI na 7 dni przed planowanym automatycznym usunięciem pliku po okresie retencji.
Kryteria akceptacji:
- UI zawiera widoczny banner/listę nadchodzących usunięć dla plików z datą usunięcia i opcją przedłużenia retencji (o ile polityka to dopuszcza).

US-030
Tytuł: Uwierzytelnienie i autoryzacja API (dla przyszłych integracji)
Opis: Jako deweloper chcę, aby API MVP wymagało uwierzytelnienia i respektowało granice zasobów, nawet jeśli integracje zewnętrzne są wyłączone.
Kryteria akceptacji:
- Wszystkie endpointy wymagające dostępu do danych użytkownika wymagają tokena sesji lub cookie sesyjnego.  
- Endpointy chroniące pliki, JSON-owe wyniki parsowania i przepisy zwracają 403 przy braku autoryzacji.  
- Interakcje z usługami modelowymi (LLM) korzystają z ustrukturyzowanego JSON zapisanego w bazie; API eksponuje bezpieczne endpointy do pobrania/minifikacji tego JSON przed wysłaniem do zewnętrznych modeli.


## 6. Metryki sukcesu
Lista kluczowych metryk i kryteriów sukcesu do monitorowania po uruchomieniu MVP:
- Metryki ekstrakcji (zbiór kontrolny):
  - 70% poprawnych tytułów (US-punkt docelowy).  
  - 60% poprawnych składników.  
  - 50% poprawnych makroskładników (tam gdzie obecne).
- Operacyjne KPI:
  - % poprawnych przepisów na zbiorze kontrolnym (monitorowane po każdej aktualizacji parsera).  
  - Liczba utworzonych planów tygodniowych (miesięcznie).  
  - Liczba wygenerowanych list zakupów (miesięcznie).  
  - Średnia latencja przetwarzania (target <2 min dla 20–30 MB).  
  - % udanych retry dla failed jobs (cel >80% przy max 3 retry).
- Produktowe KPI (adopcja/retencja):
  - % użytkowników, którzy po uploadzie tworzą co najmniej jeden plan tygodniowy (dostępność funkcji core flow).  
  - % użytkowników, którzy akceptują anonimowe rejestrowanie korekt (opcja opt-in).
- Telemetria i jakość:
  - Liczba korekt użytkowników (źródło danych dla ETL).  
  - Liczba błędów krytycznych przetwarzania na 1000 uploadów.
- Kontrola jakości i testy:
- Automatyczne testy regresyjne uruchamiane na zbiorze kontrolnym (dostarczonym przez użytkownika) po każdej istotnej zmianie parsowania, raportujące accuracy per pole.  
- Testy obciążeniowe pipeline'u asynchronicznego, by potwierdzić SLA latencji.

---

Lista kontrolna PRD (przejrzeć przed przekazaniem do developmentu):
- Czy każdą historię użytkownika można przetestować? Tak — wszystkie mają jasne kryteria akceptacji.  
- Czy kryteria akceptacji są jasne i konkretne? Tak — użyto scenariuszy GIVEN/WHEN/THEN tam gdzie to było sensowne.  
- Czy mamy wystarczająco dużo historyjek, aby zbudować funkcjonalną aplikację? Tak — uwzględniono podstawowe, alternatywne i skrajne scenariusze.  
- Czy uwzględniono uwierzytelnianie i autoryzację? Tak — US-001, US-002 i US-030 oraz wymogi bezpieczeństwa w sekcji 3.5.

Plik zapisany jako .ai/prd.md zawiera zaktualizowany PRD z opisem pipelineu: PDF → JSON → zapis w DB → LLM otrzymuje tylko JSON.
