Shortcut | Description
-|-
CTRL + 5 | Strikethrough

### Delete empty rows
1. Select data/column
2. F5 > Special... > Blanks # This selects the empty rows
3. "Delete Sheet Rows" in the Ribbon

### Create a function that examines a regex and spits out the result
In this case the regex examines if a servername contains a P before 3 digits:

Cell | Result
-| -
someservername**p**002.example.com | PROD
someotherservername**d**001.example.com | NON PROD

```
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
- https://stackoverflow.com/questions/22542834/how-to-use-regular-expressions-regex-in-microsoft-excel-both-in-cell-and-loops
- https://stackoverflow.com/questions/8146485/returning-a-regex-match-in-vba-excel

### Filter rows based on a list of values
`=IF(ISERROR(VLOOKUP(A1,Sheet2!A:A,1,FALSE)),"NotInList","InList")`
