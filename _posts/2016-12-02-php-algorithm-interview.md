---
title: 一个PHP算法面试题优化过程
categories:
 - PHP
tags:
 - 算法
 - 面试
---

```php
$listData = [
    '111' => ['a', 'b', 'c', 'a'],
    '222' => ['d', 'e', 'f', 'f', 'b'],
    '333' => ['g', 'h'],
    '444' => ['i', 'j'],
    ...
];
```
> 定义一个函数，传入$listData
如果111里面的元素，和 222/333/444... 里面的元素有重复，返回false 
如果222里面的元素，和 111/333/444... 里面的元素有重复，返回false 
如果333里面的元素，和 111/222/444... 里面的元素有重复，返回false 
如果 ...
允许 111/222/333/444 自己里面的元素重复，返回true 其他情况返回true

> 已知:
$listData长度未知
111/222/333/444... 的长度未知
111/222/333/444... 里的元素为字符串和数字

## 优化过程

```php
$listData = [
    '111' => ['a', 'b', 'c', 'a'],
    '222' => ['d', 'e', 'f', 'f', 'z'],
    '333' => ['g', 'h'],
    '444' => ['i', 'j'],
    '555' => ['p', 'q', 'x'],
    '666' => ['k', 'l', 'n', 'm'],
    '777' => ['o', 'r', 's'],
    '888' => ['t', 'u', 'v'],
    '999' => ['w', 'y']
];


/**
 * 最简单的方式
 * @param $listData
 * @return bool
 * @spent 0.001424 seconds
 */
function checkRepeat($listData)
{
    foreach ($listData as $key => $row) {
        foreach ($row as $val) {
            foreach ($listData as $subKey => $subRow) {
                if ($key != $subKey && in_array($val, $subRow)) {
                    return false;
                }
            }
        }
    }
    return true;
}

$beginTime = microtime(true);
$result = checkRepeat($listData);
$endTime = microtime(true);
echo "spent " . round($endTime - $beginTime, 6) . " seconds";
var_dump($result);
echo "<br /><br /><br />";


/**
 * 使用了array_intersect做交集比较
 * @param $listData
 * @return bool
 * @spent 0.000594 seconds
 */
function checkRepeat2($listData)
{
    foreach ($listData as $key => $row) {
        foreach ($listData as $subKey => $subRow) {
            if ($key != $subKey && array_intersect($row, $subRow)) {
                return false;
            }
        }
    }
    return true;
}

$beginTime = microtime(true);
$result = checkRepeat2($listData);
$endTime = microtime(true);
echo "spent " . round($endTime - $beginTime, 6) . " seconds";
var_dump($result);
echo "<br /><br /><br />";


/**
 * 使用了array_intersect做交集，删除重复比较
 * @param $listData
 * @return bool
 * @spent 0.000305 seconds
 */
function checkRepeat3($listData)
{
    $subListData = $listData;
    foreach ($listData as $key => $row) {
        unset($subListData[$key]);
        foreach ($subListData as $subKey => $subRow) {
            if (array_intersect($row, $subRow)) {
                return false;
            }
        }
    }
    return true;
}

$beginTime = microtime(true);
$result = checkRepeat3($listData);
$endTime = microtime(true);
echo "spent " . round($endTime - $beginTime, 6) . " seconds";
var_dump($result);
echo "<br /><br /><br />";


/**
 * 使用array_merge合并后在做交集
 * @param $listData
 * @return bool
 * @spent 0.000194 seconds
 */
function checkRepeat4($listData)
{
    $check_arr = $listData;
    foreach ($listData as $key => $val) {
        // 删除当前key
        unset($check_arr[$key]);
        if ($check_arr) {
            // 合并删除后的数组，判断是否存在交集
            //As PHP 5.6 you can use array_merge + "splat" operator to reduce a bidimensonal array to a simple array:
            if (array_intersect($val, array_merge(...$check_arr))) {
                return false;
            }
        }
    }
    return true;
}

$beginTime = microtime(true);
$result = checkRepeat4($listData);
$endTime = microtime(true);
echo "spent " . round($endTime - $beginTime, 6) . " seconds";
var_dump($result);
echo "<br /><br /><br />";


/**
 * https://segmentfault.com/q/1010000007527180 @shiji 的答案
 * 子数组先去重再合并的结果数量 和 先合并子数组再去重的结果数量 做比较。
 * 如果是相同的，意味着不存在跨子数组的重复，只存在子数组内部重复，所以`True`
 * @param $listData
 * @return bool
 * @spent 0.000153 seconds 但是当有重复项时速度很慢
 */
function checkRepeat5($listData)
{
    $result = array();
    foreach ($listData as $line) {
        //子数组内部去重,再组装回原来的格式
        $result[] = array_unique($line);
    }

    return count(array_merge(...$result)) === count(array_unique(array_merge(...$listData)));
}

$beginTime = microtime(true);
$result = checkRepeat5($listData);
$endTime = microtime(true);
echo "spent " . round($endTime - $beginTime, 6) . " seconds";
var_dump($result);
echo "<br /><br /><br />";
```
