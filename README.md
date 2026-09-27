# Лабораторна робота №1

## Робота з СУБД PostgreSQL та основи SQL

**Здобувач освіти:** Чижук Михайло  
**Група:** ІПЗ-32  
**Рівень складності:** 2

### Мета роботи

Набути практичних навичок роботи з системою керування базами даних PostgreSQL та мовою SQL, виконати основні операції вибірки, фільтрації та сортування даних.

## Хід роботи

Для виконання лабораторної роботи використано базу даних **«ТехноМарт»**. У базі даних наявні таблиці:

* `categories`
* `customers`
* `employees`
* `order_items`
* `orders`
* `products`
* `regions`
* `suppliers`

### 1. Перевірка таблиць бази даних

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```

**Результат:** отримано перелік таблиць бази даних.

*Скріншот результату*

### 2. Робота з таблицею customers

```sql
SELECT *
FROM customers;
```

*Скріншот результату*

### 3. Вибірка товарів

```sql
SELECT product_name, unit_price
FROM products;
```

*Скріншот результату*

### 4. Фільтрація даних

```sql
SELECT *
FROM customers
WHERE city = 'Київ';
```

```sql
SELECT product_name, unit_price
FROM products
WHERE unit_price > 25000;
```

*Скріншот результатів*

### 5. Робота із замовленнями

```sql
SELECT *
FROM orders
WHERE order_status = 'delivered';
```

*Скріншот результату*

### 6. Сортування та обмеження результатів

```sql
SELECT product_name, unit_price
FROM products
ORDER BY unit_price DESC
LIMIT 10;
```

```sql
SELECT *
FROM orders
ORDER BY order_date DESC
LIMIT 5;
```

*Скріншот результатів*

### 7. Додаткові запити рівня 2

```sql
SELECT product_name, unit_price
FROM products
WHERE product_name ILIKE '%Samsung%';
```

```sql
SELECT product_name, unit_price
FROM products
WHERE unit_price BETWEEN 10000 AND 30000;
```

```sql
SELECT contact_name, city
FROM customers
WHERE city IN ('Київ', 'Львів', 'Харків');
```

```sql
SELECT order_id, order_date, order_status
FROM orders
WHERE shipped_date IS NULL;
```

```sql
SELECT product_name, unit_price
FROM products
ORDER BY product_id
LIMIT 5 OFFSET 5;
```

*Скріншоти результатів*

## Висновок

У ході лабораторної роботи було виконано роботу з базою даних PostgreSQL та відпрацьовано основні SQL-запити. Було отримано та опрацьовано дані з таблиць клієнтів, товарів і замовлень, виконано їх фільтрацію та сортування.

У результаті роботи отримано практичні навички роботи з PostgreSQL та SQL.

**Самооцінка: 3.**

Вважаю, що основні завдання лабораторної роботи виконано та отримано достатні практичні навички для подальшої роботи з базами даних.

