# Wypożyczalnia multimediów

Desktopowa aplikacja w Javie (Swing) do obsługi wypożyczalni książek i audiobooków. Projekt powstał na przedmiocie **MAS (Modelowanie i Analiza Systemów)** na PJATK. Napisana w czystej Javie, bez frameworków i bez bazy danych.

## Funkcje

- **Wypożyczanie i zwroty.** Wybierasz czytelnika i dostępny egzemplarz, a aplikacja nalicza termin zwrotu zależnie od typu pozycji (książka 30 dni, audiobook 14 dni).
- **Kary za zwłokę.** Liczone osobno dla każdego typu pozycji: książka 0,50 zł/dzień, audiobook 0,30 zł/dzień po 2 dniach karencji.
- **Rejestracja czytelnika.** Numer karty nadaje się automatycznie (`K-001`, `K-002`, …), a e-mail jest walidowany i sprawdzany pod kątem unikalności.
- **Katalog pozycji.** Dodawanie książek i audiobooków, dodawanie egzemplarzy do oddziałów oraz usuwanie pozycji razem z egzemplarzami i historią wypożyczeń.
- **Trwałość danych.** Stan aplikacji zapisuje się po każdej operacji do pliku `dane.bin` (serializacja Javy). Przy pierwszym uruchomieniu ładują się dane przykładowe.

## Reguły biznesowe pilnowane w kodzie

- Czytelnik może mieć najwyżej **5 aktywnych wypożyczeń**.
- Nie da się wypożyczyć egzemplarza, który ma stan `WYPOZYCZONY`.
- ISBN, e-mail i numer karty są unikalne, a sygnatura jest unikalna w obrębie oddziału.
- Ocena w recenzji musi być z zakresu 1–5. Jeden czytelnik może wystawić jedną recenzję danej pozycji.
- Jeśli walidacja w konstruktorze się nie powiedzie, obiekt jest usuwany z ekstensji. W systemie nie zostają „pół-utworzone" obiekty.

## Model obiektowy

| Konstrukcja | Realizacja w kodzie |
|---|---|
| Dziedziczenie overlapping / dynamiczne role `{OR}` | `Osoba` może być jednocześnie `Czytelnikiem` i `Pracownikiem` (kompozycja ról, `asCzytelnik()` / `asPracownik()`) |
| Wielodziedziczenie | `AudiobookCyfrowy extends Audiobook implements IPlikCyfrowy` |
| Klasa abstrakcyjna + polimorfizm | `Pozycja` → `Ksiazka`, `Audiobook` (`okresWypozyczenia()`, `oplataZaZwloke()`) |
| Dynamiczny stan `{XOR}` | `Wypozyczenie` ma albo stan `Wypozyczone`, albo `Zwrocone` |
| Klasa asocjacji | `Wypozyczenie` łączy `Czytelnik`, `Pracownik` i `Egzemplarz` |
| Asocjacja kwalifikowana | `Oddzial.znajdzEgzemplarz(sygnatura)` (mapa po sygnaturze) |
| Kompozycja | usunięcie `Pozycji` kaskadowo usuwa jej `Egzemplarze` i powiązane wypożyczenia |
| Atrybut złożony | `Adres` (ulica, miasto, kod) |
| Atrybut wielowartościowy | gatunki pozycji (`List<String>`) |
| Atrybut i metoda klasowa | domyślny limit wypożyczeń, `Wypozyczenie.liczbaAktywnych()` |
| Ekstensja klasy | klasa bazowa `ObjectPlus` trzyma wszystkie obiekty każdej klasy i zapisuje je do pliku |

Asocjacje są dwukierunkowe, a połączenia zwrotne są aktualizowane automatycznie.

## Architektura

```
pja/edu/
├── ObjectPlus.java          # ekstensje klas + serializacja
├── DataInitializer.java     # dane przykładowe
├── Main.java                # demo konsolowe (zapis/odczyt, kary, usuwanie kaskadowe)
├── model/                   # klasy domenowe
├── gui/                     # Swing: widoki + WypozyczalniaController (MVC)
└── storage/ObjectManager.java  # zapis/odczyt pliku dane.bin
```

Widoki (`GlowneOkno`, `KatalogOkno`, `RejestracjaOkno`) nie dotykają modelu bezpośrednio. Każda operacja przechodzi przez `WypozyczalniaController`.

## Uruchomienie

Wymagana Java 17+ (testowane na JDK 21).

```bash
cd WypozyczalniaMultimedia
javac -d out $(find pja -name "*.java")

# aplikacja okienkowa
java -cp out pja.edu.gui.App

# demo w konsoli
java -cp out pja.edu.Main
```

W IntelliJ wystarczy otworzyć folder i uruchomić `pja.edu.gui.App`.

## Technologie

Java · Swing · MVC · serializacja obiektów

---
Autor: Hubert Żukowski
