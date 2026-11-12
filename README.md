# django_szablony_i_formularze

Materiał 11 – Django: szablony i formularze. Projekt `config`, aplikacja `shop` (kontynuacja materiału 10).

## Uruchomienie

```powershell
python --version                 # wymagane 3.12+
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python manage.py runserver
```

Serwer: http://127.0.0.1:8000

## Notatki

### Zadanie 3 – Jinja2 → szablony Django

Zmiany potrzebne przy przepisywaniu `product_list.html` z Flaska na DTL (na podstawie tabeli z materiału):

| Jinja2 (Flask) | DTL (Django) |
|---|---|
| `{{ url_for('produkt', id=p.id) }}` | `{% url 'shop:product_detail' p.id %}` |
| `{{ url_for('static', filename='style.css') }}` | `{% static 'shop/style.css' %}` + `{% load static %}` |
| `loop.index` | `forloop.counter` |
| `{{ price\|round(2) }}` | `{{ price\|floatformat:2 }}` |
| `{{ x\|default("brak") }}` | `{{ x\|default:"brak" }}` |
| `{{ "tak" if ok else "nie" }}` | `{{ ok\|yesno:"tak,nie" }}` |
| `{{ p.gross_price() }}` | `{{ p.gross_price }}` (bez nawiasów) |
| `{{ items[0] }}` | `{{ items.0 }}` |
| `{{ price * 1.23 }}` | brak arytmetyki – liczone w widoku |

Liczba zmian w moim `product_list.html` z Flaska: **UZUPEŁNIJ po przepisaniu własnego pliku**.

### Zadania 4 i 5 – formularz `ProductForm` i walidacja wbudowana

Formularz jest pod `/products/add/` (GET → pusty formularz, POST → walidacja, przy sukcesie redirect na listę). Klasa ma 5 pól: `name` (`CharField`, 3–100 znaków), `price` (`DecimalField`, min. 0,01), `category` (`ChoiceField`), `is_available` (`BooleanField(required=False)`) i `description` (`CharField` z `Textarea`, `required=False`).

Sprawdzane przypadki (zrzuty ekranu: **DODAJ do `docs/` i podlinkuj tutaj**):

- pusty formularz → „To pole jest wymagane.” przy polach `name` i `price`,
- za krótka nazwa („ab”) → komunikat o minimalnej długości 3 znaków,
- ujemna cena („-5”) → komunikat, że wartość musi być większa lub równa 0,01,
- tekst w polu liczbowym (po zmianie `type="number"` na `type="text"` w F12) → komunikat o konieczności wpisania liczby.

Po usunięciu atrybutu `required` w F12 i wysłaniu pustego formularza dalej pojawia się „To pole jest wymagane.”. Walidacja działa **po stronie serwera** w Pythonie, a atrybuty HTML tylko pomagają użytkownikowi i można je dowolnie zmienić w przeglądarce, więc nie chronią aplikacji.

### Zadanie 6 – walidacja własna

- `clean_name()` – nazwa nie może być „test” ani „asdf” (bez względu na wielkość liter); metoda zwraca też nazwę po `strip()`.
- `clean()` – reguła między polami: produkt z kategorii „Laptopy” nie może kosztować mniej niż 500 zł (błąd przypięty do pola `price` przez `add_error`).

Test ręczny: nazwa „test” → błąd przy polu Nazwa; kategoria Laptopy + cena 300 → błąd przy polu Cena; kategoria Akcesoria + cena 300 → zapis przechodzi.

### Zadanie 7 – CSRF

Po usunięciu `{% csrf_token %}` wysłanie formularza kończy się odpowiedzią **403 Forbidden** („CSRF verification failed”) – zrzut ekranu: **DODAJ do `docs/`**. Token przywrócony.

CSRF to atak, w którym obca strona wysyła w tle formularz POST do serwisu, w którym użytkownik jest zalogowany, a przeglądarka sama dołącza jego ciasteczka, więc serwis uznaje żądanie za prawdziwe. Obrona polega na tym, że każdy formularz POST dostaje losowy token zapisany w ciasteczku i w ukrytym polu, a Django sprawdza, czy się zgadzają. Obca strona nie zna tego tokena, więc jej podrobione żądanie dostaje 403, zanim widok się uruchomi.

### Zadanie 8 – wyszukiwarka (GET)

`SearchForm` (pola `q` i `only_available`) jest związany z `request.GET`, więc po wysłaniu pola zostają wypełnione bez ręcznego `value="{{ q }}"`. Formularz jest nad listą na `/products/`, np. `/products/?q=lap&only_available=on`.

Nie ma w nim `{% csrf_token %}`, bo GET nie zmienia danych na serwerze – nie ma czego podrobić. Token jest potrzebny tylko dla metod zmieniających stan (POST, PUT, PATCH, DELETE).
