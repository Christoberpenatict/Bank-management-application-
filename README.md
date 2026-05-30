# Bank-management-application-
c++
#include <iostream>
#include <fstream>
#include <string>
#include <iomanip>
#include <limits>

using namespace std;

class Account {
private:
    int accountNumber;
    string accountHolderName;
    string accountType;
    double balance;

public:
    void createAccount();
    void displayAccountDetails() const;
    void setBalance(double newBalance) { balance = newBalance; }
    
    int getAccountNumber() const { return accountNumber; }
    string getAccountHolderName() const { return accountHolderName; }
    string getAccountType() const { return accountType; }
    double getBalance() const { return balance; }

    void writeToFile(ofstream& outFile) const {
        outFile << accountNumber << "|" << accountHolderName << "|" << accountType << "|" << balance << "\n";
    }

    bool readFromFile(ifstream& inFile) {
        string accNumStr, name, type, balStr;
        if (getline(inFile, accNumStr, '|') &&
            getline(inFile, name, '|') &&
            getline(inFile, type, '|') &&
            getline(inFile, balStr)) {
            
            accountNumber = stoi(accNumStr);
            accountHolderName = name;
            accountType = type;
            balance = stod(balStr);
            return true;
        }
        return false;
    }
};

void Account::createAccount() {
    cout << "\n--- Create New Account ---\n";
    cout << "Enter Account Number: ";
    while (!(cin >> accountNumber)) {
        cout << "Invalid input. Enter a valid integer for Account Number: ";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
    
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    cout << "Enter Account Holder Full Name: ";
    getline(cin, accountHolderName);
    
    cout << "Enter Account Type (Savings / Current): ";
    getline(cin, accountType);
    
    cout << "Enter Initial Deposit Amount: ";
    while (!(cin >> balance) || balance < 0) {
        cout << "Invalid amount. Enter a valid positive initial deposit: ";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

void Account::displayAccountDetails() const {
    cout << "\n--- Account Details ---\n";
    cout << "Account Number : " << accountNumber << endl;
    cout << "Holder Name    : " << accountHolderName << endl;
    cout << "Account Type   : " << accountType << endl;
    cout << "Current Balance: $" << fixed << setprecision(2) << balance << endl;
}

void createNewAccount();
void displayAllAccounts();
void checkBalance();
void depositMoney();
void withdrawMoney();
void deleteAccount();
void clearInputBuffer();

const string FILE_NAME = "bank_records.txt";

int main() {
    int choice;

    while (true) {
        cout << "\n=========================================\n";
        cout << "         BANK MANAGEMENT SYSTEM          \n";
        cout << "=========================================\n";
        cout << "1. Open New Account\n";
        cout << "2. Display All Accounts\n";
        cout << "3. Balance Inquiry\n";
        cout << "4. Deposit Money\n";
        cout << "5. Withdraw Money\n";
        cout << "6. Close an Account\n";
        cout << "7. Exit\n";
        cout << "=========================================\n";
        cout << "Enter your choice (1-7): ";

        if (!(cin >> choice)) {
            cout << "Invalid input! Please enter a number.\n";
            clearInputBuffer();
            continue;
        }

        switch (choice) {
            case 1: createNewAccount(); break;
            case 2: displayAllAccounts(); break;
            case 3: checkBalance(); break;
            case 4: depositMoney(); break;
            case 5: withdrawMoney(); break;
            case 6: deleteAccount(); break;
            case 7:
                cout << "\nThank you for using our banking services. Goodbye!\n";
                return 0;
            default:
                cout << "Invalid choice! Please select between 1 and 7.\n";
        }
    }
    return 0;
}

void clearInputBuffer() {
    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

void createNewAccount() {
    ofstream outFile(FILE_NAME, ios::app);
    if (!outFile) {
        cout << "Error opening file for transaction validation!\n";
        return;
    }

    Account acc;
    acc.createAccount();
    acc.writeToFile(outFile);
    outFile.close();

    cout << "\nAccount created and saved successfully!\n";
}

void displayAllAccounts() {
    ifstream inFile(FILE_NAME);
    if (!inFile) {
        cout << "\nNo records found. Open an account first.\n";
        return;
    }

    Account acc;
    cout << "\n-----------------------------------------------------------------\n";
    cout << left << setw(12) << "Acc No" << setw(25) << "Name" << setw(15) << "Type" << setw(12) << "Balance" << endl;
    cout << "-----------------------------------------------------------------\n";

    bool hasRecords = false;
    while (acc.readFromFile(inFile)) {
        hasRecords = true;
        cout << left << setw(12) << acc.getAccountNumber()
             << setw(25) << acc.getAccountHolderName()
             << setw(15) << acc.getAccountType()
             << "$" << fixed << setprecision(2) << acc.getBalance() << endl;
    }

    if (!hasRecords) {
        cout << "No accounts available currently.\n";
    }
    cout << "-----------------------------------------------------------------\n";
    inFile.close();
}

void checkBalance() {
    ifstream inFile(FILE_NAME);
    if (!inFile) {
        cout << "\nNo records found.\n";
        return;
    }

    int targetAcc;
    cout << "\nEnter Account Number for balance inquiry: ";
    cin >> targetAcc;

    Account acc;
    bool found = false;

    while (acc.readFromFile(inFile)) {
        if (acc.getAccountNumber() == targetAcc) {
            acc.displayAccountDetails();
            found = true;
            break;
        }
    }

    if (!found) {
        cout << "\nAccount Number " << targetAcc << " not found.\n";
    }
    inFile.close();
}

void depositMoney() {
    ifstream inFile(FILE_NAME);
    if (!inFile) {
        cout << "\nNo records found.\n";
        return;
    }

    int targetAcc;
    cout << "\nEnter Account Number for deposit: ";
    cin >> targetAcc;

    ofstream tempFile("temp_bank.txt");
    Account acc;
    bool found = false;

    while (acc.readFromFile(inFile)) {
        if (acc.getAccountNumber() == targetAcc) {
            found = true;
            double amount;
            cout << "\nCurrent Balance: $" << fixed << setprecision(2) << acc.getBalance() << endl;
            cout << "Enter Amount to Deposit: ";
            while (!(cin >> amount) || amount <= 0) {
                cout << "Invalid amount. Enter a positive transaction amount: ";
                clearInputBuffer();
            }
            acc.setBalance(acc.getBalance() + amount);
            cout << "\n$" << amount << " deposited successfully!\n";
            cout << "New Balance: $" << acc.getBalance() << endl;
        }
        acc.writeToFile(tempFile);
    }

    inFile.close();
    tempFile.close();

    remove(FILE_NAME.c_str());
    rename("temp_bank.txt", FILE_NAME.c_str());

    if (!found) {
        cout << "\nAccount Number " << targetAcc << " not found.\n";
    }
}

void withdrawMoney() {
    ifstream inFile(FILE_NAME);
    if (!inFile) {
        cout << "\nNo records found.\n";
        return;
    }

    int targetAcc;
    cout << "\nEnter Account Number for withdrawal: ";
    cin >> targetAcc;

    ofstream tempFile("temp_bank.txt");
    Account acc;
    bool found = false;

    while (acc.readFromFile(inFile)) {
        if (acc.getAccountNumber() == targetAcc) {
            found = true;
            double amount;
            cout << "\nCurrent Balance: $" << fixed << setprecision(2) << acc.getBalance() << endl;
            cout << "Enter Amount to Withdraw: ";
            while (!(cin >> amount) || amount <= 0) {
                cout << "Invalid amount. Enter a positive transaction amount: ";
                clearInputBuffer();
            }

            if (amount > acc.getBalance()) {
                cout << "\nTransaction Declined: Insufficient balance available!\n";
            } else {
                acc.setBalance(acc.getBalance() - amount);
                cout << "\n$" << amount << " withdrawn successfully!\n";
                cout << "New Balance: $" << acc.getBalance() << endl;
            }
        }
        acc.writeToFile(tempFile);
    }

    inFile.close();
    tempFile.close();

    remove(FILE_NAME.c_str());
    rename("temp_bank.txt", FILE_NAME.c_str());

    if (!found) {
        cout << "\nAccount Number " << targetAcc << " not found.\n";
    }
}

void deleteAccount() {
    ifstream inFile(FILE_NAME);
    if (!inFile) {
        cout << "\nNo records found to perform deletion.\n";
        return;
    }

    int targetAcc;
    cout << "\nEnter Account Number to close: ";
    cin >> targetAcc;

    ofstream tempFile("temp_bank.txt");
    Account acc;
    bool found = false;

    while (acc.readFromFile(inFile)) {
        if (acc.getAccountNumber() == targetAcc) {
            found = true;
            cout << "\nAccount containing account number " << targetAcc << " has been permanently dissolved.\n";
        } else {
            acc.writeToFile(tempFile);
        }
    }

    inFile.close();
    tempFile.close();

    remove(FILE_NAME.c_str());
    rename("temp_bank.txt", FILE_NAME.c_str());

    if (!found) {
        cout << "\nAccount Number " << targetAcc << " not found.\n";
    }
}
