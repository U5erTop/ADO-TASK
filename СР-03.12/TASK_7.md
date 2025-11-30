# Решение Задания 7: Навигация по отношению многие-ко-многим в обе стороны

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;

class Program
{
    static void Main()
    {
        // Создаём DataSet и таблицы
        DataSet ds = CreateDataSet();
        
        // Заполняем таблицы тестовыми данными
        FillTestData(ds);
        
        // Создаём отношения многие-ко-многим
        CreateRelationships(ds);
        
        Console.WriteLine("=== ДЕМОНСТРАЦИЯ НАВИГАЦИИ ПО ОТНОШЕНИЮ МНОГИЕ-КО-МНОГИМ ===\n");
        
        // 1. Для студента: получите все его курсы и оценки
        Console.WriteLine("1. ВСЕ КУРСЫ И ОЦЕНКИ СТУДЕНТА:");
        Console.WriteLine("=====================================");
        GetStudentCoursesAndGrades(ds, 101);
        Console.WriteLine();
        
        // 2. Для курса: получите всех его студентов и их оценки
        Console.WriteLine("2. ВСЕ СТУДЕНТЫ И ОЦЕНКИ НА КУРСЕ:");
        Console.WriteLine("=====================================");
        GetCourseStudentsAndGrades(ds, "C001");
        Console.WriteLine();
        
        // 3. Поиск студентов, которые учатся на одних и тех же курсах
        Console.WriteLine("3. СТУДЕНТЫ, УЧАЩИЕСЯ НА ОДНИХ И ТЕХ ЖЕ КУРСАХ:");
        Console.WriteLine("=====================================");
        FindStudentsWithSameCourses(ds, 101);
        Console.WriteLine();
        
        // 4. Вывод полной информации о регистрации
        Console.WriteLine("4. ПОЛНАЯ ИНФОРМАЦИЯ О РЕГИСТРАЦИЯХ:");
        Console.WriteLine("=====================================");
        PrintAllRegistrations(ds);
        Console.WriteLine();
        
        // 5. Средняя оценка для каждого студента
        Console.WriteLine("5. СРЕДНЯЯ ОЦЕНКА СТУДЕНТОВ:");
        Console.WriteLine("=====================================");
        CalculateStudentAverageGrades(ds);
        Console.WriteLine();
        
        // 6. Средняя оценка студентов на каждом курсе
        Console.WriteLine("6. СРЕДНЯЯ ОЦЕНКА ПО КУРСАМ:");
        Console.WriteLine("=====================================");
        CalculateCourseAverageGrades(ds);
        Console.WriteLine();
        
        // 7. Лучшие студенты
        Console.WriteLine("7. ЛУЧШИЕ СТУДЕНТЫ (средняя оценка > 4.5):");
        Console.WriteLine("=====================================");
        FindTopStudents(ds, 4.5);
        Console.WriteLine();
    }
    
    // Создание DataSet с таблицами
    static DataSet CreateDataSet()
    {
        DataSet ds = new DataSet("UniversityDB");
        
        // Таблица Студенты
        DataTable students = new DataTable("Студенты");
        students.Columns.Add("StudentID", typeof(int));
        students.Columns.Add("StudentName", typeof(string));
        students.Columns.Add("Email", typeof(string));
        students.PrimaryKey = new DataColumn[] { students.Columns["StudentID"] };
        
        // Таблица Курсы
        DataTable courses = new DataTable("Курсы");
        courses.Columns.Add("CourseID", typeof(string));
        courses.Columns.Add("CourseName", typeof(string));
        courses.Columns.Add("Instructor", typeof(string));
        courses.PrimaryKey = new DataColumn[] { courses.Columns["CourseID"] };
        
        // Таблица Регистрация (промежуточная)
        DataTable registration = new DataTable("Регистрация");
        registration.Columns.Add("RegistrationID", typeof(int));
        registration.Columns.Add("StudentID", typeof(int));
        registration.Columns.Add("CourseID", typeof(string));
        registration.Columns.Add("EnrollmentDate", typeof(DateTime));
        registration.Columns.Add("Grade", typeof(double));
        registration.PrimaryKey = new DataColumn[] { registration.Columns["RegistrationID"] };
        
        ds.Tables.Add(students);
        ds.Tables.Add(courses);
        ds.Tables.Add(registration);
        
        return ds;
    }
    
    // Заполнение тестовыми данными
    static void FillTestData(DataSet ds)
    {
        DataTable students = ds.Tables["Студенты"];
        DataTable courses = ds.Tables["Курсы"];
        DataTable registration = ds.Tables["Регистрация"];
        
        // Добавляем студентов
        students.Rows.Add(101, "Иван Петров", "ivan@example.com");
        students.Rows.Add(102, "Мария Сидорова", "maria@example.com");
        students.Rows.Add(103, "Петр Иванов", "petr@example.com");
        students.Rows.Add(104, "Анна Смирнова", "anna@example.com");
        
        // Добавляем курсы
        courses.Rows.Add("C001", "C# Programming", "Дмитрий Волков");
        courses.Rows.Add("C002", "Database Design", "Светлана Морозова");
        courses.Rows.Add("C003", "Web Development", "Алексей Новиков");
        courses.Rows.Add("C004", "OOP Principles", "Петр Сергеев");
        
        // Добавляем регистрации с оценками
        registration.Rows.Add(1, 101, "C001", new DateTime(2024, 01, 15), 4.5);
        registration.Rows.Add(2, 101, "C002", new DateTime(2024, 01, 20), 3.8);
        registration.Rows.Add(3, 101, "C004", new DateTime(2024, 02, 10), 4.9);
        
        registration.Rows.Add(4, 102, "C001", new DateTime(2024, 01, 15), 4.8);
        registration.Rows.Add(5, 102, "C003", new DateTime(2024, 02, 05), 4.2);
        
        registration.Rows.Add(6, 103, "C002", new DateTime(2024, 01, 20), 3.5);
        registration.Rows.Add(7, 103, "C003", new DateTime(2024, 02, 05), 4.0);
        registration.Rows.Add(8, 103, "C004", new DateTime(2024, 02, 10), 4.7);
        
        registration.Rows.Add(9, 104, "C001", new DateTime(2024, 01, 15), 4.6);
        registration.Rows.Add(10, 104, "C002", new DateTime(2024, 01, 20), 4.3);
        registration.Rows.Add(11, 104, "C003", new DateTime(2024, 02, 05), 4.1);
    }
    
    // Создание отношений многие-ко-многим
    static void CreateRelationships(DataSet ds)
    {
        DataTable students = ds.Tables["Студенты"];
        DataTable courses = ds.Tables["Курсы"];
        DataTable registration = ds.Tables["Регистрация"];
        
        // Отношение: Студенты → Регистрация (один студент → много регистраций)
        DataRelation rel1 = new DataRelation(
            "FK_Students_Registration",
            students.Columns["StudentID"],
            registration.Columns["StudentID"],
            true
        );
        ds.Relations.Add(rel1);
        
        // Отношение: Курсы → Регистрация (один курс → много регистраций)
        DataRelation rel2 = new DataRelation(
            "FK_Courses_Registration",
            courses.Columns["CourseID"],
            registration.Columns["CourseID"],
            true
        );
        ds.Relations.Add(rel2);
    }
    
    // 1. Для студента: получите все его курсы и оценки
    static void GetStudentCoursesAndGrades(DataSet ds, int studentID)
    {
        DataTable students = ds.Tables["Студенты"];
        DataTable courses = ds.Tables["Курсы"];
        DataTable registration = ds.Tables["Регистрация"];
        
        // Находим студента
        DataRow studentRow = students.Rows.Find(studentID);
        
        if (studentRow == null)
        {
            Console.WriteLine($"Студент с ID {studentID} не найден");
            return;
        }
        
        Console.WriteLine($"Студент: {studentRow["StudentName"]}");
        Console.WriteLine($"Email: {studentRow["Email"]}");
        Console.WriteLine("\nКурсы:");
        Console.WriteLine("─────────────────────────────────────────────────");
        
        // Получаем все регистрации студента (GetChildRows от таблицы Студенты)
        DataRow[] registrationRows = studentRow.GetChildRows("FK_Students_Registration");
        
        if (registrationRows.Length == 0)
        {
            Console.WriteLine("У студента нет зарегистрированных курсов");
            return;
        }
        
        foreach (DataRow regRow in registrationRows)
        {
            // Получаем информацию о курсе (GetParentRows от таблицы Регистрация)
            DataRow[] courseRows = regRow.GetParentRows("FK_Courses_Registration");
            
            if (courseRows.Length > 0)
            {
                DataRow courseRow = courseRows[0];
                double grade = (double)regRow["Grade"];
                DateTime enrollmentDate = (DateTime)regRow["EnrollmentDate"];
                
                Console.WriteLine($"  • {courseRow["CourseName"]}");
                Console.WriteLine($"    Преподаватель: {courseRow["Instructor"]}");
                Console.WriteLine($"    Дата регистрации: {enrollmentDate:dd.MM.yyyy}");
                Console.WriteLine($"    Оценка: {grade:F1}");
                Console.WriteLine();
            }
        }
    }
    
    // 2. Для курса: получите всех его студентов и их оценки
    static void GetCourseStudentsAndGrades(DataSet ds, string courseID)
    {
        DataTable students = ds.Tables["Студенты"];
        DataTable courses = ds.Tables["Курсы"];
        
        // Находим курс
        DataRow courseRow = courses.Rows.Find(courseID);
        
        if (courseRow == null)
        {
            Console.WriteLine($"Курс с ID {courseID} не найден");
            return;
        }
        
        Console.WriteLine($"Курс: {courseRow["CourseName"]}");
        Console.WriteLine($"Преподаватель: {courseRow["Instructor"]}");
        Console.WriteLine("\nСтуденты:");
        Console.WriteLine("─────────────────────────────────────────────────");
        
        // Получаем все регистрации на этот курс (GetChildRows от таблицы Курсы)
        DataRow[] registrationRows = courseRow.GetChildRows("FK_Courses_Registration");
        
        if (registrationRows.Length == 0)
        {
            Console.WriteLine("На этом курсе нет студентов");
            return;
        }
        
        foreach (DataRow regRow in registrationRows)
        {
            // Получаем информацию о студенте (GetParentRows от таблицы Регистрация)
            DataRow[] studentRows = regRow.GetParentRows("FK_Students_Registration");
            
            if (studentRows.Length > 0)
            {
                DataRow studentRow = studentRows[0];
                double grade = (double)regRow["Grade"];
                
                Console.WriteLine($"  • {studentRow["StudentName"]}");
                Console.WriteLine($"    Email: {studentRow["Email"]}");
                Console.WriteLine($"    Оценка: {grade:F1}");
                Console.WriteLine();
            }
        }
    }
    
    // 3. Поиск студентов, которые учатся на одних и тех же курсах
    static void FindStudentsWithSameCourses(DataSet ds, int studentID)
    {
        DataTable students = ds.Tables["Студенты"];
        DataRow studentRow = students.Rows.Find(studentID);
        
        if (studentRow == null)
        {
            Console.WriteLine($"Студент с ID {studentID} не найден");
            return;
        }
        
        // Получаем все курсы студента
        DataRow[] studentCourses = studentRow.GetChildRows("FK_Students_Registration");
        var studentCourseIDs = new HashSet<string>();
        
        foreach (DataRow regRow in studentCourses)
        {
            studentCourseIDs.Add((string)regRow["CourseID"]);
        }
        
        Console.WriteLine($"Студент: {studentRow["StudentName"]}");
        Console.WriteLine($"Его курсы: {string.Join(", ", studentCourseIDs)}");
        Console.WriteLine("\nСоучащиеся:");
        Console.WriteLine("─────────────────────────────────────────────────");
        
        // Находим других студентов, учащихся на тех же курсах
        foreach (DataRow student in students.Rows)
        {
            if ((int)student["StudentID"] == studentID)
                continue;
            
            DataRow[] otherStudentCourses = student.GetChildRows("FK_Students_Registration");
            var otherCourseIDs = new HashSet<string>();
            
            foreach (DataRow regRow in otherStudentCourses)
            {
                otherCourseIDs.Add((string)regRow["CourseID"]);
            }
            
            // Находим пересечение курсов
            var commonCourses = studentCourseIDs.Intersect(otherCourseIDs).ToList();
            
            if (commonCourses.Count > 0)
            {
                Console.WriteLine($"  • {student["StudentName"]}");
                Console.WriteLine($"    Общие курсы: {string.Join(", ", commonCourses)}");
                Console.WriteLine();
            }
        }
    }
    
    // 4. Вывод полной информации о регистрации
    static void PrintAllRegistrations(DataSet ds)
    {
        DataTable registration = ds.Tables["Регистрация"];
        
        Console.WriteLine("Номер | Студент              | Курс             | Дата         | Оценка");
        Console.WriteLine("─────────────────────────────────────────────────────────────────────────");
        
        foreach (DataRow regRow in registration.Rows)
        {
            int studentID = (int)regRow["StudentID"];
            string courseID = (string)regRow["CourseID"];
            DateTime enrollmentDate = (DateTime)regRow["EnrollmentDate"];
            double grade = (double)regRow["Grade"];
            
            // Получаем информацию о студенте
            DataRow[] studentRows = regRow.GetParentRows("FK_Students_Registration");
            string studentName = studentRows.Length > 0 ? 
                (string)studentRows[0]["StudentName"] : "Неизвестен";
            
            // Получаем информацию о курсе
            DataRow[] courseRows = regRow.GetParentRows("FK_Courses_Registration");
            string courseName = courseRows.Length > 0 ? 
                (string)courseRows[0]["CourseName"] : "Неизвестен";
            
            Console.WriteLine($"{regRow["RegistrationID"],5} | {studentName,-20} | {courseName,-15} | {enrollmentDate:dd.MM.yyyy} | {grade:F1}");
        }
    }
    
    // 5. Средняя оценка для каждого студента
    static void CalculateStudentAverageGrades(DataSet ds)
    {
        DataTable students = ds.Tables["Студенты"];
        
        Console.WriteLine("Студент              | Количество курсов | Средняя оценка");
        Console.WriteLine("─────────────────────────────────────────────────────────");
        
        foreach (DataRow studentRow in students.Rows)
        {
            string studentName = (string)studentRow["StudentName"];
            DataRow[] registrationRows = studentRow.GetChildRows("FK_Students_Registration");
            
            if (registrationRows.Length == 0)
            {
                Console.WriteLine($"{studentName,-20} | {0,16} | Нет оценок");
                continue;
            }
            
            double sumGrades = 0;
            foreach (DataRow regRow in registrationRows)
            {
                if (regRow["Grade"] != DBNull.Value)
                {
                    sumGrades += (double)regRow["Grade"];
                }
            }
            
            double averageGrade = sumGrades / registrationRows.Length;
            Console.WriteLine($"{studentName,-20} | {registrationRows.Length,16} | {averageGrade,13:F2}");
        }
    }
    
    // 6. Средняя оценка студентов на каждом курсе
    static void CalculateCourseAverageGrades(DataSet ds)
    {
        DataTable courses = ds.Tables["Курсы"];
        
        Console.WriteLine("Курс                 | Количество студентов | Средняя оценка");
        Console.WriteLine("────────────────────────────────────────────────────────────");
        
        foreach (DataRow courseRow in courses.Rows)
        {
            string courseName = (string)courseRow["CourseName"];
            DataRow[] registrationRows = courseRow.GetChildRows("FK_Courses_Registration");
            
            if (registrationRows.Length == 0)
            {
                Console.WriteLine($"{courseName,-20} | {0,20} | Нет оценок");
                continue;
            }
            
            double sumGrades = 0;
            foreach (DataRow regRow in registrationRows)
            {
                if (regRow["Grade"] != DBNull.Value)
                {
                    sumGrades += (double)regRow["Grade"];
                }
            }
            
            double averageGrade = sumGrades / registrationRows.Length;
            Console.WriteLine($"{courseName,-20} | {registrationRows.Length,20} | {averageGrade,13:F2}");
        }
    }
    
    // 7. Найдите лучших студентов (средняя оценка выше указанного порога)
    static void FindTopStudents(DataSet ds, double minAverageGrade)
    {
        DataTable students = ds.Tables["Студенты"];
        
        Console.WriteLine("Студент              | Средняя оценка | Курсы");
        Console.WriteLine("──────────────────────────────────────────────────────────");
        
        bool foundStudents = false;
        
        foreach (DataRow studentRow in students.Rows)
        {
            string studentName = (string)studentRow["StudentName"];
            DataRow[] registrationRows = studentRow.GetChildRows("FK_Students_Registration");
            
            if (registrationRows.Length == 0)
                continue;
            
            double sumGrades = 0;
            foreach (DataRow regRow in registrationRows)
            {
                if (regRow["Grade"] != DBNull.Value)
                {
                    sumGrades += (double)regRow["Grade"];
                }
            }
            
            double averageGrade = sumGrades / registrationRows.Length;
            
            if (averageGrade >= minAverageGrade)
            {
                foundStudents = true;
                
                // Получаем информацию о курсах студента
                var courseNames = new List<string>();
                foreach (DataRow regRow in registrationRows)
                {
                    DataRow[] courseRows = regRow.GetParentRows("FK_Courses_Registration");
                    if (courseRows.Length > 0)
                    {
                        courseNames.Add((string)courseRows[0]["CourseName"]);
                    }
                }
                
                string courses = string.Join(", ", courseNames);
                Console.WriteLine($"{studentName,-20} | {averageGrade,14:F2} | {courses}");
            }
        }
        
        if (!foundStudents)
        {
            Console.WriteLine($"Студентов со средней оценкой ≥ {minAverageGrade} не найдено");
        }
    }
}
```

## Основные концепции решения

### 1. **GetChildRows() и GetParentRows()**
```csharp
// GetChildRows - получить все регистрации студента
DataRow[] registrationRows = studentRow.GetChildRows("FK_Students_Registration");

// GetParentRows - получить курс регистрации
DataRow[] courseRows = regRow.GetParentRows("FK_Courses_Registration");
```

### 2. **Структура отношений многие-ко-многим**
```
Студенты (1) ──→ Регистрация (∞)
Курсы (1) ──────→ Регистрация (∞)
```

### 3. **Двусторонняя навигация**
- От студента к курсам: `Student → GetChildRows(Registration) → GetParentRows(Course)`
- От курса к студентам: `Course → GetChildRows(Registration) → GetParentRows(Student)`

### 4. **Обработка NULL значений**
```csharp
if (regRow["Grade"] != DBNull.Value)
{
    sumGrades += (double)regRow["Grade"];
}
```
