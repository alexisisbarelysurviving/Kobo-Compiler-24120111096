Report on My Parser (src/parser.rs)

For this part of the project, I wrote the parser. It takes the tokens from the scanner and builds the tree in ast.rs. Most of the rules are one function each, and the order of the calls is the precedence table from section 3.2. The helpers like matches, consume and peek were already given, so I used them.

1. The dangling else

The dangling else is handled in if_statement, at src/parser.rs:157. The then branch is parsed first, by statement() on src/parser.rs:156. When that call returns, the function looks at the next token. If it is else, matches consumes it and statement() on src/parser.rs:158 parses the branch after it. If it is not else, else_branch stays None. Section 3.1 writes if with an optional else, so the grammar says an else may be there. It does not say which if should take it when one if is inside another. The recursion decides that. The inner if_statement is still running when the else token is reached, so it consumes the else before the outer if can see it. For if (false) if (true) print "a"; else print "b"; the parser prints (if false (if true (print "a") (print "b"))). The else is on the inner if. If it had bound to the outer if, the same program would print (if false (if true (print "a")) (print "b")).

2. Assignment is not LL(1)

The check on the left side is the match at src/parser.rs:221. assignment does not look for a name first. It parses the left side as a normal expression, by calling or() on src/parser.rs:217, and only if the next token is = does it look at what that expression was. The check cannot come any earlier. A name, a call and a number all start out as expressions, and one token of lookahead cannot tell you that an = is still coming. a, f() and 1 have to be read before you know. Once the left side is a tree, a bare variable becomes an Assign node. Anything else is rejected, and the original expression is kept. For 1 = 2; the message from section 5.1 is Cannot assign to this expression. It is reported on the =, at src/parser.rs:230.

3. Synchronising after an error

synchronise is at src/parser.rs:417. parse calls it on src/parser.rs:18 when a declaration fails, and a block calls it on src/parser.rs:189. It steps over one token first, then stops. It stops when the token just passed is a semicolon, at src/parser.rs:422, because a semicolon ends a statement. It also stops on fun, var, if, while, print or return, at src/parser.rs:425 to src/parser.rs:430. Those six tokens start a declaration or a statement in section 3.1, so the next statement is kept and parsed. else is left out, because an else by itself does not start a statement.

In 8725aad the only stop was if self.previous().kind == TokenType::Semicolon, and the function set self.depth = 0, so it walked past a later var or print. In 6c70120 the depth line is gone and the loop also returns on those six keywords.

Deleting the call on src/parser.rs:18 changes tests/phase-2/invalid/expect_expression.kobo. The file is print 1 + * 2;, and the error is [line 1] Error at '*': Expect expression. The * is not consumed. With the call, that line is printed once. Without it, the parser stays on the * and prints the same line over and over, so the test does not finish. tool/run-golden says it timed out after 5s.
