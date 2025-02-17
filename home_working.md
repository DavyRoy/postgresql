# Postgres

---

### Задание 1 - Введение в PostgreSQL

1. `Создай базу для интернет-магазина с таблицами users, products, orders, reviews. Используй внешние ключи, представления, триггеры и индексы.`

``` ~Решение~
1.
-- Таблица пользователей
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица товаров
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) CHECK (price >= 0),
    stock INT CHECK (stock >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица заказов
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    total_price DECIMAL(10,2) CHECK (total_price >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица товаров в заказах (связь many-to-many)
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id) ON DELETE CASCADE,
    quantity INT CHECK (quantity > 0),
    price DECIMAL(10,2) CHECK (price >= 0)
);

-- Таблица отзывов
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id) ON DELETE CASCADE,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
2.
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_products_price ON products(price);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_reviews_product_id ON reviews(product_id);
3.
CREATE VIEW sales_report AS
SELECT 
    o.id AS order_id,
    u.name AS customer,
    SUM(oi.price * oi.quantity) AS total_spent,
    o.created_at
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id, u.name, o.created_at;
4.
CREATE OR REPLACE FUNCTION check_order_total()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.total_price < 0 THEN
        RAISE EXCEPTION 'Итоговая сумма заказа не может быть отрицательной';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_check_order_total
BEFORE INSERT OR UPDATE ON orders
FOR EACH ROW
EXECUTE FUNCTION check_order_total();
5.
CREATE INDEX idx_reviews_text_search ON reviews USING GIN(to_tsvector('russian', comment));

SELECT * FROM reviews WHERE to_tsvector('russian', comment) @@ to_tsquery('интересно');

---