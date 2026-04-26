# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FootFusion — a Django 5.1 eCommerce platform for selling shoes. Features product catalog with variants, cart/checkout, bank-transfer checkout (THB), email verification, social auth (Google/Facebook), PDF invoices, reviews, wishlist, and coupon discounts.

## Common Commands

```bash
# Activate virtual environment (Windows)
.\venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Database migrations
python manage.py makemigrations
python manage.py migrate

# Run development server
python manage.py runserver

# Create admin user
python manage.py createsuperuser

# Run tests
python manage.py test                        # all tests
python manage.py test accounts               # single app
python manage.py test accounts.tests.MyTest  # single class
```

## Environment Setup

Copy `.env.example` to `.env` and fill in all values. Required variables:

- `SECRET_KEY` — Django secret key
- `DEBUG` — `True` for local dev
- `BASE_URL` — e.g. `http://127.0.0.1:8000` (used in emails and social auth callbacks)
- `ALLOWED_HOSTS` — comma-separated hosts
- `SOCIAL_AUTH_FACEBOOK_KEY` / `SOCIAL_AUTH_FACEBOOK_SECRET` — Facebook OAuth
- `EMAIL_HOST_USER` / `EMAIL_HOST_PASSWORD` — Gmail app password for transactional email

Config is loaded via `python-decouple` from `.env`.

## Architecture

### App Structure

| App | Responsibility |
|-----|---------------|
| `ecomm/` | Project settings, root URLs |
| `base/` | Abstract `BaseModel` (UUID PK + timestamps), shared email utilities |
| `home/` | Landing page, search, contact/about/terms pages, `ShippingAddress` model |
| `products/` | `Product`, `Category`, `ColorVariant`, `SizeVariant`, `ProductReview`, `Wishlist`, `Coupon` |
| `accounts/` | `Profile` (1:1 User), `Cart`, `CartItem`, `Order`, `OrderItem` |

### Key Patterns

**BaseModel**: All custom models inherit from `base.models.BaseModel`, which provides a UUID primary key and `created_at`/`updated_at` timestamps.

**Product variants**: `SizeVariant` and `ColorVariant` have a `price` field that adds to the base product price. Products relate to variants via ManyToMany. Cart items store the chosen size/color as ForeignKeys.

**Images**: All images use `URLField` — no file uploads. Products, categories, and user profiles store image URLs.

**Cart lifecycle**: A `Cart` is created per user. `is_paid=False` means active/unpaid. On checkout (`place_order` POST), `is_paid=True` and an `Order` record is created from cart items. Prices are in Thai Baht (THB, ฿).

**Email verification**: Registration sends a token-based activation link. The token is stored on `Profile`; users must verify before login is allowed.

**Social auth**: Handled by `django-allauth` + `social-auth-app-django`. Google uses allauth; Facebook uses python-social-auth. Both require site domain configured in Django Sites framework (`python manage.py shell` → set `Site` domain).

**PDF invoices**: WeasyPrint converts an HTML template (`templates/payment_success/`) to PDF, returned as a download response.

### URL Layout

```
/                   → home app (index, search, contact, about)
/accounts/          → accounts app (login, register, cart, orders, profile)
/product/           → products app (detail, review, wishlist)
/admin/             → Django admin
```

### Templates

Base template is `templates/base/base.html`. All app templates extend it. Error pages (`404.html`, `500.html`) are at the root of `templates/`.

Static files (CSS, FontAwesome fonts) live in `public/media/`. `STATICFILES_DIRS` points there; `collectstatic` outputs to `staticfiles/`.

## Known Setup Issues (from README)

- **`Site.DoesNotExist`**: Run `python manage.py shell` and set `Site.objects.update_or_create(id=1, defaults={'domain': '127.0.0.1:8000', 'name': 'localhost'})` before using social auth.
- **WeasyPrint on macOS**: Requires `brew install pango` and `brew install gdk-pixbuf`.
- **`pkg_resources` error**: Upgrade pip — `pip install --upgrade pip setuptools`.
