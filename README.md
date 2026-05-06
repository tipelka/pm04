import sys, pymysql, os
from PyQt6.QtWidgets import *
from PyQt6.QtCore import Qt
from PyQt6.QtGui import QPixmap

DB = {'host': 'localhost', 'user': 'root', 'password': 'Tipelka.006', 'database': 'clothing_store',
      'cursorclass': pymysql.cursors.DictCursor}


def db(): return pymysql.connect(**DB)


class Main(QMainWindow):
    def __init__(s):
        super().__init__()
        s.setWindowTitle("Магазин")
        s.resize(1200, 750)
        w = QWidget()
        s.setCentralWidget(w)
        l = QVBoxLayout(w)

        s.srch = QLineEdit()
        s.srch.setPlaceholderText("🔍 Поиск...")
        s.srch.textChanged.connect(s.load)
        l.addWidget(s.srch)

        s.scroll = QScrollArea()
        s.scroll.setWidgetResizable(True)
        w2 = QWidget()

        s.grid = QVBoxLayout(w2)
        s.scroll.setWidget(w2)
        l.addWidget(s.scroll)
        s.load()

    def load(s):
        while s.grid.count():
            i = s.grid.takeAt(0)
            if i.widget(): i.widget().deleteLater()
        c = db()
        cur = c.cursor()
        t = s.srch.text().strip()

        cur.execute(f"SELECT * FROM products WHERE name LIKE '%{t}%'" if t else "SELECT * FROM products")
        p = cur.fetchall()
        c.close()

        r = QHBoxLayout()
        for i, q in enumerate(p):
            f = QFrame()
            f.setStyleSheet("background:#f9f9f9;border:1px solid #ddd;padding:10px;")
            lr = QVBoxLayout(f)

            img = QLabel()
            img.setFixedSize(80, 80)
            img.setAlignment(Qt.AlignmentFlag.AlignCenter)
            img.setStyleSheet("background:#e0e0e0;")

            if q.get('image_path') and os.path.exists(q['image_path']):
                pm = QPixmap(q['image_path'])
                pm = pm.scaled(80, 80, Qt.AspectRatioMode.KeepAspectRatio)
                img.setPixmap(pm)

            else:
                img.setText("НЕТ ФОТО")
            lr.addWidget(img)
            lr.addWidget(QLabel(f"<b style='color:#000;'>{q['name']}</b>"))
            lr.addWidget(QLabel(f"<span style='color:#555;'>{q.get('manufacturer', '-')}</span>"))
            lr.addWidget(QLabel(f"<b style='color:#dc3545;'>{float(q['price'])} ₽</b>"))

            btn = QPushButton("В корзину")
            btn.setStyleSheet("background:#28a745;color:white;")
            btn.clicked.connect(lambda ch, x=q: QMessageBox.information(s, "Выбрано", f"Товар {x['name']} добавлен"))

            lr.addWidget(btn)
            r.addWidget(f)
            if (i + 1) % 2 == 0:
                s.grid.addLayout(r)
                r = QHBoxLayout()

        if r.count():
            s.grid.addLayout(r)
        s.grid.addStretch()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = Main()
    window.show()
    sys.exit(app.exec())


import sys, pymysql, os
from PyQt6.QtWidgets import *
from PyQt6.QtCore import Qt
from PyQt6.QtGui import QPixmap

DB = {'host': 'localhost', 'user': 'root', 'password': 'Tipelka.006', 'database': 'clothing_store',
      'cursorclass': pymysql.cursors.DictCursor}


def db(): return pymysql.connect(**DB)


class Main(QMainWindow):
    def __init__(s):
        super().__init__()
        s.setWindowTitle("Магазин")
        s.resize(1200, 750)
        w = QWidget()
        s.setCentralWidget(w)
        l = QVBoxLayout(w)

        s.srch = QLineEdit()
        s.srch.setPlaceholderText("🔍 Поиск...")
        s.srch.textChanged.connect(s.load)
        l.addWidget(s.srch)

        s.scroll = QScrollArea()
        s.scroll.setWidgetResizable(True)
        w2 = QWidget()

        s.grid = QVBoxLayout(w2)
        s.scroll.setWidget(w2)
        l.addWidget(s.scroll)
        s.load()

    def load(s):
        while s.grid.count():
            i = s.grid.takeAt(0)
            if i.widget(): i.widget().deleteLater()
        c = db()
        cur = c.cursor()
        t = s.srch.text().strip()

        cur.execute(f"SELECT * FROM products WHERE name LIKE '%{t}%'" if t else "SELECT * FROM products")
        p = cur.fetchall()
        c.close()

        r = QHBoxLayout()
        for i, q in enumerate(p):
            f = QFrame()
            f.setStyleSheet("background:#f9f9f9;border:1px solid #ddd;padding:10px;")
            lr = QVBoxLayout(f)

            img = QLabel()
            img.setFixedSize(80, 80)
            img.setAlignment(Qt.AlignmentFlag.AlignCenter)
            img.setStyleSheet("background:#e0e0e0;")

            if q.get('image_path') and os.path.exists(q['image_path']):
                pm = QPixmap(q['image_path'])
                pm = pm.scaled(80, 80, Qt.AspectRatioMode.KeepAspectRatio)
                img.setPixmap(pm)

            else:
                img.setText("НЕТ ФОТО")
            lr.addWidget(img)
            lr.addWidget(QLabel(f"<b style='color:#000;'>{q['name']}</b>"))
            lr.addWidget(QLabel(f"<span style='color:#555;'>{q.get('manufacturer', '-')}</span>"))
            lr.addWidget(QLabel(f"<b style='color:#dc3545;'>{float(q['price'])} ₽</b>"))

            btn = QPushButton("В корзину")
            btn.setStyleSheet("background:#28a745;color:white;")
            btn.clicked.connect(lambda ch, x=q: QMessageBox.information(s, "Выбрано", f"Товар {x['name']} добавлен"))

            lr.addWidget(btn)
            r.addWidget(f)
            if (i + 1) % 2 == 0:
                s.grid.addLayout(r)
                r = QHBoxLayout()

        if r.count():
            s.grid.addLayout(r)
        s.grid.addStretch()


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = Main()
    window.show()
    sys.exit(app.exec())

исправь бд для этого кода-чтобы там было только нужное CREATE DATABASE IF NOT EXISTS clothing_store;
USE clothing_store;

CREATE TABLE roles (
    role_id INT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role_id INT,
    FOREIGN KEY (role_id) REFERENCES roles(role_id) ON DELETE SET NULL
);

CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    image_path VARCHAR(255),
    manufacturer VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10, 2) DEFAULT 0.00,
    status VARCHAR(50) DEFAULT 'Новый',
    delivery_address TEXT,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE SET NULL
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price_per_item DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE
);

-- Вставка ролей (только те, что используются: 1-админ, 3-клиент)
INSERT INTO roles VALUES 
(1, 'Администратор'),
(3, 'Клиент');

-- Вставка пользователей
INSERT INTO users (user_id, username, password_hash, role_id) VALUES 
(1, 'admin', 'admin', 1),
(2, 'manager', 'manager', 1),  -- менеджер тоже как админ (role_id=1)
(3, 'client', 'client', 3);

-- Вставка товаров (без category_id, supplier, unit, stock_quantity)
INSERT INTO products (product_id, name, description, price, image_path, manufacturer) VALUES
(1, 'Футболка белая', 'Классическая футболка из хлопка', 1200.00, 'images/tshirt.jpg', 'Zara'),
(2, 'Джинсы синие', 'Прямые джинсы', 3500.00, 'images/jeans.jpg', 'Levis'),
(3, 'Кроссовки', 'Спортивные кроссовки', 4500.00, 'images/sneakers.jpg', 'Nike'),
(4, 'Куртка', 'Осенняя куртка', 6500.00, 'images/jacket.jpg', 'Adidas'),
(5, 'Ремень', 'Кожаный ремень', 1500.00, 'images/belt.jpg', 'Gucci'),
(6, 'Кепка', 'Спортивная кепка', 800.00, 'images/cap.jpg', 'Puma');
