# Student Portal Management Registration System  
A simple and efficient web-based application developed using **ASP.NET MVC** and **SQL Server** to manage student registration, student records, and administrative tasks using **Stored Procedures (CRUD)**.

##  Project Overview
This system allows admin to register students, update details, view records, and generate simple reports. The application is built without Entity Framework and uses **ADO.NET + Stored Procedures** for all database operations.

##  Technology Stack
- **Frontend:** HTML, CSS, Bootstrap  
- **Backend:** ASP.NET MVC  
- **Database:** SQL Server  
- **Architecture:** MVC (Model–View–Controller)  
- **ORM:** Not used (ADO.NET only)  

## 🎯 Core Functionalities

### **1️⃣ Create Student**
- Enter Name, Email, Department, DOB, Mobile  
- Saves data using SQL **INSERT Stored Procedure**

### **2️⃣ Read Students**
- Displays student list  
- Search & sort filters  
- Uses **SELECT Stored Procedure**

### **3️⃣ Update Student**
- Edit student details  
- Uses **UPDATE Stored Procedure**

### **4️⃣ Delete Student**
- Delete a student entry  
- Uses **DELETE Stored Procedure**

---

##  **Project Structure**

/Controllers
StudentController.cs
DashboardController.cs
SettingsController.cs
ReportsController.cs

/Models
Student.cs
Dashboard.cs
StudentRepository.cs

/Views
/Student
Add.cshtml
Edit.cshtml
Delete.cshtml
Index.cshtml
Details.cshtml

/Dashboard
Index.cshtml

/Reports
Index.cshtml
FilteredReport.cshtml

/Settings
Index.cshtml



---

# SQL SCRIPT (Full File)

##  Create Table
```sql
CREATE TABLE Student (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    Email NVARCHAR(100) NOT NULL,
    Phone NVARCHAR(10) NOT NULL,
    DateOfBirth DATE NOT NULL,   
    Course NVARCHAR(50) NOT NULL,
    RegisteredAt DATETIME NOT NULL DEFAULT GETDATE()
);

📌 Stored Procedures (CRUD)
CREATE  PROC sp_AddStudent
(
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50),
    @Email NVARCHAR(100),
    @Phone NVARCHAR(10),
    @DateOfBirth DATE,
    @Course NVARCHAR(50)
)
AS
BEGIN
    INSERT INTO Student (FirstName, LastName, Email, Phone, DateOfBirth, Course)
    VALUES (@FirstName, @LastName, @Email, @Phone, @DateOfBirth, @Course);
END;
GO

CREATE  PROCEDURE sp_UpdateStudent
(
    @Id INT,
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50),
    @Email NVARCHAR(100),
    @Phone NVARCHAR(10),
    @DateOfBirth DATE,
    @Course NVARCHAR(50)
)
AS
BEGIN
    UPDATE Student
    SET FirstName = @FirstName,
        LastName = @LastName,
        Email = @Email,
        Phone = @Phone,
        DateOfBirth = @DateOfBirth,
        Course = @Course
    WHERE Id = @Id;
END;
GO

CREATE  PROCEDURE sp_DeleteStudent
    @Id INT
AS
BEGIN
    DELETE FROM Student WHERE Id = @Id;
END;
GO

CREATE  PROCEDURE sp_GetStudentById
    @Id INT
AS
BEGIN
    SELECT * FROM Student WHERE Id = @Id;
END;
GO

ALTER PROCEDURE sp_GetAllStudents
AS
BEGIN
    SELECT * 
    FROM Student 
    ORDER BY Id ASC;
END;

EXEC sp_GetAllStudents;

GO

CREATE  PROCEDURE sp_SearchStudents
    @Search NVARCHAR(100)
AS
BEGIN
    SELECT *
    FROM Student
    WHERE FirstName LIKE '%' + @Search + '%'
       OR LastName LIKE '%' + @Search + '%'
       OR Email LIKE '%' + @Search + '%'
    ORDER BY RegisteredAt ASC;
END;
GO

CREATE  PROCEDURE sp_FilterStudentsByCourse
    @Course NVARCHAR(50)
AS
BEGIN
    SELECT *
    FROM Student
    WHERE Course = @Course
    ORDER BY RegisteredAt ASC;
END;
GO


CREATE PROCEDURE sp_TotalStudent
AS
BEGIN
    SELECT COUNT(*) AS TotalStudent FROM Student;
END;
GO


CREATE  PROCEDURE sp_StudentsByCourses
AS
BEGIN
    SELECT Course, COUNT(*) AS Total
    FROM Student
    GROUP BY Course;
END;
GO

CREATE  PROCEDURE sp_TodaysStudent
AS
BEGIN
    SELECT COUNT(*) AS TodayCount
    FROM Student
    WHERE CAST(RegisteredAt AS DATE) = CAST(GETDATE() AS DATE);
END;
GO

CREATE  PROCEDURE sp_StudentsAgeRanges
AS
BEGIN
    SELECT COUNT(*) AS AgeRangeCount
    FROM Student
    WHERE DATEDIFF(YEAR, DateOfBirth, GETDATE()) BETWEEN 18 AND 25;
END;
GO

ALTER PROCEDURE sp_LatestStudent
AS
BEGIN
    SELECT *
    FROM (
        SELECT TOP 5 *
        FROM Student
        ORDER BY RegisteredAt ASC -- newest 5
    ) AS latest
    ORDER BY Id ASC; -- show in 1,2,3,4 order
END;

EXEC sp_LatestStudent;

GO
---------

**UI Design Code**

/Views
/Student

AddStudent.cshtml

@model StudentManagement.Models.Student
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2 class="page-title mb-4">@ViewBag.Title</h2>
<h2 class="text-center mb-4">Student Registration Form</h2>

<div class="card p-4">
    @using (Html.BeginForm())
    {
        @Html.AntiForgeryToken()

        <div class="row">
            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.FirstName)
                @Html.TextBoxFor(m => m.FirstName, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.LastName)
                @Html.TextBoxFor(m => m.LastName, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.Email)
                @Html.TextBoxFor(m => m.Email, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.Phone)
                @Html.TextBoxFor(m => m.Phone, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.DateOfBirth)
                @Html.TextBoxFor(m => m.DateOfBirth, "{0:yyyy-MM-dd}", new { type = "date", @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.Course)
                @Html.DropDownListFor(m => m.Course,
                    new SelectList(new[] { "Computer Science", "Electronics", "Mechanical" }),
                    "-- Select Course --",
                    new { @class = "form-control" })
            </div>
        </div>

        <button class="btn btn-success mt-3">Save</button>
        <a href="@Url.Action("Index")" class="btn btn-secondary mt-3">Cancel</a>
    }
</div>

Delete.cshtml

@model StudentManagement.Models.Student
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h3>Are you sure you want to delete this student?</h3>

<table class="table table-bordered">
    <tr><th>Id</th><td>@Model.Id</td></tr>
    <tr><th>Name</th><td>@Model.FirstName @Model.LastName</td></tr>
    <tr><th>Email</th><td>@Model.Email</td></tr>
</table>

@using (Html.BeginForm("Delete", "Student", new { id = Model.Id }, FormMethod.Post))
{
    <button type="submit"
            onclick="return confirm('Delete this student?')"
            class="btn btn-danger btn-sm">
        Delete
    </button>
}

Edit.cshtml

@model StudentManagement.Models.Student
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2 class="page-title mb-4">@ViewBag.Title</h2>

<div class="card p-4">
    @using (Html.BeginForm())
    {
        @Html.AntiForgeryToken()

        <div class="row">
            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.FirstName)
                @Html.TextBoxFor(m => m.FirstName, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.LastName)
                @Html.TextBoxFor(m => m.LastName, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.Email)
                @Html.TextBoxFor(m => m.Email, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.Phone)
                @Html.TextBoxFor(m => m.Phone, new { @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.DateOfBirth)
                @Html.TextBoxFor(m => m.DateOfBirth, "{0:yyyy-MM-dd}", new { type = "date", @class = "form-control" })
            </div>

            <div class="col-md-6 mb-3">
                @Html.LabelFor(m => m.Course)
                @Html.DropDownListFor(m => m.Course,
                    new SelectList(new[] { "Computer Science", "Electronics", "Mechanical" }),
                    "-- Select Course --",
                    new { @class = "form-control" })
            </div>
        </div>

        <button class="btn btn-success mt-3">Save</button>
        <a href="@Url.Action("Index")" class="btn btn-secondary mt-3">Cancel</a>
    }
</div>

Details.cshtml

@model StudentManagement.Models.Student
@{
    ViewBag.Title = "Student Details";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2>Student Details</h2>

<table class="table table-bordered">
    <tr><th>Id</th><td>@Model.Id</td></tr>
    <tr><th>Name</th><td>@Model.FirstName @Model.LastName</td></tr>
    <tr><th>Email</th><td>@Model.Email</td></tr>
    <tr><th>Phone</th><td>@Model.Phone</td></tr>
    <tr><th>DOB</th><td>@Model.DateOfBirth.ToString("dd/MM/yyyy")</td></tr>
    <tr><th>Course</th><td>@Model.Course</td></tr>
    <tr><th>Registered</th><td>@Model.RegisteredAt</td></tr>
</table>

<a href="@Url.Action("Edit", new { id = Model.Id })" class="btn btn-warning">Edit</a>
<a href="@Url.Action("Index")" class="btn btn-secondary">Back</a>

Index.cshtml

@model IEnumerable<StudentManagement.Models.Student>
@{
    ViewBag.Title = "Students";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="main-content">

    <h2 class="page-title mb-4">Student Records</h2>

    <!-- Search + Page Size -->
    <form method="get" class="search-row">
        <input type="text" name="search" class="form-control search-box"
               placeholder="Search name / email / course..."
               value="@ViewBag.Search" />

        

        <button class="btn btn-search"><i class="fa fa-search"></i> Search</button>
    </form>

    <div class="action-row">
        <a href="@Url.Action("Create")" class="btn btn-primary btn-add">
            <i class="fa fa-plus"></i> Add Student
        </a>

        <a href="@Url.Action("ExportToExcel")" class="btn btn-success btn-excel">
            <i class="fa fa-file-excel"></i> Export Excel
        </a>
    </div>

    <!-- Table -->
    <table class="table custom-table">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>Phone</th>
                <th>DateOfBirth</th>
                <th>Course</th>
                <th>Registered</th>
                <th width="200">Actions</th>
            </tr>
        </thead>

        <tbody>
            @foreach (var item in Model)
            {
                <tr>
                    <td>@item.Id</td>
                    <td>@item.FirstName @item.LastName</td>
                    <td>@item.Email</td>
                    <td>@item.Phone</td>
                    <td>@item.DateOfBirth.ToString("dd/MM/yyyy")</td>
                    <td>@item.Course</td>
                    <td>@item.RegisteredAt.ToString("dd/MM/yyyy")</td>
                    <td>
                    <td>
                        <a href="@Url.Action("Details", "Student", new { id = item.Id })" class="btn btn-info btn-sm">
                            <i class="fa fa-eye"></i>
                        </a>

                        <a href="@Url.Action("Edit", "Student", new { id = item.Id })" class="btn btn-warning btn-sm">
                            <i class="fa fa-edit"></i>
                        </a>
              
                    <a href="@Url.Action("Delete", "Student", new { id = item.Id })" class="btn btn-danger btn-sm">
                        <i class="fa fa-trash"></i>
                    </a>
                    </td>


                </tr>
            }
        </tbody>
    </table>


    <!-- Pagination -->
    <nav aria-label="Page navigation example" class="mt-4">
        <ul class="pagination justify-content-center">

            <!-- Previous Button -->
            <li class="page-item @(ViewBag.Page == 1 ? "disabled" : "")">
                <a class="page-link"
                   href="@Url.Action("Index", new { page = ViewBag.Page - 1, search = ViewBag.Search })">
                    ‹ Prev
                </a>
            </li>

            <!-- Page Numbers -->
            @for (int i = 1; i <= ViewBag.TotalPages; i++)
            {
                <li class="page-item @(i == ViewBag.Page ? "active" : "")">
                    <a class="page-link"
                       href="@Url.Action("Index", new { page = i, search = ViewBag.Search })">
                        @i
                    </a>
                </li>
            }

            <!-- Next Button -->
            <li class="page-item @(ViewBag.Page == ViewBag.TotalPages ? "disabled" : "")">
                <a class="page-link"
                   href="@Url.Action("Index", new { page = ViewBag.Page + 1, search = ViewBag.Search })">
                    Next ›
                </a>
            </li>

        </ul>
    </nav>

</div>

/Dashboard

Index.cshtml

@model StudentManagement.Models.Dashboard
@{
    ViewBag.Title = "Dashboard";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" />

<style>
    .dashboard-container {
        padding: 20px;
    }

    .card-box {
        padding: 20px;
        border-radius: 10px;
        color: white;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .card-1 {
        background: #007bff;
    }

    .card-2 {
        background: #28a745;
    }

    .card-3 {
        background: #ffc107;
    }

    .card-box i {
        font-size: 40px;
        opacity: .7;
    }

    .section-title {
        font-size: 22px;
        margin-top: 20px;
        margin-bottom: 15px;
        font-weight: bold;
    }

    .latest-students table {
        background: white;
        border-radius: 5px;
        overflow: hidden;
    }
</style>

<div class="dashboard-container">

    <h2 class="mb-4">Dashboard Overview</h2>

    <!-- TOP CARDS -->
    <div class="row mb-4">
        <div class="col-md-4">
            <div class="card-box card-1">
                <div>
                    <h4>Total Students</h4>
                    <h2>@Model.TotalStudents</h2>
                </div>
                <i class="fa fa-users"></i>
            </div>
        </div>

        <div class="col-md-4">
            <div class="card-box card-2">
                <div>
                    <h4>Today's Registrations</h4>
                    <h2>@Model.TodayCount</h2>
                </div>
                <i class="fa fa-user-plus"></i>
            </div>
        </div>

        <div class="col-md-4">
            <div class="card-box card-3">
                <div>
                    <h4>Age 18 - 25 Count</h4>
                    <h2>@Model.AgeRangeCount</h2>
                </div>
                <i class="fa fa-chart-line"></i>
            </div>
        </div>
    </div>

    <!-- COURSE CHART -->
    <div class="section-title">Students by Course</div>
    <canvas id="courseChart" height="100"></canvas>

    <!-- LATEST STUDENTS TABLE -->
    <div class="section-title mt-4">Latest Registered Students</div>

    <div class="latest-students">
        <table class="table table-bordered table-striped">
            <thead class="table-dark">
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Course</th>
                    <th>Registered</th>
                </tr>
            </thead>
            <tbody>
                @foreach (var s in Model.LatestStudents)
                {
                    <tr>
                        <td>@s.Id</td>
                        <td>@s.FirstName @s.LastName</td>
                        <td>@s.Email</td>
                        <td>@s.Course</td>
                        <td>@s.RegisteredAt.ToString("dd-MM-yyyy")</td>
                    </tr>
                }
            </tbody>
        </table>
    </div>

</div>

<!-- CHART JS -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
    var labels = @Html.Raw(Newtonsoft.Json.JsonConvert.SerializeObject(
                Model.StudentsByCourse.Select(x => x.Key)
            ));

    var values = @Html.Raw(Newtonsoft.Json.JsonConvert.SerializeObject(
                Model.StudentsByCourse.Select(x => x.Value)
            ));

    var ctx = document.getElementById("courseChart").getContext("2d");
    new Chart(ctx, {
        type: 'bar',
        data: {
            labels: labels,
            datasets: [{
                label: "Students",
                data: values,
                backgroundColor: ["#007bff", "#28a745", "#ffc107", "#dc3545", "#17a2b8"]
            }]
        }
    });
</script>


/Reports

Index.cshtml

@{
    ViewBag.Title = "Reports";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h2 class="mb-4">Reports & Analytics</h2>

<!-- Summary -->
<div class="row mb-4">
    <div class="col-md-4">
        <div class="alert alert-primary">
            <b>Total Students:</b> @ViewBag.Total
        </div>
    </div>

    <div class="col-md-4">
        <div class="alert alert-warning">
            <b>Age 18-25 Students:</b> @ViewBag.AgeCount
        </div>
    </div>
</div>

<!-- Course Report -->
<h4>Course-wise Students</h4>
<table class="table table-bordered">
    <thead>
        <tr>
            <th>Course</th>
            <th>Total</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var c in ViewBag.CourseReport)
        {
            <tr>
                <td>@c.Key</td>
                <td>@c.Value</td>
            </tr>
        }
    </tbody>
</table>

<!-- Date Filter -->
<h4 class="mt-4">Filter by Registration Date</h4>

<form action="/Reports/FilterDates" method="post" class="row g-3">
    <div class="col-md-3">
        <label>From Date</label>
        <input type="date" name="fromDate" class="form-control" required />
    </div>

    <div class="col-md-3">
        <label>To Date</label>
        <input type="date" name="toDate" class="form-control" required />
    </div>

    <div class="col-md-3 align-self-end">
        <button class="btn btn-primary">Generate</button>
    </div>

    <div class="col-md-3 align-self-end">
        <a href="/Reports/DownloadExcel" class="btn btn-success">Download Excel</a>
    </div>
</form>

ReportsFiltered.cshtml

@model IEnumerable<StudentManagement.Models.Student>
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h3 class="mb-3">Filtered Report</h3>

<table class="table table-bordered">
    <thead>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Course</th>
            <th>Registered</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var s in Model)
        {
            <tr>
                <td>@s.Id</td>
                <td>@s.FirstName @s.LastName</td>
                <td>@s.Email</td>
                <td>@s.Course</td>
                <td>@s.RegisteredAt.ToString("dd/MM/yyyy")</td>
            </tr>
        }
    </tbody>
</table>

/Settings

Index.cshtml

@{
    ViewBag.Title = "Settings";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="settings-container">

    <!-- PAGE HEADER -->
    <div class="settings-header mb-4">
        <h2><i class="fa fa-gear"></i> Application Settings</h2>
        <p class="subtitle">Customize theme, table size and app preferences</p>
    </div>

    <!-- CARD -->
    <div class="card shadow-lg p-4">

        <form method="post" action="/Settings/SaveSettings">

            <!-- Theme Selection -->
            <div class="form-group mb-4">
                <label class="font-weight-bold"><i class="fa fa-palette"></i> Select Theme</label>
                <select name="theme" class="form-control custom-select">
                    <option value="light">🌤️ Light Mode</option>
                    <option value="dark">🌙 Dark Mode</option>
                    <option value="blue">🔵 Blue Theme</option>
                </select>
            </div>

            <!-- Page Size -->
            <div class="form-group mb-4">
                <label class="font-weight-bold"><i class="fa fa-table"></i> Default Table Page Size</label>
                <select name="pageSize" class="form-control custom-select">
                    <option value="5">5 Records</option>
                    <option value="10">10 Records</option>
                    <option value="20">20 Records</option>
                </select>
            </div>

            <button class="btn btn-primary btn-lg w-100">
                <i class="fa fa-save"></i> Save Settings
            </button>
        </form>

        @if (TempData["Success"] != null)
        {
            <div class="alert alert-success mt-3">
                <i class="fa fa-check-circle"></i> @TempData["Success"]
            </div>
        }

    </div>
</div>

<style>
    .settings-container {
        max-width: 600px;
        margin: auto;
        padding-top: 20px;
        animation: fadeIn 0.5s;
    }

    .settings-header h2 {
        font-weight: 700;
        color: #333;
    }

    .settings-header .subtitle {
        color: #777;
        margin-top: -5px;
    }

    .card {
        border-radius: 12px;
        background: #fff;
    }

    .form-group label {
        font-size: 16px;
    }

    .custom-select {
        height: 45px;
        font-size: 16px;
    }

    @@keyframes fadeIn {
        from { opacity: 0; transform: translateY(15px); }
        to   { opacity: 1; transform: translateY(0); }
    }
</style>

/Shared

Layout.cshtml

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Student Portal</title>

    <link href="~/Content/bootstrap.min.css" rel="stylesheet" />
    <link href="~/Content/site.css" rel="stylesheet" />
    <link rel="stylesheet"
          href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css" />

    <style>
        body {
            background: #f5f7fa;
        }

        .wrapper {
            display: flex;
        }

        /* DARK SIDEBAR */
        /* ================= SIDEBAR FIX ================= */
        .sidebar {
            width: 230px; /* reduce width */
            background: #10121b;
            color: white;
            height: 100vh;
            padding: 20px;
            position: fixed; /* important */
            left: 0;
            top: 0;
        }

        .sidebar-header h3 {
            font-size: 22px;
            margin: 0;
            font-weight: 700;
        }

        .sidebar-title {
            font-size: 22px;
            font-weight: bold;
            color: #ffffff;
            padding: 20px 15px;
            line-height: 1.5;
        }

            .sidebar-title .line1 {
                display: block;
            }

            .sidebar-title .line2 {
                display: block;
                font-size: 16px;
                font-weight: 500;
                opacity: 0.8;
            }


        .sidebar-menu {
            margin-top: 30px;
            list-style: none;
            padding-left: 0;
        }

            .sidebar-menu li {
                margin: 18px 0;
            }

                .sidebar-menu li a {
                    text-decoration: none;
                    color: white;
                    font-size: 15px;
                    display: flex;
                    align-items: center;
                    gap: 10px;
                }

                    .sidebar-menu li a:hover {
                        color: #4cc3ff;
                    }

        /* ================= ADMIN BOX AT BOTTOM ================= */
        .admin-box {
            position: absolute;
            bottom: 20px;
            left: 20px;
            display: flex;
            gap: 10px;
            color: white;
        }

        .admin-name {
            margin: 0;
            font-size: 15px;
            font-weight: 600;
        }

        .admin-role {
            margin-top: -5px;
            font-size: 12px;
            color: #a8a8a8;
        }

        /* ================= CONTENT AREA FIX ================= */
        .content-wrapper {
            margin-left: 240px; /* match sidebar width */
            padding: 30px;
            width: calc(100% - 240px);
        }

        
    </style>
</head>

<body>

    <div class="wrapper">

        <!-- SIDEBAR -->
        <div class="sidebar">

            <div>
                <div class="sidebar-title">
                    <span class="line1">Student Portal</span>
                    <span class="line2">Management Registration Form</span>
                </div>

                <ul class="sidebar-menu">
                    <li>
                        <a href="/Dashboard/Index">
                            <i class="fa fa-chart-line"></i> Dashboard
                        </a>
                    </li>

                    <li>
                        <a href="/Student/Index">
                            <i class="fa fa-users"></i> Students
                        </a>
                    </li>

                    <li>
                        <a href="/Reports/Index">
                            <i class="fa fa-file-alt"></i> Reports
                        </a>
                    </li>

                    <li>
                        <a href="/Settings/Index">
                            <i class="fa fa-gear"></i> Settings
                        </a>
                    </li>
                </ul>
            </div>

            <!-- ADMIN USER (BOTTOM FIXED) -->
            <div class="admin-box">
                <i class="fa fa-user-shield"></i>
                <div>
                    <p class="admin-name">Admin User</p>
                    <p class="admin-role">Administrator</p>
                </div>
            </div>

        </div>

        <!-- CONTENT -->
        <div class="content-wrapper">
            @RenderBody()
        </div>

    </div>

</body>
</html>

/Models
Dashboard.cs

using System;
using System.Collections.Generic;

namespace StudentManagement.Models
{
    public class Dashboard
    {
        public int TotalStudents { get; set; }
        public int TodayCount { get; set; }
        public int AgeRangeCount { get; set; }
        public Dictionary<string, int> StudentsByCourse { get; set; }
        public List<Student> LatestStudents { get; set; }
    }

}

Student.cs

using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

namespace StudentManagement.Models
{
    public class Student
    {
        public int Id { get; set; }
        public string FirstName { get; set; } = "";
        public string LastName { get; set; } = "";
        public string Email { get; set; } = "";
        public string Phone { get; set; } = "";
        public DateTime DateOfBirth { get; set; }
        public string Course { get; set; } = "";
        public DateTime RegisteredAt { get; set; }
    }
}

StudentRepository.cs

using System;
using System.Collections.Generic;
using System.Configuration;
using System.Data;
using System.Data.SqlClient;
using System.Linq;
using System.Web;

namespace StudentManagement.Models
{
    public class StudentRepository
    {
        private readonly string _connectionString =
            ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;

        // Get all
        public List<Student> GetAllStudents()
        {
            var list = new List<Student>();
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_GetAllStudents", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                using (var dr = cmd.ExecuteReader())
                {
                    while (dr.Read())
                    {
                        list.Add(ReadStudent(dr));
                    }
                }
            }
            return list;
        }


        

        // Get by id
        public Student GetById(int id)
        {
            Student s = null;
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_GetStudentById", con) { CommandType = CommandType.StoredProcedure })
            {
                cmd.Parameters.AddWithValue("@Id", id);
                con.Open();
                using (var dr = cmd.ExecuteReader())
                {
                    if (dr.Read()) s = ReadStudent(dr);
                }
            }
            return s;
        }

        // Add
        public void Add(Student s)
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_AddStudent", con) { CommandType = CommandType.StoredProcedure })
            {
                cmd.Parameters.AddWithValue("@FirstName", s.FirstName);
                cmd.Parameters.AddWithValue("@LastName", s.LastName);
                cmd.Parameters.AddWithValue("@Email", s.Email);
                cmd.Parameters.AddWithValue("@Phone", s.Phone);
                cmd.Parameters.AddWithValue("@DateOfBirth", s.DateOfBirth);
                cmd.Parameters.AddWithValue("@Course", s.Course);

                con.Open();
                cmd.ExecuteNonQuery();
            }
        }

        // Update
        public void Update(Student s)
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_UpdateStudent", con) { CommandType = CommandType.StoredProcedure })
            {
                cmd.Parameters.AddWithValue("@Id", s.Id);
                cmd.Parameters.AddWithValue("@FirstName", s.FirstName);
                cmd.Parameters.AddWithValue("@LastName", s.LastName);
                cmd.Parameters.AddWithValue("@Email", s.Email);
                cmd.Parameters.AddWithValue("@Phone", s.Phone);
                cmd.Parameters.AddWithValue("@DateOfBirth", s.DateOfBirth);
                cmd.Parameters.AddWithValue("@Course", s.Course);

                con.Open();
                cmd.ExecuteNonQuery();
            }
        }

        // Delete
        public void Delete(int id)
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_DeleteStudent", con) { CommandType = CommandType.StoredProcedure })
            {
                cmd.Parameters.AddWithValue("@Id", id);
                con.Open();
                cmd.ExecuteNonQuery();
            }
        }

        // Search
        public List<Student> Search(string search)
        {
            var list = new List<Student>();
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_SearchStudents", con) { CommandType = CommandType.StoredProcedure })
            {
                cmd.Parameters.AddWithValue("@Search", search ?? "");
                con.Open();
                using (var dr = cmd.ExecuteReader())
                {
                    while (dr.Read()) list.Add(ReadStudent(dr));
                }
            }
            return list;
        }

        // Dashboard helpers
        public int GetTotalStudents()
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_TotalStudent", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                return Convert.ToInt32(cmd.ExecuteScalar());
            }
        }

        public int GetTodayStudents()
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_TodaysStudent", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                return Convert.ToInt32(cmd.ExecuteScalar());
            }
        }

        public int GetAgeRangeStudents()
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_StudentsAgeRanges", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                return Convert.ToInt32(cmd.ExecuteScalar());
            }
        }

        public Dictionary<string, int> GetStudentsByCourse()
        {
            var dict = new Dictionary<string, int>();
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_StudentsByCourses", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                using (var dr = cmd.ExecuteReader())
                {
                    while (dr.Read()) dict.Add(dr["Course"].ToString(), Convert.ToInt32(dr["Total"]));
                }
            }
            return dict;
        }

        public int GetStudentAgeRange()
        {
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_StudentAgeRanges", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                return Convert.ToInt32(cmd.ExecuteScalar());
            }
        }


        public List<Student> GetLatestStudent()
        {
            var list = new List<Student>();
            using (var con = new SqlConnection(_connectionString))
            using (var cmd = new SqlCommand("sp_LatestStudent", con) { CommandType = CommandType.StoredProcedure })
            {
                con.Open();
                using (var dr = cmd.ExecuteReader())
                {
                    while (dr.Read()) list.Add(ReadStudent(dr));
                }
            }
            return list;
        }

        // Excel: GetAll is used

        private Student ReadStudent(SqlDataReader dr)
        {
            return new Student
            {
                Id = Convert.ToInt32(dr["Id"]),
                FirstName = dr["FirstName"].ToString(),
                LastName = dr["LastName"].ToString(),
                Email = dr["Email"].ToString(),
                Phone = dr["Phone"].ToString(),
                DateOfBirth = Convert.ToDateTime(dr["DateOfBirth"]),
                Course = dr["Course"].ToString(),
                RegisteredAt = Convert.ToDateTime(dr["RegisteredAt"])
            };
        }
    }
}

/Controllers

DashboardController.cs

using StudentManagement.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace StudentManagement.Controllers
{
    public class DashboardController : Controller
    {
        private readonly StudentRepository repo = new StudentRepository();

        public ActionResult Index()
        {
            var data = new Dashboard
            {
                TotalStudents = repo.GetTotalStudents(),
                TodayCount = repo.GetTodayStudents(),
                AgeRangeCount = repo.GetAgeRangeStudents(),
                StudentsByCourse = repo.GetStudentsByCourse(),
                LatestStudents = repo.GetLatestStudent()
            };

            return View(data);
        }

    }
}

ReportsController.cs

using ClosedXML.Excel;
using StudentManagement.Models;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace StudentManagement.Controllers
{
    public class ReportsController : Controller
    {
        StudentRepository repo = new StudentRepository();

        public ActionResult Index()
        {
            ViewBag.CourseReport = repo.GetStudentsByCourse();
            ViewBag.AgeCount = repo.GetAgeRangeStudents();
            ViewBag.Total = repo.GetTotalStudents();

            return View();
        }

        // Date Filter Report
        [HttpPost]
        public ActionResult FilterDates(DateTime fromDate, DateTime toDate)
        {
            var all = repo.GetAllStudents();
            var result = all.Where(x => x.RegisteredAt.Date >= fromDate
                                     && x.RegisteredAt.Date <= toDate)
                            .ToList();

            return View("FilteredReport", result);
        }

        // Excel Download
        public FileResult DownloadExcel()
        {
            var students = repo.GetAllStudents();

            using (var wb = new XLWorkbook())
            {
                var ws = wb.Worksheets.Add("Report");
                ws.Cell(1, 1).Value = "ID";
                ws.Cell(1, 2).Value = "Name";
                ws.Cell(1, 3).Value = "Email";
                ws.Cell(1, 4).Value = "Course";
                ws.Cell(1, 5).Value = "Registered";

                int r = 2;
                foreach (var s in students)
                {
                    ws.Cell(r, 1).Value = s.Id;
                    ws.Cell(r, 2).Value = s.FirstName + " " + s.LastName;
                    ws.Cell(r, 3).Value = s.Email;
                    ws.Cell(r, 4).Value = s.Course;
                    ws.Cell(r, 5).Value = s.RegisteredAt.ToString("dd/MM/yyyy");
                    r++;
                }

                using (var ms = new MemoryStream())
                {
                    wb.SaveAs(ms);
                    return File(ms.ToArray(),
                        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                        "StudentReport.xlsx");
                }
            }

        }
    }
}

StudentController.cs

using StudentManagement.Models;
using System;
using System.Collections.Generic;
using System.IO;
using ClosedXML.Excel;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace StudentManagement.Controllers
{
    public class StudentController : Controller
    {
        private readonly StudentRepository repo = new StudentRepository();

        public ActionResult Index(string search, int page = 1, int pageSize = 5)
        {
            var allStudents = string.IsNullOrEmpty(search)
                                ? repo.GetAllStudents()
                                : repo.Search(search);

            int totalRecords = allStudents.Count;
            int totalPages = (int)Math.Ceiling((double)totalRecords / pageSize);

            var pagedData = allStudents
                            .Skip((page - 1) * pageSize)
                            .Take(pageSize)
                            .ToList();

            ViewBag.Page = page;
            ViewBag.TotalPages = totalPages;
            ViewBag.Search = search;

            return View(pagedData);
        }


        public ActionResult Details(int id) => View(repo.GetById(id));

        public ActionResult Create() => View();

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult Create(Student s)
        {
            if (!ModelState.IsValid) return View(s);

            // Age check
            var today = DateTime.Today;
            var age = today.Year - s.DateOfBirth.Year;
            if (s.DateOfBirth > today.AddYears(-age)) age--;
            if (age < 18 || age > 25)
            {
                ModelState.AddModelError("DateOfBirth", "Age must be between 18 and 25.");
                return View(s);
            }

            repo.Add(s);
            TempData["Success"] = "Student added.";
            return RedirectToAction("Index");
        }

        public ActionResult Edit(int id) => View(repo.GetById(id));

        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult Edit(Student s)
        {
            if (!ModelState.IsValid) return View(s);
            repo.Update(s);
            TempData["Success"] = "Student updated.";
            return RedirectToAction("Index");
        }

        public ActionResult Delete(int id)
        {
            var student = repo.GetById(id);
            return View(student);
        }


        [HttpPost, ActionName("Delete")]
        public ActionResult DeleteConfirmed(int id)
        {
            repo.Delete(id);
            return RedirectToAction("Index");
            

        }





        // Excel export
        public FileResult ExportToExcel()
        {
            var students = repo.GetAllStudents();
            using (var wb = new XLWorkbook())
            {
                var ws = wb.Worksheets.Add("Students");
                ws.Cell(1, 1).Value = "Id";
                ws.Cell(1, 2).Value = "FirstName";
                ws.Cell(1, 3).Value = "LastName";
                ws.Cell(1, 4).Value = "Email";
                ws.Cell(1, 5).Value = "Phone";
                ws.Cell(1, 6).Value = "DateOfBirth";
                ws.Cell(1, 7).Value = "Course";
                ws.Cell(1, 8).Value = "RegisteredAt";

                int r = 2;
                foreach (var s in students)
                {
                    ws.Cell(r, 1).Value = s.Id;
                    ws.Cell(r, 2).Value = s.FirstName;
                    ws.Cell(r, 3).Value = s.LastName;
                    ws.Cell(r, 4).Value = s.Email;
                    ws.Cell(r, 5).Value = s.Phone;
                    ws.Cell(r, 6).Value = s.DateOfBirth.ToString("dd/MM/yyyy");
                    ws.Cell(r, 7).Value = s.Course;
                    ws.Cell(r, 8).Value = s.RegisteredAt.ToString("dd/MM/yyyy HH:mm");
                    r++;
                }

                ws.Columns().AdjustToContents();
                using (var ms = new MemoryStream())
                {
                    wb.SaveAs(ms);
                    return File(ms.ToArray(),
                                "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                                "Students.xlsx");
                }
            }
        }
    }
}

SettingsController.cs

using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace StudentManagement.Controllers
{
    public class SettingsController : Controller
    {
        // Page Load
        public ActionResult Index()
        {
            return View();
        }

        // Save Settings
        [HttpPost]
        public ActionResult SaveSettings(string theme, int pageSize)
        {
            // TEMP SAVE (can use DB also)
            Session["theme"] = theme;
            Session["pageSize"] = pageSize;

            TempData["Success"] = "Settings Saved Successfully!";
            return RedirectToAction("Index");
        }
    }
}



Web.Config (Connection String)

<connectionStrings>
	<add name="DefaultConnection" connectionString="Data Source=LAPTOP-GFRGLR8G\SQLEXPRESS01;Initial Catalog=StudentPortal;Integrated Security=SSPI " providerName="System.Data.SqlClient" />
</connectionStrings>

RouteConfig.cs

defaults: new { controller = "Dashboard", action = "Index", id = UrlParameter.Optional }



