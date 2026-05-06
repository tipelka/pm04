import sys,pymysql,os
from PyQt6.QtWidgets import *
from PyQt6.QtCore import Qt
from PyQt6.QtGui import QPixmap
from functools import partial

DB={'host':'localhost','user':'root',
    'password':'Tipelka.006',
    'database':'clothing_store',
    'cursorclass':pymysql.cursors.DictCursor}
def db(): return pymysql.connect(**DB)

class Card(QFrame):
    def __init__(s,p,buy,ed,dl,bf,ef,df):
        super().__init__()
        s.setStyleSheet("background:white;border:1px solid #ccc;")
        l=QHBoxLayout(s); l.setContentsMargins(10,10,10,10)
        i=QLabel(); i.setFixedSize(80,80)
        i.setStyleSheet("background:#f0f0f0;")
        i.setAlignment(Qt.AlignmentFlag.AlignCenter)
        if p.get('image_path') and os.path.exists(p['image_path']):
            pm=QPixmap(p['image_path'])
            pm=pm.scaled(80,80,Qt.AspectRatioMode.KeepAspectRatio)
            i.setPixmap(pm)
        else: i.setText("НЕТ ФОТО")
        l.addWidget(i)
        v=QVBoxLayout()
        t=QLabel(f"<b>{p['name']}</b>"); t.setStyleSheet("color:black;")
        v.addWidget(t)
        if p.get('description'):
            d=QLabel(p['description'][:50])
            d.setStyleSheet("color:#555;"); v.addWidget(d)
        m=QLabel(f"Производитель: {p.get('manufacturer','-')}")
        m.setStyleSheet("color:#777;"); v.addWidget(m)
        l.addLayout(v,2)
        a=QVBoxLayout()
        a.addWidget(QLabel(f"<b style='color:#dc3545'>{float(p['price'])} ₽</b>"))
        if buy and bf:
            btn=QPushButton("В корзину")
            btn.setStyleSheet("background:#28a745;color:white;")
            btn.clicked.connect(bf); a.addWidget(btn)
        if ed or dl:
            r=QHBoxLayout()
            if ed and ef:
                btn=QPushButton("✏️")
                btn.setStyleSheet("background:#ffc107;color:black;")
                btn.clicked.connect(ef); r.addWidget(btn)
            if dl and df:
                btn=QPushButton("🗑️")
                btn.setStyleSheet("background:#dc3545;color:white;")
                btn.clicked.connect(df); r.addWidget(btn)
            a.addLayout(r)
        l.addLayout(a)

class Main(QMainWindow):
    def __init__(s,u):
        super().__init__()
        s.user=u; s.role=u['role_id']; s.cart=[]
        s.setWindowTitle(f"Магазин - {u['username']}")
        s.resize(1200,750); s.tabs=QTabWidget()
        s.setCentralWidget(s.tabs)
        c=QWidget(); s.tabs.addTab(c,"📦 КАТАЛОГ")
        s.setup_catalog(c)
        if s.role==1 or s.role==3:
            o=QWidget(); s.tabs.addTab(o,"📋 ЗАКАЗЫ")
            s.setup_orders(o)
    def setup_catalog(s,p):
        l=QVBoxLayout(p)
        s.srch=QLineEdit()
        s.srch.setPlaceholderText("🔍 Поиск...")
        s.srch.setStyleSheet("background:white;color:black;padding:8px;")
        s.srch.textChanged.connect(s.load); l.addWidget(s.srch)
        s.scroll=QScrollArea(); s.scroll.setWidgetResizable(True)
        w=QWidget(); s.grid=QVBoxLayout(w)
        s.scroll.setWidget(w); l.addWidget(s.scroll)
        if s.role==3:
            btn=QPushButton("🛒 КОРЗИНА")
            btn.setStyleSheet("background:#28a745;color:white;padding:12px;")
            btn.clicked.connect(s.show_cart); l.addWidget(btn)
        s.load()
    def setup_orders(s,p):
        l=QVBoxLayout(p)
        s.orders=QTableWidget()
        s.orders.setColumnCount(5)
        s.orders.setHorizontalHeaderLabels(
            ["ID","Дата","Сумма","Статус","Адрес"])
        s.orders.setStyleSheet("background:white;color:black;")
        l.addWidget(s.orders)
        btn=QPushButton("🔄 ОБНОВИТЬ")
        btn.setStyleSheet("background:#0078d7;color:white;padding:8px;")
        btn.clicked.connect(s.refresh_orders); l.addWidget(btn)
        s.refresh_orders()
    def refresh_orders(s):
        c=db(); cur=c.cursor()
        if s.role==3:
            uid=s.user['user_id']
            cur.execute(f"SELECT * FROM orders WHERE user_id={uid} ORDER BY order_date DESC")
        else:
            cur.execute("SELECT * FROM orders ORDER BY order_date DESC")
        o=cur.fetchall(); c.close()
        s.orders.setRowCount(len(o))
        for i,r in enumerate(o):
            s.orders.setItem(i,0,QTableWidgetItem(str(r['order_id'])))
            s.orders.setItem(i,1,QTableWidgetItem(str(r['order_date'])[:19]))
            s.orders.setItem(i,2,QTableWidgetItem(f"{float(r['total_amount'])} ₽"))
            s.orders.setItem(i,3,QTableWidgetItem(r['status']))
            s.orders.setItem(i,4,QTableWidgetItem(r.get('delivery_address','-')))
        s.orders.resizeColumnsToContents()
    def load(s):
        # Очистка
        while s.grid.count():
            i=s.grid.takeAt(0)
            if i.widget():
                i.widget().deleteLater()
            elif i.layout():
                while i.layout().count():
                    j=i.layout().takeAt(0)
                    if j.widget():
                        j.widget().deleteLater()
        # Загрузка из БД
        c=db()
        if not c: return
        cur=c.cursor()
        t=s.srch.text().strip()
        if t:
            cur.execute(f"SELECT * FROM products WHERE name LIKE '%{t}%'")
        else:
            cur.execute("SELECT * FROM products")
        p=cur.fetchall()
        c.close()
        # Создание карточек
        r=QHBoxLayout()
        for i,q in enumerate(p):
            buy=(s.role==3)
            ed=(s.role==1)
            dl=(s.role==1)
            bf=partial(s.add_cart,q) if buy else None
            ef=partial(s.edit,q) if ed else None
            df=partial(s.del_p,q) if dl else None
            r.addWidget(Card(q,buy,ed,dl,bf,ef,df))
            if(i+1)%2==0:
                s.grid.addLayout(r)
                r=QHBoxLayout()
        if r.count():
            s.grid.addLayout(r)
        s.grid.addStretch()
    def add_cart(s,p):
        d=QDialog(s); d.setWindowTitle("В корзину")
        d.resize(300,180); d.setStyleSheet("background:white;")
        l=QVBoxLayout(d)
        l.addWidget(QLabel(f"<b>{p['name']}</b>"))
        q=QSpinBox(); q.setMinimum(1); q.setMaximum(99)
        q.setStyleSheet("background:white;color:black;")
        l.addWidget(QLabel("Количество:")); l.addWidget(q)
        def add():
            s.cart.append({'product':p,'qty':q.value(),
                          'total':float(p['price'])*q.value()})
            d.accept()
        btn=QPushButton("Добавить")
        btn.setStyleSheet("background:#28a745;color:white;")
        btn.clicked.connect(add); l.addWidget(btn); d.exec()
    def edit(s,p):
        d=QDialog(s); d.setWindowTitle("Редактирование")
        d.resize(400,400); d.setStyleSheet("background:white;")
        l=QVBoxLayout(d)
        n=QLineEdit(p['name']); pr=QLineEdit(str(p['price']))
        ds=QTextEdit(); ds.setText(p.get('description',''))
        m=QLineEdit(p.get('manufacturer',''))
        for t,w in [("Название:",n),("Цена:",pr),
                    ("Описание:",ds),("Производитель:",m)]:
            l.addWidget(QLabel(f"<b>{t}</b>"))
            w.setStyleSheet("padding:8px;background:white;"
                           "color:black;border:1px solid #ccc;")
            l.addWidget(w)
        def save():
            c=db(); cur=c.cursor()
            cur.execute(f"UPDATE products SET name='{n.text()}',"
                       f"price={float(pr.text())},"
                       f"description='{ds.toPlainText()}',"
                       f"manufacturer='{m.text()}' "
                       f"WHERE product_id={p['product_id']}")
            c.commit(); c.close(); s.load(); d.accept()
        btn=QPushButton("Сохранить")
        btn.setStyleSheet("background:#28a745;color:white;padding:10px;")
        btn.clicked.connect(save); l.addWidget(btn); d.exec()
    def del_p(s,p):
        c=db(); cur=c.cursor()
        cur.execute(f"DELETE FROM products WHERE product_id={p['product_id']}")
        c.commit(); c.close(); s.load()
    def show_cart(s):
        if not s.cart:
            QMessageBox.information(s,"Корзина","Пуста")
            return
        t=sum(i['total'] for i in s.cart)
        d=QDialog(s); d.setWindowTitle("Корзина")
        d.resize(400,400); d.setStyleSheet("background:white;")
        l=QVBoxLayout(d); lst=QListWidget()
        lst.setStyleSheet("background:white;color:black;")
        for i in s.cart:
            lst.addItem(f"{i['product']['name']} x{i['qty']} = {i['total']} ₽")
        l.addWidget(lst); l.addWidget(QLabel(f"<b>ИТОГО: {t} ₽</b>"))
        a=QLineEdit(); a.setPlaceholderText("Адрес доставки")
        a.setStyleSheet("background:white;color:black;padding:8px;"
                       "border:1px solid #ccc;")
        l.addWidget(a)
        def cf():
            if not a.text():
                QMessageBox.warning(d,"Ошибка","Укажите адрес!")
                return
            c=db(); cur=c.cursor()
            uid=s.user['user_id'] if s.user['user_id'] else 'NULL'
            cur.execute(f"INSERT INTO orders (user_id,total_amount,"
                       f"status,delivery_address) VALUES ({uid},{t},"
                       f"'Новый','{a.text()}')")
            oid=cur.lastrowid
            for i in s.cart:
                cur.execute(f"INSERT INTO order_items (order_id,product_id,"
                           f"quantity,price_per_item) VALUES ({oid},"
                           f"{i['product']['product_id']},{i['qty']},"
                           f"{i['product']['price']})")
            c.commit(); c.close(); s.cart.clear(); d.accept()
            if hasattr(s,'refresh_orders'):
                s.refresh_orders()
        btn=QPushButton("Подтвердить")
        btn.setStyleSheet("background:#28a745;color:white;padding:10px;")
        btn.clicked.connect(cf); l.addWidget(btn); d.exec()

class Login(QDialog):
    def __init__(s):
        super().__init__(); s.setWindowTitle("Вход")
        s.resize(400,300); s.setStyleSheet("background:#f0f2f5;")
        l=QVBoxLayout(s)
        title=QLabel("<h1 style='color:#333;'>👕 МАГАЗИН</h1>")
        title.setAlignment(Qt.AlignmentFlag.AlignCenter)
        l.addWidget(title)
        s.login=QLineEdit(); s.login.setPlaceholderText("Логин")
        s.login.setStyleSheet("background:white;color:black;"
                             "padding:8px;border:1px solid #ccc;")
        s.pwd=QLineEdit(); s.pwd.setPlaceholderText("Пароль")
        s.pwd.setEchoMode(QLineEdit.EchoMode.Password)
        s.pwd.setStyleSheet("background:white;color:black;"
                           "padding:8px;border:1px solid #ccc;")
        l.addWidget(s.login); l.addWidget(s.pwd)
        btn=QPushButton("ВОЙТИ")
        btn.setStyleSheet("background:#0078d7;color:white;padding:10px;")
        btn.clicked.connect(s.login_user); l.addWidget(btn)
        btn2=QPushButton("ГОСТЬ")
        btn2.setStyleSheet("background:#6c757d;color:white;padding:10px;")
        btn2.clicked.connect(s.guest_mode); l.addWidget(btn2)
        s.user_data=None
    def login_user(s):
        c=db(); cur=c.cursor()
        cur.execute(f"SELECT user_id,role_id,username FROM users "
                   f"WHERE username='{s.login.text()}' "
                   f"AND password_hash='{s.pwd.text()}'")
        s.user_data=cur.fetchone(); c.close()
        if s.user_data:
            s.accept()
        else:
            QMessageBox.warning(s,"Ошибка","Неверный логин/пароль")
    def guest_mode(s):
        s.user_data={'user_id':None,'role_id':0,'username':'Гость'}
        s.accept()

if __name__=="__main__":
    app=QApplication(sys.argv)
    login=Login()
    if login.exec()==QDialog.DialogCode.Accepted:
        window=Main(login.user_data)
        window.show()
    sys.exit(app.exec())



CREATE DATABASE IF NOT EXISTS clothing_store;
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
