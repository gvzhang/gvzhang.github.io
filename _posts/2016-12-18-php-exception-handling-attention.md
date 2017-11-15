---
title: PHP异常处理注意项
categories:
 - PHP
tags:
 - 异常
 - exception
---

在实际开发中，错误及异常捕捉仅仅靠try{}catch()是远远不够的。
所以引用以下几中函数。
1. set_error_handler
一般用于捕捉  E_NOTICE 、E_USER_ERROR、E_USER_WARNING、E_USER_NOTICE
不能捕捉：
E_ERROR, E_PARSE, E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR and E_COMPILE_WARNING。
2. set_exception_handler 
设置默认的异常处理程序，用于没有用 try/catch 块来捕获的异常。 在 exception_handler 调用后异常会中止。 
与throw new Exception('Uncaught Exception occurred')，连用。
3. register_shutdown_function 
执行机制是：php把要调用的函数调入内存。当页面所有ＰＨＰ语句都执行完成时，再调用此函数。
一般与trigger_error("...", E_USER_ERROR)，配合使用。

但是在写日志的处理过程中，也需要注意其中出现异常的情况。
```php
<?php
set_error_handler('_error_handler');
set_exception_handler('_exception_handler');
register_shutdown_function('_shutdown_handler');

function _exception_handler($exception)
{
	//避免循环处理异常
	restore_error_handler();
	restore_exception_handler();
	
	write_log();
	exit(1);
}

function _error_handler($severity, $message, $filepath, $line)
{
	write_log();
	exit(1);
}

function _shutdown_handler()
{
	$last_error = error_get_last();
	if (isset($last_error) &&
		($last_error['type'] & (E_ERROR | E_PARSE | E_CORE_ERROR | E_CORE_WARNING | E_COMPILE_ERROR | E_COMPILE_WARNING)))
	{
		var_dump($last_error['message']);
		exit(1);
	}
}

throw new Exception("test");


/**
 * 写入日志
 * _error_handler发生Exception，_exception_handler不会捕获
 * _exception_handler发生E_NOTICE，_error_handler会捕获
 * 抛出自身级别的异常，不会循环处理
 */
function write_log()
{
	$filepath = "test.log";
	if (!$fp = @fopen($filepath, 'ab')) {
		return FALSE;
	}

	flock($fp, LOCK_EX);

	$description = [1, 2, 3];
	//此处抛出 E_NOTICE 错误
	$message = sprintf("错误描述：%s", $description) . "\n";

	$result = fwrite($fp, $message);

	flock($fp, LOCK_UN);
	fclose($fp);

	return is_int($result);
}
```
