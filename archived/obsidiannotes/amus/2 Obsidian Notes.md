
## Spacing and Lines

Multiple spaces/lines are formatting to be single spaces/lines automatically. To force multiple spaces, use `&nbsp;`. To force multiple lines, use `<br>`.

## Headers

To create headers for a section, use `#`. Up to six can be used, descending in size and scope. All text under a header up to another header of the same or greater scope can be hidden or shown by clicking the dropdown icon next to the header name.

## Styling

#### Bold

`**Bolded Text**` 
`__Bolded Text__` 

**Bolded Text**

#### Italic

`*Italicized Text*`
`_Italicized Text_`

*Italicized Text*

#### Strikethrough

`~~Struck Text~~`

~~Struck Text~~

#### Highlight

`==Highlighted Text==`

==Highlighted Text==

#### Bold and Italic

`***Bolded and Italicized Text***`
`___Bolded and Italicized Text___`

***Bolded and Italicized Text***.

Styling choices can be nested inside of other styles.

## Quotes

To create a quote, use `>` before the quoted text.

> This is a quote.

## Callouts

To create a callout, use `> [!TYPE]`, where TYPE denotes the type identifier. The title text comes directly after. The rest of the text within the callout should also have `>` at the beginning. The available type identifiers are:

- note
- abstract
- info
- todo
- tip
- success
- question
- warning
- failure
- danger
- bug
- example
- quote

Adding a `+` or `-` directly after the closing square bracket `]` allows the callout to be foldable. The `+` sign has the callout expanded by default, and the `-` sign has it collapsed by default.

> [!note]+ This is the title of the note callout.
> And this is the text inside it.

## Code

To format a line of text like code, use ` `` `. To format a block of text like code, 
use ` ``` `.

To add syntax highlighting to a code block, add a language in front of the first triple back ticks.

```cpp
// C++ Example
int main()
{
	int a = 2;
	cout << creamCheese(a) << endl;
	return 0;
}
```

```python
# Python Example
def main():

	a = 2
	print(creamCheese(a))
```


## External Links

Create an external link by surrounding the link text with `[]`, followed by the URL surrounded by `()`.

[Clicking on this text will take you to Google.](https://www.google.com)

Your external link can also take you to a note within a different vault. To obtain the URL that directs to a note within a different vault, first navigate to the desired vault. Next, find the desired note, right click, and click `Copy Obsidian URL`. Navigate back to the starting note, and surround the Obsidian URL with `()`. ^c7a9d5

If the URL contains blank spaces, they can be replaced with `%20`, or the entire link can be surrounded by `< >`.

## Internal Links

#### Linking to File

To link to a file, use the syntax:

`[[FILE_NAME.FILE_EXTENSION]]`

The file can have any of the following extensions:

1. Markdown files: `md`
2. Image files: `png`, `webp`, `jpg`, `jpeg`, `gif`, `bmp`, `svg`
3. Audio files: `mp3`, `webm`, `wav`, `m4a`, `ogg`, `3gp`, `flac`
4. Video files: `mp4`, `webm`, `ogv`, `mov`, `mkv`
5. PDF files: `pdf`

Any files of these types can be imported by simply clicking and dragging into the desired location within the Obsidian file system.

#### Linking to Note

To link to a note other than the one in the current view, use the following syntax:

`[[NOTE_NAME]]`

After inserting the `[[]]`, a list of notes within the current vault will become available to scroll through and select from.

#### Linking to Heading

To link to a heading, use the following syntax:

`[[#]]`

After inserting the `#`, a list of headings within the current note will become available to scroll through and select from.

For example, [[#Internal Links]] will take you back to the start of the Internal Links section, while highlighting everything included within the section.

#### Linking to Text Block in a Note

To link to a heading, use the following syntax:

`[[^]]`

After inserting the `^`, a list of text blocks within the current note will become available to scroll through and select from.

For example, [[#^c7a9d5]] will take you back to the large paragraph within the External Links section.

#### Changing Link Display Text

After inserting the header/block into the internal link, the following syntax can be used to change the display text for the link:

`[[...|DISPLAY_TEXT]]`

For example, [[#Internal Links|this]] link will also take you back to the start of the Internal Links section, just like the internal link in the Linking to Heading section, but given different display text.

#### Combining Internal Links

Multiple techniques shown above can be used in tandem in order to link to a different note at a particular heading/block, with chosen display text. The syntax is shown below:

`[[NOTE_NAME#HEADING_NAME|DISPLAY_TEXT]]`
`[[NOTE_NAME^TEXT_BLOCK_ID|DISPLAY_TEXT]]`

#### Embedded Notes

Most of the techniques shown in [[#Internal Links]] can be applied for embedded notes, using the syntax:

`![[]]`

For example, the large paragraph in the External Links section is embedded below.

![[#^c7a9d5]]

## External Images

Include an external image by surrounding `width x height` in pixels with `![]`, followed by the image URL surrounded by `()`. If no width or height is specified, the default width and height of the image will be used. ***The image will not render if the computer is offline.***

![](https://history-computer.com/ModernComputer/Basis/images/Engelbart.jpg)

## Embedded File

To embed an image, use the following syntax:

`![[FILE_NAME.FILE_EXTENSION|WIDTHxHEIGHT]]`

If `|WIDTHxHEIGHT` is not included, the original width and height (in pixels) of the image will be used. If only `|WIDTH` is included, the image will scale according to its original aspect ratio.

To embed an audio file, use the following syntax:

`![[FILE_NAME.FILE_EXTENSION]]`

To embed a PDF, use the following syntax:

`![[FILE_NAME.pdf#page=PAGENUMBER]]`

The inclusion of `#page=PAGENUMBER` forces the embedded PDF to be open at the specified page. If `#page=PAGENUMBER` is not included, the PDF will be open at the first page.

## Lists

Create an unordered list by starting a line of text with `-`, `*`, or `+`.

- First Item
- Second Item
- Third Item

Create an ordered list by starting a line of text with a number followed by `.` .

1. First Item
2. Second Item
10. Tenth Item

You can create a nested list by indenting one or more items.

1. First Item
	1. First Nested Item
	2. Second Nested Item
2. Second Item
	- Third Nested Item

## Task List

Create a task list by starting the line with `- [ ]` for an unchecked task, and 
`- [CHARACTER]` for a checked task. The rest of the text after the checkbox is the task text. You can check and uncheck the check boxes after making them.

- [ ] Unchecked Task
- [x] Checked Task

## Horizontal Bar

Using `***`, `---`, or `___` will create a horizontal line that splits the screen to divide sections.

---

## Footnotes

You can add footnotes to notes by including `[^NUMBER]` at any point within a block of text, and `[^NUMBER]:`, followed by the footnote text at the end of the block of text. Names can also be used instead of numbers for the footnotes, but they will still be displayed as numbers in reading mode.

This is a line of text[^note].

[^note]: This is the footnote for the above line of text.

## Comments

You can add a comment by surrounding text with `%% %%`. These comments will not be visible in reading mode, only in editing mode.

%%
This is a comment.
%%