# lc的hot100（一）

### 哈希表

​		**哈希表**是根据关键码的值而直接进行访问的数据结构。

​		**哈希函数**：通过hashCode把其他数据格式转化为不同的数值，hashCode取得的数值大于哈希表的大小则再次取模。

​		**哈希碰撞**：拉链法和线性探测法。

​				1.拉链法：被储存在冲突索引处的链表中。

​				2.线性探测法：冲突位置向下找空位存放冲突数据。

​		**常见哈希结构**：数组，set集合，map映射。

#### [1. 两数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/two-sum/description)

方法一：暴力枚举

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
  for(let i=0;i<nums.length;i++){
     for(let j=i+1;j<nums.length;j++){
       if(nums[i]+nums[j] == target)
         return [i,j];
     }
  }
};
```

方法二：哈希表

```javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
  let map = new Map();
  for(let i=0; i<nums.length; i++){
     if(map.has(target - nums[i])){
       return [map.get(target - nums[i]), i];
     }else{
       map.set(nums[i], i);
     }
  }
  return [];
};
```

注：忘记哈希表的知识还去复习。

#### [49. 字母异位词分组 - 力扣（LeetCode）](https://leetcode.cn/problems/group-anagrams)

方法一：排序作为键

```javascript
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
var groupAnagrams = function(strs) {
  let map = new Map();
  for(let str of strs){
     let array = Array.from(str);
     array.sort();
     let key = array.toString();
     let list = map.get(key) ? map.get(key) : new Array();
     list.push(str);
     map.set(key, list);
  }
  return Array.from(map.values());
};
```

方法二：字母计数作为键

```javascript
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
var groupAnagrams = function(strs) {
    const map = new Object();
    for (let s of strs) {
        const count = new Array(26).fill(0);
        for (let c of s) {
            count[c.charCodeAt() - 'a'.charCodeAt()]++;
        }
        map[count] ? map[count].push(s) : map[count] = [s];
    }
    return Object.values(map);
};
```

注：思路虽然知道是哈希表，如果是个人不看题解的话，可能对于键的确定不太流畅，卡在了最关键的地方。

#### [128. 最长连续序列 - 力扣（LeetCode）](https://leetcode.cn/problems/longest-consecutive-sequence/description)

脑子里面都是哈希+叠判断，这个题解是真神的，抄了，明天整理动态规划。

```
/**
 * @param {number[]} nums
 * @return {number}
 */
var longestConsecutive = function(nums) {
    let map=new Map(),maxCount=0;
    for(let i=0;i<nums.length;i++){
        // 避免重复计算相同的元素
        if(!map.get(nums[i])){
            preLen=map.get(nums[i]-1)||0;
            lastLen=map.get(nums[i]+1)||0;
            curLen=preLen+1+lastLen;
            maxCount=Math.max(maxCount,curLen);
            map.set(nums[i],curLen);
            map.set(nums[i]-preLen,curLen);
            map.set(nums[i]+lastLen,curLen);
        }
    }
    return maxCount;
};
```

