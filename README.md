#C++程序设计--教务信息管理系统
```C++
#include <iostream>
#include <vector>
#include <string>
#include <map>
#include <fstream>
#include <iomanip>
#include <algorithm>
#include <sstream>
#include <conio.h>
#include <cstdlib>
using namespace std;

// 学生结构体
struct Student
{
    string id;
    string name;
    string gender;
    int age;
    string dorm;
    string phone;
    string className;
    string password = "12345"; // 默认密码
};

// 课程结构体
struct Course
{
    string id;                     // 课程编号
    string name;                   // 课程名称
    string time;                   // 上课时间
    string classroom;              // 上课教室
    string teacherId;              // 授课教师工号
    string teacherName;            // 授课教师姓名
    double credit;                 // 学分
    int capacity;                  // 课程容量
    int currentEnrollment = 0;     // 当前选课人数
    bool isRetakeRequired = false; // 是否需要重修
    double retakePassScore = 0;    // 重修通过的最低分数
};

// 选课记录结构体
struct Selection
{
    string studentId;
    string courseId;
    double examScore = 0;
    double usualScore = 0;
    double totalScore = 0;
    bool isRetake = false;
    int retakeCount = 0;
    string retakeSemester;
};

// 教师结构体
struct Teacher
{
    string id;                 // 工号
    string name;               // 姓名
    string password = "12345"; // 默认密码
    vector<string> courseIds;  // 教师所教课程编号列表
};

// 全局变量
vector<Student> students;
vector<Course> courses;
vector<Selection> selections;
vector<Teacher> teachers;

// 文件路径
const string STUDENT_FILE = "students.txt";
const string COURSE_FILE = "courses.txt";
const string SELECTION_FILE = "selections.txt";
const string TEACHER_FILE = "teachers.txt";

// 函数声明
void adminMenu();
void sclass_name(Student &stu);
void teacherMenu(Teacher &t);
void studentMenu(Student &stu);
void saveStudents();
void saveCourses();
void saveSelections();
void saveTeachers();
void loadStudents();
void loadCourses();
void loadSelections();
void loadTeachers();
string getPassword();
double scoreToGPA(double score);
Student *findStudentById(const string &id);
Course *findCourseById(const string &id);
Teacher *findTeacherById(const string &id);
void selectCourse(Student &stu);
void viewSchedule(Student &stu);                               // 声明 viewSchedule 函数
void queryGrades(Student &stu);                                // 声明 queryGrades 函数
void dropCourse(Student &stu);                                 // 声明 dropCourse 函数
void modifyPassword(Student &stu);
void printHeader(const string &header);
void printColoredText(const string &text, const string &color);
bool isTimeConflict(const string &time1, const string &time2); // 声明时间冲突检测函数
void clearScreen()
{
#if defined(_WIN32) || defined(_WIN64)
    system("cls");  // Windows-specific
#else
    clearScreen();
#endif
}
#define RESET   "\033[0m"
#define RED     "\033[31m"
#define GREEN   "\033[32m"
#define YELLOW  "\033[33m"
#define BLUE    "\033[34m"
#define MAGENTA "\033[35m"
#define CYAN    "\033[36m"
#define WHITE   "\033[37m"
#define BOLD    "\033[1m"
#define GRAY    "\033[90m"
#define UNDERLINE "\033[4m"
void printHeader(const string &header)
{
    cout << BOLD << YELLOW << header << RESET << endl;
}
void printColoredText(const string &text, const string &color)
{
    cout << color << text << RESET << endl;
}
void printCentered(const string &text, const string &color = RESET)
{
    int terminalWidth = 80;  // Default width, you can adjust it as needed
    int padding = (terminalWidth - text.length()) / 2;  // Calculate the spaces before text
    cout << string(padding, ' ') << color << text << RESET << endl;  // Print the text with calculated padding
}
void modifyPassword(Student &stu)
{
    clearScreen();

    string pad(15, ' ');
    cout << BOLD << GREEN;
    cout << pad << "╔════════════════════════════════════╗\n";
    cout << pad << "║         修改密码ing                ║\n";
    cout << pad << "╚════════════════════════════════════╝\n" << RESET;

    string newPass, confirmPass;
    while (true)
    {
        cout << CYAN << "\n请输入新密码: ";
        cin >> newPass;
        cout << "请再次输入新密码: ";
        cin >> confirmPass;

        if (newPass == confirmPass) break;
        cout << RED << "两次输入的密码不一致，请重新输入。\n" << RESET;
    }

    stu.password = newPass;
    saveStudents();

    cout << GREEN << "\n密码修改成功！\n" << RESET;

    char choice;
    cout << CYAN << "\n按 'b' 返回学生主菜单：";
    cin >> choice;
    if (choice == 'b' || choice == 'B') return;
    else modifyPassword(stu);
}

void syncTeacherCourses()
{
    for (auto &teacher : teachers)
    {
        teacher.courseIds.clear();
    }

    for (const auto &course : courses)
    {
        Teacher *teacher = findTeacherById(course.teacherId);
        if (teacher)
        {
            teacher->courseIds.push_back(course.id);
        }
    }

    saveTeachers();
}

// 初始化文件
void initFileIfNotExist(const string &filename)
{
    ifstream f(filename);
    if (!f.good())
    {
        ofstream newFile(filename);
        newFile.close();
    }
}

// 保存学生数据
void saveStudents()
{
    ofstream file(STUDENT_FILE);
    for (const auto &s : students)
    {
        file << s.id << " " << s.name << " " << s.gender << " "
             << s.age << " " << s.dorm << " " << s.phone << " "
             << s.className << " " << s.password << "\n";
    }
    file.close();
}

// 保存课程数据
void saveCourses()
{
    ofstream file(COURSE_FILE);
    if (!file.is_open())
    {
        cout << "无法打开课程文件！\n";
        return;
    }

    for (const auto &c : courses)
    {
        file << c.id << "|" << c.name << "|" << c.time << "|"
             << c.classroom << "|" << c.teacherId << "|" << c.teacherName << "|"
             << c.credit << "|" << c.capacity << "|" << c.currentEnrollment << "|"
             << c.isRetakeRequired << "|" << c.retakePassScore << "\n";
    }
    file.close();
}

// 保存选课记录
void saveSelections()
{
    ofstream file(SELECTION_FILE);
    for (const auto &sel : selections)
    {
        file << sel.studentId << " " << sel.courseId << " "
             << sel.examScore << " " << sel.usualScore << " "
             << sel.totalScore << " " << sel.isRetake << " "
             << sel.retakeCount << " " << sel.retakeSemester << "\n";
    }
    file.close();
}

// 保存教师数据
void saveTeachers()
{
    ofstream file(TEACHER_FILE);
    for (const auto &t : teachers)
    {
        file << t.id << " " << t.name << " " << t.password;
        for (const auto &courseId : t.courseIds)
        {
            file << " " << courseId;
        }
        file << "\n";
    }
    file.close();
}

// 加载学生数据
void loadStudents()
{
    ifstream file(STUDENT_FILE);
    students.clear();
    string line;
    while (getline(file, line))
    {
        stringstream ss(line);
        Student s;
        ss >> s.id >> s.name >> s.gender >> s.age >> s.dorm >> s.phone >> s.className >> s.password;
        students.push_back(s);
    }
    file.close();
}

// 加载课程数据
void loadCourses()
{
    ifstream file(COURSE_FILE);
    if (!file.is_open())
    {
        cout << "无法打开课程文件！\n";
        return;
    }

    courses.clear();
    string line;
    while (getline(file, line))
    {
        // 跳过空行
        if (line.empty())
        {
            continue;
        }

        stringstream ss(line);
        Course c;

        // 按顺序解析字段
        if (!getline(ss, c.id, '|'))
        {
            cout << "解析课程编号失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, c.name, '|'))
        {
            cout << "解析课程名称失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, c.time, '|'))
        {
            cout << "解析上课时间失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, c.classroom, '|'))
        {
            cout << "解析上课教室失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, c.teacherId, '|'))
        {
            cout << "解析授课教师工号失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, c.teacherName, '|'))
        {
            cout << "解析授课教师姓名失败，跳过该行：" << line << "\n";
            continue;
        }

        // 解析数字字段
        string creditStr, capacityStr, currentEnrollmentStr, isRetakeRequiredStr, retakePassScoreStr;
        if (!getline(ss, creditStr, '|'))
        {
            cout << "解析学分失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, capacityStr, '|'))
        {
            cout << "解析课程容量失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, currentEnrollmentStr, '|'))
        {
            cout << "解析当前选课人数失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, isRetakeRequiredStr, '|'))
        {
            cout << "解析是否需要重修失败，跳过该行：" << line << "\n";
            continue;
        }
        if (!getline(ss, retakePassScoreStr, '|'))
        {
            cout << "解析重修通过的最低分数失败，跳过该行：" << line << "\n";
            continue;
        }

        // 转换数字字段
        try
        {
            c.credit = stod(creditStr);
            c.capacity = stoi(capacityStr);
            c.currentEnrollment = stoi(currentEnrollmentStr);
            c.isRetakeRequired = (isRetakeRequiredStr == "1");
            c.retakePassScore = stod(retakePassScoreStr);
        }
        catch (const exception &e)
        {
            cout << "数字字段转换失败，跳过该行：" << line << "\n";
            continue;
        }

        courses.push_back(c);
    }
    file.close();
}

// 加载选课记录
void loadSelections()
{
    ifstream file(SELECTION_FILE);
    selections.clear();
    string line;
    while (getline(file, line))
    {
        stringstream ss(line);
        Selection sel;
        ss >> sel.studentId >> sel.courseId >> sel.examScore >> sel.usualScore >> sel.totalScore >> sel.isRetake >> sel.retakeCount >> sel.retakeSemester;
        selections.push_back(sel);
    }
    file.close();
}

// 加载教师数据
void loadTeachers()
{
    ifstream file(TEACHER_FILE);
    teachers.clear();
    string line;
    while (getline(file, line))
    {
        stringstream ss(line);
        Teacher t;
        ss >> t.id >> t.name >> t.password;

        string courseId;
        while (ss >> courseId)
        {
            t.courseIds.push_back(courseId);
        }

        teachers.push_back(t);
    }
    file.close();
}
void modifyPassword(Teacher &t)
{
    clearScreen();

    string pad(15, ' ');
    cout << BOLD << GREEN;
    cout << pad << "╔════════════════════════════════════╗\n";
    cout << pad << "║         修改密码ing            ║\n";
    cout << pad << "╚════════════════════════════════════╝\n" << RESET;

    string newPass, confirmPass;
    while (true)
    {
        cout << CYAN << "\n请输入新密码: ";
        cin >> newPass;
        cout << "请再次输入新密码: ";
        cin >> confirmPass;

        if (newPass == confirmPass) break;
        cout << RED << "两次输入的密码不一致，请重新输入。\n" << RESET;
    }

    t.password = newPass;
    saveTeachers();

    cout << GREEN << "\n密码修改成功！\n" << RESET;

    char choice;
    cout << CYAN << "\n按 'b' 返回教师主菜单，按任意键继续修改密码：";
    cin >> choice;
    if (choice == 'b' || choice == 'B') return;
    else modifyPassword(t);
}


// 密码输入函数
string getPassword()
{
    string password;
    char ch;
    while ((ch = _getch()) != '\r')
    {
        if (ch == '\b')
        {
            if (!password.empty())
            {
                cout << "\b \b";
                password.pop_back();
            }
        }
        else
        {
            cout << '*';
            password.push_back(ch);
        }
    }
    cout << endl;
    return password;
}

// 查找学生
Student *findStudentById(const string &id)
{
    for (auto &s : students)
    {
        if (s.id == id)
            return &s;
    }
    return nullptr;
}

// 查找课程
Course *findCourseById(const string &id)
{
    for (auto &c : courses)
    {
        if (c.id == id)
            return &c;
    }
    return nullptr;
}

// 查找教师
Teacher *findTeacherById(const string &id)
{
    for (auto &t : teachers)
    {
        if (t.id == id)
            return &t;
    }
    return nullptr;
}
double scoreToGPA(double examScore, double usualScore)
{
    double totalScore = usualScore * 0.5 + examScore * 0.5;

    if (examScore < 60)
    {
        // Fail if exam score is below 60
        return 0.0;
    }
    else if (totalScore >= 90)
    {
        return 4.0 + (totalScore - 90) / 10.0;
    }
    else if (totalScore >= 80)
    {
        return 3.0 + (totalScore - 80) / 10.0;
    }
    else if (totalScore >= 70)
    {
        return 2.0 + (totalScore - 70) / 10.0;
    }
    else if (totalScore >= 60)
    {
        return 1.0 + (totalScore - 60) / 10.0;
    }
    else
    {
        return 0.0; // Shouldn't reach here because of the exam score condition
    }
}


// 管理员菜单
void adminMenu()
{
    int choice;
    do
    {
        clearScreen();
        string welcomeText = "Hello，管理员，欢迎登录教务系统！";
        int width = 80;
        int padding = (width - welcomeText.length()) / 2;
        cout << BOLD << YELLOW;
        cout << string(padding, ' ') << welcomeText << "\n\n" << RESET;
        printHeader(" 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓 管理员菜单 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓");
        printf("\n");
        printf("                 ┌────────────────────────────────────────────────┐\n");
        printColoredText("                 │                 1. 添加学生                ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 2. 添加课程         ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 3. 添加教师         ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 4. 查看所有学生          ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 5. 查看所有课程                         ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 6. 查看所有教师                ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 0. 返回               ", CYAN);
        printf("                 └────────────────────────────────────────────────┘\n\n\n\n\n");
        cout << "\n选择: ";
        cin >> choice;

        // 清理输入缓冲区，防止输入错误导致问题
        if (cin.fail())
        {
            cin.clear();                                         // 清除错误标志
            cin.ignore(numeric_limits<streamsize>::max(), '\n'); // 丢弃无效输入
            cout << "无效输入，请重新选择！\n";
            continue;
        }

        switch (choice)
        {
        case 1:
{
    while (true)
    {
        clearScreen();
        cout << BOLD << GREEN;
        cout << "╔════════════════════════════════════════════════════════════════════╗\n";
        cout << "║                          添加学生信息                              ║\n";
        cout << "╚════════════════════════════════════════════════════════════════════╝\n" << RESET;

        Student s;
        cout << CYAN << "\n输入学生学号: " << RESET;
        cin >> s.id;

        // 检查学生学号是否已存在
        if (findStudentById(s.id))
        {
            cout << RED << "\n学生学号已存在！\n" << RESET;
            cout << CYAN << "\n请选择操作：\n";
            cout << "1. 继续添加学生\n";
            cout << "2. 返回管理员菜单\n";
            cout << "选择: " << RESET;

            int option;
            cin >> option;

            if (option == 2)
                break; // 返回管理员菜单
            else
                continue; // 继续添加学生
        }

        cout << CYAN << "输入学生姓名: " << RESET;
        cin >> s.name;
        cout << CYAN << "输入学生性别: " << RESET;
        cin >> s.gender;
        cout << CYAN << "输入学生年龄: " << RESET;
        cin >> s.age;
        cout << CYAN << "输入学生宿舍: " << RESET;
        cin >> s.dorm;
        cout << CYAN << "输入学生电话: " << RESET;
        cin >> s.phone;
        cout << CYAN << "输入学生班级: " << RESET;
        cin >> s.className;

        // 添加学生到列表
        students.push_back(s);

        // 保存学生数据
        saveStudents();
        cout << GREEN << "\n学生添加成功！\n" << RESET;

        char choice;
        cout << CYAN << "\n输入 'C' 继续添加学生，输入 'B' 返回管理员菜单: " << RESET;
        cin >> choice;
        if (choice == 'b' || choice == 'B')
            break;
    }
    break;
}
      case 2:
{
    while (true)
    {
        clearScreen();
        cout << BOLD << GREEN;
        cout << "╔════════════════════════════════════════════════════════════════════╗\n";
        cout << "║                          添加课程信息                              ║\n";
        cout << "╚════════════════════════════════════════════════════════════════════╝\n" << RESET;

        Course c;
        cout << CYAN << "\n输入课程编号: " << RESET;
        cin >> c.id;

        // 检查课程编号是否已存在
        if (findCourseById(c.id))
        {
            cout << RED << "\n课程编号已存在！\n" << RESET;
            cout << CYAN << "\n请选择操作：\n";
            cout << "1. 继续添加课程\n";
            cout << "2. 返回管理员菜单\n";
            cout << "选择: " << RESET;

            int option;
            cin >> option;

            if (option == 2)
                break; // 返回管理员菜单
            else
                continue; // 继续添加课程
        }

        cout << CYAN << "输入课程名称: " << RESET;
        cin.ignore();
        getline(cin, c.name);
        cout << CYAN << "输入上课时间 (格式: 周X第Y-Z节): " << RESET;
        getline(cin, c.time);
        cout << CYAN << "输入上课教室: " << RESET;
        getline(cin, c.classroom);
        cout << CYAN << "输入授课教师工号: " << RESET;
        cin >> c.teacherId;

        // 检查教师是否存在
        Teacher *teacher = findTeacherById(c.teacherId);
        if (!teacher)
        {
            cout << RED << "教师工号不存在！\n" << RESET;
            continue;
        }
        c.teacherName = teacher->name;

        cout << CYAN << "输入学分: " << RESET;
        cin >> c.credit;
        cout << CYAN << "输入课程容量: " << RESET;
        cin >> c.capacity;

        // 是否需要重修
        char retakeOption;
        cout << CYAN << "该课程是否需要重修？(y/n): " << RESET;
        cin >> retakeOption;
        if (retakeOption == 'y' || retakeOption == 'Y')
        {
            c.isRetakeRequired = true;
            cout << CYAN << "请输入重修通过的最低分数: " << RESET;
            cin >> c.retakePassScore;
        }

        // 添加课程到列表
        courses.push_back(c);

        // 保存课程数据
        saveCourses();
        saveTeachers();
        cout << GREEN << "\n课程添加成功！\n" << RESET;

        char choice;
        cout << CYAN << "\n输入 'C' 继续添加课程，输入 'B' 返回管理员菜单: " << RESET;
        cin >> choice;
        if (choice == 'b' || choice == 'B')
            break;
    }
    break;
}
        case 3:
{
    while (true)
    {
        clearScreen();
        cout << BOLD << GREEN;
        cout << "╔════════════════════════════════════════════════════════════════════╗\n";
        cout << "║                          添加教师信息                              ║\n";
        cout << "╚════════════════════════════════════════════════════════════════════╝\n" << RESET;

        Teacher t;
        cout << CYAN << "\n输入教师工号: " << RESET;
        cin >> t.id;

        // 检查教师工号是否已存在
        if (findTeacherById(t.id))
        {
            cout << RED << "\n教师工号已存在！\n" << RESET;
            cout << CYAN << "\n请选择操作：\n";
            cout << "1. 继续添加教师\n";
            cout << "2. 返回管理员菜单\n";
            cout << "选择: " << RESET;

            int option;
            cin >> option;

            if (option == 2)
                break; // 返回管理员菜单
            else
                continue; // 继续添加教师
        }

        cout << CYAN << "输入教师姓名: " << RESET;
        cin >> t.name;

        // 添加教师到列表
        teachers.push_back(t);

        // 保存教师数据
        saveTeachers();
        cout << GREEN << "\n教师添加成功！\n" << RESET;

        char choice;
        cout << CYAN << "\n输入 'C' 继续添加教师，输入 'B' 返回管理员菜单: " << RESET;
        cin >> choice;
        if (choice == 'b' || choice == 'B')
            break;
    }
    break;
}
        case 4:
        {
            while (true)
            {
                clearScreen();
                cout << BOLD << GREEN;

                // 顶部边框 = 每列总宽度之和（精确匹配）
                cout << "╔════════════════════════════════════════════════════════════════════════════════════════════╗\n";

                // 字段行对齐
                cout << left
                     << "║ " << setw(10) << "学号"
                     << setw(15) << "姓名"
                     << setw(8)  << "性别"
                     << setw(6)  << "年龄"
                     << setw(12) << "宿舍"
                     << setw(15) << "电话"
                     << setw(10) << "班级"
                     << setw(15) << "密码" << "║\n";

                // 底部边框
                cout << "╚════════════════════════════════════════════════════════════════════════════════════════════╝\n";

                // 内容行
                for (const auto& s : students)
                {
                    cout << left
                         << "║ " << setw(10) << s.id
                         << setw(15) << s.name
                         << setw(8)  << s.gender
                         << setw(6)  << s.age
                         << setw(12) << s.dorm
                         << setw(15) << s.phone
                         << setw(10) << s.className
                         << setw(15) << s.password << "║\n";
                }

                cout << CYAN << "\n按 'B' 返回管理员菜单: ";
                char choice;
                cin >> choice;
                if (choice == 'b' || choice == 'B') break;
            }
            break;
        }

        case 5:
            while (true)
            {
                clearScreen(); // 清空屏幕
                cout << BOLD << GREEN;
                cout << "╔══════════════════════════════════════════════════════════════════════════════════════════════════\n";
                cout << "║ 课程编号   课程名称           授课教师       学分    容量   已选人数   上课时间               \n";
                cout << "╟──────────────────────────────────────────────────────────────────────────────────────────────────\n";

                for (const auto &c : courses)
                {
                    cout << "║ " << left
                         << setw(10) << c.id
                         << setw(20) << c.name
                         << setw(15) << c.teacherName
                         << fixed << setprecision(1)
                         << setw(8) << c.credit
                         << setw(8) << c.capacity
                         << setw(10) << c.currentEnrollment
                         << setw(20) << c.time << "\n";
                }

                cout << "╚══════════════════════════════════════════════════════════════════════════════════════════════════\n";

                cout << CYAN << "\n共有课程数：" << courses.size() << " 门";
                cout << CYAN << "\n按 'B' 返回管理员菜单: ";
                char choice;
                cin >> choice;
                if (choice == 'b' || choice == 'B')
                    break; // 返回管理员菜单
            }
            break;
        case 6:
            while (true)
            {
                clearScreen();
                cout << BOLD << GREEN;

                // 表头上边框
                cout << "╔════════════════════════════════════════════════════════════════════╗\n";
                cout << "║ 工号        姓名                    密码                          \n";
                cout << "╟────────────────────────────────────────────────────────────────────╢\n";

                // 输出内容（无右边框）
                for (const auto &t : teachers)
                {
                    cout << "║ " << left
                         << setw(10) << t.id
                         << setw(24) << t.name
                         << setw(30) << t.password
                         << endl;
                }

                // 下边框删掉右角
                cout << "╚════════════════════════════════════════════════════════════════════╝\n";

                cout << GREEN << "\n共有教师人数：" << teachers.size() << " 人\n";
                cout << CYAN << "\n按 'B' 返回管理员菜单: ";
                char choice;
                cin >> choice;
                if (choice == 'b' || choice == 'B')
                    break; // 返回管理员菜单
            }
            break;
        case 0:
            return;
        default:
            cout << "无效选择！\n";
        }
    }
    while (true);
}

void teacherMenu(Teacher &t)
{
    int choice;
    do
    {
        clearScreen(); // 清空屏幕
        string welcomeText = "Hello，" + t.name + "老师，欢迎登录教务系统！";
        int width = 80;
        int padding = (width - welcomeText.length()) / 2;
        cout << BOLD << YELLOW;
        cout << string(padding, ' ') << welcomeText << "\n\n" << RESET;

        printHeader(" 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓 教师菜单 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓"); // Print the header in bold yellow
        printf("\n");
        printf("                 ┌────────────────────────────────────────────────┐\n");
        printColoredText("                 │             1. 录入学生成绩                ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │         2. 按课程统计学生名单及成绩         ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │        3. 按班级统计学生选课情况及成绩         ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │       4. 查看学生已修学分及不及格课程           ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │               5. 修改密码                          ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │                 0. 返回                          ", CYAN);
        printf("                 └────────────────────────────────────────────────┘\n\n\n\n\n");
        cout << "\n选择: ";
        cin >> choice;

        switch (choice)
        {
        case 1:
        {
            clearScreen();
            cout << BOLD << GREEN;
            cout << "\n╔══════════════════════════════╗\n";
            cout <<   "║      录入学生成绩 ing        ║\n";
            cout <<   "╚══════════════════════════════╝\n" << RESET;

            string courseId;
            cout << CYAN << "\n请输入课程编号：";
            cin >> courseId;

            Course* course = findCourseById(courseId);
            if (!course || course->teacherId != t.id)
            {
                cout << RED << (course ? "您无权录入该课程的成绩！" : "课程编号不存在！") << endl;
            }
            else
            {
                bool found = false;
                cout << BOLD << YELLOW;
                cout << "\n╔══════════════════════════════════════════════════╗\n";
                cout <<   "║ 学号           姓名           平时成绩   考试成绩║\n";
                cout <<   "╟──────────────────────────────────────────────────╢\n";

                for (auto& sel : selections)
                {
                    if (sel.courseId == courseId)
                    {
                        Student* student = findStudentById(sel.studentId);
                        if (student)
                        {
                            found = true;
                            cout << WHITE << "║ " << left << setw(14) << student->id
                                 << setw(14) << student->name;
                            cout << CYAN << "\n  平时成绩：";
                            cin >> sel.usualScore;
                            cout << "  考试成绩：";
                            cin >> sel.examScore;
                            sel.totalScore = min(100.0, sel.usualScore * 0.5 + sel.examScore * 0.5);
                            cout << GREEN << "  成绩已录入：" << fixed << setprecision(1) << sel.totalScore << "\n";
                            cout << YELLOW << "╟──────────────────────────────────────────────────╢\n";
                        }
                    }
                }

                if (!found)
                    cout << RED << "该课程暂无学生选课。\n";
                else
                {
                    saveSelections();
                    cout << GREEN << " 成绩录入完成！\n";
                }
            }

            cout << CYAN << "\n按任意键返回教师菜单：";
            cin.ignore();
            cin.get();
            break;
        }


        case 2:
        {
            clearScreen(); // 清空屏幕
            string pad(15, ' '); // 居中用 padding

            // 顶部框放在输入之前
            cout << BOLD << GREEN;
            cout << pad << "╔════════════════════════════════════════╗\n";
            cout << pad << "║       查看课程成绩 ing...              ║\n";
            cout << pad << "╚════════════════════════════════════════╝\n\n" << RESET;

            string courseId;
            cout << CYAN << "请输入课程编号：";
            cin >> courseId;

            Course *course = findCourseById(courseId);
            if (!course)
            {
                cout << RED << "课程编号不存在！\n";
                break;
            }

            // 表头输出
            cout << BOLD << WHITE;
            cout << pad << "╔══════════════════════════════════════════════════════════════════════\n";
            cout << pad << "║ 学生名单及成绩（课程：" << course->name << "）\n";
            cout << pad << "╟──────────────────────────────────────────────────────────────────────\n";

            // 列头
            cout << pad << WHITE << "║ "
                 << left
                 << setw(10) << "学号"
                 << setw(15) << "姓名"
                 << setw(12) << "平时成绩"
                 << setw(12) << "考试成绩"
                 << setw(12) << "总成绩" << "\n";

            cout << pad << "╟──────────────────────────────────────────────────────────────────────\n";

            bool hasData = false;
            for (const auto &sel : selections)
            {
                if (sel.courseId == courseId)
                {
                    Student *stu = findStudentById(sel.studentId);
                    if (stu)
                    {
                        hasData = true;
                        string color = (sel.totalScore >= 60 ? GREEN : RED);

                        cout << pad << "║ "
                             << left
                             << setw(10) << stu->id
                             << setw(15) << stu->name
                             << fixed << setprecision(1)
                             << setw(12) << sel.usualScore
                             << setw(12) << sel.examScore
                             << color << setw(12) << sel.totalScore << RESET << "\n";
                    }
                }
            }

            if (!hasData)
            {
                cout << pad << RED << "║        该课程暂无学生成绩记录。\n" << RESET;
            }

            cout << pad << BOLD << WHITE;
            cout << "╚══════════════════════════════════════════════════════════════════════\n" << RESET;

            cout << CYAN << "\n按 'b' 返回教师菜单: ";
            char choice;
            cin >> choice;
            if (choice == 'b' || choice == 'B') break;
        }
        case 3:
        {
            clearScreen();
            string pad(15, ' '); // 居中空格

            // 顶部标题框
            cout << BOLD << GREEN;
            cout << pad << "╔════════════════════════════════════════╗\n";
            cout << pad << "║        查看班级成绩 ing                ║\n";
            cout << pad << "╚════════════════════════════════════════╝\n\n" << RESET;

            string className;
            cout << CYAN << "请输入班级名称：";
            cin >> className;

            cout << BOLD << WHITE;
            cout << pad << "╔══════════════════════════════════════════════════════════════════════\n";
            cout << pad << "║ 班级学生选课情况及成绩（" << className << "）\n";
            cout << pad << "╟──────────────────────────────────────────────────────────────────────\n";

            cout << pad << WHITE
                 << "║ " << left
                 << setw(10) << "学号"
                 << setw(15) << "姓名"
                 << setw(12) << "课程编号"
                 << setw(20) << "课程名称"
                 << setw(10) << "总成绩" << "\n";

            cout << pad << "╟──────────────────────────────────────────────────────────────────────\n";

            bool found = false;
            for (const auto &sel : selections)
            {
                Student *student = findStudentById(sel.studentId);
                if (student && student->className == className)
                {
                    Course *course = findCourseById(sel.courseId);
                    if (course)
                    {
                        found = true;
                        string color = (sel.totalScore >= 60 ? GREEN : RED);
                        cout << pad << "║ "
                             << left
                             << setw(10) << student->id
                             << setw(15) << student->name
                             << setw(12) << course->id
                             << setw(20) << course->name
                             << color << setw(10) << sel.totalScore << RESET << "\n";
                    }
                }
            }

            if (!found)
            {
                cout << pad << RED << "║            该班级无成绩记录。\n" << RESET;
            }

            cout << pad << BOLD << WHITE;
            cout << "╚══════════════════════════════════════════════════════════════════════\n" << RESET;

            cout << CYAN << "\n按 'b' 返回教师菜单，按任意键继续查看其他：";
            string back;
            cin >> back;
            if (back == "b" || back == "B") break;
        }

        case 4:
        {
            clearScreen(); // 清空屏幕
            string pad(15, ' '); // 用于居中
            string studentId;

            cout << BOLD << GREEN;
            cout << pad << "╔════════════════════════════════════════╗\n";
            cout << pad << "║        查看学生成绩 ing                ║\n";
            cout << pad << "╚════════════════════════════════════════╝\n\n" << RESET;

            cout << CYAN << "请输入学生学号：";
            cin >> studentId;

            Student *student = findStudentById(studentId);
            if (!student)
            {
                cout << RED << "学生学号不存在！\n";
                break;
            }

            cout << BOLD << WHITE;
            cout << pad << "╔══════════════════════════════════════════════════════════════════════\n";
            cout << pad << "║ 学生成绩记录（" << student->name << "）\n";
            cout << pad << "╟──────────────────────────────────────────────────────────────────────\n";

            cout << pad << "║ " << left
                 << setw(12) << "课程编号"
                 << setw(20) << "课程名称"
                 << setw(12) << "平时成绩"
                 << setw(12) << "考试成绩"
                 << setw(12) << "总成绩" << "\n";

            cout << pad << "╟──────────────────────────────────────────────────────────────────────\n";

            bool found = false;
            double totalCredits = 0;
            for (const auto &sel : selections)
            {
                if (sel.studentId == studentId)
                {
                    Course *course = findCourseById(sel.courseId);
                    if (course)
                    {
                        found = true;
                        string color = (sel.totalScore >= 60 ? GREEN : RED);
                        cout << pad << "║ "
                             << left
                             << setw(12) << course->id
                             << setw(20) << course->name
                             << fixed << setprecision(1)
                             << setw(12) << sel.usualScore
                             << setw(12) << sel.examScore
                             << color << setw(12) << sel.totalScore << RESET << "\n";

                        if (sel.totalScore >= 60)
                            totalCredits += course->credit;
                    }
                }
            }

            if (!found)
            {
                cout << pad << RED << "║          该学生暂无成绩记录。\n" << RESET;
            }

            cout << pad << BOLD << WHITE;
            cout << "╚══════════════════════════════════════════════════════════════════════\n" << RESET;

            if (found)
            {
                cout << CYAN << "\n已修学分：" << fixed << setprecision(1) << totalCredits << RESET << endl;
            }

            cout << CYAN << "\n按任意键返回教师菜单：";
            cin.ignore();
            cin.get();
            break;
        }

        case 5:
        {
            clearScreen(); // 清空屏幕
            modifyPassword(t);
        }
        case 0:
            return; // 返回到教师登录后的主菜单
        default:
            cout << "无效选择！\n";
        }
    }
    while (true);
}

// 学生菜单
void studentMenu(Student &stu)
{
    int choice;
    do
    {
        clearScreen();
        string welcomeText = "Hello，" + stu.name + "同学，欢迎登录教务系统！";
        int width = 80;
        int padding = (width - welcomeText.length()) / 2;
        cout << BOLD << YELLOW;
        cout << string(padding, ' ') << welcomeText << "\n\n" << RESET;
        printHeader(" 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓 学生菜单 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓"); // Print the header in bold yellow
        printf("\n");
        printf("                 ┌────────────────────────────────────────────────┐\n");
        printColoredText("                 │             1. 选课              ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │             2. 查看课表        ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │             3. 查询成绩         ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │             4. 修改密码           ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │             5. 退选课程           ", CYAN);
        printf("                 ┌────────────────────────────────────────────────┤\n");
        printColoredText("                 │             0. 返回                          ", CYAN);
        printf("                 └────────────────────────────────────────────────┘\n\n\n\n\n");
        cout << "\n选择: ";
        cin >> choice;

        switch (choice)
        {
        case 1:
            clearScreen(); // 清空屏幕
            selectCourse(stu); // 调用选课功能
            break;
        case 2:
            clearScreen(); // 清空屏幕
            viewSchedule(stu); // 调用查看课表功能
            break;
        case 3:
            clearScreen(); // 清空屏幕
            queryGrades(stu); // 调用查询成绩功能
            break;
        case 4:
            clearScreen(); // 清空屏幕
            {
                modifyPassword(stu);
                break;
            }
        case 5:
            clearScreen(); // 清空屏幕
            dropCourse(stu); // 调用退选功能
            break;
        case 0:
            return;
        default:
            cout << "无效选择！\n";
        }
    }
    while (true);
}

// 时间冲突检测函数
bool isTimeConflict(const string &time1, const string &time2)
{
    // 假设时间格式为 "周X第Y-Z节"
    // 解析时间字符串
    auto parseTime = [](const string &time)
    {
        char day;
        int start, end;
        sscanf(time.c_str(), "周%c第%d-%d节", &day, &start, &end);
        return make_tuple(day, start, end);
    };

    auto parsedTime1 = parseTime(time1);
    auto parsedTime2 = parseTime(time2);

    char day1 = std::get<0>(parsedTime1);
    int start1 = std::get<1>(parsedTime1);
    int end1 = std::get<2>(parsedTime1);

    char day2 = std::get<0>(parsedTime2);
    int start2 = std::get<1>(parsedTime2);
    int end2 = std::get<2>(parsedTime2);

    // 检查是否在同一天且时间段有重叠
    if (day1 == day2 && !(end1 < start2 || end2 < start1))
    {
        return true;
    }
    return false;
}
void selectCourse(Student &stu)
{
    if (courses.empty())
    {
        cout << "当前没有可选课程！\n";
        return;
    }
    while (true)
    {
        cout << BOLD << YELLOW << "\n〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓 可选课程列表 〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓\n";
        cout << BOLD << YELLOW << left << setw(10) << "课程编号" << setw(20) << "课程名称" << setw(15) << "授课教师"
             << setw(10) << "学分" << setw(10) << "容量" << setw(10) << "已选人数" << setw(15) << "上课时间" << RESET << endl;

        for (const auto &c : courses)
        {
            cout << left << setw(10) << c.id << setw(20) << c.name << setw(15) << c.teacherName
                 << setw(10) << c.credit << setw(10) << c.capacity << setw(10) << c.currentEnrollment
                 << setw(15) << c.time << endl;
        }
        cout << BOLD << YELLOW << string(90, '_') << RESET << endl;
        cout << endl;
        string courseId;
        cout << "请输入要选择的课程编号（输入0返回）：";
        cin >> courseId;

        if (courseId == "0")
            return; // 返回学生菜单

        Course *course = findCourseById(courseId);
        if (!course)
        {
            cout << "课程编号不存在，请重新输入！\n";
            continue;
        }

        // 检查课程是否已满
        if (course->currentEnrollment >= course->capacity)
        {
            cout << "该课程已满，无法选择！请重新选择其他课程。\n";
            continue;
        }

        // 检查是否已选
        bool alreadySelected = false;
        for (const auto &sel : selections)
        {
            if (sel.studentId == stu.id && sel.courseId == courseId)
            {
                alreadySelected = true;
                break;
            }
        }
        if (alreadySelected)
        {
            cout << "您已选择该课程，无法重复选择！请重新选择其他课程。\n";
            continue;
        }

        // 检查时间冲突
        bool timeConflict = false;
        for (const auto &sel : selections)
        {
            if (sel.studentId == stu.id)
            {
                Course *selectedCourse = findCourseById(sel.courseId);
                if (selectedCourse && selectedCourse->time == course->time)
                {
                    cout << RED << "时间冲突！您已选择了 " << selectedCourse->name
                         << "，上课时间为 " << selectedCourse->time << "。\n" << RESET;

                    // 提供继续选课和退出选项
                    char conflictChoice;
                    cout << CYAN << "\n是否继续选课？(y: 继续, b: 返回学生菜单): " << RESET;
                    cin >> conflictChoice;

                    if (conflictChoice == 'y' || conflictChoice == 'Y')
                    {
                        timeConflict = true;
                        break; // 继续选课
                    }
                    else if (conflictChoice == 'b' || conflictChoice == 'B')
                    {
                        return; // 返回学生菜单
                    }
                    else
                    {
                        cout << "无效输入，返回学生菜单。\n";
                        return; // 默认返回学生菜单
                    }
                }
            }
        }

        if (timeConflict)
            continue;

        // 添加选课记录
        Selection sel;
        sel.studentId = stu.id;
        sel.courseId = courseId;
        selections.push_back(sel);

        // 更新课程的已选人数
        course->currentEnrollment++;

        saveSelections();
        saveCourses();

        cout << GREEN << "选课成功！\n" << RESET;

        // 提示是否继续选课
        char continueChoice;
        cout << "\n是否继续选课？(y: 继续, b: 返回学生菜单): ";
        cin >> continueChoice;

        if (continueChoice == 'y' || continueChoice == 'Y')
        {
            continue; // 继续选课
        }
        else if (continueChoice == 'b' || continueChoice == 'B')
        {
            return; // 返回学生菜单
        }
        else
        {
            cout << "无效输入，返回学生菜单。\n";
            return; // 默认返回学生菜单
        }
    }
}
void viewSchedule(Student &stu)
{
    while (true)
    {
        clearScreen(); // 清空屏幕
        cout << BOLD << GREEN;
        cout << "╔══════════════════════════════════════════════════════════════════════════════════\n";
        cout << "║                             已 选 课 程 列 表                                   \n";
        cout << "╟──────────────────────────────────────────────────────────────────────────────────\n";
        cout << "║ 课程编号   课程名称           授课教师         学分    上课时间                \n";
        cout << "╟──────────────────────────────────────────────────────────────────────────────────\n";

        int count = 0;
        for (const auto &sel : selections)
        {
            if (sel.studentId == stu.id)
            {
                Course *course = findCourseById(sel.courseId);
                if (course)
                {
                    cout << "║ " << left
                         << setw(10) << course->id
                         << setw(20) << course->name
                         << setw(17) << course->teacherName
                         << fixed << setprecision(1)
                         << setw(8)  << course->credit
                         << setw(24) << course->time << "\n";
                    count++;
                }
            }
        }

        if (count == 0)
        {
            cout << "║                                您当前没有已选课程                               \n";
        }

        cout << "╚══════════════════════════════════════════════════════════════════════════════════\n";
        cout << CYAN << "\n按 'b' 返回学生主菜单，按任意键继续查看课表：";
        char choice;
        cin >> choice;
        if (choice == 'b' || choice == 'B')
            break;
    }
}


void queryGrades(Student &stu)
{
    while (true)
    {
        clearScreen(); // 清空屏幕
        double totalCredits = 0;
        double weightedGPA = 0;
        int courseCount = 0;

        cout << BOLD << GREEN;
        cout << "╔════════════════════════════════════════════════════════════════════\n";
        cout << "║ 课程编号   课程名称           学分    总成绩   绩点   排名        \n";
        cout << "╟────────────────────────────────────────────────────────────────────\n";


        for (const auto &sel : selections)
        {
            if (sel.studentId == stu.id)
            {
                Course *course = findCourseById(sel.courseId);
                if (course && sel.examScore >= 0)
                {
                    double gpa = scoreToGPA(sel.examScore, sel.usualScore);

                    // 计算课程排名
                    int rank = 1;
                    for (const auto &otherSel : selections)
                    {
                        if (otherSel.courseId == sel.courseId && otherSel.totalScore > sel.totalScore)
                        {
                            rank++;
                        }
                    }

                    string scoreColor = sel.totalScore >= 60 ? GREEN : RED;

                    cout << WHITE<<"║ " << left<<
                         course->id<< setw(10) << WHITE
                         << WHITE << setw(20) << course->name << RESET  // 确保是白色
                         << fixed << setprecision(1)
                         << setw(8)  << course->credit;

                    cout << (sel.totalScore >= 60 ? GREEN : RED)
                         << setw(8) << sel.totalScore << RESET;

                    cout << setw(7) << gpa
                         << setw(12) << rank << "\n";




                    totalCredits += course->credit;
                    weightedGPA += gpa * course->credit;
                    courseCount++;
                }
            }
        }

        cout << "╚════════════════════════════════════════════════════════════════════\n";

        if (courseCount > 0)
        {
            double studentGPA = weightedGPA / totalCredits;

            // 计算所有学生的 GPA
            vector<pair<string, double>> studentGPAs; // 存储学生学号和 GPA
            for (const auto &student : students)
            {
                double totalCreditsForStudent = 0;
                double weightedGPAForStudent = 0;

                for (const auto &sel : selections)
                {
                    if (sel.studentId == student.id)
                    {
                        Course *course = findCourseById(sel.courseId);
                        if (course && sel.examScore >= 0)
                        {
                            double gpa = scoreToGPA(sel.examScore, sel.usualScore);
                            weightedGPAForStudent += gpa * course->credit;
                            totalCreditsForStudent += course->credit;
                        }
                    }
                }

                if (totalCreditsForStudent > 0)
                {
                    double gpa = weightedGPAForStudent / totalCreditsForStudent;
                    studentGPAs.push_back({student.id, gpa});
                }
            }

            // 按 GPA 降序排序
            sort(studentGPAs.begin(), studentGPAs.end(), [](const pair<string, double> &a, const pair<string, double> &b)
            {
                return a.second > b.second;
            });

            // 找到当前学生的排名
            int gradeRank = 1;
            for (const auto &entry : studentGPAs)
            {
                if (entry.first == stu.id)
                {
                    break;
                }
                gradeRank++;
            }

            cout << CYAN << "\n总学分: " << totalCredits
                 << "，平均绩点(GPA): " << fixed << setprecision(2) << studentGPA
                 << "，年级排名: " << gradeRank << "/" << studentGPAs.size() << RESET << endl;
        }
        else
        {
            cout << RED << "您当前没有成绩记录。\n"
                 << RESET;
        }

        cout << WHITE << "\n按 'b' 返回学生菜单，按任意键继续查看成绩：";
        char choice;
        cin >> choice;
        if (choice == 'b' || choice == 'B') break;
    }
}
void dropCourse(Student &stu)
{
    if (selections.empty())
    {
        cout << BOLD << RED << "\n您当前没有已选课程！\n" << RESET;
        return;
    }

    clearScreen();
    cout << BOLD << GREEN;
    cout << "╔══════════════════════════════════════════════════════════════════════════════════╗\n";
    cout << "║                             已 选 课 程 列 表                                   \n";
    cout << "╟──────────────────────────────────────────────────────────────────────────────────╢\n";
    cout << "║ 课程编号   课程名称           授课教师         学分    上课时间                \n";
    cout << "╟──────────────────────────────────────────────────────────────────────────────────╢\n";

    int count = 0;
    for (const auto &sel : selections)
    {
        if (sel.studentId == stu.id)
        {
            Course *course = findCourseById(sel.courseId);
            if (course)
            {
                cout << "║ " << left
                     << setw(10) << course->id
                     << setw(20) << course->name
                     << setw(17) << course->teacherName
                     << fixed << setprecision(1)
                     << setw(8) << course->credit
                     << setw(24) << course->time << "\n";
                count++;
            }
        }
    }

    if (count == 0)
    {
        cout << "║                                您当前没有已选课程                               ║\n";
    }

    cout << "╚══════════════════════════════════════════════════════════════════════════════════╝\n";

    if (count == 0)
    {
        cout << CYAN << "\n按任意键返回学生菜单：";
        cin.ignore();
        cin.get();
        return;
    }

    string courseId;
    cout << CYAN << "\n请输入要退选的课程编号（输入 0 返回）：";
    cin >> courseId;

    if (courseId == "0")
        return;

    auto it = find_if(selections.begin(), selections.end(), [&](const Selection &sel)
    {
        return sel.studentId == stu.id && sel.courseId == courseId;
    });

    if (it == selections.end())
    {
        cout << RED << "\n您未选择该课程，无法退选！\n" << RESET;
        cout << CYAN << "\n按任意键返回：";
        cin.ignore();
        cin.get();
        return;
    }

    // 更新课程的已选人数
    Course *course = findCourseById(courseId);
    if (course)
    {
        course->currentEnrollment--;
    }

    // 删除选课记录
    selections.erase(it);

    saveSelections();
    saveCourses();

    cout << GREEN << "\n退选成功！\n" << RESET;
    cout << CYAN << "\n按任意键返回学生菜单：";
    cin.ignore();
    cin.get();
}
// 通用登录处理函数：查找用户，核对密码，进入菜单
template<typename UserType>
bool loginUser(const string& roleName, UserType* (*findFunc)(const string&), void (*menuFunc)(UserType&))
{
    string id, password;

    while (true)
    {
        clearScreen();
        cout << BOLD << GREEN;
        cout << "                 ┌────────────────────────────────────────────────┐\n";
        cout << "                 │                                                │\n";
        cout << "                 │           欢迎进入" << roleName << "登录界面                 │\n";
        cout << "                 │                                                │\n";
        cout << "                 └────────────────────────────────────────────────┘\n";
        cout << GREEN << "账号: ";
        cin >> id;

        UserType* user = findFunc(id);
        if (!user)
        {
            cout << RED << "\n账号不存在！\n";
            cout << GREEN << "1. 重新输入账号\n0. 返回主菜单\n选择: ";
            int option;
            cin >> option;
            if (option == 0) return false;
            continue;
        }

        while (true)
        {
            cout << GREEN << "密码: ";
            password = getPassword();

            if (user->password == password)
            {
                clearScreen();
                menuFunc(*user);
                return true;
            }
            else
            {
                cout << RED << "\n密码错误！\n";
                cout << GREEN << "1. 重新输入密码\n0. 返回登录界面\n选择: ";
                int subOption;
                cin >> subOption;
                if (subOption == 0) break;
            }
        }
    }
}

bool loginStudent()
{
    return loginUser<Student>("学生", findStudentById, studentMenu);
}

bool loginTeacher()
{
    return loginUser<Teacher>("教师", findTeacherById, teacherMenu);
}

bool loginAdmin()
{
    // 管理员账号密码硬编码
    string id, password;
    while (true)
    {
        clearScreen();
        cout << BOLD << GREEN;
        cout << "                 ┌────────────────────────────────────────────────┐\n";
        cout << "                 │                                                │\n";
        cout << "                 │           欢迎进入管理员登录界面               │\n";
        cout << "                 │                                                │\n";
        cout << "                 └────────────────────────────────────────────────┘\n";
        cout << GREEN << "账号: ";
        cin >> id;

        if (id != "12345")
        {
            cout << RED << "\n账号不存在！\n";
            cout << GREEN << "1. 重新输入账号\n0. 返回主菜单\n选择: ";
            int option;
            cin >> option;
            if (option == 0) return false;
            continue;
        }

        while (true)
        {
            cout << GREEN << "密码: ";
            password = getPassword();

            if (password == "12345")
            {
                clearScreen();
                adminMenu();
                return true;
            }
            else
            {
                cout << RED << "\n密码错误！\n";
                cout << GREEN << "1. 重新输入密码\n0. 返回登录界面\n选择: ";
                int subOption;
                cin >> subOption;
                if (subOption == 0) break;
            }
        }
    }
}
void printWelcomeBanner()
{
    clearScreen();  // 清空屏幕
    const string GREEN_BOLD = "\033[1m\033[32m";
    cout << GREEN_BOLD;

    // 中间图案
    cout << "                              ***          ***\n";
    cout << "                           ***....**     **...***\n";
    cout << "                          **........** **.......**\n";
    cout << "                   ***    **..........*.........**    ***\n";
    cout << "                **.....**  **..................**  **.....**\n";
    cout << "              **.........**  **..............**  **.........**\n";
    cout << "             *..............*   *..........*   *..............*\n";
    cout << "              **..............*   *......*   *..............**\n";
    cout << "                **..............** *....* **..............**\n";
    cout << "                  *......................................*\n";
    cout << "                **..............**........**..............**\n";
    cout << "              **..............*    *....*....*..............**\n";
    cout << "             *..............*    *........* ...*..............*\n";
    cout << "              **.........**    *............* ...**.........**\n";
    cout << "                **.....**   **...............**....**.....**\n";
    cout << "                   ***    **...................**...*  ***\n";
    cout << "                        **...........*...........**...*\n";
    cout << "                         **.........* *.........**  *...*..*..*..*\n";
    cout << "                           *......**   **......*      *........*\n";
    cout << "                             **  *       * **            *...*\n";
    cout << "                                                           *\n";

    // 教务信息管理系统 居中大标题
    string pad = "                           ";
    cout << "\n";
    cout << pad << " __          __         _                                       _            ╔════════════════════╗\n";
    cout << pad << " \\ \\        / /        | |                                     | |           ║  教务信息管理系统  ║\n";
    cout << pad << "  \\ \\  /\\  / /    ___  | |   ___    ___    _ __ ___     ___    | |_    ___   ╚════════════════════╝\n";
    cout << pad << "   \\ \\/  \\/ /    / _ \\ | |  / __|  / _ \\  | '_ ` _ \\   / _ \\   | __|  / _ \\  \n";
    cout << pad << "    \\  /\\  /    |  __/ | | | (__  | (_) | | | | | | | |  __/   | |_  | (_) | \n";
    cout << pad << "     \\/  \\/      \\___| |_|  \\___|  \\___/  |_| |_| |_|  \\___|    \\__|  \\___/  \n";
    cout << "\n" << RESET;
}

void printLoginOptions()
{
    // 绿色加粗定义
    const string GREEN_BOLD = "\033[1m\033[32m";

    cout << "                 ┌────────────────────────────────────────────────┐\n";
    cout << GREEN_BOLD << "                 │                 1. 管理员登录                  │\n" << RESET;
    cout << "                 ├────────────────────────────────────────────────┤\n";
    cout << GREEN_BOLD << "                 │                 2. 学生登录                    │\n" << RESET;
    cout << "                 ├────────────────────────────────────────────────┤\n";
    cout << GREEN_BOLD << "                 │                 3. 教师登录                    │\n" << RESET;
    cout << "                 ├────────────────────────────────────────────────┤\n";
    cout << GREEN_BOLD << "                 │                 0. 退出系统                    │\n" << RESET;
    cout << "                 └────────────────────────────────────────────────┘\n\n";
}

void showLoginInterface()
{
    printWelcomeBanner();
    printLoginOptions();
    cout << "\n请输入您的选择: ";
}

// 主函数
int main()
{
    // 初始化文件
    initFileIfNotExist(STUDENT_FILE);
    initFileIfNotExist(COURSE_FILE);
    initFileIfNotExist(SELECTION_FILE);
    initFileIfNotExist(TEACHER_FILE);

    // 加载数据
    loadStudents();
    loadCourses();
    loadSelections();
    loadTeachers();
    // 同步教师课程关系
    syncTeacherCourses();
    while (true)
    {
        showLoginInterface();
        cout<<"输入您的选择"<<endl;
        int role;
        cin >> role;

        if (role == 0)
        {
            cout << "系统已退出！\n";
            break;
        }

        switch (role)
        {
        case 1:
            loginAdmin();
            break;
        case 2:
            loginStudent();
            break;
        case 3:
            loginTeacher();
            break;
        default:
            break;
        }

    }

    return 0;

}
```
