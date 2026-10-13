# Лабораторна робота №2. Створення складних SQL-запитів

**Здобувач освіти:** Чижук Михайло  
**Група:** ІПЗ 32  
**Обраний рівень:** 1  
**СУБД:** PostgreSQL (Supabase)

## Мета роботи

Попрактикуватися у з'єднанні таблиць, групуванні даних і написанні підзапитів. Додатково перевірити кілька віконних і рекурсивних запитів.

## Виконання роботи

### 1. З'єднання таблиць

#### 1.1. Товари, категорії та постачальники

```sql
SELECT
	p.product_name,
	c.category_name,
	s.company_name,
	p.unit_price
FROM products p
JOIN categories c ON p.category_id = c.category_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
ORDER BY c.category_name, p.product_name;
```

У результаті для кожного товару видно його категорію, постачальника та ціну. На скриншоті, наприклад, показані товари категорії «Аксесуари та комплектуючі».

Скріншот: [запит і результат](<screenshots/Знімок екрана 2026-10-13 155613.png>).

#### 1.2. Клієнти та їхні замовлення

```sql
SELECT
	c.contact_name,
	c.customer_type,
	r.region_name,
	COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.contact_name, c.customer_type, r.region_name
ORDER BY order_count DESC;
```

`LEFT JOIN` залишає у вибірці клієнтів навіть без замовлень. Кількість рахується через `COUNT`.

Скріншот: [запит і результат](<screenshots/Знімок екрана 2026-10-13 155607.png>).

#### 1.3. Деталі замовлень

```sql
SELECT
	o.order_id,
	o.order_date,
	c.contact_name,
	p.product_name,
	cat.category_name,
	oi.quantity,
	oi.unit_price,
	oi.quantity * oi.unit_price AS item_total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
ORDER BY o.order_date DESC, o.order_id;
```

Тут об'єднано п'ять таблиць. У результаті відображаються замовлення, клієнти, товари та сума кожної позиції без урахування знижки. Запит повернув 47 рядків.

Скріншот: [запит і результат](<screenshots/Знімок екрана 2026-10-13 155601.png>).

### 2. Агрегатні функції

#### 2.1. Кількість і ціни товарів за категоріями

```sql
SELECT
	c.category_name,
	COUNT(p.product_id) AS product_count,
	SUM(p.unit_price) AS total_price,
	AVG(p.unit_price) AS avg_price,
	MIN(p.unit_price) AS min_price,
	MAX(p.unit_price) AS max_price
FROM categories c
LEFT JOIN products p ON c.category_id = p.category_id
GROUP BY c.category_id, c.category_name
ORDER BY product_count DESC;
```

Для кожної категорії виводиться кількість товарів і статистика їхніх цін. Для категорій без товарів кількість дорівнює нулю, а інші агрегатні значення відображаються як `NULL`.

Скріншот: [запит і результат](<screenshots/Знімок екрана 2026-10-13 155552.png>).

#### 2.2. Продажі за регіонами

```sql
SELECT
	r.region_name,
	COUNT(DISTINCT o.order_id) AS orders_count,
	SUM(oi.quantity * oi.unit_price) AS total_sales
FROM regions r
JOIN customers c ON c.region_id = r.region_id
JOIN orders o ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY r.region_id, r.region_name
HAVING SUM(oi.quantity * oi.unit_price) > 0
ORDER BY total_sales DESC;
```

У результаті показано кількість замовлень і загальну суму позицій у кожному регіоні. `HAVING` відбирає групи після підрахунку суми.

Скріншот: [результат](<screenshots/Знімок екрана 2026-10-13 155544.png>).

#### 2.3. Постачальники з більш ніж двома товарами

```sql
SELECT
	s.supplier_id,
	s.company_name,
	COUNT(p.product_id) AS product_count
FROM suppliers s
JOIN products p ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.company_name
HAVING COUNT(p.product_id) > 2
ORDER BY product_count DESC;
```

У вибірці залишилися чотири постачальники; найбільше товарів на скриншоті має ТОВ «Електроніка Плюс» — 6.

Скріншот: [результат](<screenshots/Знімок екрана 2026-10-13 155541.png>).

### 3. Підзапити

#### 3.1. Товари дорожчі за середню ціну у своїй категорії

```sql
SELECT
	p.product_name,
	p.unit_price,
	c.category_name
FROM products p
JOIN categories c ON p.category_id = c.category_id
WHERE p.unit_price > (
	SELECT AVG(p2.unit_price)
	FROM products p2
	WHERE p2.category_id = p.category_id
)
ORDER BY c.category_name, p.unit_price DESC;
```

Підзапит рахує середню ціну для категорії поточного товару. У результаті залишаються товари, ціна яких вища за це середнє значення.

Скріншот: [результат](<screenshots/Знімок екрана 2026-10-13 155548.png>).

#### 3.2. Клієнти із замовленнями у 2024 році

```sql
SELECT
	c.customer_id,
	c.contact_name,
	c.customer_type
FROM customers c
WHERE c.customer_id IN (
	SELECT o.customer_id
	FROM orders o
	WHERE o.order_date >= DATE '2024-01-01'
	  AND o.order_date < DATE '2025-01-01'
)
ORDER BY c.contact_name;
```

Запит повертає клієнтів, для яких знайшлося хоча б одне замовлення у вказаному проміжку дат.

Той самий період можна перевірити через `LEFT JOIN`, додавши умову за датою до `ON`. Цей варіант показує клієнтів разом із датами їхніх замовлень за 2024 рік.

Скріншоти: [підзапит із `IN`](<screenshots/Знімок екрана 2026-10-13 155537.png>), [варіант із `LEFT JOIN`](<screenshots/Знімок екрана 2026-10-13 155516.png>).

#### 3.3. Кількість проданих одиниць товару

```sql
SELECT
	p.product_id,
	p.product_name,
	p.unit_price,
	COALESCE((
		SELECT SUM(oi.quantity)
		FROM order_items oi
		WHERE oi.product_id = p.product_id
	), 0) AS total_sold
FROM products p
ORDER BY total_sold DESC, p.product_name;
```

Для кожного товару підраховано продану кількість. `COALESCE` показує 0, якщо для товару немає позицій у замовленнях.

Скріншот: [результат](<screenshots/Знімок екрана 2026-10-13 155533.png>).

### 4. Додаткові запити

#### 4.1. Категорії та товари

Перевірив `FULL OUTER JOIN`, щоб отримати рядки з обох таблиць, навіть якщо відповідної пари немає. Також окремо використав `RIGHT JOIN` і порахував для категорій кількість товарів та середню ціну.

Скріншоти: [FULL OUTER JOIN](<screenshots/Знімок екрана 2026-10-13 155524.png>), [RIGHT JOIN та агрегати](<screenshots/Знімок екрана 2026-10-13 155528.png>).

#### 4.2. Співробітники та керівники

Таблиця `employees` з'єднана сама з собою за полями `reports_to` і `employee_id`. Це дає змогу вивести співробітника поруч із його керівником. Для генерального директора керівник не зазначений.

Скріншот: [результат](<screenshots/Знімок екрана 2026-10-13 155520.png>).

#### 4.3. Віконні функції

Для товарів обчислив номер рядка та ранг ціни в межах категорії за допомогою `ROW_NUMBER`, `RANK` і `DENSE_RANK`. В іншому запиті порівняв ціну товару із середньою та максимальною ціною категорії.

Скріншоти: [ранжування](<screenshots/Знімок екрана 2026-10-13 155510.png>), [порівняння із середнім](<screenshots/Знімок екрана 2026-10-13 155457.png>).

Для замовлень також використав `LAG` і `LEAD`, щоб показати попередню та наступну дату замовлення клієнта. Для першого замовлення попередньої дати немає, тому результат — `NULL`.

Скріншот: [дати замовлень](<screenshots/Знімок екрана 2026-10-13 155504.png>).

#### 4.4. Ієрархія співробітників

Рекурсивний CTE починає з працівника без керівника, а потім знаходить його підлеглих. У результаті видно рівень працівника та шлях від керівника до нього.

Скріншот: [результат рекурсивного запиту](<screenshots/Знімок екрана 2026-10-13 155444.png>).

### 5. Перевірка плану виконання

Для перегляду плану запустив `EXPLAIN (ANALYZE, BUFFERS)` для запиту з'єднання замовлень, клієнтів і позицій замовлень та групування продажів за клієнтами. У показаному плані загальний час був близько 2 мс. Це значення стосується мого запуску в Supabase і може змінюватися.

Скріншот: [план виконання](<screenshots/Знімок екрана 2026-10-13 155432.png>).

## Висновок

Під час роботи я потренувався з'єднувати таблиці, групувати записи та рахувати агрегатні значення. Також перевірив підзапити, кілька типів `JOIN`, віконні функції, рекурсивний запит і план виконання. Результати запусків збережені у папці `screenshots`.
