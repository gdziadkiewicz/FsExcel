<img src="https://raw.githubusercontent.com/misterspeedy/FsExcel/main/assets/logo.png"
     alt="FsExcel Logo"
     style="width: 150px;" />
     
[![Twitter URL](https://img.shields.io/twitter/url/https/twitter.com/fsexcel.svg?style=social&label=Twitter%20%40FsExcel)](https://twitter.com/fsexcel)
[![Nuget](https://img.shields.io/nuget/v/Fsexcel)](https://www.nuget.org/packages/FsExcel/)


## Welcome! 

Welcome to FsExcel, a library for generating Excel spreadsheets using very simple code.

FsExcel is based on [ClosedXML](https://github.com/ClosedXML/ClosedXML) but abstracts away many of the complications of building spreadsheets cell by cell.


This tutorial is also available as an <a href="https://raw.githubusercontent.com/misterspeedy/FsExcel/main/src/Notebooks/Tutorial.dib" download="Tutorial.dib">interactive notebook</a>. Download it, open in Visual Studio Code, and start generating spreadsheets for real!

* *Contributors* - please see [Contributing.md](/Contributing.md) for getting-started information.

* *Usage example* - for an example of FsExcel in action, see http://www.pushbuttonreceivetables.com. Source code on [GitHub](https://github.com/misterspeedy/HtmlExcel).

---
## Hello World

Here's the complete code to generate a spreadsheet with a single cell containing a string.

Run this and you should find a spreadsheet called `HelloWorld.xlsx` in your `/temp` folder. (Change the path to suit.)
<!-- Test -->

```fsharp
// For scripts only; for programs, use NuGet to install FsExcel:
#r "nuget: FsExcel"

let savePath = "/temp"

open System.IO
open FsExcel

[
    Cell [ String "Hello world!" ]
]
|> Render.AsFile (Path.Combine(savePath, "HelloWorld.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/HelloWorld.PNG?raw=true"
     alt="Hello World example"
     style="width: 120px;" />

This example embodies the main stages of building a spreadsheet using FsExcel:

1) Build a list using a list comprehension: `[ ... ]`
2) In the list make cells using `Cell`
3) Each cell gets a list of properties, in this case just the cell content, which here is a string: `String "Hello world!"`

If you've used `Fable.React`, or a similar library, you'll already be familiar with the concepts so far.

4) Send the resulting list to `FsExcel.Render.AsFile`, providing a path.

---
## Multiple Cells
<!-- Test -->

```fsharp
open System.IO
open FsExcel

[
    for i in 1..10 do
        Cell [ Integer i ]
]
|> Render.AsFile (Path.Combine(savePath, "MultipleCells.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/MultipleCells.PNG?raw=true"
     alt="Multiple Cells example"
     style="width: 500px;" />

Here we use a `for...` comprehension to build multiple cells. (Don't panic: we could have used `List.map` instead!)

By default each new cell is put on the right of its predecessor.

---
## Vertical Movement

If you want the next cell to be rendered below instead of to the right, you can add a `Next(DownBy 1)` property to the cell:

<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel

[
    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
            Next(DownBy 1)
        ]
]
|> Render.AsFile (Path.Combine(savePath, "VerticalMovement.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/VerticalMovement.PNG?raw=true"
     alt="Vertical Movement example"
     style="width: 100px;" />

---
The `Next` property overrides the default behaviour of rendering each successive cell one to the right. In this case we override it with a 'go down by 1' behaviour.

But what if you want a table of cells? Use the default behaviour for each cell in a row except the last. In the last cell use `Next NewRow`. This causes the next cell to be rendered in column 1 of the next row.

<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel

[
    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [
            String monthName
        ]
        Cell [
            Integer monthName.Length
            Next NewRow
        ]
]
|> Render.AsFile (Path.Combine(savePath, "Rows.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Rows.PNG?raw=true"
     alt="Rows example"
     style="width: 150px;" />

---
Maybe you don't like the idea of saying where to go next in the properties of a cell. No problem, you can have standalone position-control with the `Go` instruction:
<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel

[
    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [ String monthName ]
        Cell [ Integer monthName.Length ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "RowsGo.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/RowsGo.PNG?raw=true"
     alt="Rows Go example"
     style="width: 150px;" />

---
## Indentation

Maybe you want a series of rows that don't start in column 1.  Use `Indent`:
<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel

[
    Go(Indent 2)

    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [ String monthName ]
        Cell [ Integer monthName.Length ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "Indentation.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Indentation.PNG?raw=true"
     alt="Indentation example"
     style="width: 220px;" />

Now each row begins at column 2.

Indents apply to all `NewRow` operations until some other indent value is set using `Go(Indent n)`. Specify no indenting with `Go(Indent 1)`.

You can specify indents relative to the current indent level using `Go(IndentBy n)` where _n_ can be a positive or negative integer.

---
## Border and Font Styling

You can add border styling and font emphasis (bold, italic, underline or strikethrough) styling using `Border (...)` and `FontEmphasis ...` cell properties.

The border style values are in `ClosedXML.Excel.XLBorderStyleValues` and the underline values are in `ClosedXML.Excel.XLFontUnderlineValues`.
<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel
open ClosedXML.Excel

[
    for heading in ["Month"; "Letter Count"] do
        Cell [
            String heading
            Border (Border.Bottom XLBorderStyleValues.Medium)
            FontEmphasis Bold
            FontEmphasis Italic
        ]
    Go NewRow
    
    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [ 
            String monthName
            FontEmphasis (Underline XLFontUnderlineValues.DoubleAccounting)
            if monthName = "May" then
                FontEmphasis StrikeThrough
        ]
        Cell [ Integer monthName.Length ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "Styling.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Styling.PNG?raw=true"
     alt="Styling example"
     style="width: 150px;" />

---
As they are just list items, styles can be composed and applied together as a list. You'll need a `yield!` to include these multiple elements in your cell property list.
<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel
open ClosedXML.Excel

let headingStyle = 
    [
        Border(Border.Bottom XLBorderStyleValues.Medium)
        FontEmphasis Bold
        FontEmphasis Italic 
    ]

[
    for heading in ["Month"; "Letter Count"] do
        Cell [
            String heading
            yield! headingStyle
        ]
    Go NewRow
    
    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [ String monthName ]
        Cell [ Integer monthName.Length ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "ComposedStyling.xlsx"))

```
## Font Name and Size

You can set the font name using `FontName` and the size using `FontSize`:

```fsharp
open System.IO
open System.Globalization
open FsExcel
open ClosedXML.Excel

// ClosedXml currently depends on SixLabors.Fonts - 
// we use that to enumerate fonts so this code works cross-platform:
let fontNames = 
    SixLabors.Fonts.SystemFonts.Collection.Families
    |> Seq.map (fun font -> font.Name)
    |> Seq.sort
    |> Seq.truncate 20

[
    for i, fontName in fontNames |> Seq.indexed do
        Cell [
            String fontName
            FontName fontName
            FontSize (10 + (i * 2) |> float)
        ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "FontNameSize.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/FontNameSize.PNG?raw=true"
     alt="Number Format and Alignment example"
     style="width: 250px;" />

---
## Wrap Text

You can wrap text in cells with long sentences/many words using `WrapText true`:

<!-- Test -->

```fsharp
open System.IO
open FsExcel
open ClosedXML.Excel

[
    Cell [ String "Without wrap text:"
           HorizontalAlignment Center
           VerticalAlignment Middle
           CellSize (ColWidth 16) ]
    Cell [ String "The quick brown fox jumps over the lazy dog."
           HorizontalAlignment Center
           VerticalAlignment Middle ]
    Go NewRow
    Cell [ String "With wrap text:"
           HorizontalAlignment Center
           VerticalAlignment Middle 
           CellSize (ColWidth 16) ]
    Cell [ String "The quick brown fox jumps over the lazy dog."
           HorizontalAlignment Center
           VerticalAlignment Middle
           WrapText true ]
]
|> Render.AsFile (Path.Combine(savePath, "WrapText.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/WrapText.PNG?raw=true"
     alt="Wrap Text Example"
     style="width: 350px;" />

---
## Text Rotation

You can rotate text between -90° and +90° with `TextRotation n`.

<!-- Test -->

```fsharp
open System
open FsExcel

let p, m, g = "⏺", "◑", "⭘"
let performances = 
    [|
        [| p; m; g; g; p;  p; g; p; p; g |]
        [| g; m; g; m; g;  p; g; p; p; g |]
        [| g; m; m; g; g;  p; g; g; p; g |]
        [| m; m; m; p; p;  p; g; m; p; g |]
    
        [| p; p; p; p; g;  g; m; m; p; g |]
        [| p; g; p; g; g;  g; p; g; m; m |]
        [| g; p; g; p; m;  p; m; p; p; g |]
        [| p; p; m; g; p;  p; p; m; p; m |]
    |]

let getPerformance (categoryIndex : int) (supplierIndex : int) =
    performances[supplierIndex-1][categoryIndex-1]

[
    Go (RC(1, 2))
    for category in 1..10 do
        Cell [String $"Category {category}"; TextRotation 45; CellSize (RowHeight 45)]
    Go NewRow
    for supplier in 1..8 do
        Cell [String $"Supplier {supplier}"; CellSize (ColWidth 10)]
        Go NewRow
    Go (RC(2, 2))
    Go (Indent 2)
    for supplier in 1..8 do
        for category in 1..10 do
            Cell [ String (getPerformance category supplier); HorizontalAlignment Center]
        Go NewRow
]
|> Render.AsFile (System.IO.Path.Combine(savePath, "TextRotation.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/TextRotation.PNG?raw=true"
     alt="Wrap Text Example"
     style="width: 500px;" />

---
## Number Formatting and Alignment

Number styling can be applied using standard Excel format strings.  You can also apply horizontal alignment.
<!-- Test -->

```fsharp
open System
open System.IO
open FsExcel
open ClosedXML.Excel

module PseudoRandom =

    let mutable state = 1u
    let mangle (n : UInt64) = (n &&& (0x7fffffff |> uint64)) + (n >>> 31)

    let nextDouble() =
        state <- (state |> uint64) * 48271UL |> mangle |> mangle |> uint32
        (float state) / (float Int32.MaxValue)

let headingStyle = 
    [
        Border(Border.Bottom XLBorderStyleValues.Medium)
        FontEmphasis Bold
        FontEmphasis Italic 
    ]

[
    for heading, alignment in ["Stock Item", Left; "Price", Right ; "Count", Right] do
        Cell [
            String heading
            yield! headingStyle
            HorizontalAlignment alignment
        ]
    
    Go NewRow

    for item in ["Apples"; "Oranges"; "Pears"] do
        Cell [
            String item
        ]
        Cell [
            Float ((PseudoRandom.nextDouble()*1000.))
            FormatCode "$0.00"
        ]
        Cell [
            Integer (int (PseudoRandom.nextDouble()*100.))
            FormatCode "#,##0"
        ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "NumberFormatAndAlignment.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/NumberFormatAndAlignment.PNG?raw=true"
     alt="Number Format and Alignment example"
     style="width: 250px;" />

---
## Formulae

You can add a formula to a cell using `FormulaA1(...)`.  

Currently only the `A1` style of cell referencing is supported, meaning that you will need to keep track of the column letter and row number you want to refer to:
<!-- Test -->

```fsharp
open System
open System.IO
open FsExcel
open ClosedXML.Excel

module PseudoRandom =

    let mutable state = 1u
    let mangle (n : UInt64) = (n &&& (0x7fffffff |> uint64)) + (n >>> 31)

    let nextDouble() =
        state <- (state |> uint64) * 48271UL |> mangle |> mangle |> uint32
        (float state) / (float Int32.MaxValue)

let headingStyle = 
    [
        Border(Border.Bottom XLBorderStyleValues.Medium)
        FontEmphasis Bold
        FontEmphasis Italic 
    ]

[
    for heading, alignment in ["Stock Item", Left; "Price", Right ; "Count", Right; "Total", Right] do
        Cell [
            String heading
            yield! headingStyle
            HorizontalAlignment alignment
        ]
    
    Go NewRow

    for index, item in ["Apples"; "Oranges"; "Pears"] |> List.indexed do
        Cell [
            String item
        ]
        Cell [
            Float (PseudoRandom.nextDouble()*1000.)
            FormatCode "$0.00"
        ]
        Cell [
            Integer (int (PseudoRandom.nextDouble()*1000.))
            FormatCode "#,##0"
        ]
        Cell [
            FormulaA1 $"=B{index+2}*C{index+2}"
            FormatCode "$#,##0.00"
        ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "Formulae.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Formulae.PNG?raw=true"
     alt="Styling example"
     style="width: 300px;" />

---
## Color

Set the font color with `FontColor` and the background color with the `BackgroundColor` property.  Set the border color with `BorderColor`.

The color values and color creation functions are in `ClosedXml.Excel.XLColor`.
<!-- Test -->

```fsharp
open System.IO
open FsExcel
open ClosedXML.Excel

[
    let values = [0..32..224] @ [255]
    for r in values do
        for g in values do
            for b in values do
                // N.B. the API refuses to fill a cell with black if its font is black
                // so the very first cell won't be colored.
                let backgroundColor = ClosedXML.Excel.XLColor.FromArgb(0, r, g, b)
                let fontColor = ClosedXML.Excel.XLColor.FromArgb(0, b, r, g)
                let borderColor = ClosedXML.Excel.XLColor.FromArgb(0, g, b, r)
                Cell [
                    String $"R={r};G={g};B={b}"
                    FontColor fontColor
                    BackgroundColor backgroundColor
                    Border (Border.Top XLBorderStyleValues.Thick)
                    Border (Border.Right XLBorderStyleValues.Thick)
                    Border (Border.Bottom XLBorderStyleValues.Thick)
                    Border (Border.Left XLBorderStyleValues.Thick)
                    // Could also have used Border.All:
                    // Border (Border.All XLBorderStyleValues.Thick)
                    BorderColor (BorderColor.Top borderColor)
                    BorderColor (BorderColor.Right borderColor)
                    BorderColor (BorderColor.Bottom borderColor)
                    BorderColor (BorderColor.Left borderColor)
                    // Could also have used BorderColor.All:
                    // BorderColor (BorderColor.All borderColor)
                ]
            Go NewRow
        Go NewRow

]
|> Render.AsFile (Path.Combine(savePath, "Color.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Color.PNG?raw=true"
     alt="Color example"
     style="width: 400px;" />

---
## Range Styles

You can apply any properties to all cells from a point in your code using `Style [ prop; prop...]`. Don't forget to reset style with `Style []` afterwards.
<!-- Test -->

```fsharp
open System
open System.IO
open FsExcel
open ClosedXML.Excel

module PseudoRandom =

    let mutable state = 1u
    let mangle (n : UInt64) = (n &&& (0x7fffffff |> uint64)) + (n >>> 31)

    let nextDouble() =
        state <- (state |> uint64) * 48271UL |> mangle |> mangle |> uint32
        (float state) / (float Int32.MaxValue)

[
    Style [
        Border(Border.Bottom XLBorderStyleValues.Medium)
        FontEmphasis Bold
        FontEmphasis Italic 
    ]
    for heading in ["Stock Item"; "Price"; "Count"] do
        Cell [ String heading ]
    Style []
    
    Go NewRow
    for item in ["Apples"; "Oranges"; "Pears"] do
        Cell [
            String item
        ]
        Style [ FontEmphasis Italic ]        
        Cell [
            Float ((PseudoRandom.nextDouble()*1000.))
            FormatCode "$0.00"
        ]
        Cell [
            Integer (int (PseudoRandom.nextDouble()*100.))
            FormatCode "#,##0"
        ]
        Style []
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "RangeStyle.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/RangeStyle.PNG?raw=true"
     alt="Range Style example"
     style="width: 250px;" />

---
## Adding a Border to Merged Cells

To add a border to all cells in an Item list that includes *merged cells*, use:

`BorderMergedCell [ BorderType (Border.x XLBorderStyleValues.[......]); ColorBorder (BorderColor.x (XLColor.FromArgb(0, 68, 114, 196)))]`

where *x* = [All, Top, Right, Bottom, Left].  Include this border styling *after* any merged cells.

Other styling such as background color, font emphasis, font style etc. can be applied to all cells (including merged cells) using `Style [prop; prop...]` at the *start* of the Item list as outlined above. These styling propeties are retained when a cell is merged, unlike with cell borders.
<!-- Test -->

```fsharp
open System.IO
open System
open ClosedXML.Excel
open FsExcel 


[   Go NewRow
    for heading, colWidth in ["ID", 3.22; "Car Name", 10.33; "Car Description", 49.33; "Car Regestration", 16.89 ] do
        Cell [
            String heading
            FontEmphasis Bold
            FontName "Calibri"
            FontSize 11
            HorizontalAlignment Center
            FontColor (XLColor.FromArgb(0, 255, 255, 255))
            BackgroundColor (XLColor.FromArgb(0, 68, 114, 196))
            Border (Border.All XLBorderStyleValues.Thin)
            CellSize (ColWidth colWidth)
        ]
    Go NewRow
    Style [ HorizontalAlignment Center
            VerticalAlignment Middle
            BackgroundColor (XLColor.FromArgb(0, 240, 240, 210))]
    Cell [  Integer 1
            //HorizontalAlignment Left
            //VerticalAlignment TopMost
            Name "ID" ] 
    Cell [  String "Ford Fiesta" ]
            //HorizontalAlignment Center
            //VerticalAlignment Middle ] 
    Cell [  String "Car Technical Details:"
            Next (DownBy 1) ]
    Cell [  String "Technical Detail 1"
            Next (DownBy 1) ]
    Cell [  String "Technical Detail 2"
            Next (DownBy 1)]
    Cell [  String "Technical Detail 3"
            Name "LastL" ]
    Go (RC (3, 4))
    Cell [  String "AB12 CDE" 
            //HorizontalAlignment Right
            //VerticalAlignment Base
            Name "Reg" ]
    Go (RC (6, 4))
    Cell [Name "RegEnd"]
    Go (RC (7, 3))
    Cell [  String "Another Technical Detail"
            FontEmphasis Italic
            //VerticalAlignment Middle
            Name "TD" 
            Next Stay]
    Go (DownBy 1)
    Cell [ Name "info"]

    // Merging between named and specific cells
    MergeCells (ColRowLabel ("B", 3), ColRowLabel ("B", 6))
    MergeCells (NamedCell "ID", ColRowLabel ("A", 6))
    MergeCells (ColRowLabel ("C", 7), NamedCell "info")
    MergeCells (NamedCell "Reg", NamedCell "RegEnd") 
    // Adding a border to merged cells - any original border around a single cell is lost post merging cells
    BorderMergedCell [ BorderType (Border.All XLBorderStyleValues.Thin)
                       ColorBorder (BorderColor.All (XLColor.FromArgb(0, 68, 114, 196)))]
]
|> Render.AsFile (Path.Combine(savePath, "BorderMergedCells.xlsx"))  

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/BorderMergedCells.PNG?raw=true"
     alt="Adding a border to merged cells"
     style="width: 500px;" />

---
## Absolute Positioning

FsExcel is designed to save you from having to keep track of absolute row- and column-numbers. However sometimes you might want to position a cell at an absolute row or column position - or both.

After the explicitly-positioned cell, subsequent cells are by default rendered to the right again.
<!-- Test -->

```fsharp
open System.IO
open FsExcel
open ClosedXML.Excel

[
    Go (Col 3)
    Cell [ String "Col 3"]
    Go (Row 4)
    Cell [ String "Row 4"]
    Go (RC(6, 5))
    Cell [ String "R6C5"]
    Cell [ String "R6C6"]
]
|> Render.AsFile (Path.Combine(savePath, "AbsolutePositioning.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/AbsolutePositioning.PNG?raw=true"
     alt="Absolute Positioning example"
     style="width: 350px;" />    

---
Remember that, by default, successive cells are placed to the right of their predecessors? Sometimes (rarely) you might want to suppress that behaviour completely. To do that use `Next Stay`.
<!-- Test -->

```fsharp
open System.IO
open FsExcel

[
    for i in 1..5 do
        Cell [
            Integer i
            Next Stay
        ]
        Go(DownBy i)
]
|> Render.AsFile (Path.Combine(savePath, "Stay.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Stay.PNG?raw=true"
     alt="Stay example"
     style="width: 150px;" />

---
## Named cells

To create worksheet scoped name use  
`
Name "Username"
`  
or  
`
ScopedName ("Email", NameScope.Worksheet)
`  

To create workbook scoped name use  
`
ScopedName ("Email", NameScope.Workbook)
`
<!-- Test -->

```fsharp
open System.IO
open FsExcel

[
    Cell [ 
        String "JohnDoe"
        Name "Username" ]
    Cell [ 
        String "john.doe@company.com"
        ScopedName ("Email", NameScope.Workbook) ]
]
|> Render.AsFile (Path.Combine(savePath, "NamedCells.xlsx"))

```
## Worksheets (Tabs)

By default, all cells are placed into a worksheet (tab) called "Sheet1".  You can override this, and create additional worksheets, using `Worksheet ...`.

If you do not want a "Sheet1" tab you'll need to use `Worksheet` to create an explicitly named sheet - before creating any cells.

Each new worksheet starts at the top-left cell, has an indent setting of 1 (no indent), and has an empty list as its current `Style [...]` value.

If you use `Worksheet` with the name of a worksheet that already exists, that worksheet becomes active with a current position of `RC(1, 1)`, no indent and an empty `Style [...]` value.
<!-- Test -->

```fsharp
open System.IO
open FsExcel

let britishCultureNativeName = "English (United Kingdom)"
let ukrainianCultureNativeName = "українська"

let britishCultureDateTimeFormatGetMonthName =
    [ "January"; "February"; "March"; "April"; "May"; "June"; "July";
       "August"; "September"; "October"; "November"; "December" ]

let britishCultureDateTimeFormatAbbreviatedMonthNames =
    [ "Jan"; "Feb"; "Mar"; "Apr"; "May"; "Jun"; "Jul"; "Aug"; "Sep"; "Oct";
      "Nov"; "Dec" ]

let ukrainianCultureDateTimeFormatGetMonthName =
    [ "січень"; "лютий"; "березень"; "квітень"; "травень"; "червень";
      "липень"; "серпень"; "вересень"; "жовтень"; "листопад"; "грудень" ]

let ukrainianCultureDateTimeFormatAbbreviatedMonthNames =
    [ "січ"; "лют"; "бер"; "кві"; "тра"; "чер"; "лип"; "сер"; "вер"; "жов";
      "лис"; "гру" ]

[
    Worksheet britishCultureNativeName
    for m in 0..11 do
        let monthName = britishCultureDateTimeFormatGetMonthName.[m]
        Cell [ String monthName ]
        Cell [ Integer monthName.Length ]
        Go NewRow

    Worksheet ukrainianCultureNativeName
    for m in 0..11 do
        let monthName = ukrainianCultureDateTimeFormatGetMonthName.[m]
        Cell [ String monthName ]
        Cell [ Integer monthName.Length ]
        Go NewRow

    Worksheet britishCultureNativeName // Switch back to the first worksheet
    Go (RC(13, 1))
    for m in 0..11 do
        let monthAbbreviation = britishCultureDateTimeFormatAbbreviatedMonthNames.[m]
        Cell [ String monthAbbreviation ]
        Cell [ Integer monthAbbreviation.Length ]
        Go NewRow

    Worksheet ukrainianCultureNativeName // Switch back to the second worksheet
    Go (RC(13, 1))
    for m in 0..11 do
        let monthAbbreviation = ukrainianCultureDateTimeFormatAbbreviatedMonthNames.[m]
        Cell [ String monthAbbreviation ]
        Cell [ Integer monthAbbreviation.Length ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "Worksheets.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/Worksheets.PNG?raw=true"
     alt="Worksheets example"
     style="width: 350px;" />

---
## Working with an existing workbook 

You can update an existing workbook by using `Workbook ...` as the first item in the list. By default, this will set the first worksheet in the workbook as the active sheet. 

Typically, you will want to immediately follow with a `Worksheet ...` to either move to the worksheet you want to update or create a new worksheet.

`Workbook`  requires you to pass in a reference to a valid `ClosedXML.Excel.XLWorkbook` object. See [Inserting blank rows](#insertingblankrows) below for an example.

<a name="insertingblankrows"></a>
## Inserting blank rows

One common task when working with existing workbooks is inserting rows of data above existing rows. You can use `InsertRowsAbove n` which will insert `n` blank rows above the current row. 

`InsertRowsAbove` does not change the current position. However, the row at that position is now the first inserted (blank) row. Note that if a formula refers to a cell that is moved, the formula is automatically updated.

<!-- Test -->

```fsharp
open System.IO
open ClosedXML.Excel
open FsExcel

// Open Worksheets.xlsx created in the previous snippet:
let workbook = new XLWorkbook(Path.Combine(savePath, "Worksheets.xlsx"))

let britishCultureNativeName = "English (United Kingdom)"
let ukrainianCultureNativeName = "українська"

let altMonthNames = [| "Vintagearious"; "Fogarious"; "Frostarious"; "Snowous"; "Rainous"; "Windous"; "Buddal"; "Floweral"; "Meadowal"; "Reapidor"; "Heatidor"; "Fruitidor" |]

[
    Workbook workbook
    Worksheet ukrainianCultureNativeName
    Go(RC(1,3))
    Cell [FormulaA1 $"='{britishCultureNativeName}'!B1*2" ]
    Worksheet britishCultureNativeName
    InsertRowsAbove 12 // The cell reference in the  formula above will be updated to B13
    for m in 0..11 do
        Cell [ String altMonthNames[m] ]
        Cell [ Integer altMonthNames[m].Length ]
        Go NewRow
]
|> Render.AsFile (Path.Combine(savePath, "WorksheetsRevised.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/InsertRowsAbove.PNG?raw=true"
     alt="InsertRowsAbove example"
     style="width: 300px;" />

---
## Column Widths and Row Heights for All Cells

You can set a specific width for *all* columns and a specific height for *all* rows with `SizeAll (ColWidth x)` and `SizeAll (RowHeight x)`.

Excel and ClosedXml documentation is not clear on what units are used. The width unit correlates with ~10% of the width given number of pixels. For example, if you need a cell to be ~50 pixels wide, then you would set the ColWidth to ~5. The width unit appears to be, roughly, average character width. The height unit is 60% of the pixel height. For example, if you want the height to be 45 pixels, then you would set the RowHeight to 27.
<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel

[
    for x in 1..12 do
        for y in 0..12 do
            Cell [ Integer (x * y) ]
        Go NewRow

    SizeAll (ColWidth 5)
    SizeAll (RowHeight 20)
]
|> Render.AsFile (Path.Combine(savePath, "ColumnWidthRowHeight.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/ColumnWidthRowHeight.PNG?raw=true"
     alt="Column Width and Row Height example"
     style="width: 350px;" />

---
## Individual Cell Sizing

To size individual cells within an Item list (e.g. `[ Cell [....]; Go NewRow; Cell [...]; Go NewRow etc.`]`)`, use `CellSize (ColWidth 10)` and `CellSize (RowHeight 10)` as part of a Cell's list of properties. 
<!-- Test -->

```fsharp
open System.IO
open System
open ClosedXML.Excel
open FsExcel

[   Go NewRow
    for heading, colWidth in ["ID", 3.22; "Car Name", 10.33; "Car Description", 49.33; "Car Registration", 16.89 ] do
        Cell [
            String heading
            FontEmphasis Bold
            FontName "Calibri"
            FontSize 11
            HorizontalAlignment Center
            FontColor (XLColor.FromArgb(0, 255, 255, 255))
            BackgroundColor (XLColor.FromArgb(0, 68, 114, 196))
            Border(Border.All XLBorderStyleValues.Thin)
            CellSize (ColWidth colWidth)
        ]
    Go NewRow
    Cell [ Integer 1
           HorizontalAlignment Center ] 
    Cell [ String "Ford Fiesta" ]
    Cell [ String "Car Technical Details..."]  
    Cell [ String "AB12 CDE" 
           HorizontalAlignment Center]
]
|> Render.AsFile (Path.Combine(savePath, "IndividualCellSize.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/IndividualCellSize.PNG?raw=true"
     alt="Individual Cell Size example"
     style="width: 400px;" />

---
## Autofitting

You can set the widths of columns to fit their contents using ``AutoFit AllCols``. You can auto fit a range of columns with ``AutoFit (ColRange(<c1>, <c2>))``.

You can autofit heights of rows with ``AutoFit AllRows`` and ``AutoFit (RowRange(<r1>,<r2>))``.

You can autofit all columns *and* all rows with ``AutoFit All``.

Perform ``AutoFit`` operations *after* the cells have been populated!
<!-- Test -->

```fsharp
open System.IO
open System.Globalization
open FsExcel
open ClosedXML.Excel

// For non-Windows runtime environments you will have to add these lines to use AutoFit.
// This is because ClosedXML needs a font to work with when computing sizes. You may have
// to use a different font name if Liberation Sans is not installed on the target system.
open System.Runtime.InteropServices
if not (RuntimeInformation.IsOSPlatform(OSPlatform.Windows)) then
    LoadOptions.DefaultGraphicEngine <- new ClosedXML.Graphics.DefaultGraphicEngine("Liberation Sans") 
//

let headingStyle = 
    [
        Border(Border.Bottom XLBorderStyleValues.Medium)
        FontEmphasis Bold
        FontEmphasis Italic 
    ]

[
    for heading in ["Month"; "Letter Count"] do
        Cell [
            String heading
            yield! headingStyle
        ]
    Go NewRow
    
    for m in 1..12 do
        let monthName = CultureInfo.GetCultureInfoByIetfLanguageTag("en-GB").DateTimeFormat.GetMonthName(m)
        Cell [ String monthName ]
        Cell [ Integer monthName.Length ]
        Go NewRow

    AutoFit AllCols
]
|> Render.AsFile (Path.Combine(savePath, "AutosizeColumns.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/AutosizeColumns.PNG?raw=true"
     alt="Autosize Columns example"
     style="width: 200px;" />

---
## Merging Cells and Vertical Alignment
You can merge cells by using `MergeCells (CellLabel , CellLabel)` where a `CellLabel` can be a:
* specific cell - `ColRowLabel ("<column letter>", <row number>)`
* named cell - `NamedCell "CellName"`
* specified depth and span - `SpanDepth (<column span>, <row depth>)`

Cells can be merged before or after placing text in them, however, as per Excel cell naming convention with merged cells, the following rules must be observed:

* **Merging cells vertically** - text must be assigned to the *top-most cell label* of the merged cell. For example, if there is a vertically merged cell spanning from cell A3 to A6, any text within the range A3 to A6 must be assigned to cell A3.
* **Merging cells horizontally** - text must be assigned to the *left-most cell label* of the merged cell. For example, if there is a horizontally merged cell spanning from cell A2 to E2, any text within the range A2 to E2 must be assigned to cell A2.
* **Merging cells horizontally & vertically** - text must be assigned to the *top-left cell label* of the merged cell. For example, if there is a merged cell spanning from cell A2 to E10, any text within the range A2 to E2 must be assigned to cell A2.

A `SpanDepth` of e.g. (1, 3) creates a merged cell spanning one column and a depth of three rows. The column span and row depth of a merged cell from a starting cell can be specified in two ways:
* **Forward Merging** i.e. merging from top left to bottom right with the starting cell being in the top left hand corner of the merged cell. This method retains cell name, cell contents, cell shading & the top left hand corner of the original starting cell border. This is achieved by having SpanDepth as the second item in the Merge tuple:
    *    `MergeCells ((NamedCell "CellName", SpanDepth (3, 3)))`
    *    `MergeCells ((ColRowLabel ("B", 15), SpanDepth (1, 2)))`
* **Reverse Merging** i.e. merging from bottom right to top left with the starting cell being in the bottom right hand corner of the merged cell. This method loses the cell name, cell contents, cell shading of the original starting cell. However, the bottom right hand corner of the original starting cell border is retained. In the case of starting with e.g cell B2 and requesting a reverse merge SpanDepth of (5, 5), i.e. merging to beyond the excel sheet boundaries, the cell to merge to will be defaulted to cell A1. The Reverse Merging is achieved by having SpanDepth as the first item in the Merge tuple:
    *    `MergeCells ((SpanDepth (3, 3)), NamedCell "CellName")`
    *    `MergeCells ((SpanDepth (1, 2)), (ColRowLabel ("B", 15))`

**Vertical Alignment** for a given cell can be achieved by using `Vertical Alignment [Base, Middle, TopMost]`.

Note that when using two `NamedCell` references in a `MergeCells ()` call, the cell names must differ between the first and second cell.  For example the following construct will not result in merged cells (though it does not cause an error):

```fsharp
[
    Cell [String "Hello"; Name "name"]
    Cell [String "Hello2"; Name "name"]
    MergeCells ((NamedCell "name", NamedCell "name"))
] 
```

Merges which lead to a column reference beyond the maximum supported by Excel ("XFD") will result in an `ArgumentException`.

<!-- Test -->

```fsharp
open System.IO
open System
open ClosedXML.Excel
open FsExcel

[   Go NewRow
    for heading, colWidth in ["ID", 3.22; "Car Name", 10.33; "Car Description", 49.33; "Car Registration", 16.89 ] do
        Cell [
            String heading
            FontEmphasis Bold
            FontName "Calibri"
            FontSize 11
            HorizontalAlignment Center
            FontColor (XLColor.FromArgb(0, 255, 255, 255))
            BackgroundColor (XLColor.FromArgb(0, 68, 114, 196))
            Border(Border.All XLBorderStyleValues.Thin)
            CellSize (ColWidth colWidth)
        ]
    Go NewRow
    Cell [  Integer 1
            HorizontalAlignment Left
            VerticalAlignment TopMost
            Name "ID" ] 
    Cell [  String "Ford Fiesta"
            HorizontalAlignment Center
            VerticalAlignment Middle ] 
    Cell [  String "Car Technical Details:"
            Next (DownBy 1) ]
    Cell [  String "Technical Detail 1"
            Next (DownBy 1) ]
    Cell [  String "Technical Detail 2"
            Next (DownBy 1)]
    Cell [  String "Technical Detail 3"
            Name "LastL" ]
    Go (RC (3, 4))
    Cell [  String "AB12 CDE" 
            HorizontalAlignment Right
            VerticalAlignment Base
            Name "Reg" ]
    Go (RC (6, 4))
    Cell [Name "RegEnd"]
    Go (RC (7, 3))
    Cell [  String "Another Technical Detail"
            FontEmphasis Italic
            VerticalAlignment Middle
            Name "TD" 
            Next Stay]
    Go (DownBy 1)
    Cell [ Name "info"]

    // Merging between named and specific cells
    MergeCells ((ColRowLabel ("B", 3), ColRowLabel ("B", 6)))
    MergeCells ((NamedCell "ID", ColRowLabel ("A", 6)))
    MergeCells ((ColRowLabel ("C", 7), NamedCell "info")) 
    MergeCells ((NamedCell "Reg", NamedCell "RegEnd")) 
    
    Go (RC (10, 1))
    Cell [  String "Merging from a starting cell given a depth and span"
            BackgroundColor (XLColor.FromArgb(0, 80, 180, 220))
            FontEmphasis Bold
            HorizontalAlignment Center ] 
    MergeCells ((ColRowLabel ("A", 10), ColRowLabel ("D", 10)))

    Go (RC (12, 2))
    Cell [  String "The components that make up a car are: "
            Name "components" 
            HorizontalAlignment Left
            VerticalAlignment TopMost
            Border(Border.All XLBorderStyleValues.MediumDashDot)]
    Go (RC (12, 4))
    Cell [ Border(Border.All XLBorderStyleValues.MediumDashDot)]
    Go (RC (14, 4))
    Cell [ Border(Border.All XLBorderStyleValues.MediumDashDot)]

    Go (RC (15, 2))
    Cell [  String "Road Tax"
            HorizontalAlignment Center
            VerticalAlignment Middle
            Border(Border.All XLBorderStyleValues.SlantDashDot)]
    Go (RC (16, 2))
    Cell [ Border(Border.All XLBorderStyleValues.SlantDashDot)]

    // Forward merging - cell name, cell contents, shading & top LH corner of border are retained
    MergeCells ((NamedCell "components", SpanDepth (3, 3)))
    MergeCells ((ColRowLabel ("B", 15), SpanDepth (1, 2))) 

    Go (RC (17, 4))
    Cell [  String "Insurance"
            Name "insurance" // NamedCells cannot begin with a number
            Border(Border.All XLBorderStyleValues.Dashed) ]
    Go (RC (17, 3))
    Cell [ Border(Border.All XLBorderStyleValues.Dashed)]
    Go (RC (17, 2))
    Cell [ Border(Border.All XLBorderStyleValues.Dashed)] 
   
    Go (RC (16, 4))
    Cell [  String "Signature"]

    // Reverse Merging - original cell contents, cell name and cell shading are lost
    // Only bottom RH corner of the border is retained
    MergeCells ((SpanDepth (3, 1), NamedCell "insurance")) 
    MergeCells ((SpanDepth (2, 2), ColRowLabel ("D", 16))) 
]
|> Render.AsFile (Path.Combine(savePath, "MergeCellsWithVerticalAlignment.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/MergedCellWithVerticalAlignment.PNG?raw=true"
     alt="Merged Cell with Vertical Alignment example"
     style="width: 800px;" />

---
## Tables from Records

You can create a table of cells from an F# record or a sequence of F# records.

*The tables created using this approach are not proper Excel tables, simply grids of cells with potentially a little formatting. To create true Excel tables, see [Excel Table Tutorial](https://github.com/misterspeedy/FsExcel/blob/main/ExcelTableTutorial.md).*

Use `Table.fromInstance` or `Table.fromSeq` and provide

- an orientation (`Table.Direction.Horizontal` or `Table.Direction.Vertical`)
- a function which, given an index and a field name, returns a list of properties for styling. (This style can be an empty list.)
- the instance or sequence.

In horizontal tables, the values for each record appear beside one another.  In vertical tables the values for a record appear below one another.

Calls to the cell style function are given 0 for the header, 1 for the first (or only) data row, 2 for the next and so on.

Tables don't automatically autofit - you'll have to do that (if you want) after the table is built.

Regardless of table orientation, the 'current cell' (i.e. the address at which any further new cell is rendered) is always just below the bottom-left corner of the table that was just created.
<!-- Test -->

```fsharp
open System
open System.IO
open ClosedXML.Excel
open FsExcel

type JoiningInfo = {
    Name : string
    Age : int
    Fees : decimal
    DateJoined : string
}

// This works just as well if these are anonymous record instances,
// eg. {| Name = "..."; ... |}

let records = [
    { Name = "Jane Smith"; Age = 32; Fees = 59.25m; DateJoined = "2022-03-12" } // Excel will treat these strings as dates
    { Name = "Michael Nguyễn"; Age = 23; Fees = 61.2m; DateJoined = "2022-03-13" }
    { Name = "Sofia Hernández"; Age = 58; Fees = 59.25m; DateJoined = "2022-03-15" }
]

let cellStyleVertical index name =
    if index = 0 then
        [ FontEmphasis Bold ]
    elif name = "Fees" then
        [ FormatCode "$0.00" ]
    else
        []

let cellStyleHorizontal index name =
    if index = 0 then
        [
            Border(Border.Bottom XLBorderStyleValues.Medium)
            FontEmphasis Bold
        ]
    elif name = "Fees" then
        [ FormatCode "$0.00" ]
    else
        []

records
|> Table.fromSeq Table.Direction.Vertical cellStyleVertical
|> fun cells -> cells @ [ AutoFit All ]
|> Render.AsFile (Path.Combine(savePath, "RecordSequenceVertical.xlsx"))

records
|> Table.fromSeq Table.Direction.Horizontal cellStyleHorizontal
|> fun cells -> cells @ [ AutoFit All ]
|> Render.AsFile (Path.Combine(savePath, "RecordSequenceHorizontal.xlsx"))

records
|> Seq.tryHead
|> Option.iter (fun r ->

    r 
    |> Table.fromInstance Table.Direction.Vertical cellStyleVertical
    |> fun cells -> cells @ [ AutoFit All ]
    |> Render.AsFile (Path.Combine(savePath, "RecordInstanceVertical.xlsx"))

    r 
    |> Table.fromInstance Table.Direction.Horizontal cellStyleHorizontal
    |> fun cells -> cells @ [ AutoFit All ]
    |> Render.AsFile (Path.Combine(savePath, "RecordInstanceHorizontal.xlsx")))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/RecordSequenceVertical.PNG?raw=true"
     alt="Table example - vertical record sequence"
     style="width: 450px;" />

<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/RecordSequenceHorizontal.PNG?raw=true"
     alt="Table example - horizontal record sequence"
     style="width: 320px;" />

<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/RecordInstanceVertical.PNG?raw=true"
     alt="Table example - vertical record instance"
     style="width: 200px;" />
     
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/RecordInstanceHorizontal.PNG?raw=true"
     alt="Table example - horizontal record instance"
     style="width: 280px;" />

---
## Rendering in Fable Elmish and similar web applications

You can use `Render.AsStream <stream> <items>` to render to a pre-existing stream, or `Render.AsStreamBytes <items>` to render as a byte array. 

`Render.AsStreamBytes` is useful for Fable-based and other web app scenarios. Render to a byte array on the server, and transfer the bytes to the client using Fable Remoting.  On the client use the `SaveFileAs` extension function to start a browser download.  Make sure you have opened the `Fable.Remoting.Client` to get the `SaveFileAs` method of a byte array.

For a working example, see http://www.pushbuttonreceivetables.com/, in particular https://github.com/misterspeedy/HtmlExcel/blob/main/src/Server/Html.fs#L105.

```fsharp
open FsExcel

[
    Cell [ String "Hello world!" ]
]
|> Render.AsStreamBytes
|> fun bytes ->
    $"Bytes length: {bytes.Length}"

```
## Data Types

FsExcel supports the following data types for cell content:

- String
- Integer
- Float
- Boolean
- DateTime
- TimeSpan
<!-- Test -->

```fsharp
open System
open System.IO
open FsExcel

[
    Cell [ String "String"]; Cell [ String "string" ]
    Go NewRow
    Cell [ String "Integer" ]; Cell [ Integer 42 ]
    Go NewRow
    Cell [ String "Number" ]; Cell [ Float Math.PI ]
    Go NewRow
    Cell [ String "Boolean" ]; Cell [ Boolean false  ]
    Go NewRow
    Cell [ String "DateTime" ]; Cell [ DateTime (System.DateTime(1903, 12, 17)) ]
    Go NewRow
    Cell [ String "TimeSpan" ]
    Cell [ 
        TimeSpan (System.TimeSpan(hours=1, minutes=2, seconds=3)) 
        FormatCode "hh:mm:ss"
    ]
]
|> Render.AsFile (Path.Combine(savePath, "DataTypes.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/DataTypes.PNG?raw=true"
     alt="Data Types example"
     style="width: 200px;" />

---
## Rendering as HTML

You can render a workbook as a set of HTML tables. You will get one table per worksheet.

This feature is primarily for use in Dotnet Interactive Notebooks, where you can use the `HTML()` helper method to display the resulting HTML. This can be useful when experimenting with cell layouts, to avoid having to view an Excel file on every iteration.

The styling representation is approximate:

- Bold and italic font emphasis should show correctly. (Note that VS Code does not default to representing `<th>` items in bold)
- Underlining, where present, will always be be shown as a single underline.
- Cell borders, where present, will always be a single line. (Note that VS Code does not yet show borders on tables.)
- Font names, sizes, cell alignment and any kind of color are not currently supported.

The `AsHtml` function takes a function parameter which is called for every cell rendered, with a row and column index (both zero based, originating from the top-left-most occupied cell). When this function returns true, the cell is rendered as `<th>`, otherwise it is rendered as `<td>`.

```fsharp
open System
open System.IO
open FsExcel
open ClosedXML.Excel

let isHeader r c =
    r = 0 || c = 0

[
    Worksheet "Worksheet 1"

    Style [ FontEmphasis Bold ]
    Cell [ String "Item" ]
    Cell [ String "Example" ]
    Style []
    Go NewRow

    Cell [ String "String"]
    Cell [ String "string" ]
    Go NewRow

    Cell [ String "Integer" ]
    Cell [ Integer 42 ]
    Go NewRow
    
    Cell [ String "Number" ]
    Cell [ Float Math.PI ]
    Go NewRow
    
    Cell [ String "Boolean" ]
    Cell [ Boolean false  ]
    Go NewRow

    Cell [ String "DateTime" ]
    Cell [ DateTime (System.DateTime(1903, 12, 17)) ]
    Go NewRow

    Cell [ String "TimeSpan" ]
    Cell [ 
        TimeSpan (System.TimeSpan(hours=1, minutes=2, seconds=3)) 
        FormatCode "hh:mm:ss"
    ]
    Go NewRow

    Cell [ String "Bold" ]
    Cell [
        String "I am bold"
        FontEmphasis Bold
    ]
    Go NewRow

    Cell [ String "Italic" ]
    Cell [
        String "I am Italic"
        FontEmphasis Italic
    ]
    Go NewRow

    Cell [ String "Underlined" ]
    Cell [
        String "I am underlined"
        FontEmphasis (Underline XLFontUnderlineValues.Single)
    ]
    Go NewRow

    Worksheet "Worksheet 2"
    Cell [String "I am another table"]
]
|> Render.AsHtml isHeader
|> HTML

```
## AutoFilter

You can add filters to a WorkSheet.

* Enable Only: Enables but does not apply an AutoFilter.
* Apply filter: Enables and applies an AutoFilter.
* Clear filter: Clears an AutoFilter.

### AutoFilterRange

There can be multiple `AutoFilters` on a given worksheet. This means that the area to be filtered has to be specified when defining the filter. This is done with `AutoFilterRange`.

* `RangeUsed`: The entire range used in the worksheet.
* `CurrentRegion` of string: The current region around a spcified cell.
* `Range` of string: A specified range.

Examples:

```F#
AutoFilter [ EnableOnly RangeUsed ]

AutoFilter [ EnableOnly CurrentRegion ]

AutoFilter [ GreaterThanInt ("A1:E6", 2, 3) ]
```

### List of available filters

```F#
EnableOnly of AutoFilterRange
Clear of AutoFilterRange

EqualToString of range : AutoFilterRange * column : int * value : string
EqualToInt of range : AutoFilterRange * column : int * value : int
EqualToFloat of range : AutoFilterRange * column : int * value : float
EqualToDateTime of range : AutoFilterRange * column : int * value : DateTime
EqualToBool of range : AutoFilterRange * column : int * value : bool

NotEqualToString of range : AutoFilterRange * column : int * value : string
NotEqualToInt of range : AutoFilterRange * column : int * value : int
NotEqualToFloat of range : AutoFilterRange * column : int * value : float
NotEqualToDateTime of range : AutoFilterRange * column : int * value : DateTime
NotEqualToBool of range : AutoFilterRange * column : int * value : bool

BetweenInt of range : AutoFilterRange * column : int * min : int * max : int
BetweenFloat of range : AutoFilterRange * column : int * min : float * max : float
BetweenDateTime of range : AutoFilterRange * column : int * min : DateTime * max : DateTime

NotBetweenInt of range : AutoFilterRange * column : int * min : int * max : int
NotBetweenFloat of range : AutoFilterRange * column : int * min : float * max : float
NotBetweenDateTime of range : AutoFilterRange * column : int * min : DateTime * max : DateTime

ContainsString of range : AutoFilterRange * column : int * value : string
NotContainsString of range : AutoFilterRange * column : int * value : string

BeginsWithString of range : AutoFilterRange * column : int * value : string
NotBeginsWithString of range : AutoFilterRange * column : int * value : string

EndsWithString of range : AutoFilterRange * column : int * value : string
NotEndsWithString of range : AutoFilterRange * column : int * value : string

Top of range : AutoFilterRange * column : int * value : int * topType : XLTopBottomType
Bottom of range : AutoFilterRange * column : int * value : int * bottomType : XLTopBottomType

GreaterThanInt of range : AutoFilterRange * column : int * value : int
GreaterThanFloat of range : AutoFilterRange * column : int * value : float
GreaterThanDateTime of range : AutoFilterRange * column : int * value : DateTime

LessThanInt of range : AutoFilterRange * column : int * value : int
LessThanFloat of range : AutoFilterRange * column : int * value : float
LessThanDateTime of range : AutoFilterRange * column : int * value : DateTime

EqualOrGreaterThanInt of range : AutoFilterRange * column : int * value : int
EqualOrGreaterThanFloat of range : AutoFilterRange * column : int * value : float
EqualOrGreaterThanDateTime of range : AutoFilterRange * column : int * value : DateTime

EqualOrLessThanInt of range : AutoFilterRange * column : int * value : int
EqualOrLessThanFloat of range : AutoFilterRange * column : int * value : float
EqualOrLessThanDateTime of range : AutoFilterRange * column : int * value : DateTime

AboveAverage of range : AutoFilterRange * column : int
BelowAverage of range : AutoFilterRange * column : int
```

### Known Issues

EqualToDateTime:
> Works but, both Equals and Custom Filter are blank.

NotEqualToDateTime:
> Does not work. Does contains. Should be not contains.

BetweenDateTime
> Does not work. Excel filter shows 07/01/1900. Reapply hides all rows.

NotBetweenDateTime

> Works but, shows as a Custom filter with 07/01/1900 in Excel.

NotContains

> Works but, shows as a Contains filter in Excel. Reapply does Contains.

GreaterThanDateTime

> Works but, filter name is After and shows 07/01/1900.

LessThanDateTime

> Works but, filter name is Before and shows 07/01/1900.

EqualOrGreaterThanDateTime

> Works but, filter name is Custom Filter and shows 07/01/1900.

EqualOrLessThanDateTime

> Works but, filter name is Custom Filter and shows 07/01/1900.

<br />

Some of the above issues may be related to one of these:

* [Setting AutoFilter EqualTo on Date Column Doesn't Display Values When Spreadsheet Is Opened Until Filters Are Reapplied #701](https://github.com/ClosedXML/ClosedXML/issues/701)

* [Text to number coercion doesn't work correctly #1891](https://github.com/ClosedXML/ClosedXML/issues/1891)

---
### Enable Only

In the example below and `AutoFilter` is enabled for the `RangeUsed`, but no filter is applied.

<!-- Test -->

```fsharp
open System
open System.IO
open FsExcel

let headings =
    [ Cell [ String "StringCol"; HorizontalAlignment Center ]
      Cell [ String "IntCol"; HorizontalAlignment Center ]
      Cell [ String "FloatCol"; HorizontalAlignment Center ]
      Cell [ String "DateTimeCol"; HorizontalAlignment Center ]
      Cell [ String "BooleanCol"; HorizontalAlignment Center ]
      Go NewRow ]

let rows =
    [ 1 .. 5 ]
    |> Seq.map(fun i ->
        [ Cell [ String $"String{i}" ]
          Cell [ Integer i ]
          Cell [ Float ((i |> float) + 0.1) ]
          Cell [ DateTime (DateTime.Parse("15-July-2017 05:33:00").AddMinutes(i)) ]
          Cell [ Boolean (i % 2 |> Convert.ToBoolean) ]
          Go NewRow ])
    |> Seq.collect id
    |> List.ofSeq

headings @ rows @ [ AutoFit All; AutoFilter [ EnableOnly RangeUsed ] ]
|> Render.AsFile (Path.Combine(savePath, "AutoFilterEnableOnly.xlsx"))

```
<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/AutoFilterEnableOnly.PNG?raw=true"
     alt="AutoFilter Enable Only example"
     style="width: 500px;" />

---
### Apply AutoFilter

In the example below `AutoFilter` is enabled (this is automatic if you create a filter).

The following compound filter is created:

* `RangeUsed`, column 2 is filtered for greater than 3
* and `RangeUsed`, column 5 is filtered to equal `true`

<!-- Test -->

```fsharp
open System
open System.IO
open FsExcel

let headings =
    [ Cell [ String "StringCol"; HorizontalAlignment Center ]
      Cell [ String "IntCol"; HorizontalAlignment Center ]
      Cell [ String "FloatCol"; HorizontalAlignment Center ]
      Cell [ String "DateTimeCol"; HorizontalAlignment Center ]
      Cell [ String "BooleanCol"; HorizontalAlignment Center ]
      Go NewRow ]

let rows =
    [ 1 .. 5 ]
    |> Seq.map(fun i ->
        [ Cell [ String $"String{i}" ]
          Cell [ Integer i ]
          Cell [ Float ((i |> float) + 0.1) ]
          Cell [ DateTime (DateTime.Parse("15-July-2017 05:33:00").AddMinutes(i)) ]
          Cell [ Boolean (i % 2 |> Convert.ToBoolean) ]
          Go NewRow ])
    |> Seq.collect id
    |> List.ofSeq

headings @ rows @ [ AutoFit All; AutoFilter [ GreaterThanInt (RangeUsed, 2, 3); EqualToBool (RangeUsed, 5, true) ] ]
|> Render.AsFile (Path.Combine(savePath, "AutoFilterCompound.xlsx"))

```
No AutoFilter:

<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/AutoFilterBefore.PNG?raw=true"
     alt="AutoFilter Enable Only example"
     style="width: 500px;" />

AutoFilter applied:

<img src="https://github.com/misterspeedy/FsExcel/blob/main/assets/AutoFilterAfter.PNG?raw=true"
     alt="AutoFilter example"
     style="width: 500px;" />

---
## Excel Tables

To create [Excel Tables](https://support.microsoft.com/en-us/office/overview-of-excel-tables-7ab0bb7d-3a9e-4b56-a3c9-6c94334e492c), see the separate [Excel Table Tutorial](https://github.com/misterspeedy/FsExcel/blob/main/ExcelTableTutorial.md).
