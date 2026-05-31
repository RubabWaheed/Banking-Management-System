''' cpp
/*
 * Banking Management System
 * Professional OOP with admin panel, file persistence,
 * interest calculation, and transaction logging.
 */

#include <iostream>
#include <vector>
#include <string>
#include <ctime>
#include <iomanip>
#include <sstream>
#include <fstream>
#include <cstdlib>

using namespace std;

const string CUST_FILE = "customers.db";
const string ACCT_FILE = "accounts.db";
const string TRANS_FILE = "transactions.db";
const string DELIM = "|";
const string ADMIN_PASS = "admin123";

bool isAdmin = false;

string intToStr(int n) {
    stringstream ss;
    ss << n;
    return ss.str();
}

string doubleToStr(double d) {
    stringstream ss;
    ss << fixed << setprecision(2) << d;
    return ss.str();
}

string getDate() {
    time_t n = time(NULL);
    tm* l = localtime(&n);
    stringstream ss;
    ss << 1900+l->tm_year << "-" << 1+l->tm_mon << "-" << l->tm_mday;
    return ss.str();
}

string getDateTime() {
    time_t n = time(NULL);
    tm* l = localtime(&n);
    stringstream ss;
    ss << 1900+l->tm_year << "-" << 1+l->tm_mon << "-" << l->tm_mday
       << " " << l->tm_hour << ":" << l->tm_min << ":" << l->tm_sec;
    return ss.str();
}

// ==================== TRANSACTION ====================
class Transaction {
private:
    int id;
    int accountNum;
    string type;
    double amount;
    double balanceAfter;
    string dateTime;
    string description;
    static int nextId;
public:
    Transaction(int acc, string t, double amt, double bal, string desc)
        : accountNum(acc), type(t), amount(amt), balanceAfter(bal), description(desc) {
        id = nextId++;
        dateTime = getDateTime();
    }

    Transaction(int i, int acc, string t, double amt, double bal, string dt, string desc)
        : id(i), accountNum(acc), type(t), amount(amt), balanceAfter(bal), dateTime(dt), description(desc) {}

    int getId() const { return id; }
    int getAccount() const { return accountNum; }
    string getType() const { return type; }
    double getAmount() const { return amount; }

    string toFile() const {
        stringstream ss;
        ss << id << DELIM << accountNum << DELIM << type << DELIM
           << amount << DELIM << balanceAfter << DELIM << dateTime << DELIM << description;
        return ss.str();
    }

    void display() const {
        cout << "  [" << setw(5) << id << "] " << dateTime
             << " | " << left << setw(12) << type
             << " | $" << setw(10) << fixed << setprecision(2) << amount
             << " | Bal: $" << setw(10) << balanceAfter
             << " | " << description << endl;
    }
};

int Transaction::nextId = 10000;

// ==================== ACCOUNT ====================
class Account {
private:
    int number;
    string type;
    double balance;
    int customerId;
    string openDate;
    bool active;
    double interestRate;
    static int nextNum;
    vector<Transaction> transactions;

public:
    Account(string t, int cid, double deposit)
        : type(t), customerId(cid), balance(deposit), active(true) {
        number = nextNum++;
        openDate = getDate();
        interestRate = (t == "SAVINGS") ? 0.04 : 0.0;
        if(deposit > 0) {
            transactions.push_back(Transaction(number, "OPEN", deposit, balance, "Account opening deposit"));
        }
    }

    Account(int num, string t, double bal, int cid, string od, bool act, double ir)
        : number(num), type(t), balance(bal), customerId(cid), openDate(od), active(act), interestRate(ir) {}

    int getNumber() const { return number; }
    string getType() const { return type; }
    double getBalance() const { return balance; }
    int getCustomerId() const { return customerId; }
    bool isActive() const { return active; }
    double getRate() const { return interestRate; }

    static void setNextNum(int n) { nextNum = n; }

    bool deposit(double amt) {
        if(amt <= 0) {
            cout << "Invalid amount." << endl;
            return false;
        }
        balance += amt;
        transactions.push_back(Transaction(number, "DEPOSIT", amt, balance, "Cash deposit"));
        cout << "Deposited $" << fixed << setprecision(2) << amt << endl;
        return true;
    }

    bool withdraw(double amt) {
        if(amt <= 0) {
            cout << "Invalid amount." << endl;
            return false;
        }
        if(amt > balance) {
            cout << "Insufficient funds. Balance: $" << balance << endl;
            return false;
        }
        balance -= amt;
        transactions.push_back(Transaction(number, "WITHDRAW", amt, balance, "Cash withdrawal"));
        cout << "Withdrawn $" << fixed << setprecision(2) << amt << endl;
        return true;
    }

    bool transferTo(Account& target, double amt) {
        if(amt <= 0) {
            cout << "Invalid amount." << endl;
            return false;
        }
        if(amt > balance) {
            cout << "Insufficient funds." << endl;
            return false;
        }
        balance -= amt;
        string desc = "Transfer to #" + intToStr(target.getNumber());
        transactions.push_back(Transaction(number, "TRANSFER_OUT", amt, balance, desc));

        target.receiveTransfer(amt, number);
        cout << "Transferred $" << fixed << setprecision(2) << amt
             << " to Account #" << target.getNumber() << endl;
        return true;
    }

    void receiveTransfer(double amt, int fromAcc) {
        balance += amt;
        string desc = "Transfer from #" + intToStr(fromAcc);
        transactions.push_back(Transaction(number, "TRANSFER_IN", amt, balance, desc));
    }

    void addInterest() {
        if(type == "SAVINGS" && balance > 0) {
            double interest = balance * interestRate / 12.0;
            balance += interest;
            transactions.push_back(Transaction(number, "INTEREST", interest, balance, "Monthly interest"));
            cout << "Interest applied: $" << fixed << setprecision(2) << interest << endl;
        }
    }

    void addTransaction(const Transaction& t) {
        transactions.push_back(t);
    }

    void displayInfo() const {
        cout << "\n----------------------------------------" << endl;
        cout << "  Account #:    " << number << endl;
        cout << "  Type:         " << type << endl;
        cout << "  Balance:      $" << fixed << setprecision(2) << balance << endl;
        cout << "  Opened:       " << openDate << endl;
        cout << "  Status:       " << (active ? "ACTIVE" : "CLOSED") << endl;
        cout << "  Customer ID:  " << customerId << endl;
        if(type == "SAVINGS") {
            cout << "  Interest:     " << fixed << setprecision(2) << (interestRate*100) << "% annually" << endl;
        }
        cout << "----------------------------------------" << endl;
    }

    void displayTransactions(int limit) const {
        int displayCount = limit;
        if((int)transactions.size() < displayCount) displayCount = (int)transactions.size();
        cout << "\n  TRANSACTION HISTORY (Last " << displayCount << "):" << endl;
        cout << "----------------------------------------" << endl;
        if(transactions.empty()) {
            cout << "  No transactions." << endl;
            return;
        }
        int c = 0;
        for(int i = (int)transactions.size() - 1; i >= 0 && c < limit; i--) {
            transactions[i].display();
            c++;
        }
        cout << "----------------------------------------" << endl;
    }

    void close() {
        if(balance > 0) {
            cout << "Cannot close with balance $" << balance << ". Withdraw first." << endl;
            return;
        }
        active = false;
        cout << "Account #" << number << " closed." << endl;
    }

    string toFile() const {
        stringstream ss;
        ss << number << DELIM << type << DELIM << balance << DELIM
           << customerId << DELIM << openDate << DELIM << (active?"1":"0") << DELIM << interestRate;
        return ss.str();
    }
};

int Account::nextNum = 100001;

// ==================== CUSTOMER ====================
class Customer {
private:
    int id;
    string name;
    string email;
    string phone;
    string address;
    string regDate;
    static int nextId;
public:
    Customer(string n, string e, string p, string a)
        : name(n), email(e), phone(p), address(a) {
        id = nextId++;
        regDate = getDate();
    }

    Customer(int i, string n, string e, string p, string a, string rd)
        : id(i), name(n), email(e), phone(p), address(a), regDate(rd) {}

    int getId() const { return id; }
    string getName() const { return name; }
    string getEmail() const { return email; }
    string getPhone() const { return phone; }

    static void setNextId(int n) { nextId = n; }

    void display() const {
        cout << "\n----------------------------------------" << endl;
        cout << "  Customer ID:  " << id << endl;
        cout << "  Name:         " << name << endl;
        cout << "  Email:        " << email << endl;
        cout << "  Phone:        " << phone << endl;
        cout << "  Address:      " << address << endl;
        cout << "  Registered:   " << regDate << endl;
        cout << "----------------------------------------" << endl;
    }

    string toFile() const {
        return intToStr(id) + DELIM + name + DELIM + email + DELIM + phone + DELIM + address + DELIM + regDate;
    }
};

int Customer::nextId = 1001;

// ==================== BANKING SYSTEM ====================
class BankSystem {
private:
    vector<Customer> customers;
    vector<Account> accounts;
    vector<Transaction> allTransactions;

    Customer* findCustomer(int id) {
        for(size_t i = 0; i < customers.size(); i++) {
            if(customers[i].getId() == id) return &customers[i];
        }
        return 0;
    }

    Account* findAccount(int num) {
        for(size_t i = 0; i < accounts.size(); i++) {
            if(accounts[i].getNumber() == num && accounts[i].isActive()) return &accounts[i];
        }
        return 0;
    }

    int getMaxCustId() {
        int maxId = 1000;
        for(size_t i = 0; i < customers.size(); i++) {
            if(customers[i].getId() > maxId) maxId = customers[i].getId();
        }
        return maxId;
    }

    int getMaxAccNum() {
        int maxNum = 100000;
        for(size_t i = 0; i < accounts.size(); i++) {
            if(accounts[i].getNumber() > maxNum) maxNum = accounts[i].getNumber();
        }
        return maxNum;
    }

public:
    void loadData() {
        // Load customers
        ifstream cf(CUST_FILE.c_str());
        if(cf) {
            string line;
            while(getline(cf, line)) {
                if(line.empty()) continue;
                vector<string> parts;
                size_t pos = 0;
                while((pos = line.find(DELIM)) != string::npos) {
                    parts.push_back(line.substr(0, pos));
                    line.erase(0, pos + DELIM.length());
                }
                parts.push_back(line);
                if(parts.size() >= 6) {
                    customers.push_back(Customer(atoi(parts[0].c_str()), parts[1], parts[2], parts[3], parts[4], parts[5]));
                }
            }
            cf.close();
        }

        // Load accounts
        ifstream af(ACCT_FILE.c_str());
        if(af) {
            string line;
            while(getline(af, line)) {
                if(line.empty()) continue;
                vector<string> parts;
                size_t pos = 0;
                while((pos = line.find(DELIM)) != string::npos) {
                    parts.push_back(line.substr(0, pos));
                    line.erase(0, pos + DELIM.length());
                }
                parts.push_back(line);
                if(parts.size() >= 7) {
                    accounts.push_back(Account(atoi(parts[0].c_str()), parts[1], atof(parts[2].c_str()),
                        atoi(parts[3].c_str()), parts[4], parts[5]=="1", atof(parts[6].c_str())));
                }
            }
            af.close();
        }

        // Load transactions
        ifstream tf(TRANS_FILE.c_str());
        if(tf) {
            string line;
            while(getline(tf, line)) {
                if(line.empty()) continue;
                vector<string> parts;
                size_t pos = 0;
                while((pos = line.find(DELIM)) != string::npos) {
                    parts.push_back(line.substr(0, pos));
                    line.erase(0, pos + DELIM.length());
                }
                parts.push_back(line);
                if(parts.size() >= 7) {
                    Transaction t(atoi(parts[0].c_str()), atoi(parts[1].c_str()), parts[2],
                        atof(parts[3].c_str()), atof(parts[4].c_str()), parts[5], parts[6]);
                    allTransactions.push_back(t);
                    for(size_t i = 0; i < accounts.size(); i++) {
                        if(accounts[i].getNumber() == t.getAccount()) {
                            accounts[i].addTransaction(t);
                            break;
                        }
                    }
                }
            }
            tf.close();
        }

        // Fix static counters
        if(!customers.empty()) {
            int maxId = getMaxCustId();
            Customer::setNextId(maxId + 1);
        }
        if(!accounts.empty()) {
            int maxNum = getMaxAccNum();
            Account::setNextNum(maxNum + 1);
        }
    }

    void saveData() {
        ofstream cf(CUST_FILE.c_str());
        for(size_t i = 0; i < customers.size(); i++) {
            cf << customers[i].toFile() << endl;
        }
        cf.close();

        ofstream af(ACCT_FILE.c_str());
        for(size_t i = 0; i < accounts.size(); i++) {
            af << accounts[i].toFile() << endl;
        }
        af.close();

        ofstream tf(TRANS_FILE.c_str());
        for(size_t i = 0; i < allTransactions.size(); i++) {
            tf << allTransactions[i].toFile() << endl;
        }
        tf.close();
    }

    void createCustomer() {
        string name, email, phone, addr;
        cout << "\n========================================" << endl;
        cout << "         CREATE NEW CUSTOMER" << endl;
        cout << "========================================" << endl;

        cout << "\nFull Name: ";
        cin.ignore();
        getline(cin, name);
        if(name.empty()) name = "Unnamed";

        cout << "Email: ";
        getline(cin, email);

        cout << "Phone: ";
        getline(cin, phone);

        cout << "Address: ";
        getline(cin, addr);

        customers.push_back(Customer(name, email, phone, addr));
        customers.back().display();
        cout << "Customer created. ID: " << customers.back().getId() << endl;
        saveData();
    }

    void createAccount() {
        int cid = 0;
        string type;
        double deposit = 0.0;

        cout << "\n========================================" << endl;
        cout << "         OPEN NEW ACCOUNT" << endl;
        cout << "========================================" << endl;

        cout << "\nCustomer ID: ";
        cin >> cid;
        if(cid <= 0) {
            cout << "Invalid ID." << endl;
            return;
        }

        Customer* c = findCustomer(cid);
        if(!c) {
            cout << "Customer not found." << endl;
            return;
        }

        cout << "Account Type (SAVINGS/CHECKING): ";
        cin >> type;
        for(size_t i = 0; i < type.length(); i++) {
            type[i] = (char)toupper((unsigned char)type[i]);
        }
        if(type != "SAVINGS" && type != "CHECKING") {
            cout << "Invalid type. Defaulting to CHECKING." << endl;
            type = "CHECKING";
        }

        cout << "Initial Deposit: $";
        cin >> deposit;
        if(deposit < 0) deposit = 0;

        accounts.push_back(Account(type, cid, deposit));
        accounts.back().displayInfo();
        cout << "Account created. Number: " << accounts.back().getNumber() << endl;
        saveData();
    }

    void doDeposit() {
        int num = 0;
        double amt = 0.0;
        cout << "\nAccount Number: ";
        cin >> num;
        if(num <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Account* a = findAccount(num);
        if(!a) {
            cout << "Account not found or closed." << endl;
            return;
        }
        cout << "Amount: $";
        cin >> amt;
        if(amt <= 0) {
            cout << "Invalid amount." << endl;
            return;
        }
        if(a->deposit(amt)) {
            allTransactions.push_back(Transaction(a->getNumber(), "DEPOSIT", amt, a->getBalance(), "Cash deposit"));
            saveData();
        }
    }

    void doWithdraw() {
        int num = 0;
        double amt = 0.0;
        cout << "\nAccount Number: ";
        cin >> num;
        if(num <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Account* a = findAccount(num);
        if(!a) {
            cout << "Account not found or closed." << endl;
            return;
        }
        cout << "Amount: $";
        cin >> amt;
        if(amt <= 0) {
            cout << "Invalid amount." << endl;
            return;
        }
        if(a->withdraw(amt)) {
            allTransactions.push_back(Transaction(a->getNumber(), "WITHDRAW", amt, a->getBalance(), "Cash withdrawal"));
            saveData();
        }
    }

    void doTransfer() {
        int from = 0, to = 0;
        double amt = 0.0;

        cout << "\nFrom Account: ";
        cin >> from;
        if(from <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Account* src = findAccount(from);
        if(!src) {
            cout << "Source account not found." << endl;
            return;
        }

        cout << "To Account: ";
        cin >> to;
        if(to <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Account* dst = findAccount(to);
        if(!dst) {
            cout << "Destination account not found." << endl;
            return;
        }
        if(from == to) {
            cout << "Cannot transfer to same account." << endl;
            return;
        }

        cout << "Amount: $";
        cin >> amt;
        if(amt <= 0) {
            cout << "Invalid amount." << endl;
            return;
        }

        if(src->transferTo(*dst, amt)) {
            allTransactions.push_back(Transaction(src->getNumber(), "TRANSFER_OUT", amt, src->getBalance(),
                "To #" + intToStr(dst->getNumber())));
            allTransactions.push_back(Transaction(dst->getNumber(), "TRANSFER_IN", amt, dst->getBalance(),
                "From #" + intToStr(src->getNumber())));
            saveData();
        }
    }

    void viewAccount() {
        int num = 0;
        cout << "\nAccount Number: ";
        cin >> num;
        if(num <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Account* a = findAccount(num);
        if(!a) {
            for(size_t i = 0; i < accounts.size(); i++) {
                if(accounts[i].getNumber() == num) {
                    accounts[i].displayInfo();
                    return;
                }
            }
            cout << "Account not found." << endl;
            return;
        }
        a->displayInfo();
        a->displayTransactions(10);
    }

    void viewCustomer() {
        int cid = 0;
        cout << "\nCustomer ID: ";
        cin >> cid;
        if(cid <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Customer* c = findCustomer(cid);
        if(!c) {
            cout << "Customer not found." << endl;
            return;
        }
        c->display();

        cout << "\n  Accounts:" << endl;
        bool found = false;
        for(size_t i = 0; i < accounts.size(); i++) {
            if(accounts[i].getCustomerId() == cid) {
                accounts[i].displayInfo();
                found = true;
            }
        }
        if(!found) cout << "  No accounts." << endl;
    }

    void listCustomers() {
        cout << "\n========================================" << endl;
        cout << "         ALL CUSTOMERS" << endl;
        cout << "========================================" << endl;
        if(customers.empty()) {
            cout << "No customers." << endl;
            return;
        }
        cout << left << setw(8) << "ID" << setw(22) << "Name"
             << setw(25) << "Email" << setw(15) << "Phone" << endl;
        cout << "--------------------------------------------------------" << endl;
        for(size_t i = 0; i < customers.size(); i++) {
            cout << left << setw(8) << customers[i].getId()
                 << setw(22) << customers[i].getName()
                 << setw(25) << customers[i].getEmail()
                 << setw(15) << customers[i].getPhone() << endl;
        }
    }

    void listAccounts() {
        cout << "\n========================================" << endl;
        cout << "         ALL ACCOUNTS" << endl;
        cout << "========================================" << endl;
        if(accounts.empty()) {
            cout << "No accounts." << endl;
            return;
        }
        cout << left << setw(10) << "Number" << setw(12) << "Type"
             << setw(15) << "Balance" << setw(12) << "Status"
             << setw(10) << "CustID" << endl;
        cout << "--------------------------------------------------------" << endl;
        for(size_t i = 0; i < accounts.size(); i++) {
            cout << left << setw(10) << accounts[i].getNumber()
                 << setw(12) << accounts[i].getType()
                 << "$" << setw(14) << fixed << setprecision(2) << accounts[i].getBalance()
                 << setw(12) << (accounts[i].isActive() ? "ACTIVE" : "CLOSED")
                 << setw(10) << accounts[i].getCustomerId() << endl;
        }
    }

    void listTransactions() {
        cout << "\n========================================" << endl;
        cout << "         ALL TRANSACTIONS" << endl;
        cout << "========================================" << endl;
        if(allTransactions.empty()) {
            cout << "No transactions." << endl;
            return;
        }
        for(size_t i = 0; i < allTransactions.size(); i++) {
            allTransactions[i].display();
        }
    }

    void applyInterest() {
        cout << "\n========================================" << endl;
        cout << "         APPLY INTEREST" << endl;
        cout << "========================================" << endl;
        bool any = false;
        for(size_t i = 0; i < accounts.size(); i++) {
            if(accounts[i].isActive() && accounts[i].getType() == "SAVINGS") {
                accounts[i].addInterest();
                any = true;
            }
        }
        if(any) saveData();
        else cout << "No savings accounts." << endl;
    }

    void closeAccount() {
        int num = 0;
        cout << "\nAccount to close: ";
        cin >> num;
        if(num <= 0) {
            cout << "Invalid." << endl;
            return;
        }
        Account* a = findAccount(num);
        if(!a) {
            cout << "Account not found." << endl;
            return;
        }
        a->close();
        saveData();
    }

    void generateStatement() {
        int num = 0;
        cout << "\nAccount Number for statement: ";
        cin >> num;
        if(num <= 0) {
            cout << "Invalid." << endl;
            return;
        }

        string fname = "statement_" + intToStr(num) + ".txt";
        ofstream out(fname.c_str());
        if(!out) {
            cout << "Cannot create file." << endl;
            return;
        }

        out << "ACCOUNT STATEMENT" << endl;
        out << string(70, '=') << endl;

        Account* a = findAccount(num);
        if(!a) {
            out << "Account not found." << endl;
            out.close();
            return;
        }

        out << "Account #: " << a->getNumber() << endl;
        out << "Type: " << a->getType() << endl;
        out << "Current Balance: $" << fixed << setprecision(2) << a->getBalance() << endl;
        out << string(70, '-') << endl;
        out.close();

        cout << "Statement saved to " << fname << endl;
    }
};

void printMainMenu() {
    cout << "\n========================================" << endl;
    cout << "   BANKING MANAGEMENT SYSTEM" << endl;
    cout << "========================================" << endl;
    if(isAdmin) {
        cout << "  [ADMIN MODE]" << endl;
        cout << "----------------------------------------" << endl;
    }
    cout << "  1. Create Customer" << endl;
    cout << "  2. Open Account" << endl;
    cout << "  3. Deposit" << endl;
    cout << "  4. Withdraw" << endl;
    cout << "  5. Transfer" << endl;
    cout << "  6. View Account" << endl;
    cout << "  7. View Customer" << endl;
    if(isAdmin) {
        cout << "  8. List All Customers" << endl;
        cout << "  9. List All Accounts" << endl;
        cout << " 10. List All Transactions" << endl;
        cout << " 11. Apply Interest (Savings)" << endl;
        cout << " 12. Close Account" << endl;
        cout << " 13. Generate Statement" << endl;
        cout << " 14. Admin Logout" << endl;
    } else {
        cout << "  8. Admin Login" << endl;
    }
    cout << "  0. Save & Exit" << endl;
    cout << "========================================" << endl;
    cout << "Choice: ";
}

int main() {
    BankSystem bank;
    bank.loadData();

    int choice = 0;
    bool run = true;

    while(run) {
        printMainMenu();
        cin >> choice;

        if(cin.fail()) {
            cin.clear();
            cin.ignore(1000, '\n');
            cout << "Invalid input.\n" << endl;
            continue;
        }

        switch(choice) {
            case 1: bank.createCustomer(); break;
            case 2: bank.createAccount(); break;
            case 3: bank.doDeposit(); break;
            case 4: bank.doWithdraw(); break;
            case 5: bank.doTransfer(); break;
            case 6: bank.viewAccount(); break;
            case 7: bank.viewCustomer(); break;
            case 8:
                if(isAdmin) bank.listCustomers();
                else {
                    string pass;
                    cout << "\nEnter admin password: ";
                    cin >> pass;
                    if(pass == ADMIN_PASS) {
                        isAdmin = true;
                        cout << "Admin login successful.\n" << endl;
                    } else {
                        cout << "Invalid password.\n" << endl;
                    }
                }
                break;
            case 9:
                if(isAdmin) bank.listAccounts();
                else cout << "Invalid choice.\n" << endl;
                break;
            case 10:
                if(isAdmin) bank.listTransactions();
                else cout << "Invalid choice.\n" << endl;
                break;
            case 11:
                if(isAdmin) bank.applyInterest();
                else cout << "Invalid choice.\n" << endl;
                break;
            case 12:
                if(isAdmin) bank.closeAccount();
                else cout << "Invalid choice.\n" << endl;
                break;
            case 13:
                if(isAdmin) bank.generateStatement();
                else cout << "Invalid choice.\n" << endl;
                break;
            case 14:
                if(isAdmin) {
                    isAdmin = false;
                    cout << "Admin logged out.\n" << endl;
                } else {
                    cout << "Invalid choice.\n" << endl;
                }
                break;
            case 0:
                bank.saveData();
                cout << "\nData saved. Goodbye." << endl;
                run = false;
                break;
            default:
                cout << "Invalid choice.\n" << endl;
        }
    }
    return 0;
}
'''
