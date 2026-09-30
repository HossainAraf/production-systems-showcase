# Language & Naming Conventions — v1 (Bangla‑first)

**Project:** medic-bd
**Status:** Active for v1
**Audience:** Developers, Data‑entry team, Reviewers

---

## 1. Purpose of this document

This document defines **mandatory naming and language rules** for medic-bd v1, where **Bangla is the primary content language**.

The goal is to:

* Ship fast with Bangla-first data
* Avoid future schema deadlocks
* Ensure v2 bilingual migration is safe and low-cost

These rules are **non-optional** for v1.

---

## 2. Core policy (read first)

> **All patient-facing content lives in the database.**
> **All UI/system text lives in locale files.**

v1 uses **Bangla-only content**, but **schema and naming must assume bilingual future**.

---

## 3. v1 Naming Convention Checklist (MANDATORY)

### 3.1 Database columns (content)

#### ✅ Allowed in v1 (Bangla-only, language-agnostic names)

Use **neutral names** that will later become `_bn`:

* `name`
* `title`
* `description`
* `qualification`
* `designation`
* `address`
* `bio`

These fields **must contain Bangla only**.

#### ❌ Forbidden in v1

* `name_bn` (do not introduce yet)
* `name_en` (do not introduce yet)
* Mixed-language text in one field
* Hardcoded English text

---

### 3.2 Database columns (technical / non-language)

These **must NOT be duplicated** and stay single:

* `phone`
* `email`
* `registration_number`
* `years_of_experience`
* `gender`
* `availability`
* `specialization_id`
* timestamps

---

### 3.3 Relationships (critical)

#### ✅ Correct

```ruby
Doctor belongs_to :specialization
```

Specialization stores its own Bangla content.

#### ❌ Incorrect

* Storing specialization text directly on doctor
* Using enums for medical names

---

## 4. Locale file rules (strict)

### 4.1 What belongs in `config/locales/*`

* Buttons
* Headings
* Labels
* Validation messages
* Flash notices
* Navigation items

Example:

```yml
bn:
  buttons:
    submit: "জমা দিন"
```

### 4.2 What NEVER belongs in locales

* Doctor names
* Qualifications
* Specializations
* Clinic names
* Addresses

Violating this rule blocks CRUD and SEO.

---

## 5. View & rendering rules

### 5.1 Views must be language‑agnostic

Views **must not contain Bangla literals** for content.

❌ Bad:

```erb
<h1>হৃদরোগ বিশেষজ্ঞ</h1>
```

✅ Good:

```erb
<h1><%= @specialization.name %></h1>
```

---

## 6. Admin / Data‑entry rules

### 6.1 Content language

* **Only Bangla** allowed in content fields
* No English abbreviations unless medically standard

### 6.2 Form grouping (required)

Forms must be grouped as:

1. Content (Bangla)
2. Technical data

This prepares for future bilingual UI.

---

## 7. SEO rules (v1)

* Bangla content is canonical
* URLs use English slugs (stable)
* `<html lang="bn">` must be set

Slugs must NOT be Bangla.

---

## 8. v2 Migration Contract (pre‑agreed)

When bilingual support is added:

* `name` → `name_bn`
* `qualification` → `qualification_bn`
* Add parallel `_en` columns
* No data rewrite, only column migration

v1 schema is considered **migration‑ready** if these rules are followed.

---

## 9. Enforcement

Any PR that:

* Adds mixed‑language content
* Adds domain data to locale files
* Uses enums for medical names

**Must be rejected.**

---

## 10. Final note

This document is a **guardrail**, not a suggestion.

Following it ensures:

* Clean v1 delivery
* Zero‑panic v2 language expansion
* Low friction for devs and data‑entry staff

---

**File location:** `/docs/language-and-naming-v1.md`
