---
title:  "Grade calculator for MAC2312"
date:   2019-12-03 08:00:00 -0500
layout: single
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
Enter the grades received on tests 1–4, the grade for homeworks and quizzes, as well as the expected grade on the final, and obtain the resulting grade for MAC2312.
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
<label class="col-md-2 control-label" for="one"> Test 1 </label> 
<div class="col-md-3">
<input id="one" placeholder="100" class="form-control input-md" type="text">
<span class="help-block"> Grade you received on test 1</span>
</div>

<label class="col-md-2 control-label" for="two"> Test 2 </label> 
<div class="col-md-3">
<input id="two" placeholder="100" class="form-control input-md" type="text">
<span class="help-block"> Grade you received on test 2</span>
</div>
</div>

<div class="form-group">
<label class="col-md-2 control-label" for="three"> Test 3 </label> 
<div class="col-md-3">
<input id="three" placeholder="100" class="form-control input-md" type="text">
<span class="help-block"> Grade you received on test 3</span>
</div>

<label class="col-md-2 control-label" for="four"> Test 4 </label> 
<div class="col-md-3">
<input id="four" placeholder="100" class="form-control input-md" type="text">
<span class="help-block"> Grade you received on test 4</span>
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
        var hw = 0, tests = [0,0,0,0], f = 0, r = 0, m =0;

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
            hw = evaluate("#hw");
            tests[1] = evaluate("#one");
            tests[2] = evaluate("#two");
            tests[3] = evaluate("#three");
            tests[4] = evaluate("#four");
            f = evaluate("#final");

            m = Math.min(tests[1],tests[2],tests[3],tests[4]);
            r =  
                0.6*( tests[1]+tests[2]+tests[3]+tests[4] )/4
                // 0.6*( tests[1]+tests[2]+tests[3]+tests[4] + Math.max((f-m)/2, 0) )/4
                +0.1*hw
                +0.3*f;

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

            // if (r>89)
            //     mes = " (A)";
            // else if (r>88)
            //     mes = " (A-)";
            // else if (r>79)
            //     mes = " (B)";
            // else if (r>69)
            //     mes = " (C)";
            // else if (r>67)
            //     mes = " (C-)";
            // else if (r>50)
            //     mes = " (D)";
            // else if (r>0)
            //     mes = " (F)";

            $("#result").text(format(String(r))+mes);
        }

        $("#one,#two,#three,#four,#hw,#final").keyup(function (e) {
                calculate();
                });

        calculate();
});
</script>
