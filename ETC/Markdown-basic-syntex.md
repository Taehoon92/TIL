# Markdown 기본 문법
- - - 
[Markdown official document](https://www.markdownguide.org/getting-started/)

Markdown is a lightweight markup language that you can user to add formatting elements to plaintext text documents. README file or Notion app is based on markdown.

### 제목

텍스트앞에 #를 붙여서 표시한다. 최대 6개까지 붙여서 제목의 크기를 정할 수 있고, 갯수가 많아질수록 제목의 크기가 작아진다. # 뒤에 1칸의 여백을 둬야한다.
```
# Title
### Title
###### Title
```

### 가로 줄

4가지 방법 존재 (결과는 동일)
```
---
- - -
***
* * *
```

### Paragraph

Use a blank line to separate one or more lines of text

###  Line Breaks (New Line)

End a line with **two or more spaces**.

### Bold, Italic

Bold : Add two asterisks before and after a word or phrase.  
Italic : Add one asterisks before and after a word or phrase.
```
**Bold**
*Italic*
***Bold & Italic***
```
**Bold** `**Bold**`  
*Italic* `*Italic*`  
***Both*** `***Both***`      
### Blockquotes
add > in front of a paragraph
```
>Example
>
>>Test
```
>Example  
>
>>Test

### Lists
> #### Ordered Lists  
> Add line items with numbers. (The numbers don't have to be in numerical order)
>```
>1. First
>1. Second
>8. Thrid
>3. Fourth
>   3. item1
>   2. item2
>1. Fifth
>```
>1. First  
>1. Second  
>8. Thrid  
>3. Fourth  
    1. item1  
    2. item2  
>1. Fifth  
>
> #### Unordered Lists
> Add dashes(-), asterisks(*), or plus(+) in front of items. (unifying the symbol)  
>```
>* item
>* item
>   * inside item
>   * inside item
>* item
>```
>* item
>* item
>   * inside item
>   * inside item
>* item

### Code Block

Normally indented one tab or four spaces. When code blocks are in a list, indented two tabs or eight spaces.

##### Code Blocks in a list

```
* Test
* Test  

        <html>
            <head>
                <title> Test </title>
            </head>
        </html>
* Test
```

* Test 
* Test  

        <html>
            <head>
                <title> Test </title>
            </head>
        </html>
* Test  

##### Code Blocks
```
Hello

    String test = 'HELLO';
```
Hello

    String test = 'HELLO';

### Code

To detonate a word or pharase, enclose it backticks(`)
```
`hello`
```
`hello`


>If you want to escape backticks, you can escape it by enclosing the work or phrase in double backticks(``)
>```
>``Use `double backticks` for escaping.``
>```
>``Use `double backticks` for escaping.``

### Link

Enclose the link text in brackets( [Link Text] ), then follow it with the URL in parentheses.( (URL) )
```
Here is link for [Google](www.google.com)
```
Here is link for [Google](www.google.com)

### Url and Email address

Enclose the url or email in angle bracket to link 
```
<http://www.google.com>
<test@gamil.com>
```
<http://www.google.com>  
<test@gamil.com>

### Escaping Characters
Add a back slash ( \\ ) in front of the character.
```
\\ Backslash  
\` Backtick  
\* List? or Not
\!
\#
```
\\ Backslash   
\` Backtick  
\* List? or Not  
\!  
\#

