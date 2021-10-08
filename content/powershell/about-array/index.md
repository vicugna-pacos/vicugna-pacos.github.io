---
title: "配列"
date: 2021-08-19T16:13:13+09:00
---

[about Arrays - PowerShell | Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_arrays?view=powershell-5.1)

## はじめに
PowerShell の配列の各要素には、異なる型を格納できる。

## 初期化
格納する値と同時に配列を宣言する際は、値をコンマで区切ったものを変数に代入するだけでよい。

```powershell
$A = 22,5,10,8,12,9,80
```

要素が1つだけの配列を宣言したいときは、値の前にコンマを付ける。

```powershell
$B = ,7
```

さらに、範囲を表す演算子(`..`)を使って定義することも可能。下記サンプルでは、5,6,7,8 と4つの値が配列に格納される。

```powershell
$C = 5..8
```

もし何も型を指定しない場合、配列は Object 型の配列 (`System.Object[]`) となる。配列の型を調べるときは、`GetType()` メソッドを使う。

```powershell
$A.GetType()
```

要素の型を限定した配列 (string[], long[], int32[] など) を作りたい場合は、配列の変数を目的の型へキャストする。

```powershell
[int32[]]$ia = 1500,2230,3350,4000
```

配列の要素の型は、.NET でサポートされているものであればなんでもいい。例えば、`Get-Process` を実行すると、`System.Diagnostics.Process` 型の配列が戻り値として返される。
これを厳密に指定したいなら、下記のように書ける。

```PowerShell
[Diagnostics.Process[]]$zz = Get-Process
```

## 配列の部分式演算子

`@()` という演算子を使っても、配列を初期化できる。


```PowerShell
$a = @("Hello World")
$a.Count

# 出力
# 1
```

```PowerShell
$b = @()
$b.Count

# 出力
# 0
```

この演算子は、メソッド等の実行結果で配列が返ってくることが予想されるけど、その数が分からないときに有用。
例えば、`Get-Process` の結果が1件のみの場合、そのままだと戻り値の型は `Process` になってしまうが、`@()` で囲んでおけば必ず `Process[]` にさせられる。

```powershell
$p = @(Get-Process Notepad)
```

## 要素へのアクセス

```powershell
$a[0]
```

配列のインデックスは0から始まる。

範囲演算子 (`..`) を使うと、配列の一部のみを抽出できる。

```powershell
$a[1..4]
```

負の数字を指定すると、配列を後ろから数えた要素を取得できる。

```powershell
# 後ろから1番目～3番目の要素を取得

$a = 0..9
$a[-3..-1]

# 出力
# 7
# 8
# 9

# 順番を逆にすると、出力される配列の順番も変わる
$a[-1..-3]

# 出力
# 9
# 8
# 7

$a[2..-2]

# 出力
# 2
# 1
# 0
# 9
# 8
```

`+` を使うと、複数の数字の範囲を指定できる。

```PowerShell
$a = 0 .. 9
$a[0,2+4..6]

# 出力
# 0
# 2
# 4
# 5
# 6
```

## 配列を走査する

```PowerShell
# ForEach
$a = 0..9
foreach ($element in $a) {
  $element
}

# For
$a = 0..9
for ($i = 0; $i -le ($a.length - 1); $i += 2) {
  $a[$i]
}
```

## 配列のプロパティ

### Count, Length, LongLength

* Count と Length は同じ。Length の別名が Count 。
* LongLength は、配列の要素数が 2,147,483,647 を超えるときに使う。



Rank
Returns the number of dimensions in the array. Most arrays in PowerShell have one dimension, only. Even when you think you are building a multidimensional array like the following example:

PowerShell

Copy
$a = @(
  @(0,1),
  @("b", "c"),
  @(Get-Process)
)

"`$a rank: $($a.Rank)"
"`$a length: $($a.Length)"
"`$a[2] length: $($a[2].Length)"
"Process `$a[2][1]: $($a[2][1].ProcessName)"
In this example, you are creating a single-dimensional array that contains other arrays. This is also known as a jagged array. The Rank property proved that this is single-dimensional. To access items in a jagged array, the indexes must be in separate brackets ([]).

Output

Copy
$a rank: 1
$a length: 3
$a[2] length: 348
Process $a[2][1]: AcroRd32
Multidimensional arrays are stored in row-major order. The following example shows how to create a truly multidimensional array.

PowerShell

Copy
[string[,]]$rank2 = [string[,]]::New(3,2)
$rank2.rank
$rank2.Length
$rank2[0,0] = 'a'
$rank2[0,1] = 'b'
$rank2[1,0] = 'c'
$rank2[1,1] = 'd'
$rank2[2,0] = 'e'
$rank2[2,1] = 'f'
$rank2[1,1]
Output

Copy
2
6
d
To access items in a multidimensional array, separate the indexes using a comma (,) within a single set of brackets ([]).

Some operations on a multidimensional array, such as replication and concatenation, require that array to be flattened. Flattening turns the array into a 1-dimensional array of unconstrained type. The resulting array takes on all the elements in row-major order. Consider the following example:

PowerShell

Copy
$a = "red",$true
$b = (New-Object 'int[,]' 2,2)
$b[0,0] = 10
$b[0,1] = 20
$b[1,0] = 30
$b[1,1] = 40
$c = $a + $b
$a.GetType().Name
$b.GetType().Name
$c.GetType().Name
$c
The output shows that $c is a 1-dimensional array containing the items from $a and $b in row-major order.

Output

Copy
Object[]
Int32[,]
Object[]
red
True
10
20
30
40
Methods of arrays
Clear
Sets all element values to the default value of the array's element type. The Clear() method does not reset the size of the array.

In the following example $a is an array of objects.

PowerShell

Copy
$a = 1, 2, 3
$a.Clear()
$a | % { $null -eq $_ }
Output

Copy
True
True
True
In this example, $intA is explicitly typed to contain integers.

PowerShell

Copy
[int[]] $intA = 1, 2, 3
$intA.Clear()
$intA
Output

Copy
0
0
0
ForEach
Allows to iterate over all elements in the array and perform a given operation for each element of the array.

The ForEach method has several overloads that perform different operations.


Copy
ForEach(scriptblock expression)
ForEach(scriptblock expression, object[] arguments)
ForEach(type convertToType)
ForEach(string propertyName)
ForEach(string propertyName, object[] newValue)
ForEach(string methodName)
ForEach(string methodName, object[] arguments)
ForEach(scriptblock expression)
ForEach(scriptblock expression, object[] arguments)
This method was added in PowerShell v4.

 Note

The syntax requires the usage of a script block. Parentheses are optional if the scriptblock is the only parameter. Also, there must not be a space between the method and the opening parenthesis or brace.

The following example shows how use the ForEach method. In this case the intent is to generate the square value of the elements in the array.

PowerShell

Copy
$a = @(0 .. 3)
$a.ForEach({ $_ * $_})
Output

Copy
0
1
4
9
Just like the -ArgumentList parameter of ForEach-Object, the arguments parameter allows the passing of an array of arguments to a script block configured to accept them.

For more information about the behavior of ArgumentList, see about_Splatting.

ForEach(type convertToType)
The ForEach method can be used to swiftly cast the elements to a different type; the following example shows how to convert a list of string dates to [DateTime] type.

PowerShell

Copy
@("1/1/2017", "2/1/2017", "3/1/2017").ForEach([datetime])
Output

Copy

Sunday, January 1, 2017 12:00:00 AM
Wednesday, February 1, 2017 12:00:00 AM
Wednesday, March 1, 2017 12:00:00 AM
ForEach(string propertyName)
ForEach(string propertyName, object[] newValue)
The ForEach method can also be used to quickly retrieve, or set property values for every item in the collection.

PowerShell

Copy
# Set all LastAccessTime properties of files to the current date.
(dir 'C:\Temp').ForEach('LastAccessTime', (Get-Date))
# View the newly set LastAccessTime of all items, and find Unique entries.
(dir 'C:\Temp').ForEach('LastAccessTime') | Get-Unique
Output

Copy
Wednesday, June 20, 2018 9:21:57 AM
ForEach(string methodName)
ForEach(string methodName, object[] arguments)
Lastly, ForEach methods can be used to execute a method on every item in the collection.

PowerShell

Copy
("one", "two", "three").ForEach("ToUpper")
Output

Copy
ONE
TWO
THREE
Just like the -ArgumentList parameter of ForEach-Object, the Arguments parameter allows the passing of an array of values to a script block configured to accept them.

 Note

Starting in Windows PowerShell 3.0 retrieving properties and executing methods for each item in a collection can also be accomplished using "Methods of scalar objects and collections". You can read more about that here about_methods.

Where
Allows to filter or select the elements of the array. The script must evaluate to anything different than: zero (0), empty string, $false or $null for the element to show after the Where. For more information about boolean evaluation, see about_Booleans.

There is one definition for the Where method.


Copy
Where(scriptblock expression[, WhereOperatorSelectionMode mode
                            [, int numberToReturn]])
 Note

The syntax requires the usage of a script block. Parentheses are optional if the scriptblock is the only parameter. Also, there must not be a space between the method and the opening parenthesis or brace.

The Expression is scriptblock that is required for filtering, the mode optional argument allows additional selection capabilities, and the numberToReturn optional argument allows the ability to limit how many items are returned from the filter.

The acceptable values for mode are:

Default (0) - Return all items
First (1) - Return the first item
Last (2) - Return the last item
SkipUntil (3) - Skip items until condition is true, the return the remaining items
Until (4) - Return all items until condition is true
Split (5) - Return an array of two elements
The first element contains matching items
The second element contains the remaining items
The following example shows how to select all odd numbers from the array.

PowerShell

Copy
(0..9).Where{ $_ % 2 }
Output

Copy
1
3
5
7
9
This example show how to select the strings that are not empty.

PowerShell

Copy
('hi', '', 'there').Where({$_.Length})
Output

Copy
hi
there
Default
The Default mode filters items using the Expression scriptblock.

If a numberToReturn is provided, it specifies the maximum number of items to return.

PowerShell

Copy
# Get the zip files in the current users profile, sorted by LastAccessTime.
$Zips = dir $env:userprofile -Recurse '*.zip' | Sort-Object LastAccessTime
# Get the least accessed file over 100MB
$Zips.Where({$_.Length -gt 100MB}, 'Default', 1)
 Note

Both the Default mode and First mode return the first (numberToReturn) items, and can be used interchangeably.

Last
PowerShell

Copy
$h = (Get-Date).AddHours(-1)
$logs = dir 'C:\' -Recurse '*.log' | Sort-Object CreationTime
# Find the last 5 log files created in the past hour.
$logs.Where({$_.CreationTime -gt $h}, 'Last', 5)
SkipUntil
The SkipUntil mode skips all objects in a collection until an object passes the script block expression filter. It then returns ALL remaining collection items without testing them. Only one passing item is tested.

This means the returned collection contains both passing and non-passing items that have NOT been tested.

The number of items returned can be limited by passing a value to the numberToReturn argument.

PowerShell

Copy
$computers = "Server01", "Server02", "Server03", "localhost", "Server04"
# Find the first available online server.
$computers.Where({ Test-Connection $_ }, 'SkipUntil', 1)
Output

Copy
localhost
Until
The Until mode inverts the SkipUntil mode. It returns ALL items in a collection until an item passes the script block expression. Once an item passes the scriptblock expression, the Where method stops processing items.

This means that you receive the first set of non-passing items from the Where method. After one item passes, the rest are NOT tested or returned.

The number of items returned can be limited by passing a value to the numberToReturn argument.

PowerShell

Copy
# Retrieve the first set of numbers less than or equal to 10.
(1..50).Where({$_ -gt 10}, 'Until')
# This would perform the same operation.
(1..50).Where({$_ -le 10})
Output

Copy
1
2
3
4
5
6
7
8
9
10
 Note

Both Until and SkipUntil operate under the premise of NOT testing a batch of items.

Until returns the items BEFORE the first pass.

SkipUntil returns all the items AFTER the first pass, including the first passing item.

Split
The Split mode splits, or groups collection items into two separate collections. Those that pass the scriptblock expression, and those that do not.

If a numberToReturn is specified, the first collection, contains the passing items, not to exceed the value specified.

The remaining objects, even those that PASS the expression filter, are returned in the second collection.

PowerShell

Copy
$running, $stopped = (Get-Service).Where({$_.Status -eq 'Running'}, 'Split')
$running
Output

Copy
Status   Name               DisplayName
------   ----               -----------
Running  Appinfo            Application Information
Running  AudioEndpointBu... Windows Audio Endpoint Builder
Running  Audiosrv           Windows Audio
...
PowerShell

Copy
$stopped
Output

Copy
Status   Name               DisplayName
------   ----               -----------
Stopped  AJRouter           AllJoyn Router Service
Stopped  ALG                Application Layer Gateway Service
Stopped  AppIDSvc           Application Identity
...
 Note

Both foreach and where methods are intrinsic members. For more information about intrinsic members, see about_Instrinsic_Members

Get the members of an array
To get the properties and methods of an array, such as the Length property and the SetValue method, use the InputObject parameter of the Get-Member cmdlet.

When you pipe an array to Get-Member, PowerShell sends the items one at a time and Get-Member returns the type of each item in the array (ignoring duplicates).

When you use the InputObject parameter, Get-Member returns the members of the array.

For example, the following command gets the members of the $a array variable.

PowerShell

Copy
Get-Member -InputObject $a
You can also get the members of an array by typing a comma (,) before the value that is piped to the Get-Member cmdlet. The comma makes the array the second item in an array of arrays. PowerShell pipes the arrays one at a time and Get-Member returns the members of the array. Like the next two examples.

PowerShell

Copy
,$a | Get-Member

,(1,2,3) | Get-Member
Manipulating an array
You can change the elements in an array, add an element to an array, and combine the values from two arrays into a third array.

To change the value of a particular element in an array, specify the array name and the index of the element that you want to change, and then use the assignment operator (=) to specify a new value for the element. For example, to change the value of the second item in the $a array (index position 1) to 10, type:

PowerShell

Copy
$a[1] = 10
You can also use the SetValue method of an array to change a value. The following example changes the second value (index position 1) of the $a array to 500:

PowerShell

Copy
$a.SetValue(500,1)
You can use the += operator to add an element to an array. The following example shows how to add an element to the $a array.

PowerShell

Copy
$a = @(0..4)
$a += 5
 Note

When you use the += operator, PowerShell actually creates a new array with the values of the original array and the added value. This might cause performance issues if the operation is repeated several times or the size of the array is too big.

It is not easy to delete elements from an array, but you can create a new array that contains only selected elements of an existing array. For example, to create the $t array with all the elements in the $a array except for the value at index position 2, type:

PowerShell

Copy
$t = $a[0,1 + 3..($a.length - 1)]
To combine two arrays into a single array, use the plus operator (+). The following example creates two arrays, combines them, and then displays the resulting combined array.

PowerShell

Copy
$x = 1,3
$y = 5,9
$z = $x + $y
As a result, the $z array contains 1, 3, 5, and 9.

To delete an array, assign a value of $null to the array. The following command deletes the array in the $a variable.

$a = $null

You can also use the Remove-Item cmdlet, but assigning a value of $null is faster, especially for large arrays.

Arrays of zero or one
Beginning in Windows PowerShell 3.0, a collection of zero or one object has the Count and Length property. Also, you can index into an array of one object. This feature helps you to avoid scripting errors that occur when a command that expects a collection gets fewer than two items.

The following examples demonstrate this feature.

Zero objects
PowerShell

Copy
$a = $null
$a.Count
$a.Length
Output

Copy
0
0
One object
PowerShell

Copy
$a = 4
$a.Count
$a.Length
$a[0]
$a[-1]
Output

Copy
1
1
4
4
Member enumeration
You can use member enumeration to get property values from all members of a collection. When you use the member access operator (.) with a member name on a collection object, such as an array, if the collection object does not have a member of that name, the items of the collection are enumerated and PowerShell looks for that member on each item. This applies to both property and method members.

The following example creates two new files and stores the resulting objects in the array variable $files. Since the array object does not have the LastWriteTime member, the value of LastWriteTime is returned for each item in the array.

PowerShell

Copy
$files = (New-Item -Type File -Force '/temp/t1.txt'),
         (New-Item -Force -Type File '/temp/t2.txt')
$files.LastWriteTime
Output

Copy
Friday, June 25, 2021 1:21:17 PM
Friday, June 25, 2021 1:21:17 PM
Member enumeration can be used to get values from items in a collection, but it cannot be used to set values on items in a collection. For example:

PowerShell

Copy
$files.LastWriteTime = (Get-Date).AddDays(-1)
Output

Copy
The property 'LastWriteTime' cannot be found on this object. Verify that the
property exists and can be set.
At line:1 char:1
+ $files.LastWriteTime = (Get-Date).AddDays(-1)
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [], RuntimeException
    + FullyQualifiedErrorId : PropertyAssignmentException
To set the values you must use a method.

PowerShell

Copy
$files.set_LastWriteTime((Get-Date).AddDays(-1))
$files.LastWriteTime
Output

Copy
Thursday, June 24, 2021 1:23:30 PM
Thursday, June 24, 2021 1:23:30 PM
The set_LastWriteTime() method is a hidden member of the FileInfo object. The following example shows how to find members that have a hidden set method.

PowerShell

Copy
$files | Get-Member | Where-Object Definition -like '*set;*'
Output

Copy
   TypeName: System.IO.FileInfo

Name              MemberType Definition
----              ---------- ----------
Attributes        Property   System.IO.FileAttributes Attributes {get;set;}
CreationTime      Property   datetime CreationTime {get;set;}
CreationTimeUtc   Property   datetime CreationTimeUtc {get;set;}
IsReadOnly        Property   bool IsReadOnly {get;set;}
LastAccessTime    Property   datetime LastAccessTime {get;set;}
LastAccessTimeUtc Property   datetime LastAccessTimeUtc {get;set;}
LastWriteTime     Property   datetime LastWriteTime {get;set;}
LastWriteTimeUtc  Property   datetime LastWriteTimeUtc {get;set;}
 Caution

Since the method is executed for each item in the collection, care should be taken when calling methods using member enumeration.

See also
about_Assignment_Operators
about_Hash_Tables
about_Operators
about_For
about_Foreach
about_While