[[_TOC_]]

Pinyin Grep
===========

**Github version README**

> repository link: https://cs-git-research.cs.surrey.sfu.ca/cameron/parabix-devel/-/tree/alpha
>
> `pinyingrep` is a team work project leaded by **Yanwen(Steven) Chen** in Foundation of Software Engineering in Spring 2020, ZJU, advised by Professor Robert Cameron. It is based on the experimental SIMD parallelism framework `Parabix` (https://cs-git-research.cs.surrey.sfu.ca/cameron/parabix-devel/-/blob/master/CMakeLists.txt).
>
> The iteration and version history can be found in the `README-pinyingrep.md` of the repo.
>
> It is obliged for me to acknowledge the work of my teammates and the patient instructions given by Prof. Cameron.



This is a grep tool, using Chinese pinyin to "grep" lines with corresponding Chinese characters/phases in a list of files.  
It can also support simple regular-expression-like features, like:
`pinyingrep <regex> <file list>`
See below for more examples.  

How to Use Pinyin Grep
------------------

### Compilation

at root directory,

```shell
$ mkdir pinyin-build 
$ cmake -DCMAKE_BUILD_TYPE=Release .. $
$ make -j4
```

it will compile the whole `Parabix` Projects, including the **pinyingrep**

if you do not want the programs in `Parabix` like `icgrep/wc`, you can comment out the `add subdirectory` of the corresponding programs in top `CMakeList.txt`

### Functionality
**note:** 
* For inputs that are multiple syllables long, the sequence must be put in quotations(" ").
* If empty input(including only empty spaces) is given to the pinyingrep, the *whole file* will be output as the result.

#### 1. Pinyin syllables without tones specified
**Pinyingrep** supports pinyin syllables in alphabetic characters without tones sepcified.
> e.g. `zhong` or `gao`
>
As an exception of alphabetic characters in pinyin representation, `ü` is also supported as non-toned characters. `v` and `ü` are considered equivalent as un-toned input.

With such a form of input syllables, pinyingrep will 'grep' lines with Chinese characters with
readings specified by the pinyin syllables in all five tones(Tone `1-4` as well as Qingsheng/Soft/Neutral)
For instance, the result of grepping `zhong` is equivalent to those of grepping `zhong0`(Qingsheng), `zhong1`, `zhong2`, `zhong3` and `zhong4`.
Using numeric tones together with Latin characters' tones, is not supported. In this case, the number `[0-4]` is with higher priority to be considered as tones, while the previous syllable are considered illegal since there is toned Latin characters embedded other than supported alphabetic characters. In this case, pinyingrep throw errors saying that using both ways of toning is not allowed.

##### Test Sample

* Input: "zhong yao"
* Output: "grep" the lines with "zhong yao", "重要", "中药" and other possible combination of Chinese characters, whose pinyin is "zhong yao".

```
    Test
    Input: zhong wen
    Output:
    欢迎来到中文世界。
    在这里你可以感受中文的博大精深。
```

#### 2. Pinyin syllables with tones specified by numbers
**Pinyingrep** supports pinyin syllables with tones specified by Arabic numbers(`0-4`).
> e.g. `zhong1` or `zho1ng`
>
We define that`0` indicates the reading is in `Qingsheng`.
##### Test Sample

* Input: "zhong1 yao4"
* Output: "grep" the lines with "中药" and other other possible combination of Chinese characters, whose pinyin is "zhong1 yao4".

```		
    Test
    Input: zhong1 yao4
    Output: 
    中药主要用于轻症患者。
    所以把筐子中要卖的药都烧了。
```



#### 3. Pinyin syllables with tones specified by toned Latin characters 
**Pinyingrep** supports pinyin syllables with tones specified by toned Latin characters
more specifically, latin characters like `ǎ` or `ō`.

> e.g. `xuǎn` or `xiōng`
>
Toned unicode syllables like `xuǎn` is equivalent to corresponding syllables with tones specified by Arabic numbers.
##### Test Sample

* Input: "zhòng yào"
* Output: "grep" the lines with "重要" and other possible combination of Chinese characters, whose pinyin is "zhòng yào".

```
    Test
    Input: zhong4 yao4
    Output: 
    重要的事情说三遍：
    种药的人觉得现在卖不出好价钱，
```

#### 4. Regular-expression-like pinyin syllables with `.`
**Pinyingrep** supports regular-expression-like pinyin syllables with `.`.
> e.g. `zh.ng` or `x..ng`
>
Similar to normal regular expressions, `.` represent arbitrary **alphabetic** characters.
But only legal pinyin syllables in the database will be considered by **pinyingrep**. 
> e.g. `x..ng` to `xiong`, `xiang`, and etc.
##### Test Sample

* Input: “m.ng” .
* Equivalent to "mang|meng|ming|mong".

```
    Test
    Input: m.ng
    Output: 
    我名字叫小明,家里在上海。
    我的叔叔是盲的，今天我要忙着带他去看医生。
    明天我会去打篮球。
    我有个梦想做医生。
    看了我叔叔收到的歧视，我想帮盲人。
```



#### 5. Regular-expression-like pinyin syllables with `?` after tailing "g"
**Pinyingrep** supports pinyin syllables with `?` after alphabetic character `g`(only `g` after `n` is supported in this iteration).
> e.g. `zhang?`
>
The question mark indicates that both readings with `g` and without `g` are wanted as input.

##### Test Sample

* Input: “qing?” Equivalent to "qin|qing".
* "?" is only for the last "g". Equivalent to "qin|qing".

```
    Test
    Input: kang?
    Output: 
    看了我叔叔收到的歧视，我想帮盲人。
    虽然他们不能康复，但可以变更好。
```



#### 6. Sequences of pinyin syllables mentioned above
**Pinyingrep** supports sequences of the above legal syllables of arbitrary length.
> e.g. `zh.ng yao4` 
>
Any sequence of Chinese characters with readings matching the input sequences will be in result.
#### 7. Grep from a list of files
**Pinyingrep** supports one or more files as source files to grep from.



#### 8. Coloring with Command Line `-c`

* Input: with command line flag `-c`
* Coloring the matched Chinese characters or phrases in output
> Input: "zhong wen" ... -c
> 
> 欢迎来到<font color=Red>中文</font>世界。 \
> 在这里你可以感受<font color=Red>中文</font>的博大精深。
>
#### 9. Selecting different database
* The database supported currently is:
    * `XHC1983`: it is the data from the `kXHC1983` field of `Unihan_Reading.txt` in Unihan database
    * `HanyuPinyin`: it is the combination of data from the `kHanyuPinyin` field and the `kMandarin` field of `Unihan_Reading.txt` in Unihan database. `kMandarin` is only used when the codepoint has no `kHanyuPinyin` field, in case the pinyin syllable in `kMandarin` field is not included in `kHanyuPinyin` field like `U+9B0C`
    * Data differences:
        * for example, `明` has a rare reading `meng4` in `HanyuPinyin`
* the default database is `XHC1983`
    * if switching to `HanyuPinyin` is wanted, 
command line flag `-kpy` need to be in place.

#### 10. Traditional/Simplified Options
* **Pinyingrep** provides the user with options to specified the type of Chinese characters they want to grep:
    1. `-all`(Default): grep all Chinese characters, in traditional and simplified Chinese 
    2. `-trd`: grep Chinese characters used in traditional Chinese. Some of Chinese characters are used in both traditional and simplified Chinese, and this commandline option indicates that such characters are also wanted. 
    3. `-sim`: grep Chinese characters used in simplified Chinese. Some of Chinese characters are used in both traditional and simplified Chinese, and this commandline option indicates that such characters are also wanted. 
    4. `-tonly`: grep Chinese characters **only used** in traditional Chinese. This commandline option indicates that no Chinese characters that are used in simplified Chinese should be included.
    5. `-sonly`: grep Chinese characters **only used** in simplified Chinese. This commandline option indicates that no Chinese characters that were once used in traditional Chinese should be included.
* For example: `井` is used in both traditional and simplified Chinese; `書` is only used in traditional Chinese while `学` is only used in simplified Chinese.
* For more information about Traditional and Simplified Variant, please refer to <http://www.unicode.org/reports/tr38/> 

#### 11. Exact Match Mode
* **Pinyingrep** provides the user with a mode where **pinyingrep** also interpret the input string directly as a regular expression:
    * For example, `中` and `zhong` in the original text file will both be grepped with input string `zhong`.
* Command Line flag `-e` is used to turn on such a mode.

#### 12. More flexible Regular expression feature -- `?`
* Input: “xia?ng” Equivalent to "xing|xiang".
* "?" will automatically find the previous letter and skip the numbers.
* if the letter before `?` is a toned unicode character, the tone will not be changed even if this letter does not exist in some cases.
```
    Test 1: after `.`
    Input: xi.?ng
    Output: 
    心想事成
    兄友弟恭
    行云流水
```
```
    Test 2: after numbers
    Input: xi.3?ng
    Output: 
    心想事成
```
```
    Test 3: after toned unicode character
    Input: xiō?ng
    Output: 
    兄友弟恭
```
#### More: About Vague/Loose Matching
* With a more flexible regex feature `?`, Vague or Loose Matching is already available for users. Users can input `zh?i` if they are not sure whether it is `zhi` or `zi`. Such a feature gives the user more flexibility where they can specify the exact place(e.g. `h` after `z` or `g` after `n`) they are not sured about.

#### 13. Option for showing filename
* Command Line option `-h` is now supported by **pinyingrep**, which means showing the filename of *each* line grepped.
```
    Input:  
        "zhong" -h ../QA/pinyintest/testfiles/simple_pinyin 
        ../QA/pinyintest/testfiles/traditional-test  
        ../QA/pinyintest/testfiles/test2 
    Output: 
        ../QA/pinyintest/testfiles/simple_pinyin:欢迎来到中文世界。
        ../QA/pinyintest/testfiles/simple_pinyin:在这里你可以感受中文的博大精深。
            ......
        ../QA/pinyintest/testfiles/traditional-test:忧从中来，不可断绝。
        ../QA/pinyintest/testfiles/traditional-test:憂從中來，不可斷絕。
```

#### 14. Option for showing line numbers
* Command Line option `-n` is now supported, which means showing the line number in the *original* file of *each* grepped line.
```
    Input:
        "zhong" ../QA/pinyintest/testfiles/simple_pinyin 
        ../QA/pinyintest/testfiles/traditional-test 
        ../QA/pinyintest/testfiles/test2 
        ../QA/pinyintest/testfiles/exact-test -n
    Output:
        3:欢迎来到中文世界。
        4:在这里你可以感受中文的博大精深。
            ......
        11:忧从中来，不可断绝。
        28:憂從中來，不可斷絕。
        8:选择权很重要
        9:这是中文测试数据
```
#### 15. Case-insensitive input
* Uppercase and lowercase input are equivalent.
* Update: Upper Toned Characters Also Supported Now.
```
    Test
    Input: ZHŌng wen
    Output:
    欢迎来到中文世界。
    在这里你可以感受中文的博大精深。
```

#### 16. Warning Mode
* Command Line flag `-w` is used to turn on Warning Mode.
* When Warning Mode is on, users get warnings when their pinyin syllable input is not legitimate, and also when the interpreted toned syllable is not in the current database.
* The basic principle of such a warning mode is to give users more flexibility when using the software without making them bothered by unwanted warning/error messages.
```
    Input: "zhiŌng" -w 
    Output: [WARNING] zhiong matches no legitimate pinyin syllables.
```
```
    Input: "zh?ong" -w
    Output: [WARNING] zong0 matches no Chinese characters in the current database
            [WARNING] zong2 matches no Chinese characters in the current database
            [WARNING] zhong0 matches no Chinese characters in the current database
            [WARNING] zhong2 matches no Chinese characters in the current database
            欢迎来到中文世界。
            在这里你可以感受中文的博大精深。
            ......

```
