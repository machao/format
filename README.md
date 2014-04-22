极简JS模版填充引擎 format
==================

极简js模版填充引擎，30行代码，支持数组填充、对象填充和多参填充等功能。

另有一款功能更复杂的极简模版引擎：[tmpl](https://github.com/machao/tmpl/ "极简JS模版引擎tmpl")

源码：

	(function($, undefined) {
    //测试浏览器是否支持正则表达式预编译
    var baseReg = /\{([\w\.]+)\}/g, numReg = /^\d+$/,
    	//预编译核心的正则表达式，以提高正则匹配效率
        formatReg = baseReg.compile ? baseReg.compile(baseReg.source, "g") || baseReg : baseReg,
    	//其他工具函数
        toString = Object.prototype.toString, slice = Array.prototype.slice;
    //对外接口
    $.format = function(string, source){
        if( source === undefined || source === null )return string;
        var isArray = true, type = toString.call(source),
            //检测数据源
            data = type === "[object Object]" ? (isArray = false, source) : type === "[object Array]" ? source : slice.call(arguments, 1),
            N = isArray ? data.length : 0;
        //执行替换
        return String(string).replace(formatReg, function(match, index) {
            var isNumber = numReg.test(index), n, fnPath, val;
            if( isNumber && isArray ){
                n = parseInt(index, 10);
                return n < N ? data[n] : match;
            }else{ //数据源为对象，则遍历逐级查找数据
                fnPath = index.split(".");
                val = data;
                for(var i=0; i<fnPath.length; i++)
                    val = val[fnPath[i]];
                return val === undefined ? match : val;
            }
        });
    };
	})(window.jQuery || window);

对象数据填充：

	var templ = "this is a easy {name}.";
	var ret = $.format(templ, {name:"demo"});
	//ret:  "this is a easy demo."

多参数数据填充：
	
	var templ = "this is a {1} {0}.";
	var ret = $.format(templ, "demo", "easy");
	//ret:  "this is a easy demo."

数组数据源填充：
	
	var templ = "this is a {0} {1}.";
	var ret = $.format(templ, ["easy", "demo"]);
	//ret: "this is a easy demo.",

多级对象数据源填充：

	var templ = "my name is {my.fullName}.";
	var ret = $.format(templ, {
		"my":{
			"firstName" : "Ma",
			"fullName" : "MaChao"
		}
	});
	//ret: "my name is MaChao."
	
其他说明：
	
1. 模版占位符是 {key}，跟其他模版的 ${key} 不同，请留意（原因是：我想不出加一个$的必要性，罗里八嗦的）；
2. 如果没有提供对应的数据，则模版占位符原封保留；
3. 仅仅支持字符串回填，其他数据类型将被强制转化为字符串；