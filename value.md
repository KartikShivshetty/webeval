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

    var sql = "INSERT INTO student(roll,name,m1,m2,m3,total,percentage,grade) VALUES(101,'Rahul',90,80,100,270,90,'A+')";

    con.query(sql,function(err,result){

        if(err) throw err;

        console.log("Values inserted");

        con.end();

    });

});
