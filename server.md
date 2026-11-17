var express = require('express');
var mysql = require('mysql');
var bodyParser = require('body-parser');

var app = express();

var urlencodedParser = bodyParser.urlencoded({extended:true});

// Database Connection

var con = mysql.createConnection({
    host: "127.0.0.1",
    user: "root",
    password: "",
    database: "d1"
});

con.connect(function(err){
    if(err) throw err;
    console.log("Connected to MySQL");
});

// CALCULATE FUNCTION

function calculate(m1,m2,m3)
{
    var total = m1+m2+m3;

    var percentage = total/3;

    var grade;

    if(percentage >= 90)
        grade = "A+";
    else if(percentage >= 75)
        grade = "A";
    else if(percentage >= 60)
        grade = "B";
    else if(percentage >= 50)
        grade = "C";
    else
        grade = "F";

    return {
        total:total,
        percentage:percentage,
        grade:grade
    };
}

// STUDENT FORM

app.get('/send',function(req,res){

    var rr = "<html><body>";

    rr += "<h2>Student Academic Performance</h2>";

    rr += "<form method='POST' action='/thank'>";

    rr += "Name <input type='text' name='name' required><br><br>";

    rr += "Roll Number <input type='number' name='roll' required><br><br>";

    rr += "Subject 1 <input type='number' name='m1' min='0' max='100' step='any' required><br><br>";

    rr += "Subject 2 <input type='number' name='m2' min='0' max='100' step='any' required><br><br>";

    rr += "Subject 3 <input type='number' name='m3' min='0' max='100' step='any' required><br><br>";

    rr += "<input type='submit' value='Submit'>";

    rr += "</form>";

    rr += "<br><a href='/students'>View All Students</a>";

    rr += "</body></html>";

    res.send(rr);
});

// INSERT AND CALCULATE

app.post('/thank',urlencodedParser,function(req,res){

    var name = req.body.name;
    var roll = req.body.roll;

    var m1 = Number(req.body.m1);
    var m2 = Number(req.body.m2);
    var m3 = Number(req.body.m3);

    if(!name || !name.trim() || !roll ||
       !Number.isFinite(m1) ||
       !Number.isFinite(m2) ||
       !Number.isFinite(m3) ||
       m1<0 || m1>100 ||
       m2<0 || m2>100 ||
       m3<0 || m3>100)
    {
        return res.send("Invalid Details or Marks");
    }

    var result = calculate(m1,m2,m3);

    var total = result.total;
    var percentage = result.percentage;
    var grade = result.grade;

    var sql = "INSERT INTO student(roll,name,m1,m2,m3,total,percentage,grade) VALUES(?,?,?,?,?,?,?,?)";

    con.query(sql,[roll,name,m1,m2,m3,total,percentage,grade],function(err,result1){

        if(err)
        {
            if(err.code == "ER_DUP_ENTRY")
                return res.send("Roll Number Already Exists!");

            return res.send("Database Error");
        }

        var rr = "<html><body>";

        rr += "<h2>Student Result Summary</h2>";

        rr += "Name: "+name+"<br><br>";

        rr += "Roll Number: "+roll+"<br><br>";

        rr += "Subject 1: "+m1+"<br><br>";

        rr += "Subject 2: "+m2+"<br><br>";

        rr += "Subject 3: "+m3+"<br><br>";

        rr += "Total Marks: "+total+"<br><br>";

        rr += "Percentage: "+percentage.toFixed(2)+"%<br><br>";

        rr += "Grade: "+grade+"<br><br>";

        rr += "<a href='/send'>Go Back</a>";

        rr += "</body></html>";

        res.send(rr);

    });

});

// DISPLAY ALL STUDENTS

app.get('/students',function(req,res){

    var sql = "SELECT * FROM student";

    con.query(sql,function(err,result){

        if(err) throw err;

        var html = "<html><body>";

        html += "<h2>All Students</h2>";

        html += "<table border='1' cellpadding='5'>";

        html += "<tr><th>Roll</th><th>Name</th><th>M1</th><th>M2</th><th>M3</th><th>Total</th><th>Percentage</th><th>Grade</th></tr>";

        result.forEach(function(row){

            html += "<tr>";

            html += "<td>"+row.roll+"</td>";

            html += "<td>"+row.name+"</td>";

            html += "<td>"+row.m1+"</td>";

            html += "<td>"+row.m2+"</td>";

            html += "<td>"+row.m3+"</td>";

            html += "<td>"+row.total+"</td>";

            html += "<td>"+row.percentage+"</td>";

            html += "<td>"+row.grade+"</td>";

            html += "</tr>";

        });

        html += "</table>";

        html += "<br><a href='/send'>Go Back</a>";

        html += "</body></html>";

        res.send(html);

    });

});

// SERVER

app.listen(9000,function(){

    console.log("Server running at http://localhost:9000/send");

});
