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

## Struktura

```text
django_szablony_i_formularze/
├── manage.py
├── config/                  # ustawienia projektu
│   ├── settings.py
│   └── urls.py              # główna mapa URL
├── shop/                    # aplikacja
│   ├── forms.py             # formularze: ProductForm, SearchForm
│   ├── views.py
│   ├── urls.py
│   ├── templates/
│   │   └── shop/            # <- drugi raz "shop"
│   │       ├── base.html
│   │       ├── index.html
│   │       ├── item_list.html
│   │       ├── item_detail.html
│   │       ├── product_list.html
│   │       ├── product_detail.html
│   │       └── product_form.html
│   └── static/
│       └── shop/            # <- i tu też
│           └── style.css
├── requirements.txt
└── .gitignore
```

**Dlaczego dwa razy `shop`?** Django zbiera szablony (i pliki statyczne) ze wszystkich aplikacji do jednej puli. Gdyby dwie aplikacje miały `index.html` bez podfolderu, wygrałby plik z aplikacji wyżej w `INSTALLED_APPS` – po cichu, bez błędu. Podfolder z nazwą aplikacji zmienia nazwy na `shop/index.html` i `blog/index.html`, więc nie ma konfliktu. Dlatego w widoku piszemy `render(request, "shop/index.html", ...)`, a w szablonie `{% static 'shop/style.css' %}`.

## Trasy

| Metoda | Adres | Nazwa trasy | Opis |
|---|---|---|---|
| GET | `/` | `shop:index` | strona główna (szablon) |
| GET | `/about/`, `/contact/`, `/terms/` | `shop:about`, `shop:contact`, `shop:terms` | proste strony tekstowe |
| GET | `/api/info/` | `shop:info` | JSON |
| GET | `/admin-panel/` | `shop:forbidden` | zawsze 403 |
| GET | `/hello/<name>/`, `/hello/<name>/<age>/` | `shop:hello`, `shop:hello_age` | powitania |
| GET | `/add/`, `/subtract/`, `/multiply/`, `/divide/` + `<a>/<b>/` | `shop:add` … | kalkulator |
| GET | `/items/` | `shop:item_list` | lista przedmiotów, niedostępne wyróżnione |
| GET | `/items/<id>/` | `shop:item_detail` | szczegóły, brak → 404 |
| GET | `/products/?q=&only_available=` | `shop:product_list` | lista produktów z wyszukiwarką |
| GET, POST | `/products/add/` | `shop:product_add` | formularz dodawania z walidacją |
| GET | `/products/<id>/` | `shop:product_detail` | szczegóły produktu |
| GET | `/products/<slug>/` | `shop:product_slug` | produkt po slugu (po trasach z `int`) |
| GET | `/start/` | `shop:start` | redirect na `/` |
| GET | `/links/` | `shop:links` | adresy z `reverse` |
| GET | `/admin/` | – | panel admina Django |

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

### Zadania 4 i 5 – formularz `ProductForm` i walidacja wbudowana

Formularz jest pod `/products/add/` (GET → pusty formularz, POST → walidacja, przy sukcesie redirect na listę). Klasa ma 5 pól: `name` (`CharField`, 3–100 znaków), `price` (`DecimalField`, min. 0,01), `category` (`ChoiceField`), `is_available` (`BooleanField(required=False)`) i `description` (`CharField` z `Textarea`, `required=False`).

Sprawdzane przypadki:

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

Po usunięciu `{% csrf_token %}` wysłanie formularza kończy się odpowiedzią **403 Forbidden** („CSRF verification failed”). Token przywrócony.

CSRF to atak, w którym obca strona wysyła w tle formularz POST do serwisu, w którym użytkownik jest zalogowany, a przeglądarka sama dołącza jego ciasteczka, więc serwis uznaje żądanie za prawdziwe. Obrona polega na tym, że każdy formularz POST dostaje losowy token zapisany w ciasteczku i w ukrytym polu, a Django sprawdza, czy się zgadzają. Obca strona nie zna tego tokena, więc jej podrobione żądanie dostaje 403, zanim widok się uruchomi.

### Zadanie 8 – wyszukiwarka (GET)

`SearchForm` (pola `q` i `only_available`) jest związany z `request.GET`, więc po wysłaniu pola zostają wypełnione bez ręcznego `value="{{ q }}"`. Formularz jest nad listą na `/products/`, np. `/products/?q=lap&only_available=on`.

Nie ma w nim `{% csrf_token %}`, bo GET nie zmienia danych na serwerze – nie ma czego podrobić. Token jest potrzebny tylko dla metod zmieniających stan (POST, PUT, PATCH, DELETE).
