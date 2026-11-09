# Jak to działa (dla mnie, prostym językiem)

Projekt na fikcyjnych danych. Fikcyjna firma „ExamplePush” robi powiadomienia web push. Instaluje się na stronie klienta małym skryptem. Klient może po cichu dodać skrypt konkurencji albo usunąć nasz. Zwykle dowiadujemy się o tym za późno: kiedy spadają wyniki albo przychodzi wypowiedzenie. Ten monitor raz dziennie sprawdza publiczny kod stron klientów i mówi CSM-owi, co się zmieniło, zanim klient sam to zgłosi.

## Co robi każdy workflow

| Workflow | Co robi |
| --- | --- |
| SIM Mock Sites | Udaje 10 stron klientów. Każda ma 5 wersji (dni 1–5), kontener GTM i pliki service workera. Odpowiada jak prawdziwy serwer: 200, 403 albo 500. |
| Site Integration Monitor | Skaner. Pobiera stronę, szuka skryptów, pobiera kontener GTM i service workery, porównuje z tabelą sygnatur i z wczoraj. |
| SIM Drafts | Dla potwierdzonego alertu Claude pisze dwa szkice: alert dla CSM-a z dowodem i maila do klienta. Kod sprawdza oba. Nic nie jest wysyłane. |
| SIM Eval | Test: puszcza skaner przez dni 1–5 i porównuje 50 wyników z odpowiedziami, które spisałem przed testem. |
| SIM Draft Eval | Test szkiców: każdy szkic musi przejść kontrolę kodem, a sama kontrola musi dobrze ocenić 13 maili, które napisałem ręcznie (11 złych, 2 dobre). |

## Czym jest service worker

To mały plik JavaScript, który przeglądarka instaluje dla strony i trzyma w tle, nawet gdy strona jest zamknięta. Dzięki niemu powiadomienie push może się pokazać, kiedy klient sklepu nie ma otwartej karty. Każde narzędzie push ma swój plik, zwykle w głównym katalogu strony, np. `/OneSignalSDKWorker.js` albo `/pushwoosh-service-worker.js`. W środku jest zwykle jedna linijka `importScripts(...)` z adresem dostawcy. Dlatego po samym pliku widać, czyj to service worker.

Ważne: sam plik service workera nie znaczy, że narzędzie działa. W scenariuszu D nasz skrypt zniknął ze strony, a stary plik `/examplepush-sw.js` dalej leży na serwerze. Gdybym liczył plik jako „jest”, przeoczyłbym ten alert. Dlatego decyzję podejmuję po skrypcie na stronie albo w GTM, a service worker to dodatkowy dowód w alercie.

## Czemu patrzymy w GTM

Google Tag Manager to „pojemnik” na skrypty. Na stronie jest tylko jeden kawałek kodu GTM z identyfikatorem typu `GTM-XXXXXXX`. Wszystkie inne skrypty marketer dodaje w panelu GTM, bez programisty i bez zmiany kodu strony. Czyli w samym HTML strony nie widać, że doszła konkurencja.

Skaner robi tak: znajduje na stronie identyfikator GTM, pobiera kontener (plik `gtm.js?id=...`) i szuka w nim skryptów. W kontenerze kod jest zapisany „zakodowany” (np. `<` jako `<`, `/` jako `\/`), więc najpierw go odkodowuje. Scenariusz C to sprawdza: konkurencja jest tylko w GTM, a HTML strony się nie zmienia.

## Po co dwa skany z rzędu

Strony czasem robią coś jednorazowo: chwilowy błąd, test A/B, ktoś coś dodał i za godzinę usunął. Jeden dziwny wynik to jeszcze nie fakt. Alarm idzie dopiero wtedy, gdy ta sama zmiana pojawi się w dwóch udanych skanach z rzędu. Dzień z błędem 500 (scenariusz G) się nie liczy: skaner go pomija i niczego nie zmienia w zapamiętanym stanie. Dzięki temu CSM dostaje mniej alarmów, ale każdy jest potwierdzony. Koszt: alert przychodzi dzień później.

Dobre wiadomości (nasz skrypt wrócił, konkurencja zniknęła) przyjmuję od razu, bo nie mogą wywołać fałszywego alarmu.

## Czemu mail do klienta nie wspomina o skanowaniu

Gdybym napisał „widzimy, że dodaliście OneSignal”, klient poczuje się śledzony, a rozmowa zrobi się nerwowa. To podważa zaufanie, a celem jest zatrzymanie klienta. Dlatego:

- Przy zniknięciu naszego skryptu mail to zwykła pomoc techniczna: „wygląda na to, że powiadomienia mogły przestać się wyświetlać”, pytanie, czy coś zmieniali na stronie, i link do instrukcji dla ich programisty. Mail mówi „mogły”, a nie „przestały”, bo to jest przypuszczenie, a nie zgłoszenie od klienta.
- Przy konkurencji mail to zwykły mail z wartością: jeden pomysł na kampanię dla ich branży i propozycja wysłania checklisty.
- Model piszący maila w ogóle nie widzi dowodów, nazw konkurencji ani nazwy klienta. Nie może więc ich zdradzić.
- Kod i tak sprawdza każdy szkic: zakazane są nazwy konkurencji i słowa typu „scan”, „monitor”, „your code”, „tag manager”, „script”, „we noticed”, a także prośby o call, darmowa praca i obietnice.
- Szkic trafia do mnie do akceptacji. Nikt nic nie wysyła automatycznie.

## Co kod uznaje za „skrypt”

Tylko `<script>`, który przeglądarka naprawdę wykona. Pomijam tekst, linki, bloki `<code>`, komentarze HTML i bloki danych typu `<script type="application/ld+json">`. Scenariusz H to sprawdza: wpis na blogu wymienia OneSignal, w tekście i w danych JSON-LD jest nawet pełny adres skryptu OneSignal. Zwykłe szukanie słowa w całym HTML dałoby tu fałszywy alarm.

## Dwa dodatkowe scenariusze (I i J)

- **I: konkurencja tylko na jeden dzień.** Agencja klienta wrzuciła na próbę skrypt Webpushr na jeden dzień i go zdjęła. Skaner zobaczył go raz („pending”), następnego dnia skryptu już nie było, więc nic nie poszło do CSM-a. To jest dokładnie ten przypadek, dla którego są dwa skany z rzędu.
- **J: nasz skrypt ładowany „inaczej” i stary skrypt w komentarzu.** Nasz skrypt nie jest wpisany jako `<script src=...>`, tylko dokładany przez mały kawałek kodu (tzw. loader). Skaner i tak go znajduje, bo czyta też kod w środku tagów `<script>`. Do tego w kodzie strony jest stary skrypt iZooto, ale zakomentowany (`<!-- ... -->`), czyli przeglądarka go nie uruchamia. Skaner go ignoruje i nie ma fałszywego alarmu.

## Dowód braku

Kiedy nasz skrypt znika, trudno pokazać „dowód”, bo dowodem jest to, czego nie ma. Dlatego alert ma teraz osobną linijkę „not found”: w jakich plikach szukaliśmy (np. „homepage HTML + gtm.js?id=...”) i jakiego wzorca. CSM może otworzyć te same pliki i sprawdzić to sam w minutę.

## Czego nie zdążyłem sprawdzić

Po dodaniu I, J i linijki „not found” test wykrywania przeszedł 50 z 50. Potem w triale n8n skończył się limit uruchomień, więc testu szkiców AI nie puściłem już na tych nowych alertach. Szkice w repo są z wcześniejszego przebiegu. Kod szkiców się nie zmienił, ale uczciwie: ta kombinacja nie jest przetestowana.

## Co nie przeszło za pierwszym razem

1. Test zatrzymywał się po dniu 1. Węzeł wywołujący skaner nic nie zwracał, bo ostatni węzeł skanera był filtrem bez wyników, więc pętla nie szła dalej. Dodałem do skanera końcowy węzeł „Run finished” i ustawienie, żeby wywołanie zawsze coś zwracało. Sam skaner liczył dobrze.
2. Szkice maili przeszły kontrolę kodem 4 z 4, ale przeczytałem je ręcznie i 2 maile techniczne były złe: dziękowały klientowi „za kontakt” i przepraszały, jakby klient sam zgłosił problem. Dodałem regułę w prompcie i nową kontrolę w kodzie (zakaz „thank you for reaching out” i przeprosin, wymóg „it looks like / may have”), plus 2 nowe maile testowe.
3. W drugiej rundzie treść maili była już ostrożna, ale 2 tematy maili dalej mówiły wprost „Notifications stopped showing”. Dodałem kontrolę tematu i jeszcze jeden mail testowy. W trzeciej rundzie 4 z 4 szkiców przeszły kontrolę i przy czytaniu nie znalazłem już błędów.

---

## 5 pytań rekrutacyjnych (po angielsku) z krótkimi odpowiedziami

**1. Why did you use rules instead of AI to detect the integrations?**
Detection has to be repeatable and explainable. A rule gives the same answer on the same page every time, and every alert shows the exact file and code fragment. AI only writes the drafts, after the rules have decided.

**2. How do you avoid false alarms?**
Three ways. I only count `<script>` tags that actually run, so text, links and JSON-LD on a blog post are ignored. A change must show on two successful scans in a row. And a 500 error or a 403 block never becomes an alert: the day is skipped or marked "cannot check".

**3. Why do you look inside Google Tag Manager?**
Marketing teams often add tools through GTM without touching the site code. The page HTML stays the same, so a scanner that only reads HTML would miss it. I read the GTM ID from the page, fetch the container and check the tags inside it. One of my ten test scenarios is exactly this.

**4. Is it ethical to check a client's website?**
It only reads public files any browser downloads, once a day, with an honest user agent. It never logs in, and when a site blocks it with a 403, it stops and marks the site "cannot check" instead of working around the block. The client email never mentions the check or a competitor; it offers help or value, and a person approves it first.

**5. What would you do next before using it with real clients?**
Add a headless browser, because some scripts only load after cookie consent and are invisible in the raw HTML. Test it on more real-world page layouts. And connect it to delivery data from our own platform, so a drop in delivered notifications confirms the site signal.
