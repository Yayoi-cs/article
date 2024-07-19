# Emacs Command

| Command |Meaning| Description                                                       |
|:--------|:--|:------------------------------------------------------------------|
| c-f     |forward| go to next charactor                                              |
| c-b     |back| back to previous charactor                                        |
| c-n     |next| go to next line                                                   |
| c-p     |previous| back to previous line                                             |
| c-a     || go to the beginning of the line                                   |
| c-e     || go to the end of the line                                         |
| c-v     || go to the next page                                               |
| esc-v   || go to the previous page                                           |
| esc-<   || go to the beginning of the file                                   |
| esc->   || go to the end of the file                                         |
| esc-c-a || go to the beginning of the function                               |
| esc-c-e || go to the end of the function                                     |
| esc-g g || go to the specified line                                          |
| c-d     |delete| delete the charactor                                              |
| c-k     |kill| delete the line                                                   |
| c-@     || Mark                                                              |
| c-w     |wipe| delete all charactor from Mark to current position                |
| c-y     || paste                                                             |
| c-x r k |kill rectangle| delete all charactor in rectangle from Mark to current position   |
| c-x r y || paste the rectangle                                               |
| c-x u   |undo| undo the previous operation                                       |
| c-g     || cancel the input                                                  |
| c-s     |search| search strings from current position to end of the file           |
| c-r     || search strings from the beginning of the file to current position |
| esc-%   || replace strings with confirmation                                 |
| c-x c-b || show the file buff                                                |

## Example
Rename the file and continue to edit.

```Bash
c-z #suspend emacs
$mv prog prog.c
$fg
c-x c-b #show the file buff
c-x o #move to the buff window
#move to "prog"
d #give "prog" a delete mark
x #delete the marked buff
c-x c-f #open file
c-x 1 #move to the file window
```