# DSA prep

## Question 1 
### Contains Duplicate
Given an integer array nums, return true if any value appears more than once in the array, otherwise return false.

Example 1:

Input: nums = [1, 2, 3, 3]

Output: true

### Answer
- Use a set, loop through each element if exists return true, add to set. Return false at end.

```Python
class Solution:
    def hasDuplicate(self, nums: List[int]) -> bool:
        seen = set()

        for n in nums:
            if n in seen:
                return True
            seen.add(n)

        return False
```

## Question 2
### Valid Anagram
Given two strings s and t, return true if the two strings are anagrams of each other, otherwise return false.

An anagram is a string that contains the exact same characters as another string, but the order of the characters can be different.

### Answer
- Create a `fingerprint` by looping through each char of string and return true. if each fingerprint are same.

```Python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        if len(s) != len(t):
            return False
        
        fingerprintS = {}
        fingerprintT = {}

        for c in s:
            fingerprintS[c] = fingerprintS.get(c, 0) + 1
        
        for c in t:
            fingerprintT[c] = fingerprintS.get(c, 0) + 1
        
        return fingerprintS == fingerprintT
```

## Question 3

### Two Sum
Given an array of integers nums and an integer target, return the indices i and j such that nums[i] + nums[j] == target and i != j.

You may assume that every input has exactly one pair of indices i and j that satisfy the condition.

Return the answer with the smaller index first.

### Answer
- use a dict/hashmap to store the {`number`: `index`} mapping
- loop through each element of the array 
- if current number's compliment (target-number) is in the hashmap, get the index and return both indexes
- continue storing `number`: `index`

```Python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
      seen = {}

      for i, n in enumerate(nums):
        compliment = target - n
        if compliment in seen
          return [seen[compliment], i]
        
        seen[n] = i
```

## Question 4

### Group Anagrams
Given an array of strings strs, group all anagrams together into sublists. You may return the output in any order.

An anagram is a string that contains the exact same characters as another string, but the order of the characters can be different.

Example 1:

Input: strs = ["act","pots","tops","cat","stop","hat"]

Output: [["hat"],["act", "cat"],["stop", "pots", "tops"]]

### Answer
- Generate `fingerprint` and for each element of the strs.
- Use `fingerprint` as index of the hashmap and push to that hashmap

```Javascript

class Solution {
    /**
     * @param {string[]} strs
     * @return {string[][]}
     */
    groupAnagrams(strs) {
        const res = {};
        for (let s of strs) {
            const key = this.getFingerprint(s)
            if (!res[key]) {
                res[key] = [];
            }
            res[key].push(s);
        }
        return Object.values(res);
    }

    getFingerprint(s) {
      const count = new Array(26).fill(0);
      for (let c of s) {
          count[c.charCodeAt(0) - 'a'.charCodeAt(0)] += 1;
      }

      return count.join(',');
    }
}
```

## Question 5

### Top K Frequent Elements

Given an integer array nums and an integer k, return the k most frequent elements within the array.

The test cases are generated such that the answer is always unique.

You may return the output in any order.

Example 1:

Input: nums = [1,2,2,3,3,3], k = 2

Output: [2,3]

### Answer
- Use a hashmap to store frequency of each number.
- Loop through the generated hashmap and add all the frequency >= k push element to result.

```Python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
      freq = {}

      for i, n in enumerate(nums):
        freq[n] = freq.get(n, 0) + 1
      
      res = []
      for n, f in freq.items():
        if f >= k:
          res.append(n)

      return res
```

## Question 6

### Encode and Decode Strings
Design an algorithm to encode a list of strings to a string. The encoded string is then sent over the network and is decoded back to the original list of strings.

Machine 1 (sender) has the function:

string encode(vector<string> strs) {
    // ... your code
    return encoded_string;
}
Machine 2 (receiver) has the function:

vector<string> decode(string s) {
    //... your code
    return strs;
}
So Machine 1 does:

string encoded_string = encode(strs);
and Machine 2 does:

vector<string> strs2 = decode(encoded_string);
strs2 in Machine 2 should be the same as strs in Machine 1.

Implement the encode and decode methods.

### Answer
- Encode each string by `length#string` and append each in encode
- Decode by extracting length by `length = int(s[i:j])` and then extracting actual string by `s[i:i+length]`

```Python
class Solution:

    def encode(self, strs: List[str]) -> str:
        res = ""
        for s in strs:
            res += str(len(s)) + "#" + s
        return res

    def decode(self, s: str) -> List[str]:
        res = []
        i = 0

        while i < len(s):
            j = i
            while s[j] != '#':
                j += 1
            length = int(s[i:j])
            i = j + 1
            j = i + length
            res.append(s[i:j])
            i = j

        return res
```


## Question 7

### 