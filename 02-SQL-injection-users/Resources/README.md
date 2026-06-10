# SQL Injections

## 👋 Introducción

Hola

---

## 📌 Qué es

SQL Injection es una vulnerabilidad que permite manipular consultas SQL mal construidas.

---

## ⚠️ Ejemplo básico

```sql
SELECT * FROM users WHERE id = 1 OR 1=1;
