# Banking-Management-System
Banking Management System in C++
Banking Management System
Professional banking system with customer accounts, transactions, interest, and admin panel.
Features
•	Customer and Account management
•	Deposits, Withdrawals, Transfers
•	Transaction history with timestamps
•	Interest calculation for Savings (4% annual)
•	Admin Login for full records access
•	File persistence (customers.db, accounts.db, transactions.db)
•	Account statements generation
Compile & Run
g++ -o bank banking_system.cpp
./bank
Menu
Normal Mode: - 1. Create Customer - 2. Open Account - 3. Deposit - 4. Withdraw - 5. Transfer - 6. View Account - 7. View Customer - 8. Admin Login - 0. Save & Exit
Admin Mode (after login with admin123): - 8. List All Customers - 9. List All Accounts - 10. List All Transactions - 11. Apply Interest (Savings) - 12. Close Account - 13. Generate Statement - 14. Admin Logout
Admin Access
Default admin password: admin123
Admin can view all customers, accounts, transactions, apply interest, close accounts, and generate statements.

