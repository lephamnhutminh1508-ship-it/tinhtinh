#include <iostream>
#include <iomanip>
#include <string>
#include <cstdio>
using namespace std;

struct KhachHang {
    int soThuTu;
    int mucDich;
};

struct Node {
    KhachHang data;
    Node* next;
};

struct Queue {
    Node* front;
    Node* rear;
};

Node* createNode(KhachHang x) {
    Node* p = new Node;
    p->data = x;
    p->next = NULL;
    return p;
}

void initQueue(Queue& q) {
    q.front = NULL;
    q.rear = NULL;
}

bool isEmpty(Queue q) {
    return q.front == NULL;
}

void enqueue(Queue& q, KhachHang x) {
    Node* p = createNode(x);
    if (isEmpty(q)) {
        q.front = p;
        q.rear = p;
    }
    else {
        q.rear->next = p;
        q.rear = p;
    }
}

bool dequeue(Queue& q, KhachHang& x) {
    if (isEmpty(q)) {
        return false;
    }
    Node* p = q.front;
    x = p->data;
    q.front = q.front->next;
    if (q.front == NULL) {
        q.rear = NULL;
    }
    delete p;
    return true;
}

void displayQueue(Queue q) {
    Node* p = q.front;
    while (p != NULL) {
        cout << "So thu tu: " << p->data.soThuTu;
        cout << " | Muc dich: " << p->data.mucDich << endl;
        p = p->next;
    }
}

void clearQueue(Queue& q) {
    KhachHang tmp;
    while (dequeue(q, tmp)) {}
}

struct Counter {
    int counterId;
    string tenMucDich;
    int currentServing;
    Queue line;
};

int soThuTuTiepTheo = 1;

bool conKyTuThua() {
    int c = cin.peek();
    while (c == ' ' || c == '\t') {
        cin.get();
        c = cin.peek();
    }
    return c != '\n' && c != EOF;
}

void takeTicket(Counter* counters, int numCounters) {
    int mucDich;
    cout << "\n--- Danh sach muc dich ---\n";
    for (int i = 0; i < numCounters; i++) {
        cout << counters[i].counterId << ". " << counters[i].tenMucDich << "\n";
    }
    cout << "Chon muc dich (1 - " << numCounters << "): ";
    if (!(cin >> mucDich) || conKyTuThua() || mucDich < 1 || mucDich > numCounters) {
        cout << ">>> Muc dich khong hop le!\n";
        cin.clear();
        cin.ignore(10000, '\n');
        return;
    }
    cin.ignore(10000, '\n');
    KhachHang k = { soThuTuTiepTheo++, mucDich };
    enqueue(counters[mucDich - 1].line, k);
    cout << ">>> So thu tu cua ban la: " << k.soThuTu << "\n";
}

void loadSampleData(Counter* counters, int numCounters) {
    for (int i = 0; i < numCounters; i++) {
        for (int j = 0; j < 2; j++) {
            KhachHang k = { soThuTuTiepTheo++, i + 1 };
            enqueue(counters[i].line, k);
        }
    }
    cout << ">>> Da nap du lieu mau.\n";
}

void callNextCustomer(Counter* counters, int numCounters) {
    int counterId;

    cout << "\nNhap so hieu cua thu tuc muon goi khach (1 - " << numCounters << "): ";

    if (!(cin >> counterId) || conKyTuThua() || counterId < 1 || counterId > numCounters) {
        cout << ">>> Cua thu tuc khong ton tai!\n";
        cin.clear();
        cin.ignore(10000, '\n');
        return;
    }
    cin.ignore(10000, '\n');

    int idx = counterId - 1;
    KhachHang nextCust;

    if (dequeue(counters[idx].line, nextCust)) {
        counters[idx].currentServing = nextCust.soThuTu;

        cout << "\n>>> LOA THONG BAO: Moi khach hang so ["
             << nextCust.soThuTu << "] den Cua so "
             << counters[idx].counterId << " ("
             << counters[idx].tenMucDich << ")!\n";
    }
    else {
        counters[idx].currentServing = 0;

        cout << "\n>>> Thong bao: Cua so "
             << counters[idx].counterId
             << " hien khong con khach nao dang cho!\n";
    }
}

void displayStatus(Counter* counters, int numCounters) {
    cout << "\n=============== BANG HIEN THI TRANG THAI CAC CUA THU TUC ===============\n";
    cout << left << setw(10) << "Cua so"
         << setw(25) << "Muc dich"
         << setw(18) << "Dang phuc vu"
         << setw(18) << "So tiep theo" << "\n";

    cout << "-----------------------------------------------------------------------\n";

    for (int i = 0; i < numCounters; i++) {
        string servingStr = (counters[i].currentServing == 0)
            ? "Trong"
            : to_string(counters[i].currentServing);

        string nextStr = isEmpty(counters[i].line)
            ? "Khong co"
            : to_string(counters[i].line.front->data.soThuTu);

        string ten = counters[i].tenMucDich;
        if (ten.size() > 23) ten = ten.substr(0, 23);

        cout << left << setw(10) << counters[i].counterId
             << setw(25) << ten
             << setw(18) << servingStr
             << setw(18) << nextStr << "\n";
    }

    cout << "-----------------------------------------------------------------------\n";
}

int main() {
    const int numCounters = 3;
    Counter* counters = new Counter[numCounters];
    string ten[numCounters] = { "Nop ho so", "Dong tien", "Tu van" };

    for (int i = 0; i < numCounters; i++) {
        counters[i].counterId = i + 1;
        counters[i].tenMucDich = ten[i];
        counters[i].currentServing = 0;
        initQueue(counters[i].line);
    }

    int choice = -1;

    do {
        cout << "\n=============== MENU CHUC NANG ===============\n";
        cout << "1. Khach hang lay so thu tu\n";
        cout << "2. Cua thu tuc goi khach tiep theo\n";
        cout << "3. Hien thi man hinh cho\n";
        cout << "4. Nap du lieu mau\n";
        cout << "0. Thoat chuong trinh\n";
        cout << "==============================================\n";
        cout << "Nhap lua chon cua ban: ";

        if (!(cin >> choice) || conKyTuThua()) {
            if (cin.eof()) break;
            cin.clear();
            cin.ignore(10000, '\n');
            choice = -1;
        }
        else {
            cin.ignore(10000, '\n');
        }

        switch (choice) {
            case 1:
                takeTicket(counters, numCounters);
                break;

            case 2:
                callNextCustomer(counters, numCounters);
                break;

            case 3:
                displayStatus(counters, numCounters);
                break;

            case 4:
                loadSampleData(counters, numCounters);
                break;

            case 0:
                cout << "Dang thu hoi bo nho va thoat...\n";
                break;

            default:
                cout << "Lua chon khong hop le!\n";
        }
    } while (choice != 0);

    for (int i = 0; i < numCounters; i++) {
        clearQueue(counters[i].line);
    }
    delete[] counters;

    return 0;
}
