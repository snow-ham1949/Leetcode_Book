---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 題目選解

#### [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

這題要求檢查給定的括號字符串是否有效。有效的括號必須以正確的順序閉合，例如 "()" 和 "()\[]{}" 是有效的，但 "(]" 和 "(\[)]" 則不是。因為最後開啟的括號必須首先閉合，所以這是一個典型的後進先出（LIFO）結構。

```cpp
class Solution {
public:
    bool isValid(std::string s) {
        stack<char> stack;
        for (char c : s) {
            switch (c) {
                case '(': 
                case '{': 
                case '[': stack.push(c); break;
                case ')': if (stack.empty() || stack.top() != '(') return false; else stack.pop(); break;
                case '}': if (stack.empty() || stack.top() != '{') return false; else stack.pop(); break;
                case ']': if (stack.empty() || stack.top() != '[') return false; else stack.pop(); break;
                default: ;
            }
        }
        return stack.empty();
    }
};
```

#### [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/)

遇到運算子時，我們需要對最近的兩個操作數進行計算。所以可以利用 stack 來解決。

遍歷每個元素：

* 如果是操作數，將其推入堆疊。
* 如果是運算子，從堆疊中彈出兩個操作數，執行相應的運算，然後將結果推回堆疊。

最後，堆疊頂部的元素就是表達式的結果。

```cpp
class Solution {
public:
    int evalRPN(std::vector<std::string>& tokens) {
        stack<int> stack;
        for (string token : tokens) {
            if (token == "+" || token == "-" || token == "*" || token == "/") {
                int op2 = stack.top();
                stack.pop();
                int op1 = stack.top();
                stack.pop();
                if (token == "+") stack.push(op1 + op2);
                if (token == "-") stack.push(op1 - op2);
                if (token == "*") stack.push(op1 * op2);
                if (token == "/") stack.push(op1 / op2);
            } else {
                stack.push(stoi(token));
            }
        }
        return stack.top();
    }
};
```

#### [Decode String](https://leetcode.com/problems/decode-string/)

由於解碼的過程涉及到處理嵌套的括號結構，stack 就很適合處理。

遍歷每個字元：

* 如果是數字，則取出完整的數字。
* 如果是字母或 \[，則將其推入堆疊。
* 如果是 ] ，則從堆疊中彈出字符串，直到遇到 \[ ，並記錄此字符串；接著繼續彈出堆疊頂部的數字，並將字串重複相應的次數，最後將結果字串再次推回堆疊。

最後，將堆疊中的所有字串拼接起來，得到最終結果。

```cpp
class Solution {
public:
    string decodeString(string s) {
        stack<string> substring;
        stack<int> mul;
        string res = "";
        int m = 0;
        for (char c : s) {
            if (isdigit(c)) m = m * 10 + c - '0';
            else if (c == '[') {
                substring.push(res);
                mul.push(m);
                res = "";
                m = 0;
            } else if (c == ']') {
                int cnt = mul.top();
                mul.pop();
                string prv = substring.top();
                substring.pop();
                for (int i = 0; i < cnt; i++) {
                    prv += res;
                }
                res = prv;
            } else res += c;
        }

        return res;
    }
};
```

#### [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/)

當一個行星向右移動，另一個行星向左移動時，它們可能會相撞。我們可以使用堆疊來追踪這些行星的移動，當一次新的碰撞發生時，可以使用堆疊的頂部元素來判斷碰撞的結果。

遍歷每個行星：

* 如果堆疊為空或當前行星與堆疊頂部行星同方向移動，則將其推入堆疊。
* 如果當前行星向左移動而堆疊頂部行星向右移動，則根據它們的大小決定哪個行星被摧毀，或者是否都被摧毀。

最後，堆疊中剩餘的行星就是碰撞後生存下來的行星。

```cpp
class Solution {
public:
    vector<int> asteroidCollision(vector<int>& asteroids) {
        stack<int> stk;
        vector<int> res;
        for (int &x : asteroids) {
            if (stk.empty() || stk.top() < 0) stk.push(x);
            else {
                if (x > 0) stk.push(x);
                else {
                    bool ok = true;
                    while (!stk.empty() && stk.top() > 0) {
                        if (stk.top() + x < 0) {
                            stk.pop();
                        } else if (stk.top() + x > 0) {
                            ok = false;
                            break;
                        } else {
                            stk.pop();
                            ok = false;
                            break;
                        }
                    }
                    if (ok) stk.push(x);
                }
            }
        }
        while (!stk.empty()) {
            res.push_back(stk.top());
            stk.pop();
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```

每當遇到一個 '\n' 字符時，就知道目錄的名字已經結束，所以可以將目錄的長度推入 deque，它只會儲存正在處理中的檔案結構，所以如果 level 小於 deque 的大小（就是檔案結構的深度），就可以彈出這些值，它們就不再當前處理的檔案結構裡了。

最後，當遇到一個 '.' 字符時，我們知道這是一個文件，所以就可以通過將 deque 中所有目錄名的長度相加來取 max，我們將在最終結果中加上 level 作為 '/'，以及加上當前文件名的長度作為總長度。

```cpp
class Solution {
public:
    int lengthLongestPath(string input) {
        deque<int> dq;
        int len = 0, res = 0, level = 0;
        bool isFile = false;
        for(char c: input){
            if(c == '\n') {
                dq.push_back(len);
                level = len = file = 0;
            }
            else if (c == '\t') { 
                ++level;
            }
            else if (c == '.') { 
                ++len;
                isFile = true;
            }
            else {
                ++len;
                if(isFile) res = max(res, len + accumulate(dq.begin(), dq.end(), 0) + level);
                while(level < dq.size()){
                    dq.pop_back();
                }
            }
        }        
        return res;
    }
};
```

#### [Basic Calculator](https://leetcode.com/problems/basic-calculator/)

這題跟前面的 RPN 蠻像的，大概的解法就是

遍歷每個字元：

* 如果是數字，則解析完整的數字。
* 如果遇到加號或減號，將先前的數字和運算符推入堆疊。
* 如果遇到左括號，將先前的結果和運算符推入堆疊。
* 如果遇到右括號，則計算括號內的表達式值。

```cpp
class Solution {
public:
    int calculate(std::string s) {
        stack<int> nums, ops;
        int num = 0;
        int res = 0; // For the on-going result
        int sign = 1; // 1 means positive, -1 means negative  

        for (char c : s) {
            if (isdigit(c)) {
                num = num * 10 + (c - '0');
            } else if (c == '+' || c == '-') {
                res += sign * num;
                num = 0;
                sign = (c == '+') ? 1 : -1;
            } else if (c == '(') {
                nums.push(res);
                ops.push(sign);
                res = 0;
                sign = 1;
            } else if (c == ')') {
                res += sign * num;
                num = 0;
                res *= ops.top(); ops.pop();
                res += nums.top(); nums.pop();
            }
        }
        res += sign * num;
        return res;
    }

};
```
