Report on My Scanner (src/scanner.rs)
For this part of the project, I wrote the scanner for the compiler. The scanner reads the source code one character at a time and turns it into tokens. The starter code had five empty functions marked todo!(), and I filled them in. The helper functions like advance, peek and add were already given, so I used them.

1. run
This function runs the whole scan. I used a while loop that keeps calling scan_token() until the source code is finished. After the loop, I set the line number to the line of the last token (or line 1 if there are no tokens) and then added the Eof token to mark the end of the file.

2. scan_token
This function reads one character and decides what it is, using a match:

Whitespace: spaces, tabs and \r are ignored. A \n adds 1 to the line number.
Single characters: ( ) { } , ; + - * each become their own token.
Operators with two possibilities: for !, =, > and <, I check if the next character is =. If it is, I make the longer token (like != or >=). If not, I make the short one (like ! or >).
Slash and comments: if / is followed by another /, it is a comment, so I skip everything until the end of the line. If not, it is just a Slash token.
Numbers, strings and names: a digit calls number(), a " calls string(), and a letter or _ calls identifier().
Anything else: it gives the error "Character is not part of any token."
3. string
This scans text inside quotes. It keeps reading until it finds the closing ". If it passes a new line, it adds 1 to the line number, so strings can go over more than one line. If the closing quote is never found, it reports "String is never closed." using the line where the string started.

4. number
This reads digits. If there is a . and a digit comes right after it, it also reads the decimal part. So 3.14 is one number, but 3. is not treated as a decimal.

5. identifier
This reads letters, digits and underscores to build a word. Then it checks the word using keyword() from token.rs. If it is a keyword (like if or while), it gets the keyword token type. If not, it becomes a normal Identifier.

Things I could still improve

The Eof token gets the line of the last token, so I should check that this matches what the spec says in section 6.1.
For strings that go over many lines, the token gets the line where the string ends, not where it started. I should check the spec for this too.