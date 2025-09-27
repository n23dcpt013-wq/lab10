 Final Report – ATM Mini Project

 1. Giới thiệu
Dự án ATM Mini Project nhằm mô phỏng hệ thống máy rút tiền tự động (ATM) cơ bản, giúp sinh viên thực hành toàn bộ quy trình phát triển phần mềm:


 Phân tích yêu cầu & thiết kế UML  
 Thiết kế cơ sở dữ liệu & lập trình module  
 Kiểm thử phần mềm  
 Quản lý dự án bằng Jira (Scrum)  
 

Chức năng chính:  
 Form Login  
 Rút tiền (Withdraw)  
 Kiểm tra số dư (Balance Inquiry)  

Công cụ sử dụng:  
 UML (draw.io / PlantUML)  
 MySQL Workbench  
 Python/Java (module + test)  
 HTML/CSS (form login)  
 Jira (quản lý sprint)  
 GitHub (lưu trữ source code & báo cáo)  



 2. Mô hình UML

 2.1 Use Case Diagram (Lab 02)
![Hình Use Case Diagram (Khách hàng, Kỹ thuật viên, chức năng chính)](https://github.com/n23dcpt013-wq/lab02/blob/main/UseCase_OnlineShop.drawio.png)

 2.2 Sequence Diagram (Lab 03)
 Sequence Diagram – ATM Rút tiền  
 
```mermaid
sequenceDiagram
    autonumber
    actor KhachHang as Khách hàng
    participant ATM_UI as ATM UI
    participant ATM_Core as ATM Controller
    participant Card as Card Reader
    participant Core as Hệ thống Ngân hàng
    participant Dispenser as Bộ phát tiền
    participant Printer as Máy in biên lai

    KhachHang->>Card: InsertCard()
    Card-->>ATM_Core: CardInserted
    ATM_Core->>ATM_UI: Display("Nhập PIN")
    KhachHang->>ATM_UI: EnterPIN(pin)
    ATM_UI-->>ATM_Core: SubmitPIN(pin)
    ATM_Core->>Core: AuthorizePIN(PAN, pin)
    Core-->>ATM_Core: AuthResult(OK/FAIL)

    alt PIN sai
        ATM_Core-->>ATM_UI: Display("PIN sai, nhập lại")
    else PIN đúng
        KhachHang->>ATM_UI: Select("Withdraw")
        ATM_Core->>ATM_UI: Display("Nhập số tiền")
        KhachHang->>ATM_UI: EnterAmount(amount)
        ATM_UI-->>ATM_Core: SubmitAmount(amount)
        ATM_Core->>Core: DebitRequest(PAN, amount)
        Core-->>ATM_Core: Approved(txId, newBalance) / Declined(reason)

        alt Declined
            ATM_Core-->>ATM_UI: Display("Không đủ số dư / vượt hạn mức")
            ATM_Core->>Card: Eject()
        else Approved
            ATM_Core->>Dispenser: Dispense(amount)
            Dispenser-->>ATM_Core: DispenseResult(OK/JAM/OutOfCash)

            alt Lỗi phát tiền
                ATM_Core->>Core: Reversal(txId)
                ATM_Core-->>ATM_UI: Display("Lỗi phát tiền")
            else Thành công
                opt In biên lai
                    ATM_Core->>Printer: Print(txId, amount, time, balance)
                    Printer-->>ATM_Core: PrintDone()
                end
                ATM_Core->>Card: Eject()
                ATM_Core-->>ATM_UI: Display("Vui lòng nhận tiền & thẻ")
            end
        end
    end



 2.3 Class Diagram (Lab 06)
![Hình Class Diagram ATM, Account, Transaction](https://github.com/n23dcpt013-wq/lab06/blob/main/atm%20class.png)


 3. Database & Code minh hoạ

 3.1 ERD + Database (Lab 05)
![Hình ERD](https://github.com/n23dcpt013-wq/lab10/blob/main/erd.png)

script 

@startuml

!theme plain
skinparam linetype ortho
skinparam roundcorner 8
left to right direction

top to bottom direction
hide circle
hide methods
hide stereotypes


entity "KHACHHANG" as KH {
  * maKH : CHAR(10) <<PK>>
  
  hoTen : VARCHAR(100)
  ngaySinh : DATE
  diaChi : VARCHAR(200)
  soDienThoai : VARCHAR(20)
}

entity "TAIKHOAN" as TK {
  * soTK : CHAR(14) <<PK>>
  --
  maKH : CHAR(10) <<FK -> KH.maKH>>
  loaiTK : ENUM('SAVING','CHECKING')
  soDu : DECIMAL(15,2)
  trangThai : ENUM('ACTIVE','LOCKED')
}

entity "THE_ATM" as THE {
  * soThe : CHAR(16) <<PK>>
  --
  soTK : CHAR(14) <<FK -> TK.soTK>>
  pinHash : CHAR(64)
  ngayPhatHanh : DATE
  ngayHetHan : DATE
  trangThai : ENUM('ACTIVE','BLOCKED')
}

entity "GIAODICH" as GD {
  * maGD : CHAR(14) <<PK>>
  --
  soTK : CHAR(14) <<FK -> TK.soTK>>
  loaiGD : ENUM('WITHDRAW','DEPOSIT','TRANSFER_OUT','TRANSFER_IN','BAL_INQ')
  soTien : DECIMAL(15,2)
  thoiDiem : DATETIME
  moTa : VARCHAR(255)
  soTK_DoiUng : CHAR(14)  ' 
}

' 
KH ||--o{ TK : "1–N\n(mỗi KH nhiều tài khoản)"
TK ||--|| THE : "1–1\n(mỗi TK có 1 thẻ ATM)"
TK ||--o{ GD  : "1–N\n(mỗi TK nhiều giao dịch)"

@enduml


3.2 Form Login (Lab 04)
 ![HTML form](https://github.com/n23dcpt013-wq/lab04/blob/main/login.html)
![Ảnh giao diện form login](https://github.com/n23dcpt013-wq/lab10/blob/main/Screenshot%202025-09-27%20114210.png) 

3.3 Withdraw Module (Lab 07)
 Code minh họa Python/Java:  python
def withdraw(card_no, amount):
    conn = mysql.connector.connect(user="root", password="123456", database="atm_demo")
    cur = conn.cursor()
    try:
        conn.start_transaction()
        cur.execute("SELECT balance FROM accounts WHERE card_no=%s FOR UPDATE", (card_no,))
        balance = cur.fetchone()[0]
        if balance < amount:
            raise Exception("Insufficient funds")
        cur.execute("UPDATE accounts SET balance=balance-%s WHERE card_no=%s", (amount, card_no))
        conn.commit()
        return True
    except Exception as e:
        conn.rollback()
        return False
    finally:
        conn.close()

