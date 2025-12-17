
#include <iostream>
#include <fstream>
#include <cctype>
#include <iomanip>
using namespace std;

// ***************************************************************
//                       CLASS USED IN PROJECT
// ***************************************************************

class account
{
    int acno;
    char name[50];
    int deposit;
    char type;

public:
    void create_account();
    void show_account() const;
    void modify();
    void dep(int);
    void draw(int);
    void report() const;
    int retacno() const;
    int retdeposit() const;
    char rettype() const;
};

// ***************************************************************
//                  FUNCTION DECLARATIONS
// ***************************************************************

void write_account();
void display_sp(int);
void modify_account(int);
void delete_account(int);
void display_all();
void deposit_withdraw(int, int);
void intro();

// ***************************************************************
//                  MEMBER FUNCTION DEFINITIONS
// ***************************************************************

void account::create_account()
{
    cout << "\nEnter The account No. : ";
    cin >> acno;
    cout << "\n\nEnter The Name of The account Holder : ";
    cin.ignore();
    cin.getline(name, 50);
    cout << "\nEnter Type of The account (C/S) : ";
    cin >> type;
    type = toupper(type);
    cout << "\nEnter The Initial amount (>=500 for Saving and >=1000 for Current) : ";
    cin >> deposit;
    cout << "\n\nAccount Created..";
}

void account::show_account() const
{
    cout << "\nAccount No. : " << acno;
    cout << "\nAccount Holder Name : " << name;
    cout << "\nType of Account : " << type;
    cout << "\nBalance amount : " << deposit;
}

void account::modify()
{
    cout << "\nAccount No. : " << acno;
    cout << "\nModify Account Holder Name : ";
    cin.ignore();
    cin.getline(name, 50);
    cout << "\nModify Type of Account : ";
    cin >> type;
    type = toupper(type);
    cout << "\nModify Balance amount : ";
    cin >> deposit;
}

void account::dep(int x)
{
    deposit += x;
}

void account::draw(int x)
{
    deposit -= x;
}

void account::report() const
{
    cout << acno << setw(15) << name << setw(10) << type << setw(10) << deposit << endl;
}

int account::retacno() const
{
    return acno;
}

int account::retdeposit() const
{
    return deposit;
}

char account::rettype() const
{
    return type;
}

// ***************************************************************
//                  WRITE ACCOUNT
// ***************************************************************

void write_account()
{
    account ac;
    ofstream outFile("account.dat", ios::binary | ios::app);
    ac.create_account();
    outFile.write(reinterpret_cast<char*>(&ac), sizeof(account));
    outFile.close();
}

// ***************************************************************
//                  DISPLAY SPECIFIC ACCOUNT
// ***************************************************************

void display_sp(int n)
{
    account ac;
    bool flag = false;
    ifstream inFile("account.dat", ios::binary);

    if (!inFile)
    {
        cout << "File could not be open!";
        return;
    }

    cout << "\nBALANCE DETAILS\n";

    while (inFile.read(reinterpret_cast<char*>(&ac), sizeof(account)))
    {
        if (ac.retacno() == n)
        {
            ac.show_account();
            flag = true;
        }
    }
    inFile.close();

    if (!flag)
        cout << "\nAccount number does not exist";
}

// ***************************************************************
//                  MODIFY ACCOUNT
// ***************************************************************

void modify_account(int n)
{
    bool found = false;
    account ac;
    fstream File("account.dat", ios::binary | ios::in | ios::out);

    if (!File)
    {
        cout << "File could not be open!";
        return;
    }

    while (File.read(reinterpret_cast<char*>(&ac), sizeof(account)) && !found)
    {
        if (ac.retacno() == n)
        {
            ac.show_account();
            cout << "\n\nEnter New Details:\n";
            ac.modify();

            int pos = -1 * static_cast<int>(sizeof(account));
            File.seekp(pos, ios::cur);
            File.write(reinterpret_cast<char*>(&ac), sizeof(account));
            cout << "\nRecord Updated";
            found = true;
        }
    }
    File.close();

    if (!found)
        cout << "\nRecord Not Found";
}

// ***************************************************************
//                  DELETE ACCOUNT
// ***************************************************************

void delete_account(int n)
{
    account ac;
    ifstream inFile("account.dat", ios::binary);
    ofstream outFile("Temp.dat", ios::binary);

    if (!inFile)
    {
        cout << "File could not be open!";
        return;
    }

    while (inFile.read(reinterpret_cast<char*>(&ac), sizeof(account)))
    {
        if (ac.retacno() != n)
            outFile.write(reinterpret_cast<char*>(&ac), sizeof(account));
    }

    inFile.close();
    outFile.close();
    remove("account.dat");
    rename("Temp.dat", "account.dat");
    cout << "\nRecord Deleted";
}

// ***************************************************************
//                  DISPLAY ALL ACCOUNTS
// ***************************************************************

void display_all()
{
    account ac;
    ifstream inFile("account.dat", ios::binary);

    if (!inFile)
    {
        cout << "File could not be open!";
        return;
    }

    cout << "\n\nACCOUNT HOLDER LIST\n";
    cout << "===========================================\n";
    cout << "A/c No.        Name        Type    Balance\n";
    cout << "===========================================\n";

    while (inFile.read(reinterpret_cast<char*>(&ac), sizeof(account)))
    {
        ac.report();
    }
    inFile.close();
}

// ***************************************************************
//                  DEPOSIT / WITHDRAW
// ***************************************************************

void deposit_withdraw(int n, int option)
{
    int amt;
    bool found = false;
    account ac;
    fstream File("account.dat", ios::binary | ios::in | ios::out);

    if (!File)
    {
        cout << "File could not be open!";
        return;
    }

    while (File.read(reinterpret_cast<char*>(&ac), sizeof(account)) && !found)
    {
        if (ac.retacno() == n)
        {
            ac.show_account();

            if (option == 1)
            {
                cout << "\nEnter amount to deposit: ";
                cin >> amt;
                ac.dep(amt);
            }
            else if (option == 2)
            {
                cout << "\nEnter amount to withdraw: ";
                cin >> amt;
                int bal = ac.retdeposit() - amt;

                if ((bal < 500 && ac.rettype() == 'S') ||
                    (bal < 1000 && ac.rettype() == 'C'))
                {
                    cout << "\nInsufficient balance";
                    return;
                }
                ac.draw(amt);
            }

            int pos = -1 * static_cast<int>(sizeof(account));
            File.seekp(pos, ios::cur);
            File.write(reinterpret_cast<char*>(&ac), sizeof(account));
            cout << "\nRecord Updated";
            found = true;
        }
    }
    File.close();

    if (!found)
        cout << "\nRecord Not Found";
}

// ***************************************************************
//                  INTRO FUNCTION
// ***************************************************************

void intro()
{
    cout << "\n\nBANK MANAGEMENT SYSTEM";
    cout << "\nMADE BY : Your Name";
    cout << "\nSCHOOL  : Your School";
    cin.get();
}

// ***************************************************************
//                  MAIN FUNCTION
// ***************************************************************

int main()
{
    char ch;
    int num;
    intro();

    do
    {
        cout << "\n\nMAIN MENU";
        cout << "\n1. New Account";
        cout << "\n2. Deposit Amount";
        cout << "\n3. Withdraw Amount";
        cout << "\n4. Balance Enquiry";
        cout << "\n5. All Account Holder List";
        cout << "\n6. Close An Account";
        cout << "\n7. Modify An Account";
        cout << "\n8. Exit";
        cout << "\nSelect Your Option (1-8): ";
        cin >> ch;

        switch (ch)
        {
        case '1': write_account(); break;
        case '2': cout << "Enter Account No: "; cin >> num; deposit_withdraw(num, 1); break;
        case '3': cout << "Enter Account No: "; cin >> num; deposit_withdraw(num, 2); break;
        case '4': cout << "Enter Account No: "; cin >> num; display_sp(num); break;
        case '5': display_all(); break;
        case '6': cout << "Enter Account No: "; cin >> num; delete_account(num); break;
        case '7': cout << "Enter Account No: "; cin >> num; modify_account(num); break;
        case '8': cout << "Thank you!"; break;
        default: cout << "Invalid option";
        }
    } while (ch != '8');

    return 0;
}
