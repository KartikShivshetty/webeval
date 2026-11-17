var mysql = require('mysql');

var con = mysql.createConnection({
    host: "127.0.0.1",
    user: "root",
    password: "",
    database: "d1"
});

con.connect(function(err){

    if(err) throw err;

    console.log("Connected");

    var sql = "CREATE TABLE student(roll INT PRIMARY KEY, name VARCHAR(50), m1 FLOAT, m2 FLOAT, m3 FLOAT, total FLOAT, percentage FLOAT, grade VARCHAR(5))";

    con.query(sql,function(err,result){

        if(err) throw err;

        console.log("Table created");

        con.end();

    });

});
