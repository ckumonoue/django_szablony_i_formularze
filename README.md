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
