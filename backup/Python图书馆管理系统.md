# Python图书馆管理系统
## 一、实验目的
1．掌握 Tkinter 库核心控件（Label、Entry、Button）的创建与属性设置；
2．理解 SQLite 数据库连接原理，掌握 book 表与 user 表的创建 SQL 语句；
3．明确图书管理系统登录模块的界面结构与数据流向；
4．了解实验报告 “概述”“需求分析” 的撰写框架。
5．掌握图书查询、增删改、借阅归还功能的 SQL 语句（SELECT/INSERT/ UPDATE/DELETE）；
6．理解 “登录验证→功能调用→数据更新” 的模块联动逻辑；
7．掌握测试用例设计方法（含测试项、预期结果、实际结果）；
8．明确实验报告 “系统设计”（含流程图）、“功能实现与测试”（含测试用例）的撰写规范。
## 二、实验项目内容（实验题目）
设计并实现一个图书管理系统，要求：（1）使用图形用户界面；（2）用数据库建立1或2个图书信息表（不限使用哪种数据库）；（3）能连接数据库并实现查询、增、删、改等功能；
## 三、运行效果示例：

<img width="375" height="294" alt="Image" src="https://github.com/user-attachments/assets/c503f14e-69e0-4c28-944c-52cdf0a34620" /> <img width="468" height="423" alt="Image" src="https://github.com/user-attachments/assets/b3b19d59-aef8-4635-a1ec-6dcf9aac31bb" />

<img width="388" height="390" alt="Image" src="https://github.com/user-attachments/assets/f18c3e31-fa87-489f-a04b-d52ea1a78a0f" /> <img width="612" height="383" alt="Image" src="https://github.com/user-attachments/assets/f45fe08d-c0f7-437d-8b3e-72744b4d3ac0" />
## 四、源程序（实验步骤/实验过程/算法）
### 1️⃣流程图：

<img width="1406" height="3136" alt="Image" src="https://github.com/user-attachments/assets/436bbe83-e593-4724-9f03-64c8520e8516" />

2️⃣源代码：
```python
import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
import datetime
def init_db():
    conn = sqlite3.connect('library.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS user
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  username TEXT UNIQUE NOT NULL,
                  password TEXT NOT NULL,
                  is_admin INTEGER DEFAULT 0)''')
    c.execute('''CREATE TABLE IF NOT EXISTS book
                 (book_id INTEGER PRIMARY KEY AUTOINCREMENT,
                  title TEXT NOT NULL,
                  author TEXT NOT NULL,
                  quantity INTEGER NOT NULL DEFAULT 0,
                  price REAL NOT NULL,
                  borrow_count INTEGER DEFAULT 0)''')
    c.execute("INSERT OR IGNORE INTO user (username, password, is_admin) VALUES ('admin', '123456', 1)")
    books = [('红楼梦', '曹雪芹', 100, 25.0),
             ('三国演义', '罗贯中', 120, 35.0),
             ('西游记', '吴承恩', 50, 52.0),
             ('神秘复苏', '佛前献花', 1000, 520.0),
             ('斗破苍穹', '天蚕土豆', 2000, 520.0),
             ('元尊', '天蚕土豆', 10, 5.0),
             ('武动乾坤', '天蚕土豆', 1, 2.0),
             ('大王饶命', '我爱吃肘子', 1, 1.0),
             ('我有一座冒险屋', '我会修空调', 1, 3.0)]
    for b in books:
        c.execute("INSERT OR IGNORE INTO book (title, author, quantity, price) VALUES (?,?,?,?)", b)
    conn.commit()
    conn.close()
class LoginWindow(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("图书管理系统 - 登录")
        self.geometry("400x300")
        self.resizable(False, False)
        style = ttk.Style(self)
        style.theme_use('clam')

        tk.Label(self, text="账号：", font=("Arial", 12)).place(x=80, y=80)
        self.username_var = tk.StringVar()
        tk.Entry(self, textvariable=self.username_var, font=("Arial", 12), width=20).place(x=140, y=80)

        tk.Label(self, text="密码：", font=("Arial", 12)).place(x=80, y=130)
        self.password_var = tk.StringVar()
        tk.Entry(self, textvariable=self.password_var, font=("Arial", 12), width=20, show="*").place(x=140, y=130)

        tk.Button(self, text="用户登录", command=self.user_login, font=("Arial", 11), width=10).place(x=90, y=180)
        tk.Button(self, text="管理员登录", command=self.admin_login, font=("Arial", 11), width=10).place(x=210, y=180)
        tk.Button(self, text="注册", command=self.open_register, font=("Arial", 11), width=22).place(x=90, y=220)

    def user_login(self):
        self.login(0)

    def admin_login(self):
        self.login(1)

    def login(self, is_admin):
        username = self.username_var.get().strip()
        password = self.password_var.get().strip()
        if not username or not password:
            messagebox.showwarning("警告", "账号和密码不能为空！")
            return
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("SELECT * FROM user WHERE username=? AND password=? AND is_admin=?", (username, password, is_admin))
        user = c.fetchone()
        conn.close()
        if user:
            messagebox.showinfo("成功", f"{'管理员' if is_admin else '用户'}登录成功！")
            self.destroy()
            MainWindow(username, is_admin)
        else:
            messagebox.showerror("错误", f"{'管理员' if is_admin else '用户'}账号或密码错误！")

    def open_register(self):
        RegisterWindow(self)
class RegisterWindow(tk.Toplevel):
    def __init__(self, parent):
        super().__init__(parent)
        self.title("用户注册")
        self.geometry("400x350")
        self.resizable(False, False)
        self.parent = parent
        tk.Label(self, text="账号：", font=("Arial", 12)).place(x=80, y=60)
        self.username_var = tk.StringVar()
        tk.Entry(self, textvariable=self.username_var, font=("Arial", 12), width=20).place(x=140, y=60)
        tk.Label(self, text="密码：", font=("Arial", 12)).place(x=80, y=110)
        self.password_var = tk.StringVar()
        tk.Entry(self, textvariable=self.password_var, font=("Arial", 12), width=20, show="*").place(x=140, y=110)
        tk.Label(self, text="确认密码：", font=("Arial", 12)).place(x=60, y=160)
        self.confirm_pwd_var = tk.StringVar()
        tk.Entry(self, textvariable=self.confirm_pwd_var, font=("Arial", 12), width=20, show="*").place(x=140, y=160)
        tk.Button(self, text="注册", command=self.register, font=("Arial", 11), width=22).place(x=90, y=220)

    def register(self):
        username = self.username_var.get().strip()
        password = self.password_var.get().strip()
        confirm_pwd = self.confirm_pwd_var.get().strip()
        if not all([username, password, confirm_pwd]):
            messagebox.showwarning("警告", "所有字段不能为空！")
            return
        if password != confirm_pwd:
            messagebox.showwarning("警告", "两次输入的密码不一致！")
            return
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        try:
            c.execute("INSERT INTO user (username, password, is_admin) VALUES (?, ?, 0)", (username, password))
            conn.commit()
            messagebox.showinfo("成功", "注册成功！请返回登录")
            self.destroy()
        except sqlite3.IntegrityError:
            messagebox.showerror("错误", "该账号已存在！")
        finally:
            conn.close()

class MainWindow(tk.Tk):
    def __init__(self, username, is_admin):
        super().__init__()
        self.title(f"图书管理系统 - {'管理员' if is_admin else '用户'}端")
        self.geometry("800x600")
        self.username = username
        self.is_admin = is_admin
        style = ttk.Style(self)
        style.theme_use('clam')

        nav_frame = tk.Frame(self, bg="#f0f0f0")
        nav_frame.pack(fill=tk.X, padx=10, pady=10)
        tk.Label(nav_frame, text=f"当前用户：{username}", bg="#f0f0f0", font=("Arial", 10)).pack(side=tk.LEFT, padx=20)
        tk.Button(nav_frame, text="退出登录", command=self.logout, font=("Arial", 10)).pack(side=tk.RIGHT, padx=20)

        notebook = ttk.Notebook(self)
        notebook.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)

        self.search_frame = ttk.Frame(notebook)
        notebook.add(self.search_frame, text="图书查询")
        self.init_search_frame()

        if is_admin:
            self.manage_frame = ttk.Frame(notebook)
            notebook.add(self.manage_frame, text="图书管理")
            self.init_manage_frame()

        self.borrow_return_frame = ttk.Frame(notebook)
        notebook.add(self.borrow_return_frame, text="借阅归还")
        self.init_borrow_return_frame()

    def logout(self):
        if messagebox.askyesno("确认", "是否要退出登录？"):
            self.destroy()
            LoginWindow()

    def init_search_frame(self):
        search_frame = tk.Frame(self.search_frame)
        search_frame.pack(pady=10)
        tk.Label(search_frame, text="书名：", font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        self.search_title_var = tk.StringVar()
        tk.Entry(search_frame, textvariable=self.search_title_var, font=("Arial", 12), width=30).pack(side=tk.LEFT)
        tk.Button(search_frame, text="查找", command=self.search_book, font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        result_frame = tk.Frame(self.search_frame)
        result_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)
        tk.Label(result_frame, text="图书信息：", font=("Arial", 12)).pack(anchor=tk.W)
        self.book_info_text = tk.Text(result_frame, font=("Arial", 11), wrap=tk.WORD)
        self.book_info_text.pack(fill=tk.BOTH, expand=True, pady=5)

    def search_book(self):
        title = self.search_title_var.get().strip()
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("SELECT * FROM book WHERE title LIKE ?", (f"%{title}%",)) if title else c.execute("SELECT * FROM book")
        books = c.fetchall()
        conn.close()
        self.book_info_text.delete(1.0, tk.END)
        if not books:
            self.book_info_text.insert(tk.END, "未查询到相关图书！")
            return
        for book in books:
            info = f"编号：{book[0]}\n书名：{book[1]}\n作者：{book[2]}\n数量：{book[3]} 本\n价格：{book[4]} 元\n\n"
            self.book_info_text.insert(tk.END, info)

    def init_manage_frame(self):
        form_frame = tk.Frame(self.manage_frame)
        form_frame.pack(pady=10)
        labels = ["书名：", "作者：", "数量：", "价格："]
        self.manage_vars = [tk.StringVar() for _ in range(4)]
        for i, (label, var) in enumerate(zip(labels, self.manage_vars)):
            tk.Label(form_frame, text=label, font=("Arial", 12)).grid(row=0, column=i*2, padx=5, pady=5)
            tk.Entry(form_frame, textvariable=var, font=("Arial", 12), width=15).grid(row=0, column=i*2+1, padx=5, pady=5)
        btn_frame = tk.Frame(self.manage_frame)
        btn_frame.pack(pady=10)
        tk.Button(btn_frame, text="添加图书", command=self.add_book, font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        tk.Button(btn_frame, text="修改图书", command=self.update_book, font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        tk.Button(btn_frame, text="删除图书", command=self.delete_book, font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        list_frame = tk.Frame(self.manage_frame)
        list_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)
        self.book_list_box = tk.Listbox(list_frame, font=("Arial", 11), width=80)
        self.book_list_box.pack(fill=tk.BOTH, expand=True, side=tk.LEFT)
        tk.Scrollbar(list_frame, command=self.book_list_box.yview).pack(fill=tk.Y, side=tk.RIGHT)
        self.load_book_list()

    def load_book_list(self):
        self.book_list_box.delete(0, tk.END)
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("SELECT * FROM book")
        for book in c.fetchall():
            self.book_list_box.insert(tk.END, f"编号：{book[0]} | 书名：{book[1]} | 作者：{book[2]} | 数量：{book[3]} | 价格：{book[4]}")
        conn.close()

    def add_book(self):
        title, author, quantity, price = [v.get().strip() for v in self.manage_vars]
        if not all([title, author, quantity, price]):
            messagebox.showwarning("警告", "所有字段不能为空！")
            return
        try:
            quantity = int(quantity)
            price = float(price)
        except ValueError:
            messagebox.showwarning("警告", "数量必须为整数，价格必须为数字！")
            return
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("INSERT INTO book (title, author, quantity, price) VALUES (?, ?, ?, ?)", (title, author, quantity, price))
        conn.commit()
        conn.close()
        messagebox.showinfo("成功", "图书添加成功！")
        self.load_book_list()
        for var in self.manage_vars:
            var.set("")

    def update_book(self):
        selected = self.book_list_box.curselection()
        if not selected:
            messagebox.showwarning("警告", "请选择要修改的图书！")
            return
        book_id = self.book_list_box.get(selected[0]).split(" | ")[0].split("：")[1]
        title, author, quantity, price = [v.get().strip() for v in self.manage_vars]
        if not all([title, author, quantity, price]):
            messagebox.showwarning("警告", "所有字段不能为空！")
            return
        try:
            quantity = int(quantity)
            price = float(price)
        except ValueError:
            messagebox.showwarning("警告", "数量必须为整数，价格必须为数字！")
            return
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("UPDATE book SET title=?, author=?, quantity=?, price=? WHERE book_id=?", (title, author, quantity, price, book_id))
        conn.commit()
        conn.close()
        messagebox.showinfo("成功", "图书修改成功！")
        self.load_book_list()
        for var in self.manage_vars:
            var.set("")

    def delete_book(self):
        selected = self.book_list_box.curselection()
        if not selected:
            messagebox.showwarning("警告", "请选择要删除的图书！")
            return
        if not messagebox.askyesno("确认", "是否要删除该图书？"):
            return
        book_id = self.book_list_box.get(selected[0]).split(" | ")[0].split("：")[1]
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("DELETE FROM book WHERE book_id=?", (book_id,))
        conn.commit()
        conn.close()
        messagebox.showinfo("成功", "图书删除成功！")
        self.load_book_list()

    def init_borrow_return_frame(self):
        borrow_frame = tk.Frame(self.borrow_return_frame)
        borrow_frame.pack(pady=10)
        tk.Label(borrow_frame, text="借阅图书（输入图书编号）：", font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        self.borrow_book_id_var = tk.StringVar()
        tk.Entry(borrow_frame, textvariable=self.borrow_book_id_var, font=("Arial", 12), width=10).pack(side=tk.LEFT)
        tk.Button(borrow_frame, text="借阅", command=self.borrow_book, font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        return_frame = tk.Frame(self.borrow_return_frame)
        return_frame.pack(pady=10)
        tk.Label(return_frame, text="归还图书（输入图书编号）：", font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        self.return_book_id_var = tk.StringVar()
        tk.Entry(return_frame, textvariable=self.return_book_id_var, font=("Arial", 12), width=10).pack(side=tk.LEFT)
        tk.Button(return_frame, text="归还", command=self.return_book, font=("Arial", 12)).pack(side=tk.LEFT, padx=10)
        record_frame = tk.Frame(self.borrow_return_frame)
        record_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=5)
        tk.Label(record_frame, text="借阅记录：", font=("Arial", 12)).pack(anchor=tk.W)
        self.record_text = tk.Text(record_frame, font=("Arial", 11), wrap=tk.WORD)
        self.record_text.pack(fill=tk.BOTH, expand=True, pady=5)

    def borrow_book(self):
        book_id = self.borrow_book_id_var.get().strip()
        if not book_id:
            messagebox.showwarning("警告", "图书编号不能为空！")
            return
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("SELECT * FROM book WHERE book_id=?", (book_id,))
        book = c.fetchone()
        if not book:
            conn.close()
            messagebox.showerror("错误", "该图书不存在！")
            return
        if book[3] <= 0:
            conn.close()
            messagebox.showwarning("警告", "该图书已无库存！")
            return
        c.execute("UPDATE book SET quantity=quantity-1, borrow_count=borrow_count+1 WHERE book_id=?", (book_id,))
        conn.commit()
        conn.close()
        messagebox.showinfo("成功", f"借阅《{book[1]}》成功！请按时归还")
        self.record_text.insert(tk.END, f"[{datetime.datetime.now():%Y-%m-%d %H:%M:%S}] 借阅图书：{book[1]}（编号：{book_id}）\n")
        self.borrow_book_id_var.set("")

    def return_book(self):
        book_id = self.return_book_id_var.get().strip()
        if not book_id:
            messagebox.showwarning("警告", "图书编号不能为空！")
            return
        conn = sqlite3.connect('library.db')
        c = conn.cursor()
        c.execute("SELECT * FROM book WHERE book_id=?", (book_id,))
        book = c.fetchone()
        if not book:
            conn.close()
            messagebox.showerror("错误", "该图书不存在！")
            return
        c.execute("UPDATE book SET quantity=quantity+1, borrow_count=borrow_count-1 WHERE book_id=?", (book_id,))
        conn.commit()
        conn.close()
        messagebox.showinfo("成功", f"归还《{book[1]}》成功！")
        self.record_text.insert(tk.END, f"[{datetime.datetime.now():%Y-%m-%d %H:%M:%S}] 归还图书：{book[1]}（编号：{book_id}）\n")
        self.return_book_id_var.set("")

if __name__ == "__main__":
    init_db()
    LoginWindow().mainloop()
```