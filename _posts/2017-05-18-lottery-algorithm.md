---
title: 抽奖算法与超中
categories:
 - 算法
tags:
 - 算法
---

```php
/**
*    日期   奖品    权重  库存
*    3.1    1元     5000 5000
*    3.1    2元     1000 1000
*    3.1    3元     500  500
*    3.1    5元     100  100
*    3.1    iphone6s 1    1
*    3.1    未中奖 59409 59409
**/
//奖品库存
//未中奖：假设设定抽10次中一次, 未中奖权重 = 抽检概率导数奖品数-奖品数 = 106601-6601 = 59409
$awardStockMap = ["1元"=>5000,"2元"=>1000,"3元"=>500,"5元"=>100,"iphone"=>1, "未中奖"=>59409];
$awardWeightMap = $awardStockMap;//奖品权重，此处默认为库存
asort($awardWeightMap);

$userNum = 30000;//日活用户数
$drawNum = $userNum * 3;//每天抽奖次数 = 日活数*抽奖次数
$dailyWinCountMap = [];
//模拟每次抽奖
for ($j = 0; $j<$drawNum; $j++) {
    //排除掉库存为0的商品
    foreach ($awardStockMap as $key => $stock) {
        if ($stock == 0) {
            unset($awardWeightMap[$key]);
        }
    }

    //获取总权重
    $totalWeight = array_sum($awardWeightMap);
    //生成一个随机数
    $randNum = mt_rand(0, $totalWeight - 1);
    $prev = 0;
    $chooseAward = "";
    //组成区间 0-1 1-101 101-601 601-1601 1601-6601 6601-66010 做比较
    foreach ($awardWeightMap as $key => $weight) {
        if ($randNum >= $prev && $randNum < ($prev + $weight)) {
            $chooseAward = $key;
            break;
        }
        $prev += $weight;
    }
    
    //记录每天中奖
    if (isset($dailyWinCountMap[$chooseAward])) {
        $dailyWinCountMap[$chooseAward]++;
    } else {
        $dailyWinCountMap[$chooseAward] = 1;
    }

    //未中奖不用减库存
    if ($chooseAward != "未中奖") {
        //减库存，记录空库存
        //中奖后 会减库存 但假如库存只剩1个了 有10个用户同时落入一元区间 如何避免1-10=-9的情况
        //update award_stock set stock = stock - 1 where award_id = ? and stock > 0;
        if ((--$awardStockMap[$chooseAward])==0) {
            printf("奖品：%s 库存为空<br />", $chooseAward);
        }
    }
}
//因为生成的随机数可能相同，在“未中奖”不会减库存的情况下，会存在iphone一直都是1/59409的概率
//所以在超低概率的奖品中，会出现抽不中的情况
echo "每日各奖品中奖计数：";
foreach ($dailyWinCountMap as $key => $value) {
    echo $key.":".$value."   ";
}

```
###### 可见在存在库存条件的抽奖中，每次的抽奖概率并不是都一样的
