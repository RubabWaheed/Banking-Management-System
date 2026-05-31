# Banking-Management-System
Banking Management System in C++
# 🏦 Banking Management System

> A feature-rich, console-based Banking Management System built in C++ using Object-Oriented Programming principles. Supports customer management, dual account types, full transaction history, file persistence, and a secure admin panel.

---

## 📸 Preview

```
========================================
   BANKING MANAGEMENT SYSTEM
========================================
  [ADMIN MODE]
----------------------------------------
  1. Create Customer
  2. Open Account
  3. Deposit
  4. Withdraw
  5. Transfer
  6. View Account
  7. View Customer
  8. List All Customers
  9. List All Accounts
 10. List All Transactions
 11. Apply Interest (Savings)
 12. Close Account
 13. Generate Statement
 14. Admin Logout
  0. Save & Exit
========================================
```

---

## ✨ Features

| Feature | Description |
|---|---|
| 👤 Customer Management | Register customers with name, email, phone & address |
| 🏧 Dual Account Types | SAVINGS (4% annual interest) and CHECKING accounts |
| 💰 Deposits & Withdrawals | Full validation with real-time balance updates |
| 🔄 Fund Transfers | Account-to-account transfers with linked transaction logs |
| 📊 Transaction History | Per-account history with timestamps and descriptions |
| 💾 File Persistence | Data saved across sessions in `.db` flat files |
| 🔐 Admin Panel | Password-protected admin mode with elevated privileges |
| 📈 Interest Calculation | Monthly interest application for all savings accounts |
| 📄 Statement Export | Generate account statements exported to `.txt` files |
| ❌ Account Closure | Safe account closing (requires zero balance) |

---

## 🛠️ Tech Stack

- **Language:** C++
- **Paradigm:** Object-Oriented Programming (OOP)
- **Storage:** Flat-file persistence (pipe-delimited `.db` files)
- **STL Used:** `vector`, `string`, `fstream`, `sstream`, `iomanip`

---

## 🚀 Getting Started

### Prerequisites

- A C++ compiler (g++, clang++, MSVC, etc.)
- C++98 or later

### Compilation

```bash
g++ -o banking_system banking_system.cpp
```

### Run

```bash
./banking_system
```

> On Windows:
> ```cmd
> banking_system.exe
> ```

---

## 📁 Project Structure

```
banking_system/
│
├── banking_system.cpp   # Main source file (all classes + main)
│
├── customers.db         # Auto-generated: customer records
├── accounts.db          # Auto-generated: account records
└── transactions.db      # Auto-generated: transaction logs
```

> `.db` files are created automatically on first run. Do **not** manually edit them.

---

## 🔑 Default Admin Credentials

| Field | Value |
|---|---|
| Password | `admin123` |

> ⚠️ **Security Note:** Change `ADMIN_PASS` in the source code before deploying in any real environment.

---

## 🧩 Class Architecture

```
BankSystem
├── vector<Customer>       → All registered customers
├── vector<Account>        → All bank accounts
└── vector<Transaction>    → Global transaction log

Customer
├── id, name, email, phone, address, regDate

Account
├── number, type (SAVINGS/CHECKING), balance
├── customerId, openDate, active, interestRate
└── vector<Transaction>    → Per-account transaction history

Transaction
└── id, accountNum, type, amount, balanceAfter, dateTime, description
```

---

## 📋 Menu Reference

### 👤 Regular User Options

| Option | Action |
|---|---|
| 1 | Create a new customer profile |
| 2 | Open a new bank account |
| 3 | Deposit funds into an account |
| 4 | Withdraw funds from an account |
| 5 | Transfer funds between accounts |
| 6 | View account details + last 10 transactions |
| 7 | View customer profile + linked accounts |
| 8 | Login to Admin Panel |
| 0 | Save data and exit |

### 🔐 Admin-Only Options

| Option | Action |
|---|---|
| 8  | List all customers |
| 9  | List all accounts |
| 10 | View all transactions |
| 11 | Apply monthly interest to savings accounts |
| 12 | Close an account |
| 13 | Generate account statement (.txt) |
| 14 | Logout from admin mode |

---

## 💡 How It Works

1. **On startup**, `BankSystem::loadData()` reads all three `.db` files and reconstructs in-memory state.
2. **Every operation** (deposit, withdraw, transfer, etc.) immediately calls `saveData()` to persist changes.
3. **Auto-incrementing IDs**: Customer IDs start at `1001`, Account Numbers at `100001`, Transaction IDs at `10000`.
4. **Transfers** create two linked transactions: `TRANSFER_OUT` on source and `TRANSFER_IN` on destination.
5. **Interest** is calculated monthly: `balance × rate / 12` and applied only to active SAVINGS accounts.

---

## ⚠️ Known Limitations

- Passwords are stored in plaintext in source code
- No multi-user / concurrent access support
- No undo/rollback functionality for transactions
- `.db` files use a simple pipe (`|`) delimiter — fields containing `|` would cause parsing issues

---


## 👨‍💻 Author

Built with ❤️ using C++ OOP principles.
