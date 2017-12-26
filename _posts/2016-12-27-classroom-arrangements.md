---
title: 教室根据课程排位
categories:
 - 算法
tags:
 - 算法
---

这是segmentfault中的一个问题，尝试把算法写出来了。

> ![问题图片](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/post/58573ef41a98f.png)

> 十间教室 六门课程教室支持课程如图所示

> 要求举例说明
安排甲课程时 可选择 a b c d e五间教室
假如甲课程占用b c d e教室 还剩余a 教室空闲

> 安排乙课程能选三间教室 首选 f g 教室 但因a 教室空闲 所以自动 把甲课程某一项移入a教室 空出公共的教室以供 乙课程使用

> 以此类推 后者优先 当没有教室可分配时 提示 无法分配

```php
<?php

class ClassRoomArrange
{
    /**
     * 课程可用教室
     * @var array
     */
    private $courseInit = [
        "aCourse" => ["A", "B", "C", "D", "E"],
        "bCourse" => ["B", "C", "D", "E", "F", "G"],
        "cCourse" => ["H", "J", "K"],
        "dCourse" => ["A", "B", "C"],
        "eCourse" => ["C", "D", "E", "F", "G"],
        "fCourse" => ["H", "J", "K"],
    ];
    /**
     * 解决结果
     * @var array
     */
    private $solution = [];
    /**
     * 每日安排
     * @var array
     */
    private $dailyArrange = [];
    /**
     * 日常可用教室
     * @var array
     */
    public $availableClassroom = ["A", "B", "C", "D", "E", "F", "G", "H", "J", "K"];

    public function __construct($dailyArrange)
    {
        $this->dailyArrange = $dailyArrange;
    }

    /**
     * 教室安排
     * @return array | boolean
     */
    public function ArrangeClassroom()
    {
        if (empty($this->dailyArrange)) {
            return false;
        }

        //排序优先教室
        krsort($this->dailyArrange);
        foreach ($this->dailyArrange as $course => $count) {
            //交集得到课程可用教室
            $used = [];
            $intersect = array_intersect($this->availableClassroom, $this->courseInit[$course]);
            if ($intersect) {
                //从可用教室里面去所取需课程数
                $used = array_slice($intersect, 0, $count);
            }
            $this->solution[$course] = $used;
            //如果排位结果不满足所需数量，则尝试挪位操作
            if ($surplus = $count - count($this->solution[$course])) {
                //获取剩余所属教室
                $z = 0;
                foreach ($this->courseInit[$course] as $letter) {
                    //从其他的排位中尝试挪位
                    if (!in_array($letter, $this->solution[$course]) && $this->move($letter, [$course])) {
                        if (++$z > $surplus) {
                            break;
                        }
                        array_push($this->solution[$course], $letter);
                    }
                }
            }

            //差集删除已用教室
            $this->availableClassroom = array_diff($this->availableClassroom, $used);
        }
        return $this->solution;
    }

    /**
     * 如果排位结果不满足所需数量，尝试挪位
     * @param $letter
     * @param $exclude
     * @return bool
     */
    private function move($letter, $exclude)
    {
        //从排位结果中查找可挪位的结果
        foreach ($this->solution as $c => &$r) {
            //如果需要移动的教室在排位结果中的，则尝试挪位
            if (!in_array($c, $exclude) && in_array($letter, $r)) {
                $fix = $r;
                //将排除列已排序字母也作为排出项
                foreach ($exclude as $e) {
                    $fix = array_merge($fix, $this->solution[$e]);
                }
                //获取可移动的教室
                $initSurplus = array_diff($this->courseInit[$c], $fix);
                //该教室在某个安排中是不可移动了，则说明不能再挪了
                if(empty($initSurplus)){
                    return false;
                }
                //取出教室往后移
                array_push($exclude, $c);
                foreach ($initSurplus as $moveLetter) {
                    //如果移动后会影响其他排位则再尝试挪位
                    if ($this->checkMove($moveLetter, $c) || $this->move($moveLetter, $exclude)) {
                        $r = array_merge(array_diff($r, [$letter]));
                        array_push($r, $moveLetter);
                        return true;
                    }
                }
            }
        }
        return false;
    }

    /**
     * 检查是否影响其他排位
     * @param $a
     * @param $c
     * @return bool
     */
    private function checkMove($a, $c)
    {
        foreach ($this->solution as $cc => $rr) {
            if ($cc !== $c && in_array($a, $rr)) {
                return false;
            }
        }
        return true;
    }
}

//周一需要安排哪些课程，一个课程需要安排多少节
$mondayArrange = ["aCourse" => 4, "bCourse" => 2, "cCourse" => 1, "eCourse"=>1, "fCourse" => 2];
$classRoomArrange = new ClassRoomArrange($mondayArrange);
var_dump($classRoomArrange->ArrangeClassroom());

/*
返回结果：
array (size=6)
  'fCourse' =>
    array (size=1)
      0 => string 'H' (length=1)
  'eCourse' =>
    array (size=3)
      0 => string 'E' (length=1)
      1 => string 'F' (length=1)
      2 => string 'G' (length=1)
  'dCourse' =>
    array (size=1)
      0 => string 'C' (length=1)
  'cCourse' =>
    array (size=2)
      0 => string 'J' (length=1)
      1 => string 'K' (length=1)
  'bCourse' =>
    array (size=1)
      0 => string 'D' (length=1)
  'aCourse' =>
    array (size=2)
      0 => string 'A' (length=1)
      1 => string 'B' (length=1)
*/
```
