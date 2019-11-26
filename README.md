# Excel.md

| Shortcut | Description |
| :--- | :--- |
| CTRL + 5 | Strikethrough |

## Delete empty rows

1. Select data/column
2. F5 &gt; Special... &gt; Blanks \# This selects the empty rows
3. "Delete Sheet Rows" in the Ribbon

## Add VBA function to spreadsheet

1. File &gt; Options &gt; Customize Ribbon &gt; Tick "Developer" tab
2. Click on new "Developer" tab in the ribbon &gt; Visual Basic
3. Tools &gt; Reference &gt; Tick "Microsoft VBScript Regular Expressions 5.5"
4. Run Macro \(F5\) &gt; Give title &gt; Create
5. Write the VBA script \(for example the "simpleCellRegex" from [https://github.com/cardoppler/wiki/blob/master/Other/excel.md](https://github.com/cardoppler/wiki/blob/master/Other/excel.md)\)
6. Minimize the VBA editor
7. In Excel, type =simpleCellRegex and call the new function

## Create a function that examines a regex and spits out the result

In this case the regex examines if a servername contains a P before 3 digits:

| Cell | Result |
| :--- | :--- |
| someservername**p**002.example.com | PROD |
| someotherservername**d**001.example.com | NON PROD |

```text
Function simpleCellRegex(Cell As Range) As String
    Dim regEx As New RegExp
    Dim Pattern As String
    Dim strInput As String

    Pattern = "(\w)\d+"
    strInput = Cell.Value

    With regEx
        .Global = True
        .MultiLine = True
        .IgnoreCase = True
        .Pattern = Pattern
    End With

    Set Matches = regEx.Execute(strInput)
    If Matches.Count <> 0 Then
        simpleCellRegex = Matches.Item(0).SubMatches.Item(0)
        If (StrComp(Matches.Item(0).SubMatches.Item(0), "P", vbTextCompare) = 0) Then
            simpleCellRegex = "PROD"
        Else
            simpleCellRegex = "NON PROD"
        End If
    End If
End Function
```

Resources used:

* [https://stackoverflow.com/questions/22542834/how-to-use-regular-expressions-regex-in-microsoft-excel-both-in-cell-and-loops](https://stackoverflow.com/questions/22542834/how-to-use-regular-expressions-regex-in-microsoft-excel-both-in-cell-and-loops)
* [https://stackoverflow.com/questions/8146485/returning-a-regex-match-in-vba-excel](https://stackoverflow.com/questions/8146485/returning-a-regex-match-in-vba-excel)

## Filter rows based on a list of values

`=IF(ISERROR(VLOOKUP(A1,Sheet2!A:A,1,FALSE)),"NotInList","InList")`

## Expand IP Range

[![](https://github.com/cardoppler/wiki/raw/master/Images/Excel/ExpandIpRange_01.png?raw=true)](https://github.com/cardoppler/wiki/blob/master/Images/Excel/ExpandIpRange_01.png?raw=true) [![](https://github.com/cardoppler/wiki/raw/master/Images/Excel/ExpandIpRange_02.png?raw=true)](https://github.com/cardoppler/wiki/blob/master/Images/Excel/ExpandIpRange_02.png?raw=true)

VBA function:

```text
Function ExpandIpRange(Cell As range)
    Dim CellValue As String
    Dim RegEx As New RegExp
    Dim Pattern As String
    Dim OutputString As String
    Dim BaseString As String
    
    'This matches a cell containing an IP range of the format 10.11.12.249-10.11.13.5'
    Pattern = "(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})-(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})"
    With RegEx
        .Global = True
        .MultiLine = True
        .IgnoreCase = True
        .Pattern = Pattern
    End With
    
    CellValue = Cell.Value
    Set Matches = RegEx.Execute(CellValue)
    
    'Exit if no matches'
    If Matches.Count = 0 Then
        Exit Function
    End If
      
    LeftValueFirstByte = CInt(Matches.Item(0).SubMatches.Item(0))
    LeftValueSecondByte = CInt(Matches.Item(0).SubMatches.Item(1))
    LeftValueThirdByte = CInt(Matches.Item(0).SubMatches.Item(2))
    LeftValueFourthByte = CInt(Matches.Item(0).SubMatches.Item(3))
    
    RightValueFirstByte = CInt(Matches.Item(0).SubMatches.Item(4))
    RightValueSecondByte = CInt(Matches.Item(0).SubMatches.Item(5))
    RightValueThirdByte = CInt(Matches.Item(0).SubMatches.Item(6))
    RightValueFourthByte = CInt(Matches.Item(0).SubMatches.Item(7))
    
    'Only support ranges smaller than /16'
    If (RightValueFirstByte > LeftValueFirstByte) Or (RightValueSecondByte > LeftValueSecondByte) Then
        Exit Function
    End If
    
    CurrentThirdByte = LeftValueThirdByte
    CurrentFourthByte = LeftValueFourthByte
    i = CurrentFourthByte
    
    Do
        CurrentFourthByte = i Mod 256
        If CurrentFourthByte = 0 Then
            CurrentThirdByte = CurrentThirdByte + 1
        End If
        OutputString = OutputString & LeftValueFirstByte & "." & LeftValueSecondByte & "." & CurrentThirdByte & "." & CurrentFourthByte & ", "
        i = i + 1
    Loop Until ((CurrentThirdByte = RightValueThirdByte) And (CurrentFourthByte = RightValueFourthByte))
    
    OutputString = Left(OutputString, Len(OutputString) - 2) 'Trim the last comma'
    ExpandIpRange = OutputString 'Write the result to the cell'
    
End Function
```

