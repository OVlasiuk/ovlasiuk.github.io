---
title:  "Grade calculator for MAC2312"
date:   2019-03-29 08:00:00 -0500
layout: singlemath
categories: Teaching

--- 

<head>
<meta charset="utf-8">
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
<script src="https://code.jquery.com/jquery-1.11.3.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
</head>

<div class="container">
<p>
Enter the average grade received on tests 1–3, the grade for homeworks and quizzes, as well as the expected grades on the 4th
test and the final, and obtain the resulting grade for MAC2312.
</p>

<form class="form-horizontal">
<fieldset>

<div class="form-group">
<label class="col-md-4 control-label" for="hw"> HW + quizzes </label> 
<div class="col-md-4">
<input id="hw" placeholder="100" class="form-control input-md" type="text">
<span class="help-block">Percentage earned on homeworks and quizzes (combined)</span>
</div>
</div>


<div class="form-group">
<label class="col-md-4 control-label" for="average"> Tests 1–3 </label> 
<div class="col-md-4">
<input id="average" placeholder="100" class="form-control input-md" type="text">
<span class="help-block">
Average of the grades you received on tests 1–3</span>
</div>
</div>

<div class="form-group">
<label class="col-md-4 control-label" for="fourth"> 4th test </label> 
<div class="col-md-4">
<input id="fourth" placeholder="100" class="form-control input-md" type="text">
<span class="help-block">Expected grade on the 4th test</span>
</div>
</div>




<div class="form-group">
<label class="col-md-4 control-label" for="final"> Final </label> 
<div class="col-md-4">
<input id="final" placeholder="100" class="form-control input-md" type="text">
<span class="help-block">Expected grade on the final</span>
</div>
</div>

<div class="form-group">
<label class="col-md-4 control-label" for="result"> Your grade: </label> 
<div class="col-md-4">
<div id="result" class="form-control">
</div>
<span class="help-block">The resulting grade for this class, assuming you performed as
shown above</span>
</div>
</div>

</fieldset>
</form>
</div>

<script>
$(function () {
        var n = 0, p = 0, f = 0, r = 0, q=0;

        function format(x) {
        x = String(x);
        var n = x.indexOf('.');
        if (n <= 0) { return x; }
        return x.substr(0, n + 3);
        }

        function evaluate(selector) {
        var x = $(selector).val();

        if (x.indexOf('*') > 0 || x.indexOf('+') > 0 || x.indexOf('/') > 0) {
        x = eval(x);
        }

        return Number(x);
        }

        function calculate() {
            var mes = "";
            q = evaluate("#hw");
            n = evaluate("#average");
            p = evaluate("#fourth");
            f = evaluate("#final");

            r = (3*(3*n+p)/2.0 + q + 3*f)/10.0;
            if (r>89)
                mes = " (A)";
            else if (r>79)
                mes = " (B)";
            else if (r>69)
                mes = " (C)";
            else if (r>59)
                mes = " (D)";
            else if (r>0)
                mes = " (F)";

            $("#result").text(String(r)+mes);
        }

        $("#average,#fourth,#final").keyup(function (e) {
                calculate();
                });

        calculate();
});
</script>
