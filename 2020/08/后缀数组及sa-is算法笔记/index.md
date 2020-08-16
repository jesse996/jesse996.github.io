# 后缀数组及SA-IS算法笔记


## 定义

### 字符串

字符串`s`连续的一段字符组成的串叫做字符串，更广义地，任何一个由可比较大小的元素组成的数组都可称为字符串。字符串的下标从`0`开始，长度为`length(s)`

### 后缀

`suffix(i)`表示字符串从`s`第`i`个位置开始的后缀，即由`s[i]~s[n-1]`组成的子串

### 后缀数组

`sa[]`是一个一维数组，保存了字符串`s`的所有后缀**排序后**的结果

`rank[]`是一个一维数组，保存了每个后缀在`sa[]`中的排名，`rank[i]`表示`suffix(i)`的排名，即`rank[sa[i]]=i`

`height[]`是一个一维数组，保存了相邻两个后缀的最长公共前缀（Longest Common Prefix,lcp）的长度

## 一个简单的实现

```java
public String longestDupSubstring(String S) {
    int n=S.length();
    String[] sa=new String[n];
    for (int i=0;i<n;i++){
        sa[i]=S.substring(i);
    }
    Arrays.sort(sa);
    String max="";
    for (int i=1;i<n;i++){
        int len=lcp(sa[i-1],sa[i]);
        if (len>max.length()) max=sa[i].substring(0,len);
    }
    return max;
}
static int lcp(String a,String b){
    int n =Math.min(a.length(),b.length());
    for (int i=0;i<n;i++){
        if (a.charAt(i)!=b.charAt(i)) return i;
    }
    return n;
}
```

这个对后缀数组的排序是用的标准库的排序方法，时间复杂度是`O(nlogn)`，而 SA-IS 的时间复杂度是`O(n)`

## SA-IS 背景知识

在进入 SA-IS 算法的细节之前，我们需要先介绍一下 SA-IS 所建立的概念。

### S-type and L-type suffixes

当排序一些字符串的后缀数组时，SA-IS 把他们分为“S 型”和“L 型”两种。S 型后缀是指那些比他们右边的后缀更小的那些后缀（所以在最终的排好序的后缀数组中靠前）。L 型就相反。

比如`cabbage`，offset 为 1 的后缀是`abbage`，它右边的后缀是`bbage`，可见`abbage`是比`bbage`小的，所以`abbage`是一个 S 型后缀。同理，`ge`是一个 L 型后缀（g>e）。

规定最右边是一个空的（用`#`来表示），且它是 S 型。那么倒数第二个后缀，也就是不为空的最后一个字母是 L 型

我们可以写一个函数，接受一个字符串，并按照上面的规则映射出源字符串的每个字符是 S 型还是 L 型。

```python
>>> def buildTypeMap(data):
...     """
...     Builds a map marking each suffix of the data as S_TYPE or L_TYPE.
...     """
...     # The map should contain one more entry than there are characters
...     # in the string, because we also need to store the type of the
...     # empty suffix between the last character and the end of the
...     # string.
...     res = bytearray(len(data) + 1)
...
...     # The empty suffix after the last character is S_TYPE
...     res[-1] = S_TYPE
...
...     # If this is an empty string...
...     if not len(data):
...         # ...there are no more characters, so we're done.
...         return res
...
...     # The suffix containing only the last character must necessarily
...     # be larger than the empty suffix.
...     res[-2] = L_TYPE
...
...     # Step through the rest of the string from right to left.
...     for i in range(len(data)-2, -1, -1):
...         if data[i] > data[i+1]:
...             res[i] = L_TYPE
...         elif data[i] == data[i+1] and res[i+1] == L_TYPE:
...             res[i] = L_TYPE
...         else:
...             res[i] = S_TYPE
...
...     return res

>>> def showTypeMap(data):
...     print(data.decode('ascii'))
...     print(buildTypeMap(data).decode('ascii'))

>>> showTypeMap(b'cabbage')
cabbage
LSLLSLLS
```

总结如下：

-   s[n]=="#",type[n]=S_TYPE,type[n-1]=L_TYPE
-   if s[i]==s[i+1] type[i]=type[i+1]
-   if s[i]>s[i+1] type[i]=L_TYPE
-   if s[i]<s[i+1] type[i]=S_TYPE

比如：

```
s:    "cabbage"
type: "LSLLSLLS"
```

记住最后的 S 型是一个空。

### LMS 字符

就是 Left Most S 型，一串连续的 S 型中最左边的 S 型，它的左边是一个 L 型，这样的字符就是 LMS 字符

```
cabbage
LSLLSLLS
 ^  ^  ^      ^指向的就是LMS字符
```

```python
>>> def isLMSChar(offset, typemap):
...     """
...     Returns true if the character at offset is a left-most S-type.
...     """
...     if offset == 0:
...         return False
...     if typemap[offset] == S_TYPE and typemap[offset - 1] == L_TYPE:
...         return True
...
...     return False

>>> def showTypeMap(data):
...     typemap = buildTypeMap(data)
...
...     print(data.decode('ascii'))
...     print(typemap.decode('ascii'))
...
...     print("".join(
...             "^" if isLMSChar(i, typemap) else " "
...             for i in range(len(typemap))
...         ))

>>> showTypeMap(b'cabbage')
cabbage
LSLLSLLS
 ^  ^  ^
```

### LMS 子串

LMS 子串就是从一个 LMS 字符开始到下一个 LMS 字符（不包括下一个 LMS 字符）的字符串，上面的`cabbage`的 LMS 子串就是`abb`和`age`。 SA-IS 算法的神奇之处在于对 LMS 子串进行排序，但我们不能使用普通的字符串比较函数，因为我们不一定知道每个 LMS 字符串有多长。我们需要沿着字符串走，才能找到下一个 LMS 字符的开头，我们反正要遍历字符串，不如同时进行比较。

```python
>>> def lmsSubstringsAreEqual(string, typemap, offsetA, offsetB):
...     """
...     Return True if LMS substrings at offsetA and offsetB are equal.
...     """
...     # No other substring is equal to the empty suffix.
...     if offsetA == len(string) or offsetB == len(string):
...         return False
...
...     i = 0
...     while True:
...         aIsLMS = isLMSChar(i + offsetA, typemap)
...         bIsLMS = isLMSChar(i + offsetB, typemap)
...
...         # If we've found the start of the next LMS substrings...
...         if (i > 0 and aIsLMS and bIsLMS):
...             # ...then we made it all the way through our original LMS
...             # substrings without finding a difference, so we can go
...             # home now.
...             return True
...
...         if aIsLMS != bIsLMS:
...             # We found the end of one LMS substring before we reached
...             # the end of the other.
...             return False
...
...         if string[i + offsetA] != string[i + offsetB]:
...             # We found a character difference, we're done.
...             return False
...
...         i += 1

```

### 桶排序

因为我要要对后缀数组排序，所以所有以相同字符开头的后缀都会挨着。比如`cabbage`，有 2 个 a 开头的后缀，2 个 b 开头的后缀，1 个 c 开头的后缀，所以我们可以预测排好序的数组前两个是 a 开头的后缀，然后是 2 个 b 开头的，再是 1 个 c 开头的。

```python
>>> def findBucketSizes(string, alphabetSize=256):
...     res = [0] * alphabetSize
...
...     for char in string:
...         res[char] += 1
...
...     return res

    #a = 0, b = 1, c = 2...
>>> encoded_cabbage = [2, 0, 1, 1, 0, 6, 4]#cabbage
>>> findBucketSizes(encoded_cabbage, 7)
[2, 2, 1, 0, 1, 0, 1]
```

现在我们知道了每个 bucket 有多大，就可以简单地推导出一个数组，其中每个字符的索引都指向后缀数组中相应 bucket 的开头。

```python
>>> def findBucketHeads(bucketSizes):
...     offset = 1
...     res = []
...     for size in bucketSizes:
...         res.append(offset)
...         offset += size
...
...     return res

>>> encoded_cabbage = [2, 0, 1, 1, 0, 6, 4]
>>> findBucketSizes(encoded_cabbage, 7)
[2, 2, 1, 0, 1, 0, 1]

>>> cabbage_buckets = findBucketSizes(encoded_cabbage, 7)
>>> findBucketHeads(cabbage_buckets)
[1, 3, 5, 6, 6, 7, 7]
```

最后的空后缀总是在排好的后缀数组的第 0 位，所以 bucket 从 1 开始。有 2 个 a 开头的后缀所以 b 桶从 3 开始，以此类推。注意 d 桶和 e 桶都从 6 开始，但因为没有 d 开头的后缀，所以不会有问题。  
同门可以用相似的方法找到桶尾。

```python
>>> def findBucketTails(bucketSizes):
...     offset = 1
...     res = []
...     for size in bucketSizes:
...         offset += size
...         res.append(offset - 1)
...
...     return res

>>> findBucketTails(cabbage_buckets)
[2, 4, 5, 5, 6, 6, 7]
```

现在，讲完了所有的背景知识，我们来谈谈算法本身。

## SA-IS 算法

名字由来是 `Suffix Array by Induced Sorting`。什么是
`Induced Sorting`？建立后缀数组的棘手部分是 LMS 后缀。

先宏观看一下 SA-IS 算法

```python
>>> def makeSuffixArrayByInducedSorting(string, alphabetSize):
...     """
...     Compute the suffix array of 'string' with the SA-IS algorithm.
...     """
...
...     # Classify each character of the string as S_TYPE or L_TYPE
...     typemap = buildTypeMap(string)
...
...     # We'll be slotting suffixes into buckets according to what
...     # character they start with, so let's precompute that info now.
...     bucketSizes = findBucketSizes(string, alphabetSize)
...
...     # Use a simple bucket-sort to insert all the LMS suffixes into
...     # approximately the right place the suffix array.
...     guessedSuffixArray = guessLMSSort(string, bucketSizes, typemap)
...
...     # Slot all the other suffixes into guessedSuffixArray, by using
...     # induced sorting. This may move the LMS suffixes around.
...     induceSortL(string, guessedSuffixArray, bucketSizes, typemap)
...     induceSortS(string, guessedSuffixArray, bucketSizes, typemap)
...
...     # Create a new string that summarises the relative order of LMS
...     # suffixes in the guessed suffix array.
...     summaryString, summaryAlphabetSize, summarySuffixOffsets = \
...         summariseSuffixArray(string, guessedSuffixArray, typemap)
...
...     # Make a sorted suffix array of the summary string.
...     summarySuffixArray = makeSummarySuffixArray(
...         summaryString,
...         summaryAlphabetSize,
...     )
...
...     # Using the suffix array of the summary string, determine exactly
...     # where the LMS suffixes should go in our final array.
...     result = accurateLMSSort(string, bucketSizes, typemap,
...             summarySuffixArray, summarySuffixOffsets)
...
...     # ...and once again, slot all the other suffixes into place with
...     # induced sorting.
...     induceSortL(string, result, bucketSizes, typemap)
...     induceSortS(string, result, bucketSizes, typemap)
...
...     return result
```

为了帮助说明算法的运行情况，我们将使用以下函数来显示正在进行中的后缀数组的状态。我们将在未初始化的后缀数组元素中存储-1，由于我们将在短字符串上演示该算法，我们可以假设每个偏移量将是一到两位数的长度（与"-1 "的长度相同），并像这样呈现后缀数组的中间状态。

```python
>>> def showSuffixArray(arr, pos=None):
...     print(" ".join("%02d" % each for each in arr))
...
...     if pos is not None:
...         print(" ".join(
...                 "^^" if each == pos else "  "
...                 for each in range(len(arr))
...         ))

>>> showSuffixArray([2, -1, 4])
02 -1 04

>>> showSuffixArray([2, -1, 4], 2)
02 -1 04
      ^^
```

### 第一个猜测

我们还不知道我们的 LMS 后缀在 sfffix 数组中的确切位置，所以我们先用桶排序把它们放在大约正确的位置。在其他条件相同的情况下，较长的后缀（字符串中较早出现的后缀）排在较短的后缀（较晚出现的后缀）之后，所以我们将从左到右遍历字符串，并将我们找到的每个 LMS 后缀堆放在其桶的尾部。

```python
>>> def guessLMSSort(string, bucketSizes, typemap):
...     """
...     Make a suffix array with LMS-substrings approximately right.
...     """
...     # Create a suffix array with room for a pointer to every suffix of
...     # the string, including the empty suffix at the end.
...     guessedSuffixArray = [-1] * (len(string) + 1)
...
...     bucketTails = findBucketTails(bucketSizes)
...
...     # Bucket-sort all the LMS suffixes into their appropriate bucket.
...     for i in range(len(string)):
...         if not isLMSChar(i, typemap):
...             # Not the start of an LMS suffix
...             continue
...
...         # Which bucket does this suffix go into?
...         bucketIndex = string[i]
...         # Add the start position at the tail of the bucket...
...         guessedSuffixArray[bucketTails[bucketIndex]] = i
...         # ...and move the tail pointer down.
...         bucketTails[bucketIndex] -= 1
...
...         # Show the current state of the array
...         showSuffixArray(guessedSuffixArray)
...
...     # The empty suffix is defined to be an LMS-substring, and we know
...     # it goes at the front.
...     guessedSuffixArray[0] = len(string)
...
...     showSuffixArray(guessedSuffixArray)
...
...     return guessedSuffixArray
```

因此，现在我们可以猜测 LMS 子串在我们的字符串中的位置，将所有其他位置设置为-1。

```python
>>> cabbage_buckets = findBucketSizes(b'cabbage')
>>> cabbage_types = buildTypeMap(b'cabbage')
>>> cabbage_guess = guessLMSSort(b'cabbage', cabbage_buckets, cabbage_types)
-1 -1 01 -1 -1 -1 -1 -1
-1 04 01 -1 -1 -1 -1 -1
07 04 01 -1 -1 -1 -1 -1
```

-   它找到的第一个 LMS 后缀是 abbage，在字符串的索引 1 处，所以它把它放在 a 桶的末尾。
-   它找到的下一个 LMS 后缀是 index 4 的 age，所以它把它放在 abbage 之前
-   我们到了源字符串的结尾，所以我们在后缀数组的 0 位置添加了空后缀。

为了填补剩下的位置，我们将使用 "诱导排序"；也就是根据数组中已有的内容来确定其他后缀的位置。

### 诱导排序：L 型后缀

现在我们的后缀数组中已经有了 LMS 后缀，我们可以推断出所有其他后缀的去向。我们首先扫描我们的临时后缀数组，对于每一个列出的后缀，我们检查原始字符串中它左边的后缀--如果那是 L 型，我们也会对它进行桶式排序。

```python
>>> def induceSortL(string, guessedSuffixArray, bucketSizes, typemap):
...     """
...     Slot L-type suffixes into place.
...     """
...     bucketHeads = findBucketHeads(bucketSizes)
...
...     # For each cell in the suffix array....
...     for i in range(len(guessedSuffixArray)):
...         if guessedSuffixArray[i] == -1:
...             # No offset is recorded here.
...             continue
...
...         # We're interested in the suffix that begins to the left of
...         # the suffix this entry points at.
...         j = guessedSuffixArray[i] - 1
...         if j < 0:
...             # This entry in the suffix array is the suffix that begins
...             # at the start of the string, offset 0. Therefore there is
...             # no suffix to the left of it, and j is out of bounds of
...             # the typemap.
...             continue
...         if typemap[j] != L_TYPE:
...             # We're only interested in L-type suffixes right now.
...             continue
...
...         # Which bucket does this suffix go into?
...         bucketIndex = string[j]
...         # Add the start position at the head of the bucket...
...         guessedSuffixArray[bucketHeads[bucketIndex]] = j
...         # ...and move the head pointer up.
...         bucketHeads[bucketIndex] += 1
...
...         showSuffixArray(guessedSuffixArray, i)
```

如果我们将之前的猜测反馈给这个函数，就可以看着它将 L 型后缀传播到数组中。

```python
>>> induceSortL(b'cabbage', cabbage_guess, cabbage_buckets, cabbage_types)
07 04 01 -1 -1 -1 06 -1
^^
07 04 01 03 -1 -1 06 -1
   ^^
07 04 01 03 -1 00 06 -1
      ^^
07 04 01 03 02 00 06 -1
         ^^
07 04 01 03 02 00 06 05
                  ^^
```

根据上面的结果，我们可以知道发生了什么

-   我们从后缀数组中的第一个开始，它代表字符串末尾的空后缀。
-   空后缀前的字符为 e,偏移量是 6，所以我们将 6 放入 e 桶中。
-   下一个单元格的后缀 age 在偏移量为 4，它左边的后缀是偏移量为 3 的 L 型后缀 bage，所以我们将 3 放入 b 桶中。
-   接下来是偏移量 1 的 abbage，左边是偏移量 为 0 的 L 型后缀 cabbage，所以我们把 0 槽入 c 桶中
-   我们在偏移量 3 处找到了两步前存储的 bage--但偏移量 2 处的后缀(bbage)仍然是 L 型的，所以我们将它放入 b 桶中。
-   我们在偏移量 2 处找到了 bbage，但它左边的后缀不是 L 型，所以我们继续前进。
-   我们在偏移量 0 处发现了 cabbage，但它的左边没有任何东西，所以我们继续前进。
-   我们在偏移量 6 处发现 e，而偏移量 5 处的后缀(ge)是 L 型的，所以我们将其放入 g 桶中
-   我们在第 5 关找到了 ge，但它左边的后缀不是 L 型，所以我们结束了

### 诱导排序：S 型后缀

在 L 型后缀从左到右扫描后缀数组后，我们现在从右到左扫描 S 型后缀。这基本上是前面函数的镜像。

```python
>>> def induceSortS(string, guessedSuffixArray, bucketSizes, typemap):
...     """
...     Slot S-type suffixes into place.
...     """
...     bucketTails = findBucketTails(bucketSizes)
...
...     for i in range(len(guessedSuffixArray)-1, -1, -1):
...         j = guessedSuffixArray[i] - 1
...         if j < 0:
...             # This entry in the suffix array is the suffix that begins
...             # at the start of the string, offset 0. Therefore there is
...             # no suffix to the left of it, and j is out of bounds of
...             # the typemap.
...             continue
...         if typemap[j] != S_TYPE:
...             # We're only interested in S-type suffixes right now.
...             continue
...
...         # Which bucket does this suffix go into?
...         bucketIndex = string[j]
...         # Add the start position at the tail of the bucket...
...         guessedSuffixArray[bucketTails[bucketIndex]] = j
...         # ...and move the tail pointer down.
...         bucketTails[bucketIndex] -= 1
...
...         showSuffixArray(guessedSuffixArray, i)

>>> induceSortS(b'cabbage', cabbage_guess, cabbage_buckets, cabbage_types)
07 04 04 03 02 00 06 05
                     ^^
07 01 04 03 02 00 06 05
            ^^
```

cabbage 只有两个 S 型后缀，所以我们只能得到两行输出。当函数扫过后缀数组时，它最终将 LMS 后缀的顺序从 "04 01 "换成了 "01 04"，这足以将我们猜测的后缀数组转化为正确的后缀数组。不过这只是一个快乐的巧合。如果我们处理我们的另一个例子。

```python
>>> baa = b'baabaabac'
>>> showTypeMap(baa)
baabaabac
LSSLSSLSLS
 ^  ^  ^ ^

 >>> induceSortS(baa, baa_guess, baa_buckets, baa_types)
09 -1 -1 07 04 07 06 03 00 08
                           ^^
09 -1 -1 07 02 07 06 03 00 08
                     ^^
09 -1 -1 05 02 07 06 03 00 08
                  ^^
09 -1 01 05 02 07 06 03 00 08
            ^^
09 04 01 05 02 07 06 03 00 08
         ^^
```

还有一些单元格需要换一换。

### 总结猜测的后缀数组

我们从某个字母表中的一串字符开始，我们为这串字符做了一个近似的后缀数组，现在我们将用一个新的字母表中的新字符串来总结这个近似的后缀数组，它只代表了最重要的信息：在诱导 SortL()和诱导 SortS()做了它们的事情之后，LMS 后缀的最终位置。

总结的工作原理是这样的。原始字符串中的每个 LMS 后缀都有一个根据这些后缀在猜测的后缀数组中出现的顺序得到的名字。或者说，每个 LMS 后缀开头的 LMS 子串都会得到一个名字：如果两个 LMS 后缀以相同的 LMS 子串开头，它们就会得到相同的名字。这些名称按照与原始字符串中对应后缀相同的顺序组合，形成摘要字符串。

下面是相应的实现。

```python
>>> def summariseSuffixArray(string, guessedSuffixArray, typemap):
...     """
...     Construct a 'summary string' of the positions of LMS-substrings.
...     """
...     # We will use this array to store the names of LMS substrings in
...     # the positions they appear in the original string.
...     lmsNames = [-1] * (len(string) + 1)
...
...     # Keep track of what names we've allocated.
...     currentName = 0
...
...     # Where in the original string was the last LMS suffix we checked?
...     lastLMSSuffixOffset = None
...
...     # We know that the first LMS-substring we'll see will always be
...     # the one representing the empty suffix, and it will always be at
...     # position 0 of suffixOffset.
...     lmsNames[guessedSuffixArray[0]] = currentName
...     lastLMSSuffixOffset = guessedSuffixArray[0]
...
...     showSuffixArray(lmsNames)
...
...     # For each suffix in the suffix array...
...     for i in range(1, len(guessedSuffixArray)):
...         # ...where does this suffix appear in the original string?
...         suffixOffset = guessedSuffixArray[i]
...
...         # We only care about LMS suffixes.
...         if not isLMSChar(suffixOffset, typemap):
...             continue
...
...         # If this LMS suffix starts with a different LMS substring
...         # from the last suffix we looked at....
...         if not lmsSubstringsAreEqual(string, typemap,
...                 lastLMSSuffixOffset, suffixOffset):
...             # ...then it gets a new name
...             currentName += 1
...
...         # Record the last LMS suffix we looked at.
...         lastLMSSuffixOffset = suffixOffset
...
...         # Store the name of this LMS suffix in lmsNames, in the same
...         # place this suffix occurs in the original string.
...         lmsNames[suffixOffset] = currentName
...         showSuffixArray(lmsNames)
...
...     # Now lmsNames contains all the characters of the suffix string in
...     # the correct order, but it also contains a lot of unused indexes
...     # we don't care about and which we want to remove. We also take
...     # this opportunity to build summarySuffixOffsets, which tells
...     # us which LMS-suffix each item in the summary string represents.
...     # This will be important later.
...     summarySuffixOffsets = []
...     summaryString = []
...     for index, name in enumerate(lmsNames):
...         if name == -1:
...             continue
...         summarySuffixOffsets.append(index)
...         summaryString.append(name)
...
...     # The alphabetically smallest character in the summary string
...     # is numbered zero, so the total number of characters in our
...     # alphabet is one larger than the largest numbered character.
...     summaryAlphabetSize = currentName + 1
...
...     return summaryString, summaryAlphabetSize, summarySuffixOffsets
```

上面代码很多，看一个例子

```python
>>> (
...     cabbage_summary,
...     cabbage_summary_alpha_size,
...     cabbage_summary_suffix_offsets,
... ) = summariseSuffixArray(b'cabbage', cabbage_guess, cabbage_types)
-1 -1 -1 -1 -1 -1 -1 00
-1 01 -1 -1 -1 -1 -1 00
-1 01 -1 -1 02 -1 -1 00

>>> showSuffixArray(cabbage_guess)
07 01 04 03 02 00 06 05
```

在这些后缀中，我们知道前三个是 LMS 后缀，依次是空后缀、abbage 和 age。因为这三个 LMS 后缀都是以不同的 LMS 子串开始的，所以它们得到了三个不同的名字（0，1 和 2）。这些名称被存储在 lmsNames 中，位于原始字符串中相应后缀出现的位置：分别是偏移量 7、1 和 4。

计算出 lmsNames 后，我们扔掉 lmsNames 的未使用的索引，生成 摘要字符串。

```python
>>> cabbage_summary
[1, 2, 0]
```

但这不是我们函数的唯一输出。我们还知道，摘要字符串是用三个不同的字符组成的字母表书写的:

```python
>>> cabbage_summary_alpha_size
3
```

而且我们有一个列表，存储了与摘要字符串中每个项目相关联的后缀偏移。

```python
>>> cabbage_summary_suffix_offsets
[1, 4, 7]
```

但如前所述，`cabbage`是一个很简单的字符串。让我们试试我们更复杂的例子，`baabaabac`:

```python
>>> (
...     baa_summary,
...     baa_summary_alpha_size,
...     baa_summary_suffix_offsets,
... ) = summariseSuffixArray(baa, baa_guess, baa_types)
-1 -1 -1 -1 -1 -1 -1 -1 -1 00
-1 -1 -1 -1 01 -1 -1 -1 -1 00
-1 01 -1 -1 01 -1 -1 -1 -1 00
-1 01 -1 -1 01 -1 -1 02 -1 00
```

对于这个字符串，有两个 LMS 后缀是以相同的 LMS 子串开始的：aabaabac 和 aabac 都是以 aab 开始的，这些字符的类型在这两种情况下都是 SSL。因此，这两个后缀得到了相同的名称（1），我们的摘要字符串包含两个相同的字符:

```python
>>> baa_summary
[1, 1, 2, 0]
```

虽然这个摘要字符串包含 4 个字符，但它仍然只包含 3 个不同的字符，所以字母大小仍然是 3:

```python
>>> baa_summary_alpha_size
3
```

而且我们又可以得到摘要字符串的后缀偏移 map：

```python
>>> baa_summary_suffix_offsets
[1, 4, 7, 9]
```

### 得到总结的后缀数组

我们从一个字符串开始，想建立一个后缀数组，在这个过程中，我们已经建立了第二个字符串，现在我们也要把这个字符串做一个后缀数组？这不是循环推理吗？

嗯，差不多：这是递归推理。

如果我们要做递归，就需要有一个不是递归的基例。如果我们看一下我们上面的`cabbage`例子，我们得到了一个很简单的摘要字符串：

```python
>>> cabbage_summary
[1, 2, 0]
```

是的，这个特殊的例子恰好小到你可以在脑海中建立后缀数组，但还有一个有趣的特点：我们已经说过这是一个由三个字符组成的字母表中的字符串，而字母表中的每个字符都会在这个字符串中的某个地方精确地出现一次。这意味着我们可以用我们可靠的老桶排序来创建一个后缀数组。

如果反过来看我们 `baabaabac` 的例子，前景就没有那么乐观了。

```python
>>> baa_summary
[1, 1, 2, 0]
```

这个字符串仍然是三个字符的字母表，但它的长度超过了三个字符，这意味着至少有一个字符必须重复，而我们不能确定一个桶排序会做正确的事情--所以在这种情况下，我们必须递归。

值得指出的是，虽然理论上你可以在字母表中拥有一个包含重复的三个字符的字符串（比如说，[1，1，0]），但我们的 `summaryiseSuffixArray()`函数永远不会产生这样的东西，因为它总是使用字母表中的连续字符，而且它总是将字母表的大小设置为它所使用的不同字符数。

```python
>>> def makeSummarySuffixArray(summaryString, summaryAlphabetSize):
...     """
...     Construct a sorted suffix array of the summary string.
...     """
...     if summaryAlphabetSize == len(summaryString):
...         # Every character of this summary string appears once and only
...         # once, so we can make the suffix array with a bucket sort.
...         summarySuffixArray = [-1] * (len(summaryString) + 1)
...
...         # Always include the empty suffix at the beginning.
...         summarySuffixArray[0] = len(summaryString)
...
...         for x in range(len(summaryString)):
...             y = summaryString[x]
...             summarySuffixArray[y+1] = x
...
...     else:
...         # This summary string is a little more complex, so we'll have
...         # to use recursion.
...         summarySuffixArray = makeSuffixArrayByInducedSorting(
...             summaryString,
...             summaryAlphabetSize,
...         )
...
...     return summarySuffixArray
```

我们还不能在`baabaabac`摘要上测试这个函数，因为我们还没有完成递归的实现，但我们可以在`cabbage`摘要上测试它。

```python
>>> cabbage_summary_suffix_array = makeSummarySuffixArray(
...     cabbage_summary, cabbage_summary_alpha_size,
... )
>>> showSuffixArray(cabbage_summary_suffix_array)
03 02 00 01
```

和用内置排序算法比较:

```python
>>> showSuffixArray(naivelyMakeSuffixArray(cabbage_summary))
03 02 00 01
```

### 建立真正的后缀数组

最后，我们开始根据摘要字符串的后缀数组构建真正的、最终的后缀数组。和之前一样，我们先将 LMS 后缀放入后缀数组中，然后我们可以通过诱导排序来填充所有其他的后缀。与之前不同的是，我们不只是按照找到的任何顺序将它们桶式排序到位，而是按照摘要字符串的后缀数组决定的顺序插入。

请记住，当我们建立摘要字符串时（其中每个字符都是一个 LMS 子串的生成名称），我们还建立了一个相应的摘要 SuffixOffsets 数组，该数组将摘要字符串中的字符映射回原始字符串中的 LMS 子串（因此是 LMS 后缀），所以这就是我们要使用的。

这个过程是这样的。摘要字符串的后缀数组中的每个条目都指向摘要中的一个位置。我们在 summarySuffixOffsets 中查看相同的位置，以找到我们原始字符串的相应 LMS 后缀的偏移量。然后，我们将该 LMS 后缀放入正确的桶中，相信 summary 后缀数组会将它们按正确的顺序放入桶中。

```python
>>> def accurateLMSSort(string, bucketSizes, typemap,
...         summarySuffixArray, summarySuffixOffsets):
...     """
...     Make a suffix array with LMS suffixes exactly right.
...     """
...     # A suffix for every character, plus the empty suffix.
...     suffixOffsets = [-1] * (len(string) + 1)
...
...     # As before, we'll be adding suffixes to the ends of their
...     # respective buckets, so to keep them in the right order we'll
...     # have to iterate through summarySuffixArray in reverse order.
...     bucketTails = findBucketTails(bucketSizes)
...     for i in range(len(summarySuffixArray)-1, 1, -1):
...         stringIndex = summarySuffixOffsets[summarySuffixArray[i]]
...
...         # Which bucket does this suffix go into?
...         bucketIndex = string[stringIndex]
...         # Add the suffix at the tail of the bucket...
...         suffixOffsets[bucketTails[bucketIndex]] = stringIndex
...         # ...and move the tail pointer down.
...         bucketTails[bucketIndex] -= 1
...
...         showSuffixArray(suffixOffsets)
...
...     # Always include the empty suffix at the beginning.
...     suffixOffsets[0] = len(string)
...
...     showSuffixArray(suffixOffsets)
...
...     return suffixOffsets
```

注意：为了确保我们的 LMS 后缀能在后面的诱导排序中存活下来，我们需要把它们插入到它们的桶的最后，所以我们在向后插入后缀的时候，需要向后走过 summarySuffixArray，以保持它们的相对顺序。

另外，我们并不是对 summarySuffixArray 中的每个条目都进行处理：第一个条目对应的是 summary 字符串的空后缀，它不对应原始字符串的任何后缀。第二个条目对应的是原始字符串的空后缀，我们无法对它进行桶式排序，因为它是空的。不过我们已经知道它在开头，所以这不是问题。

让我们来看看这个算法，因为它适用于我们的老朋友`cabbage`。提醒一下，这里是 accounterLMSSort()的输入。

```python
>>> showSuffixArray(cabbage_summary_suffix_array)
03 02 00 01
>>> showSuffixArray(cabbage_summary)
01 02 00
>>> showSuffixArray(cabbage_summary_suffix_offsets)
01 04 07
```

现在，我们可以走一遍算法。

-   我们在摘要的后缀数组中找到的第一个条目（最后一个条目，因为我们是从后往前走）是 01。如果我们查看 summarySuffixOffsets 的偏移量 1，我们看到这对应于原始字符串第 4 位（age）的 LMS 后缀，所以我们可以在 a 桶的最后插入该后缀。
-   summary 的后缀数组的下一个条目是 00，summarySuffixOffsets 的偏移量 0 对应于原始字符串第 1 个位置的 LMS 后缀（abbage），所以我们把这个后缀插到 age 之前的 a 桶中。
-   倒数第二的条目被跳过，因为它指向原始字符串的空后缀，而我们无法将其进行桶排序。
-   最后一条被跳过，因为它代表摘要字符串末尾的空后缀，它在原始字符串中没有对应的 LMS 后缀。
-   最后，我们知道，空的后缀（在第 7 位）将出现在最后的后缀数组的开头。

当我们运行我们的函数时，我们可以看到同样的步骤发生。

```python
>>> cabbage_real = accurateLMSSort(b'cabbage', cabbage_buckets,
...     cabbage_types, cabbage_summary_suffix_array,
...     cabbage_summary_suffix_offsets)
-1 -1 04 -1 -1 -1 -1 -1
-1 01 04 -1 -1 -1 -1 -1
07 01 04 -1 -1 -1 -1 -1
```

04 是在 a 桶的末尾，然后前面紧接着是 01，最后是空后缀在开头得到它的特殊位置。

### 整合在一起

在将 LMS 后缀放到正确的位置后，所有其他后缀都会像之前一样，通过第二轮诱导排序归位。我们现在已经描述了 makeSuffixArrayByInducedSorting()调用的所有函数，所以我们可以调用它来有效地建立一个我们喜欢的任何字符串的后缀数组......以及其内部计算的详细日志（为简洁起见这里省略）。

```python
>>> showSuffixArray(makeSuffixArrayByInducedSorting(b'cabbage', 256))
-1 -1 01 -1 -1 -1 -1 -1
...
07 01 04 03 02 00 06 05
>>> showSuffixArray(makeSuffixArrayByInducedSorting(baa, 256))
-1 -1 -1 -1 -1 01 -1 -1 -1 -1
...
09 01 04 02 05 07 00 03 06 08
```

但是，由于大多数人想要建立一个排序后缀数组的人都会有一串字节，而不是任意大小的字母串，所以让我们做一个具有合理默认值的包装函数。

```python
>>> def makeSuffixArray(bytestring):
...     return makeSuffixArrayByInducedSorting(bytestring, 256)
```

现在我们完成了。

```python
>>> naivelyMakeSuffixArray(b'cabbage') == makeSuffixArray(b'cabbage')
True
>>> naivelyMakeSuffixArray(baa) == makeSuffixArray(baa)
True
```

