---
layout: post
title: Ruby的伪BNF语法
---

Here is the syntax of Ruby in pseudo BNF. For more detail, see parse.y in Ruby distribution. 

下面是Ruby的BNF语法描述，更多细节，参考Ruby分发包中的parse.y文件。

> BNF是语言的形式化描述，是最能体现设计和品味的地方，同样也是yacc的输入文件。之前，个人曾经想要自动生成一个c--的编译器，结果一直卡在语法解释器上，也不知道如何处理，最后，由于当时还有其他的事情想做，就放弃了。

{% highlight ruby %}
PROGRAM		: COMPSTMT

COMPSTMT	: STMT (TERM EXPR)* [TERM]

STMT		: CALL do [`|' [BLOCK_VAR] `|'] COMPSTMT end
                | undef FNAME
		| alias FNAME FNAME
		| STMT if EXPR
		| STMT while EXPR
		| STMT unless EXPR
		| STMT until EXPR
                | `BEGIN' `{' COMPSTMT `}'
                | `END' `{' COMPSTMT `}'
                | LHS `=' COMMAND [do [`|' [BLOCK_VAR] `|'] COMPSTMT end]
		| EXPR

EXPR		: MLHS `=' MRHS
		| return CALL_ARGS
		| yield CALL_ARGS
		| EXPR and EXPR
		| EXPR or EXPR
		| not EXPR
		| COMMAND
		| `!' COMMAND
		| ARG

CALL		: FUNCTION
                | COMMAND

COMMAND		: OPERATION CALL_ARGS
		| PRIMARY `.' OPERATION CALL_ARGS
		| PRIMARY `::' OPERATION CALL_ARGS
		| super CALL_ARGS

FUNCTION        : OPERATION [`(' [CALL_ARGS] `)']
		| PRIMARY `.' OPERATION `(' [CALL_ARGS] `)'
		| PRIMARY `::' OPERATION `(' [CALL_ARGS] `)'
		| PRIMARY `.' OPERATION
		| PRIMARY `::' OPERATION
		| super `(' [CALL_ARGS] `)'
		| super

ARG		: LHS `=' ARG
		| LHS OP_ASGN ARG
		| ARG `..' ARG
		| ARG `...' ARG
		| ARG `+' ARG
		| ARG `-' ARG
		| ARG `*' ARG
		| ARG `/' ARG
		| ARG `%' ARG
		| ARG `**' ARG
		| `+' ARG
		| `-' ARG
		| ARG `|' ARG
		| ARG `^' ARG
		| ARG `&' ARG
		| ARG `<=>' ARG
		| ARG `>' ARG
		| ARG `>=' ARG
		| ARG `<' ARG
		| ARG `<=' ARG
		| ARG `==' ARG
		| ARG `===' ARG
		| ARG `!=' ARG
		| ARG `=~' ARG
		| ARG `!~' ARG
		| `!' ARG
		| `~' ARG
		| ARG `<<' ARG
		| ARG `>>' ARG
		| ARG `&&' ARG
		| ARG `||' ARG
		| defined? ARG
		| PRIMARY

PRIMARY		: `(' COMPSTMT `)'
		| LITERAL
		| VARIABLE
		| PRIMARY `::' IDENTIFIER
		| `::' IDENTIFIER
		| PRIMARY `[' [ARGS] `]'
		| `[' [ARGS [`,']] `]'
		| `{' [(ARGS|ASSOCS) [`,']] `}'
		| return [`(' [CALL_ARGS] `)']
		| yield [`(' [CALL_ARGS] `)']
		| defined? `(' ARG `)'
                | FUNCTION
		| FUNCTION `{' [`|' [BLOCK_VAR] `|'] COMPSTMT `}'
		| if EXPR THEN
		  COMPSTMT
		  (elsif EXPR THEN COMPSTMT)*
		  [else COMPSTMT]
		  end
		| unless EXPR THEN
		  COMPSTMT
		  [else COMPSTMT]
		  end
		| while EXPR DO COMPSTMT end
		| until EXPR DO COMPSTMT end
		| case COMPSTMT
		  (when WHEN_ARGS THEN COMPSTMT)+
		  [else COMPSTMT]
		  end
		| for BLOCK_VAR in EXPR DO
		  COMPSTMT
		  end
		| begin
		  COMPSTMT
		  [rescue [ARGS] DO COMPSTMT]+
		  [else COMPSTMT]
		  [ensure COMPSTMT]
		  end
		| class IDENTIFIER [`<' IDENTIFIER]
		  COMPSTMT
		  end
		| module IDENTIFIER
		  COMPSTMT
		  end
		| def FNAME ARGDECL
		  COMPSTMT
		  end
		| def SINGLETON (`.'|`::') FNAME ARGDECL
		  COMPSTMT
		  end

WHEN_ARGS	: ARGS [`,' `*' ARG]
		| `*' ARG

THEN		: TERM
		| then
		| TERM then

DO		: TERM
		| do
		| TERM do

BLOCK_VAR	: LHS
		| MLHS

MLHS		: MLHS_ITEM `,' [MLHS_ITEM (`,' MLHS_ITEM)*] [`*' [LHS]]
                | `*' LHS

MLHS_ITEM	: LHS
		| '(' MLHS ')'

LHS		: VARIABLE
		| PRIMARY `[' [ARGS] `]'
		| PRIMARY `.' IDENTIFIER

MRHS		: ARGS [`,' `*' ARG]
		| `*' ARG

CALL_ARGS	: ARGS
		| ARGS [`,' ASSOCS] [`,' `*' ARG] [`,' `&' ARG]
		| ASSOCS [`,' `*' ARG] [`,' `&' ARG]
		| `*' ARG [`,' `&' ARG]
		| `&' ARG
		| COMMAND

ARGS 		: ARG (`,' ARG)*

ARGDECL		: `(' ARGLIST `)'
		| ARGLIST TERM

ARGLIST		: IDENTIFIER(`,'IDENTIFIER)*[`,'`*'[IDENTIFIER]][`,'`&'IDENTIFIER]
		| `*'IDENTIFIER[`,'`&'IDENTIFIER]
		| [`&'IDENTIFIER]

SINGLETON	: VARIABLE
		| `(' EXPR `)'

ASSOCS		: ASSOC (`,' ASSOC)*

ASSOC		: ARG `=>' ARG

VARIABLE	: VARNAME
		| nil
		| self

LITERAL		: numeric
		| SYMBOL
		| STRING
		| STRING2
		| HERE_DOC
		| REGEXP

TERM		: `;'
		| `\n'

The followings are recognized by lexical analizer.


OP_ASGN		: `+=' | `-=' | `*=' | `/=' | `%=' | `**='
		| `&=' | `|=' | `^=' | `<<=' | `>>='
		| `&&=' | `||='

SYMBOL		: `:'FNAME
		| `:'VARNAME

FNAME		: IDENTIFIER | `..' | `|' | `^' | `&'
		| `<=>' | `==' | `===' | `=~'
                | `>' | `>=' | `<' | `<='
		| `+' | `-' | `*' | `/' | `%' | `**'
		| `<<' | `>>' | `~'
                | `+@' | `-@' | `[]' | `[]='

OPERATION       : IDENTIFIER
                | IDENTIFIER'!'
                | IDENTIFIER'?'

VARNAME		: GLOBAL
		| `@'IDENTIFIER
		| IDENTIFIER

GLOBAL		: `$'IDENTIFIER
		| `$'any_char
		| `$''-'any_char

STRING		: `"' any_char* `"'
		| `'' any_char* `''
		| ``' any_char* ``'

STRING2		: `%'(`Q'|`q'|`x')char any_char* char

HERE_DOC        : `<<'(IDENTIFIER|STRING)
                  any_char*
                  IDENTIFIER

REGEXP		: `/' any_char* `/'[`i'|`o'|`p']
		| `%'`r' char any_char* char
{% endhighlight %}

IDENTIFIER is the sqeunce of characters in the pattern of /[a-zA-Z_][a-zA-Z0-9_]*/.

IDENTIFIER是/[a-zA-Z_][a-zA-Z0-9_]*/模式的字符序列。
