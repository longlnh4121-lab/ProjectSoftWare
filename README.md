// Thư viện xử lý file
#include <fstream>
// Thư viện xử lý chuỗi
#include <sstream>
// Thư viện mảng động (danh sách)
#include <vector>
// Thư viện map (bảng tra cứu key-value)
#include <map>
// Thư viện string
#include <string>
// Thư viện nhập xuất
#include <iostream>
// Thư viện xử lý thời gian
#include <ctime>
// Thư viện định dạng output
#include <iomanip>

// Sử dụng namespace std để không cần viết std:: trước mỗi lệnh
using namespace std;

// ===== CLASS ACCOUNT (LỚP TÀI KHOẢN CƠ BẢN) =====
// Lớp cha chung cho Student và Admin
class Account {
public:
    // Các thuộc tính của tài khoản
    string username;    // Tên đăng nhập
    string password;    // Mật khẩu
    string email;       // Email
    string fullName;    // Họ tên đầy đủ
    string role;        // Vai trò (Student hoặc Admin)

    // Constructor (hàm khởi tạo) với giá trị mặc định
    Account(string u = "", string p = "", string e = "", string f = "")
        : username(u), password(p), email(e), fullName(f) {}

    // Phương thức đăng nhập - kiểm tra username và password
    virtual bool login(const string &u, const string &p) {
        return username == u && password == p;
    }

    // Phương thức đăng xuất (chưa cài đặt)
    virtual void logout() { /* no-op */ }
    
    // Destructor ảo để có thể xóa đúng object khi dùng pointer
    virtual ~Account() = default;
};

// Khai báo trước class Answer để Question có thể dùng
class Answer;

// ===== CLASS QUESTION (LỚP CÂU HỎI) =====
class Question {
private:
    // Biến static để tạo ID tự động tăng cho mỗi câu hỏi
    static int nextQuestionID;
    
public:
    int questionID;           // ID duy nhất của câu hỏi
    string questionText;      // Nội dung câu hỏi
    int points;               // Số điểm của câu hỏi
    vector<Answer*> answers;  // Danh sách các câu trả lời

    // Constructor - khởi tạo câu hỏi và tự động gán ID
    Question(string text = "", int pts = 1) : questionText(text), points(pts) {
        questionID = ++nextQuestionID;  // Tăng và gán ID
    }

    // Thêm một câu trả lời vào câu hỏi
    Answer* addAnswer(string text, bool isCorrect);
    
    // Kiểm tra xem câu trả lời có đúng không
    bool checkAnswer(int answerID);
};

// ===== CLASS ANSWER (LỚP CÂU TRẢ LỜI) =====
class Answer {
private:
    // Biến static để tạo ID tự động tăng cho mỗi câu trả lời
    static int nextAnswerID;
    
public:
    int answerID;           // ID duy nhất của câu trả lời
    string answerText;      // Nội dung câu trả lời
    bool isCorrect;         // Đánh dấu câu trả lời đúng hay sai
    
    // Constructor - khởi tạo câu trả lời và tự động gán ID
    Answer(string text = "", bool correct = false) : answerText(text), isCorrect(correct) { 
        answerID = ++nextAnswerID; 
    }
    
    // Getter methods (lấy thông tin)
    int getID() const { return answerID; }           // Lấy ID
    string getText() const { return answerText; }    // Lấy nội dung
    bool correct() const { return isCorrect; }       // Kiểm tra đúng/sai
};

// Khai báo trước class QuizResult
class QuizResult;

// ===== CLASS QUIZ (LỚP BÀI KIỂM TRA) =====
class Quiz {
private:
    // Biến static để tạo ID tự động tăng cho mỗi quiz
    static int nextQuizID;
    
public:
    int quizID;                    // ID duy nhất của bài quiz
    string title;                  // Tiêu đề bài quiz
    vector<Question*> questions;   // Danh sách các câu hỏi

    // Constructor - khởi tạo quiz với tiêu đề
    Quiz(string t = "Quiz") : title(t) { 
        quizID = ++nextQuizID; 
    }

    // Thêm câu hỏi mới vào bài quiz
    Question* addQuestion(string text, int points) {
        Question* q = new Question(text, points);  // Tạo câu hỏi mới
        questions.push_back(q);                    // Thêm vào danh sách
        return q;                                   // Trả về pointer để có thể thêm đáp án
    }

    // Tính điểm dựa trên câu trả lời của học sinh
    float calculateScore(const map<int, int>& userAnswers);
    
    // Thêm kết quả (chưa sử dụng trong code này)
    void addResult(QuizResult* r) { (void)r; /* optional storage */ }
};

// ===== CLASS STUDENTANSWER (LỚP CÂU TRẢ LỜI CỦA HỌC SINH) =====
// Lưu trữ câu trả lời mà học sinh đã chọn
class StudentAnswer {
public:
    Answer* selectedAnswer;  // Câu trả lời được chọn
    
    // Constructor - lưu câu trả lời đã chọn
    StudentAnswer(Answer* a) : selectedAnswer(a) {}
    
    // Lấy câu trả lời đã chọn
    Answer* getSelectedAnswer() { return selectedAnswer; }
};

// ===== CLASS QUIZRESULT (LỚP KẾT QUẢ BÀI KIỂM TRA) =====
// Lưu trữ kết quả làm bài của học sinh
class QuizResult {
private:
    float score;                            // Điểm số (%)
    time_t dateTaken;                       // Thời gian làm bài
    vector<StudentAnswer*> studentAnswers;  // Danh sách câu trả lời của học sinh
    
public:
    // Constructor - khởi tạo kết quả với thời gian hiện tại
    QuizResult() : score(0), dateTaken(time(nullptr)) {}
    
    // Setter và Getter cho điểm số
    void setScore(float s) { score = s; }
    float getScore() const { return score; }
    
    // Lấy thời gian làm bài
    time_t getDate() const { return dateTaken; }
    
    // Thêm câu trả lời của học sinh vào kết quả
    void addStudentAnswer(StudentAnswer* sa) { studentAnswers.push_back(sa); }
};

// ===== CLASS STUDENT (LỚP HỌC SINH) =====
// Kế thừa từ Account, đại diện cho học sinh
class Student : public Account {
public:
    string studentID;                  // Mã số học sinh
    vector<QuizResult*> results;       // Danh sách kết quả các bài đã làm
    
    // Constructor - khởi tạo học sinh
    Student(string u = "", string p = "", string id = "") 
        : Account(u, p), studentID(id) { 
        role = "Student";  // Gán vai trò là Student
    }

    // Phương thức làm bài kiểm tra
    QuizResult* takeQuiz(Quiz* quiz) {
        map<int,int> answers;  // Map lưu <questionID, answerID>
        
        // Duyệt qua từng câu hỏi trong quiz
        for (Question* q : quiz->questions) {
            // Hiển thị câu hỏi và số điểm
            cout << "Question: " << q->questionText << " (" << q->points << " pts)\n";
            
            // Hiển thị các đáp án
            char idx = 'A';
            for (Answer* a : q->answers) {
                cout << "  " << idx << ") " << a->getText() << " (id=" << a->getID() << ")\n";
                idx++;
            }
            
            // Nhận câu trả lời từ học sinh (theo chữ A, B, C...)
            char ansChar;
            cout << "Your answer (A-" << char('A' + (int)q->answers.size() - 1) << "): ";
            cin >> ansChar;
            ansChar = toupper((unsigned char)ansChar);
            // Kiểm tra đáp án hợp lệ, nếu không hợp lệ yêu cầu nhập lại
            while (ansChar < 'A' || ansChar > char('A' + (int)q->answers.size() - 1)) {
                cout << "Invalid! Enter a letter between A and " << char('A' + (int)q->answers.size() - 1) << ": ";
                cin >> ansChar;
                ansChar = toupper((unsigned char)ansChar);
            }

            // Chuyển ký tự sang chỉ số (1-based)
            int sel = (ansChar - 'A') + 1;
            // Lưu câu trả lời vào map (lưu answerID thực tế)
            answers[q->questionID] = q->answers[sel-1]->getID();
        }
        
        // Tính điểm dựa trên các câu trả lời
        float score = quiz->calculateScore(answers);
        
        // Tạo object QuizResult để lưu kết quả
        QuizResult* res = new QuizResult();
        res->setScore(score);
        
        // Lưu từng câu trả lời vào QuizResult
        for (auto& p : answers) {
            Question* q = nullptr;
            // Tìm câu hỏi tương ứng
            for (Question* qq : quiz->questions) {
                if (qq->questionID == p.first) { 
                    q = qq; 
                    break; 
                }
            }
            // Tìm câu trả lời tương ứng và lưu vào kết quả
            if (q) {
                for (Answer* a : q->answers) {
                    if (a->getID() == p.second) {
                        res->addStudentAnswer(new StudentAnswer(a));
                    }
                }
            }
        }
        
        // Thêm kết quả vào danh sách kết quả của học sinh
        results.push_back(res);
        return res;
    }

    // Xem các kết quả đã làm
    void viewResult() {
        for (QuizResult* r : results) {
            time_t t = r->getDate();
            tm *tm_ptr = localtime(&t);  // Chuyển đổi time_t sang tm
            cout << "Score: " << r->getScore() << "% taken on " 
                 << put_time(tm_ptr, "%c") << "\n";  // Hiển thị điểm và thời gian
        }
    }
};

// ===== CLASS ADMIN (LỚP QUẢN TRỊ/GIÁO VIÊN) =====
// Kế thừa từ Account, đại diện cho giáo viên/admin
class Admin : public Account {
public:
    string teacherID;  // Mã số giáo viên
    
    // Constructor - khởi tạo admin
    Admin(string u = "", string p = "", string id = "") 
        : Account(u, p), teacherID(id) { 
        role = "Admin";  // Gán vai trò là Admin
    }
    
    // Tạo một bài quiz mới
    Quiz* makeQuiz(string title) { 
        return new Quiz(title); 
    }
    
    // Thêm câu hỏi vào quiz
    void manageQuestion(Quiz* quiz) {
        string text; 
        int pts;
        
        // Nhập nội dung câu hỏi
        cout << "Enter question text: "; 
        cin.ignore(); 
        getline(cin, text);
        
        // Nhập số điểm
        cout << "Points: "; 
        cin >> pts;
        
        // Thêm câu hỏi vào quiz
        Question* q = quiz->addQuestion(text, pts);
        cin.ignore();
        
        // Thêm các đáp án cho câu hỏi
        while (true) {
            string at; 
            cout << "Answer text (empty to stop): "; 
            getline(cin, at);
            
            // Nếu không nhập gì thì dừng
            if (at.empty()) break;
            
            // Hỏi đáp án có đúng không
            char ch; 
            cout << "Is correct? (y/n): "; 
            cin >> ch; 
            cin.ignore();
            
            // Thêm đáp án vào câu hỏi
            q->addAnswer(at, (ch == 'y' || ch == 'Y'));
        }
    }
    
    // Xem kết quả (chưa được cài đặt)
    void viewResult(Quiz* quiz) { 
        /* not implemented in this simple sample */ 
    }
};

// ===== HÀM TIỆN ÍCH =====

// Tìm câu hỏi theo ID trong quiz
Question* getQuestionById(Quiz& quiz, int qid) {
    for (Question* q : quiz.questions) {
        if (q->questionID == qid) return q;
    }
    return nullptr;  // Không tìm thấy thì trả về null
}

// ===== HÀM ĐỌC DỮ LIỆU TỪ FILE =====

// Đọc danh sách người dùng từ file users.txt
vector<Account*> loadUsers() {
    vector<Account*> users;  // Vector chứa các tài khoản
    ifstream file("users.txt");  // Mở file để đọc
    
    // Kiểm tra file có mở được không
    if (!file.is_open()) {
        cout << "Error: Cannot open users.txt\n";
        return users;
    }
    
    string u, p, r;  // username, password, role
    
    // Đọc từng dòng trong file
    // Format: username password role
    while (file >> u >> p >> r) {
        // Tạo Student hoặc Admin tùy theo role
        if (r == "Student") 
            users.push_back(new Student(u, p, u));
        else 
            users.push_back(new Admin(u, p, u));
    }
    
    return users;
}

// Tải một bài quiz mẫu (hardcode)
// Trong thực tế có thể đọc từ file hoặc database
Quiz loadQuiz() {
    Quiz q("Basic Math Quiz");  // Tạo quiz với tiêu đề
    
    // Câu hỏi 1: 2 + 2 = ?
    Question* q1 = q.addQuestion("What is 2 + 2?", 1);
    q1->addAnswer("1", false);   // Đáp án sai
    q1->addAnswer("3", false);   // Đáp án sai
    q1->addAnswer("4", true);    // Đáp án đúng

    // Câu hỏi 2: 5 * 6 = ?
    Question* q2 = q.addQuestion("What is 5 * 6?", 2);
    q2->addAnswer("11", false);  // Đáp án sai
    q2->addAnswer("30", true);   // Đáp án đúng
    q2->addAnswer("35", false);  // Đáp án sai

    return q;
}

// ===== KHỞI TẠO BIẾN STATIC =====
// Các biến static dùng để tạo ID tự động tăng
int Question::nextQuestionID = 0;
int Answer::nextAnswerID = 0;
int Quiz::nextQuizID = 0;

// ===== TRIỂN KHAI CÁC PHƯƠNG THỨC =====

// Thêm câu trả lời vào câu hỏi
Answer* Question::addAnswer(string text, bool isCorrect) {
    Answer* a = new Answer(text, isCorrect);  // Tạo object Answer mới
    answers.push_back(a);                      // Thêm vào danh sách
    return a;                                   // Trả về pointer
}

// Kiểm tra câu trả lời có đúng không dựa trên answerID
bool Question::checkAnswer(int answerID) {
    // Duyệt qua các đáp án để tìm đáp án có ID trùng khớp
    for (Answer* a : answers) {
        if (a->getID() == answerID) return a->correct();
    }
    return false;  // Không tìm thấy hoặc sai
}

float Quiz::calculateScore(const map<int, int>& userAnswers) {
    int total = 0;
    int scored = 0;
    
    for (Question* q : questions) {
        total += q->points;
    }
    
    for (auto& p : userAnswers) {
        int qid = p.first; 
        int aid = p.second;
        Question* q = getQuestionById(*this, qid);
        if (q && q->checkAnswer(aid)) {
            scored += q->points;
        }
    }
    
    if (total == 0) return 0.0f;
    return 100.0f * scored / total;
}

// Main program
int main() {
    vector<Account*> users = loadUsers();
    Quiz quiz = loadQuiz();
    
    cout << "=== QUIZ EXAMINATION SYSTEM ===\n";
    
    string u, p;
    cout << "Username: ";
    cin >> u;
    cout << "Password: ";
    cin >> p;
    
    string role = "";
    Account* auth = nullptr;
    
    for (auto acc : users) {
        if (acc->username == u && acc->password == p) { 
            auth = acc; 
            role = acc->role; 
            break; 
        }
    }
    
    if (role == "") {
        cout << "Login failed! Wrong username or password.\n";
        return 0;
    }
    
    if (role == "Student") {
        Student* s = dynamic_cast<Student*>(auth);
        if (s) {
            QuizResult* r = s->takeQuiz(&quiz);
            cout << "\n=== RESULT ===\n";
            cout << "Your score: " << r->getScore() << "%\n";
        }
    }
    else if (role == "Admin") {
        cout << "\n=== ADMIN PANEL ===\n";
        cout << "Hello Admin " << u << "!" << endl;
        cout << "\nList of questions:\n";
        for (Question* q : quiz.questions) {
            cout << "- " << q->questionText << endl;
        }
    }
    
    return 0;
}
