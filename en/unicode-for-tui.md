# Unicode Characters for TUI

A comprehensive reference of frequently used Unicode characters for building Text User Interfaces (TUI). This guide covers box drawing, block elements, arrows, symbols, and other characters commonly used in terminal applications.

## Quick Reference

### Essential Box Drawing

```
┌─┬─┐  ╔═╦═╗  ╭─┬─╮
│ │ │  ║ ║ ║  │ │ │
├─┼─┤  ╠═╬═╣  ├─┼─┤
│ │ │  ║ ║ ║  │ │ │
└─┴─┘  ╚═╩═╝  ╰─┴─╯
Single  Double Rounded
```

### Progress Bars

```
▱▱▱▱▱▱▱▱▱▱ 0%
█▱▱▱▱▱▱▱▱▱ 10%
██████████ 100%

░░░░░░░░░░ 0%
▓▓▓▓▓░░░░░ 50%
▓▓▓▓▓▓▓▓▓▓ 100%
```

### Common Symbols

```
✓ ✗ ✔ ✘    Check marks
▶ ▷ ► ▸    Play/arrows
● ○ ◉ ◎    Bullets
⚠ ⚡ ★ ☆    Icons
```

## Box Drawing Characters

### Light (Single Line)

**Horizontal and Vertical**:
```
─  U+2500  Box Drawings Light Horizontal
│  U+2502  Box Drawings Light Vertical
```

**Corners**:
```
┌  U+250C  Box Drawings Light Down and Right
┐  U+2510  Box Drawings Light Down and Left
└  U+2514  Box Drawings Light Up and Right
┘  U+2518  Box Drawings Light Up and Left
```

**T-junctions**:
```
├  U+251C  Box Drawings Light Vertical and Right
┤  U+2524  Box Drawings Light Vertical and Left
┬  U+252C  Box Drawings Light Down and Horizontal
┴  U+2534  Box Drawings Light Up and Horizontal
```

**Cross**:
```
┼  U+253C  Box Drawings Light Vertical and Horizontal
```

**Complete Light Box Set**:
```
┌───┬───┐
│   │   │
├───┼───┤
│   │   │
└───┴───┘
```

### Heavy (Thick Line)

**Horizontal and Vertical**:
```
━  U+2501  Box Drawings Heavy Horizontal
┃  U+2503  Box Drawings Heavy Vertical
```

**Corners**:
```
┏  U+250F  Box Drawings Heavy Down and Right
┓  U+2513  Box Drawings Heavy Down and Left
┗  U+2517  Box Drawings Heavy Up and Right
┛  U+251B  Box Drawings Heavy Up and Left
```

**T-junctions**:
```
┣  U+2523  Box Drawings Heavy Vertical and Right
┫  U+252B  Box Drawings Heavy Vertical and Left
┳  U+2533  Box Drawings Heavy Down and Horizontal
┻  U+253B  Box Drawings Heavy Up and Horizontal
```

**Cross**:
```
╋  U+254B  Box Drawings Heavy Vertical and Horizontal
```

**Complete Heavy Box Set**:
```
┏━━━┳━━━┓
┃   ┃   ┃
┣━━━╋━━━┫
┃   ┃   ┃
┗━━━┻━━━┛
```

### Double Line

**Horizontal and Vertical**:
```
═  U+2550  Box Drawings Double Horizontal
║  U+2551  Box Drawings Double Vertical
```

**Corners**:
```
╔  U+2554  Box Drawings Double Down and Right
╗  U+2557  Box Drawings Double Down and Left
╚  U+255A  Box Drawings Double Up and Right
╝  U+255D  Box Drawings Double Up and Left
```

**T-junctions**:
```
╠  U+2560  Box Drawings Double Vertical and Right
╣  U+2563  Box Drawings Double Vertical and Left
╦  U+2566  Box Drawings Double Down and Horizontal
╩  U+2569  Box Drawings Double Up and Horizontal
```

**Cross**:
```
╬  U+256C  Box Drawings Double Vertical and Horizontal
```

**Complete Double Box Set**:
```
╔═══╦═══╗
║   ║   ║
╠═══╬═══╣
║   ║   ║
╚═══╩═══╝
```

### Rounded Corners

**Corners**:
```
╭  U+256D  Box Drawings Light Arc Down and Right
╮  U+256E  Box Drawings Light Arc Down and Left
╰  U+2570  Box Drawings Light Arc Up and Right
╯  U+2571  Box Drawings Light Arc Up and Left
```

**Complete Rounded Box**:
```
╭───┬───╮
│   │   │
├───┼───┤
│   │   │
╰───┴───╯
```

### Mixed Light and Heavy

**Vertical Light, Horizontal Heavy**:
```
┍  U+250D  Down Light and Right Heavy
┑  U+2511  Down Light and Left Heavy
┕  U+2515  Up Light and Right Heavy
┙  U+2519  Up Light and Left Heavy
┝  U+251D  Vertical Light and Right Heavy
┥  U+2525  Vertical Light and Left Heavy
┯  U+252F  Down Light and Horizontal Heavy
┷  U+2537  Up Light and Horizontal Heavy
┿  U+253F  Vertical Light and Horizontal Heavy
```

**Vertical Heavy, Horizontal Light**:
```
┎  U+250E  Down Heavy and Right Light
┒  U+2512  Down Heavy and Left Light
┖  U+2516  Up Heavy and Right Light
┚  U+251A  Up Heavy and Left Light
┞  U+251E  Vertical Heavy and Right Light
┦  U+2526  Vertical Heavy and Left Light
┰  U+2530  Down Heavy and Horizontal Light
┸  U+2538  Up Heavy and Horizontal Light
╂  U+2542  Vertical Heavy and Horizontal Light
```

## Block Elements

### Full and Partial Blocks

**Full Block**:
```
█  U+2588  Full Block
```

**Partial Vertical Blocks** (left to right):
```
▏  U+258F  Left One Eighth Block
▎  U+258E  Left One Quarter Block
▍  U+258D  Left Three Eighths Block
▌  U+258C  Left Half Block
▋  U+258B  Left Five Eighths Block
▊  U+258A  Left Three Quarters Block
▉  U+2589  Left Seven Eighths Block
```

**Partial Horizontal Blocks** (top to bottom):
```
▔  U+2594  Upper One Eighth Block
▀  U+2580  Upper Half Block
▄  U+2584  Lower Half Block
▁  U+2581  Lower One Eighth Block
▂  U+2582  Lower One Quarter Block
▃  U+2583  Lower Three Eighths Block
▄  U+2584  Lower Half Block
▅  U+2585  Lower Five Eighths Block
▆  U+2586  Lower Three Quarters Block
▇  U+2587  Lower Seven Eighths Block
```

**Quadrants**:
```
▘  U+2598  Quadrant Upper Left
▝  U+259D  Quadrant Upper Right
▖  U+2596  Quadrant Lower Left
▗  U+2597  Quadrant Lower Right
▚  U+259A  Quadrant Upper Left and Lower Right
▞  U+259E  Quadrant Upper Right and Lower Left
```

**Shades**:
```
░  U+2591  Light Shade (25%)
▒  U+2592  Medium Shade (50%)
▓  U+2593  Dark Shade (75%)
```

### Usage Examples

**Progress Bar (Fine-grained)**:
```
▏░░░░░░░░░  1%
▎░░░░░░░░░  2%
▍░░░░░░░░░  3%
▌░░░░░░░░░  5%
█▌░░░░░░░░  15%
████▌░░░░░  45%
█████████▉  99%
```

**Volume/Level Indicator**:
```
▁▂▃▄▅▆▇█
```

**Sparklines**:
```
▁▂▃▅▂▇▆▃▅▆
```

## Arrows

### Basic Arrows

**Cardinal Directions**:
```
←  U+2190  Leftwards Arrow
→  U+2192  Rightwards Arrow
↑  U+2191  Upwards Arrow
↓  U+2193  Downwards Arrow
```

**Diagonal**:
```
↖  U+2196  North West Arrow
↗  U+2197  North East Arrow
↘  U+2198  South East Arrow
↙  U+2199  South West Arrow
```

**Double Arrows**:
```
⇐  U+21D0  Leftwards Double Arrow
⇒  U+21D2  Rightwards Double Arrow
⇑  U+21D1  Upwards Double Arrow
⇓  U+21D3  Downwards Double Arrow
⇔  U+21D4  Left Right Double Arrow
⇕  U+21D5  Up Down Double Arrow
```

**Long Arrows**:
```
⟵  U+27F5  Long Leftwards Arrow
⟶  U+27F6  Long Rightwards Arrow
⟷  U+27F7  Long Left Right Arrow
```

### Triangle Arrows

**Filled**:
```
◀  U+25C0  Black Left-Pointing Triangle
▶  U+25B6  Black Right-Pointing Triangle
▲  U+25B2  Black Up-Pointing Triangle
▼  U+25BC  Black Down-Pointing Triangle
```

**Hollow**:
```
◁  U+25C1  White Left-Pointing Triangle
▷  U+25B7  White Right-Pointing Triangle
△  U+25B3  White Up-Pointing Triangle
▽  U+25BD  White Down-Pointing Triangle
```

**Small**:
```
◂  U+25C2  Black Left-Pointing Small Triangle
▸  U+25B8  Black Right-Pointing Small Triangle
▴  U+25B4  Black Up-Pointing Small Triangle
▾  U+25BE  Black Down-Pointing Small Triangle
```

## Bullets and Markers

### Circles

```
●  U+25CF  Black Circle
○  U+25CB  White Circle
◉  U+25C9  Fisheye
◎  U+25CE  Bullseye
⦿  U+29BF  Circled Bullet
⊙  U+2299  Circled Dot Operator
⊚  U+229A  Circled Ring Operator
```

**Small Circles**:
```
•  U+2022  Bullet
‣  U+2023  Triangular Bullet
⋅  U+22C5  Dot Operator
∙  U+2219  Bullet Operator
⸰  U+2E30  Doubled Dot Punctuation
```

### Squares

```
■  U+25A0  Black Square
□  U+25A1  White Square
▪  U+25AA  Black Small Square
▫  U+25AB  White Small Square
◼  U+25FC  Black Medium Square
◻  U+25FB  White Medium Square
◾  U+25FE  Black Medium Small Square
◽  U+25FD  White Medium Small Square
```

### Diamonds

```
◆  U+25C6  Black Diamond
◇  U+25C7  White Diamond
◈  U+25C8  White Diamond Containing Black Small Diamond
⬥  U+2B25  Black Medium Diamond
⬦  U+2B26  White Medium Diamond
```

### Stars

```
★  U+2605  Black Star
☆  U+2606  White Star
✦  U+2726  Black Four Pointed Star
✧  U+2727  White Four Pointed Star
⭐  U+2B50  White Medium Star
```

## Check Marks and Crosses

### Check Marks

```
✓  U+2713  Check Mark
✔  U+2714  Heavy Check Mark
✅  U+2705  White Heavy Check Mark
☑  U+2611  Ballot Box with Check
🗸  U+1F5F8  Light Check Mark
```

### Crosses

```
✕  U+2715  Multiplication X
✖  U+2716  Heavy Multiplication X
✗  U+2717  Ballot X
✘  U+2718  Heavy Ballot X
❌  U+274C  Cross Mark
❎  U+274E  Negative Squared Cross Mark
☒  U+2612  Ballot Box with X
```

### Other Marks

```
‼  U+203C  Double Exclamation Mark
⁉  U+2049  Exclamation Question Mark
⚠  U+26A0  Warning Sign
⛔  U+26D4  No Entry
🚫  U+1F6AB  No Entry Sign
```

## Special Symbols

### Status Indicators

```
⏸  U+23F8  Pause
⏵  U+23F5  Play
⏹  U+23F9  Stop
⏺  U+23FA  Record
⏭  U+23ED  Next Track
⏮  U+23EE  Last Track
⏩  U+23E9  Fast Forward
⏪  U+23EA  Rewind
```

### UI Elements

**Brackets and Quotes**:
```
❮  U+276E  Heavy Left-Pointing Angle Quotation Mark
❯  U+276F  Heavy Right-Pointing Angle Quotation Mark
❰  U+2770  Heavy Left-Pointing Angle Bracket
❱  U+2771  Heavy Right-Pointing Angle Bracket
⟨  U+27E8  Mathematical Left Angle Bracket
⟩  U+27E9  Mathematical Right Angle Bracket
```

**Folder/File Icons**:
```
📁  U+1F4C1  File Folder
📂  U+1F4C2  Open File Folder
📄  U+1F4C4  Page Facing Up
📝  U+1F4DD  Memo
🗀  U+1F5C0  Folder
🗁  U+1F5C1  Open Folder
🗂  U+1F5C2  Card Index Dividers
```

**Misc UI**:
```
⚙  U+2699  Gear
🔧  U+1F527  Wrench
🔍  U+1F50D  Left-Pointing Magnifying Glass
🔎  U+1F50E  Right-Pointing Magnifying Glass
🔗  U+1F517  Link Symbol
⚡  U+26A1  High Voltage
🔥  U+1F525  Fire
💾  U+1F4BE  Floppy Disk
```

### Weather and Time

```
☀  U+2600  Black Sun with Rays
☁  U+2601  Cloud
☂  U+2602  Umbrella
☃  U+2603  Snowman
⛈  U+26C8  Thunder Cloud and Rain
🌙  U+1F319  Crescent Moon
⏰  U+23F0  Alarm Clock
⏱  U+23F1  Stopwatch
⏲  U+23F2  Timer Clock
⌛  U+231B  Hourglass
```

## Spinner Characters

### Line Spinners

**Basic Line** (`|/-\`):
```
|  U+007C  Vertical Line
/  U+002F  Solidus
-  U+002D  Hyphen-Minus
\  U+005C  Reverse Solidus
```

**Box Drawing Spinner**:
```
─  U+2500  Box Drawings Light Horizontal
\\  U+005C  Reverse Solidus
│  U+2502  Box Drawings Light Vertical
/  U+002F  Solidus
```

### Circle Spinners

**Dots**:
```
⠁  U+2801  Braille Pattern Dots-1
⠂  U+2802  Braille Pattern Dots-2
⠄  U+2804  Braille Pattern Dots-3
⡀  U+2840  Braille Pattern Dots-7
⢀  U+2880  Braille Pattern Dots-8
⠠  U+2820  Braille Pattern Dots-6
⠐  U+2810  Braille Pattern Dots-5
⠈  U+2808  Braille Pattern Dots-4
```

**Circle Quarters**:
```
◴  U+25F4  White Circle with Upper Left Quadrant
◷  U+25F7  White Circle with Upper Right Quadrant
◶  U+25F6  White Circle with Lower Right Quadrant
◵  U+25F5  White Circle with Lower Left Quadrant
```

**Arc Spinners**:
```
◜  U+25DC  Upper Left Quadrant Circular Arc
◝  U+25DD  Upper Right Quadrant Circular Arc
◞  U+25DE  Lower Right Quadrant Circular Arc
◟  U+25DF  Lower Left Quadrant Circular Arc
```

**Moon Phases**:
```
🌑  U+1F311  New Moon
🌒  U+1F312  Waxing Crescent Moon
🌓  U+1F313  First Quarter Moon
🌔  U+1F314  Waxing Gibbous Moon
🌕  U+1F315  Full Moon
🌖  U+1F316  Waning Gibbous Moon
🌗  U+1F317  Last Quarter Moon
🌘  U+1F318  Waning Crescent Moon
```

## Braille Patterns

Braille patterns (U+2800 to U+28FF) can create custom graphics and spinners.

**Vertical Progress**:
```
⠀  U+2800  Braille Pattern Blank
⠁  U+2801  Braille Pattern Dots-1
⠃  U+2803  Braille Pattern Dots-12
⠇  U+2807  Braille Pattern Dots-123
⠏  U+280F  Braille Pattern Dots-1234
⠟  U+281F  Braille Pattern Dots-12345
⠿  U+283F  Braille Pattern Dots-123456
⡿  U+287F  Braille Pattern Dots-1234567
⣿  U+28FF  Braille Pattern Dots-12345678
```

**Horizontal Progress**:
```
⡀  U+2840  Braille Pattern Dots-7
⣀  U+28C0  Braille Pattern Dots-78
⣄  U+28C4  Braille Pattern Dots-378
⣤  U+28E4  Braille Pattern Dots-3678
⣦  U+28E6  Braille Pattern Dots-23678
⣶  U+28F6  Braille Pattern Dots-234678
⣷  U+28F7  Braille Pattern Dots-1234678
⣿  U+28FF  Braille Pattern Dots-12345678
```

## Line Drawing

### Diagonal Lines

```
╱  U+2571  Box Drawings Light Diagonal Upper Right to Lower Left
╲  U+2572  Box Drawings Light Diagonal Upper Left to Lower Right
╳  U+2573  Box Drawings Light Diagonal Cross
```

### Dashed and Dotted Lines

**Horizontal**:
```
╌  U+254C  Box Drawings Light Double Dash Horizontal
╍  U+254D  Box Drawings Heavy Double Dash Horizontal
┄  U+2504  Box Drawings Light Triple Dash Horizontal
┅  U+2505  Box Drawings Heavy Triple Dash Horizontal
┈  U+2508  Box Drawings Light Quadruple Dash Horizontal
┉  U+2509  Box Drawings Heavy Quadruple Dash Horizontal
```

**Vertical**:
```
╎  U+254E  Box Drawings Light Double Dash Vertical
╏  U+254F  Box Drawings Heavy Double Dash Vertical
┆  U+2506  Box Drawings Light Triple Dash Vertical
┇  U+2507  Box Drawings Heavy Triple Dash Vertical
┊  U+250A  Box Drawings Light Quadruple Dash Vertical
┋  U+250B  Box Drawings Heavy Quadruple Dash Vertical
```

## Common TUI Patterns

### Window Borders

**Basic Window**:
```
┌─────────────────┐
│ Window Title    │
├─────────────────┤
│                 │
│     Content     │
│                 │
└─────────────────┘
```

**Fancy Window**:
```
╔═════════════════╗
║ Window Title    ║
╠═════════════════╣
║                 ║
║     Content     ║
║                 ║
╚═════════════════╝
```

**Rounded Window**:
```
╭─────────────────╮
│ Window Title    │
├─────────────────┤
│                 │
│     Content     │
│                 │
╰─────────────────╯
```

### Tables

**Simple Table**:
```
┌──────┬──────┬──────┐
│ Col1 │ Col2 │ Col3 │
├──────┼──────┼──────┤
│ A    │ B    │ C    │
│ D    │ E    │ F    │
└──────┴──────┴──────┘
```

**Heavy Headers**:
```
┏━━━━━━┳━━━━━━┳━━━━━━┓
┃ Col1 ┃ Col2 ┃ Col3 ┃
┣━━━━━━╋━━━━━━╋━━━━━━┫
┃ A    ┃ B    ┃ C    ┃
┃ D    ┃ E    ┃ F    ┃
┗━━━━━━┻━━━━━━┻━━━━━━┛
```

### Menus and Lists

**Checkbox List**:
```
☐ Unchecked item
☑ Checked item
☒ Checked with X
□ White square box
■ Black square box
```

**Radio Buttons**:
```
○ Unselected option
● Selected option
◉ Selected with dot
```

**Tree View**:
```
├─ Item 1
│  ├─ Subitem 1.1
│  └─ Subitem 1.2
├─ Item 2
└─ Item 3
   ├─ Subitem 3.1
   └─ Subitem 3.2
```

### Progress Indicators

**Simple Progress**:
```
[████████░░] 80%
```

**Fancy Progress**:
```
▓▓▓▓▓▓▓▓░░ 80%
```

**Block Progress**:
```
█████████▉ 99%
```

**Percentage Bar**:
```
│████████████████████          │ 67%
```

### Panels and Sections

**Nested Panels**:
```
┌─────────────────────────┐
│ Outer Panel             │
│ ┌─────────────────────┐ │
│ │ Inner Panel         │ │
│ │                     │ │
│ └─────────────────────┘ │
└─────────────────────────┘
```

**Side-by-side**:
```
┌──────────┬──────────┐
│ Left     │ Right    │
│          │          │
│          │          │
└──────────┴──────────┘
```

## Best Practices

### Cross-Platform Compatibility

1. **Test on multiple terminals**: Different terminals support different character sets
2. **Provide ASCII fallbacks**: For terminals without Unicode support
3. **Use common characters**: Stick to well-supported Unicode blocks
4. **Check font support**: Not all fonts include all Unicode characters

### Performance Considerations

1. **Cache character lookups**: Store frequently used characters
2. **Batch updates**: Update screen in chunks rather than character-by-character
3. **Use terminal capabilities**: Leverage terminal-specific optimizations

### Accessibility

1. **Don't rely solely on symbols**: Provide text alternatives
2. **Support screen readers**: Ensure symbols have proper descriptions
3. **Consider color blindness**: Don't use color as the only indicator
4. **Maintain readability**: Choose clear, distinguishable characters

### Common Issues

**Character Width**:
- Most box drawing characters are single-width
- Some emoji are double-width
- East Asian characters may vary in width

**Font Fallback**:
- Missing characters may display as boxes (□) or question marks (?)
- Provide ASCII alternatives when possible

**Terminal Support**:
- Windows Command Prompt: Limited Unicode support
- Windows Terminal: Full Unicode support
- Most Linux terminals: Excellent Unicode support
- macOS Terminal: Good Unicode support

## Character Selection Tips

### For Borders

- **Light** (`─│┌┐└┘`): General-purpose, clean look
- **Heavy** (`━┃┏┓┗┛`): Emphasis, headers
- **Double** (`═║╔╗╚╝`): Formal, database-style
- **Rounded** (`─│╭╮╰╯`): Modern, friendly appearance

### For Progress

- **Blocks** (`█▓▒░`): Classic, widely supported
- **Eighths** (`▏▎▍▌▋▊▉█`): Fine-grained control
- **Braille** (U+2800-U+28FF): Custom patterns

### For Indicators

- **Checkmarks** (`✓✔`): Success, completion
- **Crosses** (`✗✘`): Errors, failures
- **Circles** (`●○`): Status, selection
- **Triangles** (`▶▷`): Direction, playback

## References & Resources

### Unicode Resources

- [Unicode Character Table](https://unicode-table.com/) - Searchable Unicode reference
- [Unicode Box Drawing](https://en.wikipedia.org/wiki/Box-drawing_character) - Wikipedia reference
- [Unicode Block Elements](https://en.wikipedia.org/wiki/Block_Elements) - Block element reference

### TUI Libraries

**Rust**:
- [tui-rs](https://github.com/fdehau/tui-rs) - Terminal UI library
- [crossterm](https://github.com/crossterm-rs/crossterm) - Cross-platform terminal manipulation
- [ratatui](https://github.com/ratatui-org/ratatui) - Fork of tui-rs

**Python**:
- [Rich](https://github.com/Textualize/rich) - Rich text and formatting
- [Textual](https://github.com/Textualize/textual) - TUI framework
- [urwid](https://urwid.org/) - Console UI library

**Go**:
- [tview](https://github.com/rivo/tview) - Terminal UI library
- [termui](https://github.com/gizak/termui) - Terminal dashboard
- [bubbletea](https://github.com/charmbracelet/bubbletea) - TUI framework

**JavaScript/TypeScript**:
- [blessed](https://github.com/chjj/blessed) - High-level terminal interface
- [ink](https://github.com/vadimdemedes/ink) - React for CLIs
- [neo-blessed](https://github.com/embark/neo-blessed) - Modern fork of blessed

### Tools

- [Unicode Character Picker](https://unicode-explorer.com/) - Visual character selection
- [Shapecatcher](https://shapecatcher.com/) - Draw to find Unicode characters
- [Copy Paste Character](https://www.copypastecharacter.com/) - Quick character copying
